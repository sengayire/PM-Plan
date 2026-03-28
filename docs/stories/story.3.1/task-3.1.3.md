# Task 3.1.3: Build Product Catalog Data Table UI (Next.js)

## 1. Meta Information
- **Parent Story:** [User Story 3.1: Master Product (SKU) Management](./user-story-3.1.md)
- **Epic:** [Epic 3: Unified Product Catalog (FL-6)](../../epics/EPICS.md)
- **Jira:** FL-33
- **Estimate:** 6h
- **Assignee:** Unassigned
- **Status:** To Do

## 2. Objective
Build the Product Catalog list page — a paginated, searchable data table with storage type color-coding. This is the primary catalog browsing surface for Procurement Managers and Warehouse staff.

## 3. Implementation Requirements

### 3.1. Page Route
- Route: `/products` (Next.js App Router page)
- Server component for initial SSR data load; React Query for subsequent filtering/pagination.

### 3.2. Data Table Columns
| Column | Notes |
|--------|-------|
| SKU Code | Monospace font |
| Name | Bold |
| Category | Text (empty dash if null) |
| Storage Type | Color-coded badge (see 3.3) |
| Shelf Life | "{N} days" |
| Unit of Measure | Chip |
| Status | Active (green) / Inactive (gray) badge |
| Actions | "View / Edit" button |

### 3.3. Storage Type Badges
Color-coding per design spec:
- `COLD_CHAIN` → blue badge
- `DRY` → green badge
- `FROZEN` → cyan badge
- `AMBIENT` → gray badge

Badges must be accessible (WCAG AA contrast ratio) and use consistent Tailwind utility classes defined in a shared `StorageTypeBadge` component.

### 3.4. Filters & Search
- **Search bar:** Debounced (300ms), queries `GET /products?q={term}` across name and SKU
- **Storage Type filter:** Dropdown with all 4 ENUM values + "All" option
- **Status toggle:** "Show inactive" checkbox (hidden from non-Admin/non-Procurement roles)
- Filter state persisted in URL query params (supports deep-linking and browser navigation)

### 3.5. Pagination
- Server-side pagination via React Query
- "Showing X–Y of Z results" counter
- Previous / Next page buttons; page size selector (20 / 50 / 100)

### 3.6. Empty & Loading States
- Loading: skeleton rows matching table column structure
- Empty (no results for filters): "No products match your filters. Try adjusting the search or storage type filter."
- Empty (no products at all): "No products in the catalog yet. Add your first product to get started."

### 3.7. "Add Product" CTA
- Primary button in page header: "Add Product" → navigates to `/products/new`
- Visible only to Procurement Manager and Admin roles

## 4. Acceptance Criteria (Developer Checklist)
- [ ] Table renders all columns with correct data from `GET /products`
- [ ] `StorageTypeBadge` component renders correct color for all 4 ENUM values
- [ ] Search debounces correctly and updates table results
- [ ] Storage Type filter dropdown updates `?storage_type=` query param and refetches
- [ ] Pagination controls work and "Showing X–Y of Z" counter is accurate
- [ ] Skeleton rows render during data fetch
- [ ] "Add Product" button navigates to `/products/new` and is hidden for read-only roles

## 5. Dependencies
- **Requires:** FL-32 (Product Catalog REST API — `GET /products` with filter support)
- **Requires:** FL-19 (Auth.js — role context)
- **Unblocks:** FL-34 (Create/Edit Product form — navigated to from this page)
