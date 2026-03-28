# Task 1.2.3: Build Audit Log Search UI

## 1. Meta Information
- **Parent Story:** [User Story 1.2: Immutable Audit Logging](./user-story-1.2.md)
- **Epic:** [Epic 1: Identity & Access Management (IAM)](../../epics/EPICS.md)
- **Jira:** FL-24
- **Assignee:** Unassigned
- **Status:** To Do
- **Estimation:** 6h

## 2. Objective
Build the frontend interface that allows System Auditors and Admins to search, filter, and review audit log entries. This is the primary compliance tool for Rwanda FDA inspection readiness and internal discrepancy investigation.

## 3. Implementation Requirements

### 3.1. Backend API Endpoint
Build a `GET /audit-logs` endpoint with the following query parameters:
- `user_id` — filter by acting user
- `entity_type` — filter by affected entity (e.g., `Batch`, `Product`)
- `action` — filter by action type (`CREATE`, `UPDATE`, `DELETE`)
- `date_from` / `date_to` — date range filter (ISO 8601)
- `limit` / `offset` — server-side pagination
- Response must include total count for pagination controls.
- Restrict to `Admin` and `Auditor` roles via RBAC guard.

### 3.2. Frontend Data Table
- Implement a read-only data table with columns: `Timestamp`, `User`, `Action`, `Entity Type`, `Entity ID`, `Old Value`, `New Value`.
- `Old Value` and `New Value` columns display a clickable diff viewer (JSON comparison modal) when the values are non-null.
- Implement server-side pagination.

### 3.3. Filter Panel
- Date range picker (from/to).
- User dropdown (searchable, populated from `GET /users`).
- Entity Type dropdown (static list: Batch, Product, Supplier, User, Order).
- Action type multi-select checkboxes.

## 4. Acceptance Criteria (Developer Checklist)
- [ ] Filtering by date range returns only entries within the specified window.
- [ ] Filtering by `user_id` shows only that user's actions.
- [ ] Clicking a row with `old_value`/`new_value` opens a readable JSON diff modal.
- [ ] The API returns results in under 2 seconds for queries spanning 90 days of data.
- [ ] The route is restricted to Admin/Auditor roles — other roles receive `403 Forbidden`.

## 5. Security & Dependencies
- Depends on Task 1.2.1 (Audit_Logs table) and Task 1.2.2 (interceptor populating the table).
- Depends on Task 1.1.3 (RBAC middleware) for role-based route protection.
