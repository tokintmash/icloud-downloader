## ADDED Requirements

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

## MODIFIED Requirements

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
