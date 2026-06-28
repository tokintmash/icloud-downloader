## MODIFIED Requirements

### Requirement: Sorting starts for selected albums
The system SHALL provide an API endpoint that starts sorting for an explicit list of selected album IDs and makes selected-album metadata fetching observable before local file sorting begins.

#### Scenario: Sort starts successfully
- **WHEN** the user starts sorting with one or more selected album IDs
- **THEN** the backend starts an active sort operation before iterating every selected album asset
- **THEN** the backend reports metadata-fetch progress for the selected albums
- **THEN** the backend syncs per-file metadata for the selected albums
- **THEN** the backend starts the local file sorting operation after metadata sync completes
- **THEN** the response includes `total_files`

#### Scenario: Sort is already running
- **WHEN** the user starts a sort while another sort is active
- **THEN** the API returns a standard error response with a `sort_in_progress` error code
