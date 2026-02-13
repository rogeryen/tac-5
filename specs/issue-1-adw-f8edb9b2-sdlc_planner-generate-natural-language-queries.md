# Feature: Generate Natural Language Query Button

## Feature Description
Add a button that generates interesting natural language queries based on the existing database tables and their structure. The generated queries will automatically populate the query input field, replacing any existing content. This feature helps users discover what kinds of questions they can ask about their data and provides examples of natural language queries they can execute. The generated queries are limited to two sentences maximum to keep them concise and focused.

## User Story
As a user
I want to generate example natural language queries based on my data
So that I can quickly explore my data without thinking of queries from scratch

## Problem Statement
Users who upload data to the Natural Language SQL Interface may not immediately know what questions to ask about their data. They need inspiration and examples of natural language queries that are relevant to their specific tables and columns. Currently, they must either read through the table schemas manually or come up with queries themselves, which can be time-consuming and may not showcase the full capabilities of the system.

## Solution Statement
Create a "Generate Query" button that uses the existing LLM processor to analyze the current database schema and generate interesting, contextually relevant natural language queries. The button will be styled consistently with the existing "Upload Data" button and positioned separately from the primary "Query" button. When clicked, it will generate a query (maximum two sentences) and populate the query input field, overwriting any existing content. Users can then execute the generated query immediately or modify it as needed.

## Relevant Files
Use these files to implement the feature:

### Backend Files
- `app/server/core/llm_processor.py` - Contains the LLM integration logic for both OpenAI and Anthropic. Will be extended to add a new function `generate_natural_language_query()` that creates interesting queries based on table schemas.
- `app/server/main.py` - FastAPI application entry point (not actively used, server.py is the main file).
- `app/server/server.py` - Contains all API endpoints. Will add a new `POST /api/generate-query` endpoint that calls the LLM processor to generate queries.
- `app/server/core/sql_processor.py` - Contains database schema retrieval logic via `get_database_schema()`. Will be used to pass schema information to the query generator.

### Frontend Files
- `app/client/index.html` - Contains the main HTML structure with query controls. Will add a new "Generate Query" button in the query controls section.
- `app/client/src/main.ts` - Main TypeScript entry point containing all UI logic and event handlers. Will add event handler for the generate query button and logic to populate the input field.
- `app/client/src/api/client.ts` - API client for backend communication. Will add a new `generateQuery()` function to call the `/api/generate-query` endpoint.
- `app/client/src/style.css` - Styles for the application. Will add styles for the generate query button to match the Upload Data button style.

### New Files
- `.claude/commands/e2e/test_generate_query.md` - E2E test file to validate the generate query feature works correctly, following the pattern of existing E2E tests.

## Implementation Plan

### Phase 1: Foundation
Set up the backend infrastructure to support query generation by extending the LLM processor module with a new function that can analyze database schemas and generate contextually relevant natural language queries. This includes creating the prompt engineering logic to ensure queries are interesting, relevant, and limited to two sentences.

### Phase 2: Core Implementation
Implement the full stack functionality:
- Create the backend API endpoint to handle query generation requests
- Add the frontend button and integrate it with the backend
- Ensure the generated query properly replaces the content in the query input field
- Style the button to match the existing Upload Data button

### Phase 3: Integration
Integrate the feature with the existing UI flow, ensuring the generated queries work seamlessly with the existing query execution flow. Add comprehensive E2E tests to validate the feature works correctly across different scenarios (empty database, single table, multiple tables).

## Step by Step Tasks
IMPORTANT: Execute every step in order, top to bottom.

### Step 1: Extend LLM Processor for Query Generation
- Open `app/server/core/llm_processor.py`
- Add a new function `generate_natural_language_query()` that:
  - Takes schema_info as input parameter
  - Formats the schema information for the LLM prompt
  - Creates a prompt asking the LLM to generate an interesting natural language query based on the tables and columns
  - Ensures the prompt specifies: "Generate ONE interesting natural language query that a user could ask about this data. The query should be clear, specific, and no more than two sentences. Do not include any SQL, only the natural language question."
  - Implements both OpenAI and Anthropic variants (similar to existing `generate_sql_with_openai` and `generate_sql_with_anthropic`)
  - Uses routing logic similar to `generate_sql()` to choose the appropriate provider
  - Returns a string containing the generated natural language query
  - Handles errors gracefully with appropriate error messages

### Step 2: Create Backend API Endpoint
- Open `app/server/server.py`
- Add a new Pydantic model `GenerateQueryResponse` in the imports section from `core.data_models`
- Create a new endpoint `POST /api/generate-query` that:
  - Accepts no input parameters (uses current database state)
  - Calls `get_database_schema()` to get current tables and columns
  - Checks if there are any tables in the database; if not, returns an error message like "No tables available. Please upload data first."
  - Calls `generate_natural_language_query()` from the LLM processor
  - Returns a `GenerateQueryResponse` with the generated query
  - Includes proper error handling with logging
  - Uses the same logging format as other endpoints

### Step 3: Add Response Model to Data Models
- Open `app/server/core/data_models.py`
- Add a new Pydantic model `GenerateQueryResponse` with fields:
  - `query: str` - The generated natural language query
  - `error: Optional[str] = None` - Error message if generation fails
- Export the model so it can be imported in server.py

### Step 4: Add Frontend API Client Method
- Open `app/client/src/api/client.ts`
- Add a new interface for the generate query response matching the backend model
- Add a new method `generateQuery()` to the api object that:
  - Makes a POST request to `/api/generate-query`
  - Returns the response typed with the interface
  - Handles network errors appropriately

### Step 5: Add Generate Query Button to HTML
- Open `app/client/index.html`
- Find the query-controls div (around line 22)
- Add a new button after the "Upload Data" button:
  - ID: `generate-query-button`
  - Class: `secondary-button`
  - Text content: "Generate Query"
- Position it using CSS flexbox so it's visually separated from primary buttons with `justify-content: space-between` or similar spacing

### Step 6: Update CSS Styling
- Open `app/client/src/style.css`
- Locate the `.query-controls` styles
- Update to ensure proper spacing between buttons using flexbox with `justify-content: space-between` or similar
- Verify the `.secondary-button` class matches the Upload Data button style
- If needed, add specific styles for the generate query button

### Step 7: Implement Frontend Event Handler
- Open `app/client/src/main.ts`
- Create a new function `initializeGenerateQuery()` that:
  - Gets references to the generate query button and query input
  - Adds click event listener to the button
  - Shows loading state during generation (disable button, show loading indicator)
  - Calls `api.generateQuery()` to fetch a generated query
  - Populates the query input field with the generated query (overwrites existing content)
  - Handles errors by showing error messages via `displayError()`
  - Restores button state after completion
- Call `initializeGenerateQuery()` in the DOMContentLoaded event listener

### Step 8: Create E2E Test File
- Read `.claude/commands/test_e2e.md` to understand the E2E test structure
- Read `.claude/commands/e2e/test_basic_query.md` to understand the E2E test format
- Create `.claude/commands/e2e/test_generate_query.md` with:
  - User Story describing the generate query feature
  - Test Steps that:
    1. Navigate to the application
    2. Upload sample data (users or products)
    3. Click the "Generate Query" button
    4. Verify the query input is populated with a generated query
    5. Verify the query is non-empty and maximum two sentences
    6. Click the Query button to execute the generated query
    7. Verify results are displayed successfully
    8. Click Generate Query again to verify it overwrites the previous query
  - Success Criteria listing what must pass
  - Include screenshot capture points for each major step
  - Follow the exact format of existing E2E test files

### Step 9: Run Validation Commands
- Execute all validation commands listed below to ensure zero regressions
- If any tests fail, fix the issues before proceeding
- Verify the E2E test passes by reading and executing it

## Testing Strategy

### Unit Tests
While comprehensive unit tests are ideal, for this feature we'll rely on E2E testing since:
- The backend endpoint is simple and calls existing tested functions
- The frontend interaction is straightforward button click + API call
- E2E tests will validate the full integration
- Future work could add unit tests for `generate_natural_language_query()` function

### Edge Cases
Test the following edge cases:
- **No tables in database**: Should return error message "No tables available. Please upload data first."
- **Single table with few columns**: Should generate a simple, relevant query
- **Multiple tables with many columns**: Should generate an interesting query that could involve one or more tables
- **LLM API failure**: Should handle errors gracefully and display user-friendly error message
- **Network timeout**: Should handle with appropriate timeout and error message
- **Overwriting existing query**: Should completely replace any text in the query input field
- **Multiple rapid clicks**: Button should be disabled during generation to prevent multiple requests

## Acceptance Criteria
- [ ] Generate Query button is added to the UI next to Upload Data button
- [ ] Button has same visual style as Upload Data button (secondary-button class)
- [ ] Button is positioned with justify-content spacing apart from primary Query button
- [ ] Clicking the button calls the backend `/api/generate-query` endpoint
- [ ] Backend uses `llm_processor.py` to generate queries via LLM
- [ ] Generated queries are based on actual table structures in the database
- [ ] Generated queries are maximum two sentences long
- [ ] Generated query overwrites (not appends to) existing content in query input field
- [ ] If no tables exist, user sees helpful error message
- [ ] Button shows loading state during generation
- [ ] Errors are handled gracefully with user-friendly messages
- [ ] E2E test validates the feature works end-to-end
- [ ] All existing tests pass with zero regressions
- [ ] Generated queries can be immediately executed by clicking the Query button

## Validation Commands
Execute every command to validate the feature works correctly with zero regressions.

- Read `.claude/commands/test_e2e.md`, then read and execute the new E2E test file `.claude/commands/e2e/test_generate_query.md` to validate this functionality works
- `cd app/server && uv run pytest` - Run server tests to validate the feature works with zero regressions
- `cd app/client && bun tsc --noEmit` - Run frontend tests to validate the feature works with zero regressions
- `cd app/client && bun run build` - Run frontend build to validate the feature works with zero regressions

## Notes

### LLM Provider Strategy
- The query generation will use the same LLM routing logic as SQL generation (OpenAI priority, fallback to Anthropic)
- This ensures consistency and reuses existing API key configuration
- No additional environment setup required

### Prompt Engineering Considerations
- The prompt should ask for queries that demonstrate interesting capabilities (filtering, aggregation, joins when multiple tables exist)
- Queries should be natural and conversational, not technical
- The two-sentence limit ensures queries are focused and easy to read
- Consider providing variety in generated queries (don't always generate the same type)

### Future Enhancements (Not in Scope)
- Add a "Generate Another" capability to cycle through different query suggestions
- Save/favorite generated queries for later use
- Generate multiple queries at once and let user choose
- Categorize queries by complexity (basic, intermediate, advanced)
- Use query history to avoid generating similar queries repeatedly

### Dependencies
- No new backend dependencies required (uses existing OpenAI and Anthropic clients)
- No new frontend dependencies required (uses existing fetch API)
- Uses existing `uv` for backend package management
- Uses existing `bun` for frontend package management
