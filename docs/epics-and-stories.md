# Epics and Stories - TODO App Upgrade

Based on [docs/prd-todo.md](docs/prd-todo.md), this document translates MVP and Post-MVP scope into implementation-ready epics and stories.

## MVP Epics

### Epic: Extend Task Data Model for Due Date and Priority

#### Story: Add priority support to backend task model and APIs
Acceptance Criteria:
- New tasks support a priority field with allowed values P1, P2, P3.
- If priority is omitted during create, the saved value defaults to P3.
- Existing endpoints include priority in response payloads.
- Invalid priority values are rejected with a 400 response.

Technical Requirements:
- Update database schema in [packages/backend/src/app.js](packages/backend/src/app.js) to add priority column with default P3.
- Update POST and PUT handlers in [packages/backend/src/app.js](packages/backend/src/app.js) to read/write priority.
- Keep API payload compatibility with current shape already used by frontend (including existing due_date naming).
- Update backend tests in [packages/backend/__tests__/tasks.test.js](packages/backend/__tests__/tasks.test.js) to verify priority create/update/default behavior.

#### Story: Enforce due date validation for create and update
Acceptance Criteria:
- dueDate/due_date remains optional.
- Valid ISO format YYYY-MM-DD is accepted.
- Invalid date values are ignored and treated as absent.
- Tasks with invalid dates are saved with null due date rather than causing server errors.

Technical Requirements:
- Add date validation helper in [packages/backend/src/app.js](packages/backend/src/app.js) for strict YYYY-MM-DD parsing.
- Normalize incoming due_date values in POST and PUT handlers before database writes.
- Keep current list ordering behavior compatible with due_date null values.
- Add/expand backend tests in [packages/backend/__tests__/tasks.test.js](packages/backend/__tests__/tasks.test.js) for valid and invalid date paths.

#### Story: Preserve local-only storage architecture
Acceptance Criteria:
- Task persistence remains local to the running application.
- No external storage providers or backend service dependencies are introduced.
- Existing API base path remains /api/tasks.

Technical Requirements:
- Keep current in-process SQLite strategy in [packages/backend/src/app.js](packages/backend/src/app.js) (no cloud integrations).
- Do not add new external data services, SDKs, or remote persistence code.
- Ensure frontend calls stay on relative API paths in [packages/frontend/src/App.js](packages/frontend/src/App.js) and [packages/frontend/src/TaskList.js](packages/frontend/src/TaskList.js).

### Epic: Enable MVP Task Input and Editing in UI

#### Story: Add priority input to task creation and edit form
Acceptance Criteria:
- Task form allows selecting priority P1, P2, or P3.
- New task submissions default to P3 when no explicit selection is made.
- Edit mode shows the task's current priority.
- Saved tasks round-trip with the selected priority value.

Technical Requirements:
- Extend form state in [packages/frontend/src/TaskForm.js](packages/frontend/src/TaskForm.js) with priority.
- Add priority UI control using existing Material UI components in [packages/frontend/src/TaskForm.js](packages/frontend/src/TaskForm.js).
- Include priority in payload sent through onSave from [packages/frontend/src/TaskForm.js](packages/frontend/src/TaskForm.js) to [packages/frontend/src/App.js](packages/frontend/src/App.js).
- Update frontend tests in [packages/frontend/src/__tests__/App.test.js](packages/frontend/src/__tests__/App.test.js) and mock API handlers for priority.

#### Story: Keep due date input aligned with validated ISO format
Acceptance Criteria:
- Form continues to accept due date as YYYY-MM-DD.
- Editing an existing task pre-populates a valid date value.
- Invalid values do not break submit flow and are treated as no date by backend.

Technical Requirements:
- Retain and adjust normalization behavior in [packages/frontend/src/TaskForm.js](packages/frontend/src/TaskForm.js) to emit date-only strings.
- Ensure due_date remains included in create/update payloads generated in [packages/frontend/src/TaskForm.js](packages/frontend/src/TaskForm.js).
- Verify TaskList date rendering remains compatible with null due_date in [packages/frontend/src/TaskList.js](packages/frontend/src/TaskList.js).

#### Story: Keep title as required field across UI and API
Acceptance Criteria:
- User cannot submit an empty title from UI.
- API rejects empty or whitespace-only title with clear error.
- Behavior is consistent for both create and edit flows.

Technical Requirements:
- Keep existing UI validation in [packages/frontend/src/TaskForm.js](packages/frontend/src/TaskForm.js) and ensure error message display remains intact.
- Keep existing API validation guardrails in POST/PUT handlers in [packages/backend/src/app.js](packages/backend/src/app.js).
- Add regression tests in [packages/frontend/src/__tests__/App.test.js](packages/frontend/src/__tests__/App.test.js) for empty-title submission.

### Epic: Add Date-Based Filters (All, Today, Overdue)

#### Story: Add filter controls to task list view
Acceptance Criteria:
- UI exposes three filter options: All, Today, Overdue.
- All is selected by default when app loads.
- User can switch filters without page reload.

Technical Requirements:
- Add filter state and controls in [packages/frontend/src/TaskList.js](packages/frontend/src/TaskList.js) using existing Material UI patterns.
- Keep task fetch integration unchanged at /api/tasks unless optional server-side filtering is intentionally introduced.
- Add frontend tests in [packages/frontend/src/__tests__/App.test.js](packages/frontend/src/__tests__/App.test.js) for filter visibility and switching.

#### Story: Implement Today and Overdue filter logic
Acceptance Criteria:
- Today filter returns tasks with due date equal to current local date.
- Overdue filter returns tasks with due date before current local date.
- Tasks without due dates are excluded from Today and Overdue.
- All filter continues to show all tasks.

Technical Requirements:
- Implement deterministic date comparison utility in [packages/frontend/src/TaskList.js](packages/frontend/src/TaskList.js) using date-only semantics.
- Reuse existing fetchTasks data source and derive filtered lists in UI state.
- Add tests for date edge cases in [packages/frontend/src/__tests__/App.test.js](packages/frontend/src/__tests__/App.test.js) with mocked task dates.

## Post-MVP Epics

### Epic: Improve Visual Priority and Urgency Cues

#### Story: Visually highlight overdue tasks in task list
Acceptance Criteria:
- Overdue tasks are visually distinct from non-overdue tasks.
- Highlighting applies consistently wherever overdue tasks are shown.
- Completed tasks retain readable styling with overdue cues.

Technical Requirements:
- Add overdue style computation in [packages/frontend/src/TaskList.js](packages/frontend/src/TaskList.js) based on due_date and current date.
- Update item/chip styling in [packages/frontend/src/TaskList.js](packages/frontend/src/TaskList.js) to include overdue state without breaking current theme.
- Add UI tests validating overdue style markers in [packages/frontend/src/__tests__/App.test.js](packages/frontend/src/__tests__/App.test.js).

#### Story: Add color-coded priority badges for P1, P2, P3
Acceptance Criteria:
- Each task displays its priority badge.
- Badge colors map clearly by level (P1 highest urgency, P2 medium, P3 lowest).
- Priority badges display correctly in all task states.

Technical Requirements:
- Extend task row presentation in [packages/frontend/src/TaskList.js](packages/frontend/src/TaskList.js) to render priority chip/badge.
- Use a centralized mapping of priority-to-style in [packages/frontend/src/TaskList.js](packages/frontend/src/TaskList.js) for consistency.
- Add tests in [packages/frontend/src/__tests__/App.test.js](packages/frontend/src/__tests__/App.test.js) to verify badge labels and presence.

### Epic: Implement Advanced Task Sorting

#### Story: Apply Post-MVP sort precedence for task ordering
Acceptance Criteria:
- Task order is computed as: overdue first, then priority (P1 to P3), then due date ascending, then undated tasks last.
- Sorting is stable and predictable across repeated renders.
- Sorting behavior is compatible with active filters.

Technical Requirements:
- Implement comparator utility in [packages/frontend/src/TaskList.js](packages/frontend/src/TaskList.js) that composes overdue, priority, and due date rules.
- Ensure comparator handles null due_date and missing priority safely.
- Add focused test coverage in [packages/frontend/src/__tests__/App.test.js](packages/frontend/src/__tests__/App.test.js) for full precedence cases.

#### Story: Keep backend list ordering compatible with frontend advanced sorting
Acceptance Criteria:
- Frontend remains source of truth for advanced Post-MVP ordering.
- Backend default order does not conflict with frontend final order.
- API continues returning complete fields needed for sorting.

Technical Requirements:
- Keep or simplify SQL ORDER BY in [packages/backend/src/app.js](packages/backend/src/app.js) to avoid contradictory ordering guarantees.
- Ensure GET /api/tasks response includes due_date, priority, completed, and created_at.
- Add backend API tests in [packages/backend/__tests__/tasks.test.js](packages/backend/__tests__/tasks.test.js) to verify required sorting fields are present.
