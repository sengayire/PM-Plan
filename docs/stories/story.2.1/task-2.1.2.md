# Task 2.1.2: Build Supplier Registration & Profile REST API (NestJS)

## 1. Meta Information
- **Parent Story:** [User Story 2.1: Supplier Onboarding & Profile Management](./user-story-2.1.md)
- **Epic:** [Epic 2: Partner & Supplier Ecosystem (FL-5)](../../epics/EPICS.md)
- **Jira:** FL-43
- **Estimate:** 8h
- **Assignee:** Unassigned
- **Status:** To Do

## 2. Objective
Build the NestJS `SuppliersController` with full CRUD, fuzzy duplicate detection, and soft-delete with mandatory suspension reason. This API is the single source of truth for supplier status across all downstream workflows.

## 3. Implementation Requirements

### 3.1. Endpoints

| Method | Route | Description | RBAC |
|--------|-------|-------------|------|
| POST | `/suppliers` | Register new supplier | Procurement Manager, Admin |
| GET | `/suppliers` | Paginated directory with filters | All authenticated |
| GET | `/suppliers/:id` | Full supplier profile | All authenticated |
| PATCH | `/suppliers/:id` | Update profile fields | Procurement Manager, Admin |
| PATCH | `/suppliers/:id/suspend` | Toggle `is_active: false` with reason | Procurement Manager, Admin |
| PATCH | `/suppliers/:id/activate` | Re-activate suspended supplier | Admin only |

### 3.2. Fuzzy Duplicate Detection (POST /suppliers)
- Before inserting, query existing active suppliers using `ILIKE '%{name}%'` or `levenshtein()` distance ≤ 3.
- If matches found, return `200` with body:
  ```json
  {
    "warning": "Possible duplicate suppliers found. Review before confirming.",
    "matches": [{ "id": "...", "name": "..." }],
    "requiresConfirmation": true
  }
  ```
- Client must re-submit with `{ "confirmDuplicate": true }` to proceed with creation.
- On confirmed creation, proceed normally and return `201 Created`.

### 3.3. Suspend Endpoint (PATCH /suppliers/:id/suspend)
- Request body: `{ "reason": string }` — reason is **required**; return `400` if empty.
- Sets `is_active: false`, writes `suspension_reason`.
- Writes audit log entry: `{ action: "SUPPLIER_SUSPENDED", entity: "Supplier", entity_id, changes: { is_active: false, reason } }`.
- `bank_details` must be **excluded** from the `changes` payload written to `Audit_Logs`.

### 3.4. List Endpoint (GET /suppliers)
- Query params: `?page=1&limit=20&q=kigali&is_active=true`
- Returns paginated response: `{ data: [], total, page, limit }`.
- Suspended suppliers (`is_active: false`) are hidden from results by default unless `?include_suspended=true` (Admin only).

### 3.5. DTOs & Validation
- `CreateSupplierDto`: all required fields with `@IsNotEmpty()`, `@IsEmail()` on email, `@IsString()` on location.
- `SuspendSupplierDto`: `{ reason: string }` with `@MinLength(10)` — reason must be substantive.
- `UpdateSupplierDto`: all fields optional via `PartialType(CreateSupplierDto)`.

### 3.6. Audit Logging
- All `POST` and `PATCH` mutations must trigger the global audit interceptor (FL-23).
- `bank_details` field must be stripped from audit log `changes` payload before the interceptor writes to `Audit_Logs`.

## 4. Acceptance Criteria (Developer Checklist)
- [ ] `POST /suppliers` returns `{ warning, matches }` when a near-duplicate is found; proceeds on `confirmDuplicate: true`
- [ ] `POST /suppliers` returns `201` on clean create; `400` on validation failure
- [ ] `GET /suppliers` pagination returns correct `total` count and respects `is_active` filter
- [ ] `PATCH /suppliers/:id/suspend` requires a non-empty reason; returns `400` without it
- [ ] `bank_details` is never present in audit log writes
- [ ] `403 Forbidden` returned when non-Procurement/Admin role attempts a mutation
- [ ] Unit tests cover the duplicate-detection logic and suspension flow

## 5. Dependencies
- **Requires:** FL-35 (Suppliers table schema)
- **Requires:** FL-18 (RBAC middleware — Epic 1)
- **Requires:** FL-23 (Audit interceptor — Epic 1)
- **Unblocks:** FL-37 (Supplier Directory UI), FL-44 (S3 upload), FL-45 (Profile UI)
