# Task 3.2.2: Build Category Management API and Admin UI

## 1. Meta Information
- **Parent Story:** [User Story 3.2: Product Category & Storage Classification](./user-story-3.2.md)
- **Epic:** [Epic 3: Unified Product Catalog (FL-6)](../../epics/EPICS.md)
- **Jira:** FL-49
- **Estimate:** 8h
- **Assignee:** Unassigned
- **Status:** To Do

## 2. Objective
Build the `Product_Categories` table, its CRUD API, and a lightweight Admin UI for managing product categories. Categories provide a structured taxonomy for catalog filtering and are a selectable field on the Product Create/Edit form.

## 3. Implementation Requirements

### 3.1. Database Schema — `Product_Categories` Table

| Column | Type | Constraints |
|--------|------|-------------|
| `id` | UUID | Primary Key, generated |
| `name` | VARCHAR(100) | NOT NULL, UNIQUE (case-insensitive) |
| `description` | TEXT | nullable |
| `is_active` | BOOLEAN | Default: `true`, NOT NULL |
| `created_by` | UUID | FK → `Users.id`, NOT NULL |
| `created_at` | TIMESTAMPTZ | Auto-managed |
| `updated_at` | TIMESTAMPTZ | Auto-managed |

- Migration name: `CreateProductCategoriesTable`
- Add FK on `Product_Master.category_id` → `Product_Categories.id` (nullable, no cascade delete)
- Seed data: Vegetables, Fruits, Grains & Legumes, Roots & Tubers, Herbs & Spices

### 3.2. REST API — CategoriesController

| Method | Route | Description | RBAC |
|--------|-------|-------------|------|
| POST | `/categories` | Create new category | Admin |
| GET | `/categories` | List all active categories | All authenticated |
| PATCH | `/categories/:id` | Edit name/description | Admin |
| DELETE | `/categories/:id` | Soft-delete (set `is_active: false`) | Admin |

**DELETE constraint:** Before soft-deleting, check if any `Product_Master` record with `is_active: true` references this `category_id`. If yes, return `409 Conflict`:
```json
{
  "statusCode": 409,
  "message": "Cannot delete category: 3 active products still reference it. Reassign those products first."
}
```

**POST constraint:** Check `lower(name)` uniqueness before insert; return `409 Conflict` if name already exists (even if soft-deleted).

### 3.3. Admin UI — Category Management Page
- Route: `/admin/categories`
- Simple two-panel layout:
  - **Left panel:** List of all categories (name, product count, status badge, Edit / Delete buttons)
  - **Right panel:** "New Category" inline form (Name + optional Description) with Save button

**Product count:** Display `"X products"` badge next to each category name using a count query on `Product_Master`.

**Delete flow:**
- Delete button triggers confirmation modal: "Are you sure you want to deactivate '{name}'?"
- If the backend returns `409`, display the error inline: "Cannot delete — reassign the 3 active products first."

**Instant availability:** After creating a new category via the form, it must appear immediately in the Product Create/Edit form's category dropdown without a page refresh (React Query cache invalidation).

### 3.4. Product Form Integration (FL-34 Update)
Update the Create/Edit Product form to:
- Activate the `category_id` dropdown (currently disabled "Coming soon" per FL-34 spec)
- Load categories from `GET /categories` via React Query
- Category is optional (no required validation)

## 4. Acceptance Criteria (Developer Checklist)
- [ ] `CreateProductCategoriesTable` migration runs cleanly with FK to Product_Master
- [ ] `POST /categories` returns `409` on duplicate name (case-insensitive)
- [ ] `DELETE /categories/:id` returns `409 Conflict` when active products reference the category
- [ ] `DELETE /categories/:id` succeeds (soft-delete) when no active products reference it
- [ ] Category Management page lists all categories with product count
- [ ] New category appears in product form dropdown immediately after creation (no page refresh)
- [ ] Admin role can create/edit/delete categories; non-Admin receives `403`

## 5. Dependencies
- **Requires:** FL-31 (Product_Master schema — `category_id` FK column must exist)
- **Requires:** FL-18 (RBAC middleware — Admin role guard)
- **Requires:** FL-34 (Product form — category dropdown activation)
- **Unblocks:** FL-50 (dependency contract doc — categories are part of the Epic 4/9 data contract)
