## 1. Backend Progress Flow

- [ ] 1.1 Extend backend and frontend sort progress types to allow status `fetching_metadata`.
- [ ] 1.2 Update `SorterService.start()` to create an active operation before selected-album per-asset metadata sync begins.
- [ ] 1.3 Compute selected-album metadata total from lightweight album asset counts and return it as `total_files` from sort start.
- [ ] 1.4 Add metadata-fetch progress state updates for status, total files, completed files, current album, and terminal errors.

## 2. Metadata Sync Progress

- [ ] 2.1 Add an optional progress callback to `icloud_service.sync_album_metadata()`.
- [ ] 2.2 Invoke the progress callback while iterating selected album assets, including assets skipped because filenames cannot be extracted.
- [ ] 2.3 Preserve existing metadata replacement, filename extraction, and selected-album filtering behavior.

## 3. Sorting Transition

- [ ] 3.1 After metadata sync completes, reset progress counters to actual pending sorting rows and transition status to `sorting`.
- [ ] 3.2 Start the existing local file move/copy sort work after the metadata-fetch phase completes successfully.
- [ ] 3.3 Convert metadata-sync failures into terminal `error` progress events that end the SSE stream consistently.

## 4. Frontend UI

- [ ] 4.1 Update `SortProgress` to render `fetching_metadata` with the message `Fetching metadata of selected album(s)...`.
- [ ] 4.2 Reuse the existing inline progress bar and stats location for metadata-fetch progress.
- [ ] 4.3 Preserve current sorting, completion, error, session-expiry, and app-expiry rendering behavior.

## 5. Tests and Verification

- [ ] 5.1 Add or update backend tests for metadata-fetch status, progress counters, phase transition, and metadata-sync failure handling.
- [ ] 5.2 Run `./venv/Scripts/python.exe -m pytest` and fix failures.
- [ ] 5.3 Run `cd frontend && npm run build` and fix TypeScript or build failures.
