## Purpose
Define how clients observe a running sort operation and receive progress, current-work, error, and completion information.

## Requirements

### Requirement: Sort progress is streamed with Server-Sent Events
The system SHALL expose sort progress over an SSE endpoint rather than WebSockets.

#### Scenario: Sort is active
- **WHEN** the frontend connects to the sort progress endpoint during an active sort
- **THEN** the backend streams progress events with media type `text/event-stream`

#### Scenario: No sort is active
- **WHEN** the frontend requests sort progress while no sort operation is active and no terminal progress state is available
- **THEN** the API returns a standard error response indicating that no sort operation is active

### Requirement: Progress events include sorting counters
The system SHALL include total, completed, and failed file counters in each sort progress event.

#### Scenario: Progress event is emitted
- **WHEN** the backend emits a progress event
- **THEN** the event includes `status`
- **THEN** the event includes `total_files`
- **THEN** the event includes `completed_files`
- **THEN** the event includes `failed_files`

#### Scenario: Metadata fetch progress event is emitted
- **WHEN** the backend emits a metadata-fetch progress event
- **THEN** the event includes status `fetching_metadata`
- **THEN** `total_files` is the selected albums' known asset count
- **THEN** `completed_files` is the number of selected assets inspected for metadata
- **THEN** `failed_files` remains available for schema consistency

### Requirement: Progress events identify current work
The system SHALL include the currently processed file and album in progress events when that information is available.

#### Scenario: File is being sorted
- **WHEN** the sorter is processing a file
- **THEN** the progress event includes `current_file`
- **THEN** the progress event includes `current_album`

#### Scenario: Album metadata is being fetched
- **WHEN** the backend is fetching metadata for a selected album
- **THEN** the progress event includes `current_album`
- **THEN** the progress event can include an empty `current_file`

### Requirement: Metadata fetch progress is streamed before sorting
The system SHALL stream progress for selected-album metadata fetching before local file sorting begins.

#### Scenario: Metadata fetch is active
- **WHEN** the frontend connects to the sort progress endpoint after sort start and selected-album metadata is being fetched
- **THEN** the backend emits progress events with status `fetching_metadata`
- **THEN** the event includes `total_files` based on the selected albums' known asset counts
- **THEN** the event includes `completed_files` based on selected assets inspected for metadata
- **THEN** the event includes `current_album` when an album is being fetched

#### Scenario: Metadata fetch completes
- **WHEN** selected-album metadata fetching completes successfully
- **THEN** the backend emits subsequent progress events with status `sorting`
- **THEN** sorting progress counters use the actual local sorting file total

### Requirement: Progress events include file-level errors
The system SHALL include accumulated file-level errors in progress events.

#### Scenario: A file-level error has occurred
- **WHEN** the backend emits a progress event after a file-level failure
- **THEN** the event includes an `errors` array
- **THEN** each error includes `filename`, `error`, and `album`

### Requirement: Progress stream ends on terminal status
The system SHALL end the SSE stream when the sort reaches a terminal status.

#### Scenario: Sort completes
- **WHEN** a progress event has status `complete`
- **THEN** the backend ends the SSE stream after sending the event

#### Scenario: Sort encounters a terminal error
- **WHEN** a progress event has status `error`
- **THEN** the backend ends the SSE stream after sending the event

### Requirement: Sort progress reports expiry as a terminal error
The system SHALL report expiry during an active sort as a terminal SSE progress error before ending the stream.

#### Scenario: Expiry occurs while progress stream is active
- **WHEN** a sort operation is active
- **AND** the running build reaches its computed expiry date
- **THEN** the backend emits a progress event with status `error`
- **THEN** the event identifies the expiry condition with error code `app_expired`
- **THEN** the event includes message `This beta has expired. Contact the author of the app to get an up-to-date version.`
- **THEN** the backend ends the SSE stream after sending the terminal event
