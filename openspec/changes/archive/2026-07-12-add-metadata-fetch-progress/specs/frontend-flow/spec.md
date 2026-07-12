## MODIFIED Requirements

### Requirement: Sorting users can view progress
The frontend SHALL show metadata-fetch and sort progress inline within the album selection interface after a sort is started.

#### Scenario: User starts sorting
- **WHEN** the user starts sorting selected albums from the album selection interface
- **THEN** the frontend remains on the album selection interface
- **THEN** the frontend shows progress based on SSE events from the backend near the sort action
- **THEN** the frontend displays metadata-fetch progress with a message such as `Fetching metadata of selected album(s)...` while selected-album metadata is being fetched
- **THEN** the frontend displays overall sorting progress, current file or album, failures, and completion state after sorting begins

#### Scenario: Sort reaches terminal state
- **WHEN** the inline sort progress receives a terminal completion or error state
- **THEN** the frontend keeps the terminal result visible in the album selection interface
- **THEN** the frontend provides an action to dismiss the terminal result and return to normal album selection controls
