# Task 2.2.1: Create Suppliers & Supplier_Product_Map PostgreSQL Schemas

## 1. Meta Information
- **Parent Story:** [User Story 2.2: Supplier Registration & Product Price Book](./user-story-2.2.md)
- **Epic:** Epic 2: Partner & Supplier Ecosystem (FL-5)
- **Jira:** FL-35
- **Assignee:** Unassigned
- **Status:** To Do
- **Estimation:** 4h

## 2. Objective
Create the two foundational database schemas that power the Supplier Ecosystem: the `Suppliers` master registry and the `Supplier_Product_Map` junction table that establishes agreed pricing between suppliers and AgriFlow products.

## 3. Implementation Requirements

### 3.1. Suppliers Table
Define the ORM entity:
- `id`: UUID (Primary Key)
- `name`: VARCHAR (NOT NULL)
- `contact_number`: VARCHAR
- `location`: VARCHAR
- `bank_details`: JSONB (store bank name, account number, branch — structured but flexible)
- `is_active`: BOOLEAN (Default: `true`)
- `created_at`, `updated_at`: TIMESTAMP (auto-managed)

### 3.2. Supplier_Product_Map Junction Table
- `id`: UUID (Primary Key)
- `supplier_id`: UUID (FK → `Suppliers.id`, NOT NULL)
- `product_id`: UUID (FK → `Product_Master.id`, NOT NULL)
- `agreed_price`: DECIMAL(12, 2) (NOT NULL, must be > 0)
- `currency`: VARCHAR(3) (Default: `'RWF'`)
- `updated_at`: TIMESTAMP (auto-managed on every price update)
- **Unique compound index** on `(supplier_id, product_id)` — enforced at the DB level to prevent duplicate mappings.

### 3.3. Constraints
- No hard deletes on either table.
- `Supplier_Product_Map.agreed_price` must have a DB-level check constraint: `agreed_price > 0`.

## 4. Acceptance Criteria (Developer Checklist)
- [ ] Migration for `Suppliers` table runs cleanly on a fresh PostgreSQL instance.
- [ ] Migration for `Supplier_Product_Map` runs with correct FK references to both `Suppliers` and `Product_Master`.
- [ ] Attempting to insert a duplicate `(supplier_id, product_id)` pair throws a unique constraint violation.
- [ ] Seed script populates at least 2 dummy suppliers with 2 product mappings each.

## 5. Security & Dependencies
- Depends on Task 2.1.1 (Product_Master schema must exist for FK reference).
- The `Supplier_Product_Map` table is referenced by Epic 4 (Smart Receiving) when suggesting agreed price for a received shipment.
