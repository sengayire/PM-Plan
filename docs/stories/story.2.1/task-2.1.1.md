# Task 2.1.1: Product_Master Database Schema Initialization

## 1. Meta Information
- **Parent Story:** [User Story 2.1: Master Product Catalog Management](./user-story-2.1.md)
- **Epic:** Epic 2: Master Product & Supplier Catalog
- **Assignee:** Unassigned
- **Status:** To Do

## 2. Objective
Create the foundational `Product_Master` database schema in PostgreSQL. This table is the central source of truth for all physical items flowing through AgriFlow, defining the universal operational rules (like shelf-life and storage requirements) necessary for the "Digital Twin" to automate expiry logic and inventory mapping.

## 3. Implementation Requirements

### 3.1. Entity Schema Structure
Define the entity using the system's chosen ORM (e.g., TypeORM or Prisma) to generate the database migration. 
- **Required Columns:**
  - `id`: UUID (Primary Key).
  - `sku_code`: VARCHAR (Must be strictly `UNIQUE`).
  - `name`: VARCHAR (e.g., "Rwandan Round Tomato").
  - `category`: VARCHAR (Indexed for faster UI filtering, e.g., "Vegetables").
  - `shelf_life_days`: INTEGER (Must be greater than or equal to 0).
  - `storage_temp_min`: DECIMAL (For FDA temperature logging compliance).
  - `storage_temp_max`: DECIMAL.
  - `unit_of_measure`: ENUM (`KG`, `PCS`, `BUNDLE`).
  - `is_active`: BOOLEAN (Default: `true`).

### 3.2. Auditing & Constraints
- Automatically manage standard timestamp columns: `created_at`, `updated_at`.
- Implement a database-level constraint checking that `storage_temp_min` is logically less than or equal to `storage_temp_max`.
- Do NOT implement hard-delete cascading. System records (`is_active: false`) must remain intact indefinitely for ledger accuracy.

## 4. Acceptance Criteria (Developer Checklist)
- [ ] ORM Entity/Model is created for the `Product_Master` schema.
- [ ] Database migration script runs without errors on a clean PostgreSQL instance.
- [ ] Schema successfully enforces the unique `sku_code` constraint on the DB level.
- [ ] A sample seed script is provided for local development containing at least 3 dummy SKUs (e.g., Tomatoes, Potatoes, Avocados).

## 5. Security & Dependencies
- Depends on the core database instantiation.
- Serves as the foreign key target for Epic 2.2 (Supplier Mapping) and Epic 4 (Inventory).
