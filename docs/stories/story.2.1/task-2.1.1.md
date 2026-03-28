# Task 2.1.1: Create Suppliers Table PostgreSQL Schema

## 1. Meta Information
- **Parent Story:** [User Story 2.1: Supplier Onboarding & Profile Management](./user-story-2.1.md)
- **Epic:** [Epic 2: Partner & Supplier Ecosystem (FL-5)](../../epics/EPICS.md)
- **Jira:** FL-35
- **Estimate:** 4h
- **Assignee:** Unassigned
- **Status:** To Do

## 2. Objective
Create the `Suppliers` table in PostgreSQL via a TypeORM migration. This table is the gating prerequisite for all receiving, price mapping, and audit trail workflows — no delivery can be accepted without a verified supplier record.

## 3. Implementation Requirements

### 3.1. Schema Definition
Define a TypeORM entity with the following columns:

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | UUID | Primary Key, generated |
| `name` | VARCHAR(255) | NOT NULL |
| `contact_person` | VARCHAR(255) | NOT NULL |
| `phone` | VARCHAR(20) | NOT NULL |
| `email` | VARCHAR(255) | nullable |
| `location` | TEXT | NOT NULL (district/sector, Rwanda) |
| `bank_details` | JSONB | nullable — store masked representation; full details encrypted at rest |
| `is_active` | BOOLEAN | Default: `true`, NOT NULL |
| `suspension_reason` | TEXT | nullable — required when `is_active` set to `false` |
| `contract_document_url` | TEXT | nullable — S3 pre-signed URL stored here after upload |
| `created_by` | UUID | FK → `Users.id` (NOT NULL) |
| `created_at` | TIMESTAMPTZ | Auto-managed |
| `updated_at` | TIMESTAMPTZ | Auto-managed |

### 3.2. Indexes
- Unique index on `name` (case-insensitive using `lower(name)`) — soft constraint; duplicate warning is handled at API layer, but the DB must support fuzzy uniqueness checks.
- B-tree index on `is_active` for directory filtering queries.
- Index on `created_by` for audit trail joins.

### 3.3. Constraints
- `bank_details` JSONB must **never** be written to `Audit_Logs` in raw form. Add a DB comment to the column: `"SENSITIVE: masked before audit logging"`.
- No cascade deletes. If `created_by` user is deactivated, supplier record must remain intact.
- `suspension_reason` must be checked at application layer: non-null when `is_active = false`.

### 3.4. Migration
- Generate a named migration: `CreateSuppliersTable`.
- Migration must be reversible (`down()` drops the table).
- Seed script for local dev: 3 sample suppliers (Kigali Fresh, Musanze Farms, Great Lakes Produce).

## 4. Acceptance Criteria (Developer Checklist)
- [ ] TypeORM entity created at `src/suppliers/entities/supplier.entity.ts`
- [ ] Migration `CreateSuppliersTable` runs cleanly on a fresh PostgreSQL instance
- [ ] `down()` migration reverts without errors
- [ ] Seed script inserts 3 sample suppliers with `is_active: true`
- [ ] `bank_details` column has the `"SENSITIVE"` DB comment
- [ ] All indexes are present in the migration DDL

## 5. Dependencies
- **Requires:** Epic 1 Story 1.1 (FL-13) — `Users` table must exist for `created_by` FK
- **Unblocks:** FL-43 (Supplier REST API), FL-44 (S3 upload), FL-45 (Profile UI)
