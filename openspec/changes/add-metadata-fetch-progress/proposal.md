## Why

Large selected albums can spend a long time syncing per-file iCloud metadata before the actual local file sorting starts. During that period the frontend only shows a generic start message, which makes the app appear stalled even though useful progress information is available from album asset counts.

## What Changes

- Add an explicit metadata-fetch phase after the user starts sorting selected albums and before file sorting begins.
- Report metadata-fetch progress using known selected-album file counts, including a user-facing message such as `Fetching metadata of selected album(s)...`.
- Reuse the existing inline sort progress location for metadata-fetch status and progress, then transition to normal sorting progress once metadata sync is complete.
- Preserve the existing final sort completion and error behavior.

## Capabilities

### New Capabilities

- None.

### Modified Capabilities

- `photo-sorting`: Sort startup exposes selected-album metadata fetch as part of the started operation instead of blocking silently before progress can be shown.
- `sort-progress`: Progress events include a metadata-fetch phase and counters sufficient to render a progress bar before sorting starts.
- `frontend-flow`: The inline progress area shows metadata-fetch messaging and progress before switching to file sorting progress.

## Impact

- Backend sort service flow changes so metadata sync can update observable progress while the operation is active.
- SSE progress schema expands the `status` values and current-work meaning while keeping the existing endpoint and stream mechanism.
- Frontend API types and `SortProgress` rendering update to handle the metadata-fetch phase.
- Tests should cover backend progress state transitions and frontend build/type safety.
