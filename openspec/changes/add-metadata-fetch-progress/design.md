## Context

The current sort start flow performs selected-album metadata sync inside `POST /api/sort/start` before the backend creates the background sort task. For large albums, that means the frontend remains in a generic `Starting sort...` state and cannot subscribe to SSE progress until the slow metadata fetch is already complete.

The album picker already displays lightweight album counts from iCloud metadata, and the sort service can determine selected-album totals before iterating every selected asset. The existing SSE progress stream is the right place to expose this work because it already drives the inline progress area near the sort action.

## Goals / Non-Goals

**Goals:**

- Show an explicit metadata-fetch phase after the user starts sorting and before local file sorting begins.
- Stream metadata-fetch progress over the existing sort progress SSE endpoint.
- Use selected-album asset counts as the metadata-fetch total so the frontend can render a determinate progress bar.
- Preserve the existing sort endpoint paths, overall UI location, terminal completion/error behavior, and file sorting semantics.

**Non-Goals:**

- Add a separate metadata-sync endpoint or WebSocket channel.
- Download photo or video binaries from iCloud.
- Persist metadata-fetch progress in SQLite after a run ends.
- Change album listing behavior or require full per-file metadata during album picker display.

## Decisions

### Start the operation before metadata sync

`SorterService.start()` will validate authentication and settings, fetch the lightweight album list needed for folder names and selected asset counts, set the active progress status to `fetching_metadata`, and create one background task that performs metadata sync followed by sorting.

This avoids the current silent blocking period in `POST /api/sort/start`. The alternative was to keep metadata sync inside the request and only improve the static frontend message, but that would not allow real progress updates and would still make the UI feel stalled.

### Extend existing progress events with a new phase

The existing SSE payload will add `fetching_metadata` as a valid `status` value. During this phase, `total_files` represents the selected albums' expected asset count, `completed_files` represents assets inspected for metadata, `current_album` identifies the album currently being fetched, and `current_file` can remain empty.

This keeps one progress component and one stream contract. The alternative was to introduce separate metadata-specific fields, but reusing the existing counters is enough for the requested progress bar and minimizes schema churn.

### Add a progress callback to metadata sync

`icloud_service.sync_album_metadata()` will accept an optional callback that receives metadata-sync progress updates while it iterates selected assets. The sorter service will pass a callback that updates its in-memory progress state.

This keeps pyicloud-specific iteration in the iCloud service while allowing the sort service to own user-visible run state. The alternative was to move all metadata iteration into the sorter service, but that would blur service responsibilities and duplicate iCloud handling logic.

### Transition counters between phases

When metadata sync completes, the sorter service will reset sorting counters using the actual number of pending rows and change `status` to `sorting`. This means the same progress bar moves from metadata fetch progress to sorting progress rather than trying to combine two different phases into a single percentage.

The alternative was to treat metadata fetch and sorting as one combined total, but that would make the meaning of `completed_files` less clear and would complicate completion summaries.

## Risks / Trade-offs

- Metadata total can differ from actual sortable rows if filename extraction skips assets. -> Use album asset counts for fetch progress, then switch to the actual pending row count for sorting progress once sync completes.
- The lightweight album list request can still take some time before `/api/sort/start` returns. -> Keep this limited to existing lightweight album metadata and avoid per-file iteration before activating progress.
- Frontend code must handle a new status value. -> Update TypeScript API types and `SortProgress` rendering together with backend schemas.
- A metadata-sync failure now occurs inside the background run. -> Convert failures into terminal `error` progress events so the SSE stream ends consistently.
