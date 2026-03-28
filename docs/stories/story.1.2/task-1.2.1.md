# Task 1.2.1: Create Audit_Logs Table Schema

## 1. Meta Information
- **Parent Story:** [User Story 1.2: Immutable Audit Logging](./user-story-1.2.md)
- **Epic:** [Epic 1: Identity & Access Management (IAM)](../../epics/EPICS.md)
- **Jira:** FL-22
- **Assignee:** Unassigned
- **Status:** To Do
- **Estimation:** 3h

## 2. Objective
Design and create the `Audit_Logs` PostgreSQL table — the immutable, append-only ledger that records every critical state change on the platform. This table is the foundation for Rwanda FDA compliance and internal discrepancy resolution.

## 3. Implementation Requirements

### 3.1. Entity Schema Structure
Define the entity using the system ORM (TypeORM or Prisma):
- **Required Columns:**
  - `id`: UUID (Primary Key, auto-generated)
  - `user_id`: UUID (Foreign Key → `Users.id`, NOT NULL)
  - `action`: ENUM (`CREATE`, `UPDATE`, `DELETE`, `LOGIN`, `LOGOUT`, `EXPORT`)
  - `entity_type`: VARCHAR (e.g., `Batch`, `Product`, `User`, `Supplier`)
  - `entity_id`: UUID (The ID of the affected record)
  - `old_value`: JSONB (Snapshot of data before mutation — NULL for CREATE)
  - `new_value`: JSONB (Snapshot of data after mutation — NULL for DELETE)
  - `ip_address`: VARCHAR (Request origin, for security tracking)
  - `created_at`: TIMESTAMP (Immutable, set at insert time)

### 3.2. Immutability Constraints
- **No `updated_at`** column — this table is append-only.
- Apply a PostgreSQL-level **row-level security rule** or trigger to reject any `UPDATE` or `DELETE` statements on this table.
- Do NOT expose any `DELETE` or `PATCH` endpoint for audit logs in the API layer.

### 3.3. Performance Indexing
Create composite indexes on:
- `(user_id, created_at)` — for user-specific activity queries
- `(entity_type, entity_id)` — for entity-specific audit history
- `(created_at)` — for date range queries (supports date-based partitioning later)

## 4. Acceptance Criteria (Developer Checklist)
- [ ] ORM entity/model is created for `Audit_Logs`.
- [ ] Migration script runs on a clean PostgreSQL instance without errors.
- [ ] Attempting to `UPDATE` or `DELETE` an audit log row throws a database-level error.
- [ ] All required indexes are created and verified in the migration.

## 5. Security & Dependencies
- Depends on `Users` table from Task 1.1.1.
- This table serves as the target for the NestJS interceptor built in Task 1.2.2.
