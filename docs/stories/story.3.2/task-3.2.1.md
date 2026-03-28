# Task 3.2.1: Add storage_type ENUM to Product_Master Schema and Update Form UI

## 1. Meta Information
- **Parent Story:** [User Story 3.2: Product Category & Storage Classification](./user-story-3.2.md)
- **Epic:** [Epic 3: Unified Product Catalog (FL-6)](../../epics/EPICS.md)
- **Jira:** FL-48
- **Estimate:** 4h
- **Assignee:** Unassigned
- **Status:** To Do

## 2. Objective
Verify that `storage_type` is fully implemented in the `Product_Master` schema (FL-31 should have added it) and ensure the Create/Edit Product form UI (FL-34) correctly exposes the field, validates it, and shows a visual preview badge. This task is primarily a verification + integration task — if FL-31 is complete, much of the backend work is already done.

## 3. Implementation Requirements

### 3.1. Schema Verification
Confirm that FL-31 migration includes:
- `storage_type_enum` PostgreSQL ENUM with values: `COLD_CHAIN`, `DRY`, `FROZEN`, `AMBIENT`
- `storage_type` column on `Product_Master` with `NOT NULL` constraint and default `'DRY'`
- Index on `storage_type` column

If FL-31 is not yet complete, this task blocks on it and must not proceed independently.

### 3.2. Migration for Existing Products (Data Migration)
If `Product_Master` already has rows without `storage_type` (e.g., from a pre-story migration), write a data migration:
```sql
UPDATE "Product_Master"
SET storage_type = 'DRY'
WHERE storage_type IS NULL;
```
- Migration name: `BackfillStorageTypeToDry`
- Add a DB comment to the migration: `"Default agreed with Warehouse Manager — reclassify manually post-deploy"`
- This migration must be idempotent (safe to run multiple times).

### 3.3. API Enforcement
Verify that the Product Catalog REST API (FL-32) enforces:
- `storage_type` is **required** in `CreateProductDto` — `@IsEnum(StorageTypeEnum)`, `@IsNotEmpty()`
- `GET /products?storage_type=COLD_CHAIN` filter works correctly
- `PATCH /products/:id` with `{ storage_type: "FROZEN" }` updates the field and triggers an audit log change entry

### 3.4. Form UI Update (FL-34 integration check)
Verify the Create/Edit Product form (FL-34) has:
- `storage_type` dropdown with all 4 ENUM values — **required field** (no empty option allowed after initial selection)
- Live `StorageTypeBadge` preview next to the dropdown (colour-coded as per FL-33 spec)
- When `storage_type` is changed on an **existing** product with active batches, display a warning below the dropdown:
  > "This product has active warehouse batches. The Warehouse Manager must physically relocate them after changing the storage type."

### 3.5. Storage Type Change Warning (Edit Mode)
- Frontend: display the relocation warning as soon as the `storage_type` value diverges from the saved value.
- Backend: when `PATCH /products/:id` changes `storage_type`, the response should include:
```json
{
  "data": { ...updatedProduct },
  "warning": "Storage type changed. Active batches in the old zone must be physically relocated by the Warehouse Manager."
}
```
- Active batch count can be retrieved from the Inventory module in a future sprint; for now, check using a count query on `Inventory_Batches` where `product_id` matches and `status = 'ACTIVE'` (if the table exists by Sprint 2 — otherwise, skip count and always show the warning on storage_type change).

## 4. Acceptance Criteria (Developer Checklist)
- [ ] `Product_Master.storage_type` column exists with `NOT NULL` constraint and `DRY` default
- [ ] `BackfillStorageTypeToDry` migration runs idempotently and assigns `DRY` to all null rows
- [ ] `POST /products` without `storage_type` returns `400`
- [ ] `GET /products?storage_type=COLD_CHAIN` returns only cold-chain products
- [ ] Edit form shows relocation warning when `storage_type` value is changed from saved value
- [ ] `PATCH /products/:id` with changed `storage_type` returns `200` with `warning` field

## 5. Dependencies
- **Requires:** FL-31 (Product_Master schema — ENUM must exist)
- **Requires:** FL-32 (Product Catalog REST API — PATCH endpoint)
- **Requires:** FL-34 (Product form UI — dropdown integration)
- **Provides data contract for:** Epic 4 (receiving zone validation reads `storage_type`), Epic 9 (temp log scoping reads `storage_type IN ('COLD_CHAIN', 'FROZEN')`)
