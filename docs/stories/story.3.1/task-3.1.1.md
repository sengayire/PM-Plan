# Task 3.1.1: Create Product_Master PostgreSQL Schema (Includes storage_type ENUM)

## 1. Meta Information
- **Parent Story:** [User Story 3.1: Master Product (SKU) Management](./user-story-3.1.md)
- **Epic:** [Epic 3: Unified Product Catalog (FL-6)](../../epics/EPICS.md)
- **Jira:** FL-31
- **Estimate:** 4h
- **Assignee:** Unassigned
- **Status:** To Do

## 2. Objective
Create the `Product_Master` table with the `storage_type` ENUM as a first-class field. This schema is the foundational data layer for every downstream epic — receiving zone enforcement (Epic 4), FDA temperature scoping (Epic 9), and supplier price mapping (Epic 2) all depend on this table existing with full classification fields.

## 3. Implementation Requirements

### 3.1. ENUM Types
Define PostgreSQL ENUM types in the migration:

```sql
CREATE TYPE storage_type_enum AS ENUM ('COLD_CHAIN', 'DRY', 'FROZEN', 'AMBIENT');
CREATE TYPE unit_of_measure_enum AS ENUM ('KG', 'PCS', 'BUNDLE');
```

### 3.2. Schema Definition

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | UUID | Primary Key, generated |
| `sku_code` | VARCHAR(50) | UNIQUE, NOT NULL |
| `name` | VARCHAR(255) | NOT NULL |
| `category_id` | UUID | FK → `Product_Categories.id`, nullable (Story 3.2 adds categories) |
| `shelf_life_days` | INTEGER | NOT NULL, CHECK >= 0 |
| `storage_temp_min` | DECIMAL(5,2) | nullable |
| `storage_temp_max` | DECIMAL(5,2) | nullable |
| `storage_type` | `storage_type_enum` | NOT NULL, Default: `'DRY'` |
| `unit_of_measure` | `unit_of_measure_enum` | NOT NULL |
| `is_active` | BOOLEAN | Default: `true`, NOT NULL |
| `created_by` | UUID | FK → `Users.id`, NOT NULL |
| `created_at` | TIMESTAMPTZ | Auto-managed |
| `updated_at` | TIMESTAMPTZ | Auto-managed |

### 3.3. Check Constraints
- `CHECK (storage_temp_min <= storage_temp_max)` — only enforced when both values are non-null.
- `CHECK (shelf_life_days >= 0)` — negative shelf life is invalid.

### 3.4. Indexes
- Unique index on `lower(sku_code)` — case-insensitive uniqueness.
- B-tree index on `storage_type` — critical for Epic 4 and Epic 9 queries scoping to `COLD_CHAIN` and `FROZEN`.
- B-tree index on `is_active`.
- Index on `category_id` (for Story 3.2 category filtering).

### 3.5. Migration Strategy
- Migration name: `CreateProductMasterTable`.
- `category_id` is nullable at this stage — Story 3.2 (FL-42) will create the `Product_Categories` table. Products created before categories are added must not be blocked.
- `down()` must drop the table and the ENUM types in correct dependency order.

### 3.6. Seed Data
Provide a local dev seed script with at least 5 products spanning all four `storage_type` values:
- Rwandan Round Tomato (COLD_CHAIN)
- Maize Grain (DRY)
- Frozen Green Beans (FROZEN)
- Avocado (COLD_CHAIN)
- Irish Potatoes (AMBIENT)

## 4. Acceptance Criteria (Developer Checklist)
- [ ] Migration `CreateProductMasterTable` runs cleanly on fresh PostgreSQL
- [ ] `storage_type_enum` and `unit_of_measure_enum` ENUM types are created before the table
- [ ] `sku_code` unique constraint enforced at DB level (case-insensitive)
- [ ] `CHECK (shelf_life_days >= 0)` is enforced at DB level
- [ ] `CHECK (storage_temp_min <= storage_temp_max)` enforced when both non-null
- [ ] Index on `storage_type` is present (verified via `\d Product_Master` in psql)
- [ ] `down()` migration reverts cleanly without FK errors
- [ ] Seed script inserts 5 products covering all `storage_type` values

## 5. Dependencies
- **Requires:** Epic 1 Story 1.1 (FL-13) — `Users` table must exist for `created_by` FK
- **Unblocks:** FL-32 (Product Catalog REST API), FL-33 (Catalog UI), FL-34 (Form UI)
- **Architectural note:** `storage_type` field is an explicit contract for Epic 4 (receiving zone validation) and Epic 9 (temperature log scoping). Any schema change to this ENUM must be reviewed by the Epic 4 and Epic 9 leads.
