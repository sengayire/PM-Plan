# Task 3.1.4: Build Create/Edit Product Form UI (Next.js)

## 1. Meta Information
- **Parent Story:** [User Story 3.1: Master Product (SKU) Management](./user-story-3.1.md)
- **Epic:** [Epic 3: Unified Product Catalog (FL-6)](../../epics/EPICS.md)
- **Jira:** FL-34
- **Estimate:** 6h
- **Assignee:** Unassigned
- **Status:** To Do

## 2. Objective
Build the Create and Edit Product form — a single shared component that handles both new product creation and existing product editing. Strong client-side validation mirrors backend rules to prevent invalid data submission and provide immediate feedback.

## 3. Implementation Requirements

### 3.1. Page Routes
- Create: `/products/new`
- Edit: `/products/[id]/edit`
- Both routes use the same `ProductForm` component; Edit mode pre-populates via `GET /products/:id`.

### 3.2. Form Fields
| Field | Input Type | Validation |
|-------|-----------|------------|
| SKU Code | Text | Required, max 50 chars; readonly in Edit mode |
| Name | Text | Required |
| Storage Type | Select (ENUM dropdown) | Required; shows color-coded preview badge next to dropdown |
| Category | Select (async from categories API) | Optional (Story 3.2 adds this) — render as disabled "Coming soon" in Sprint 2 |
| Shelf Life (days) | Number | Required, integer ≥ 0 |
| Unit of Measure | Select (KG / PCS / BUNDLE) | Required |
| Min Temp (°C) | Number | Optional; must be ≤ Max Temp when both entered |
| Max Temp (°C) | Number | Optional; must be ≥ Min Temp when both entered |

### 3.3. Validation (react-hook-form + Zod)
- All required fields block submission with inline error messages below the field
- Cross-field validation: if `storage_temp_min` and `storage_temp_max` both have values, validate `min ≤ max` at schema level
- SKU Code: pattern validation — alphanumeric + hyphens only (no spaces)
- `shelf_life_days`: reject decimal input

### 3.4. Storage Type Preview
- When the user selects a `storage_type` value, render the corresponding color-coded badge (from `StorageTypeBadge` component built in FL-33) next to the dropdown as a live preview.

### 3.5. Submission Behavior
- On submit: disable button + show spinner to prevent double-submit
- Success (`201` / `200`): toast "Product saved successfully" → redirect to `/products`; React Query cache invalidated
- Error (`400`): map API field errors to form field errors (inline display); generic toast for unexpected errors
- Error (`409` — duplicate SKU): inline error on SKU Code field: "This SKU code already exists."

### 3.6. Edit Mode — Shelf Life Change Warning
- If editing an existing product and `shelf_life_days` value is changed, display an inline warning below the field:
  > "Changing shelf life only affects future batches. Existing inventory retains original expiry dates."
- This warning appears as soon as the value diverges from the original (no save required).

### 3.7. Deactivate Product (Edit Mode Only)
- In Edit mode, show a "Deactivate Product" button (destructive, red, bottom of form)
- Triggers a confirmation modal: "Deactivating this product will remove it from all future receiving and ordering flows. Existing inventory is unaffected."
- On confirm: `PATCH /products/:id` with `{ is_active: false }` → redirect to `/products`

## 4. Acceptance Criteria (Developer Checklist)
- [ ] Form renders all fields; Edit mode pre-populates from API
- [ ] SKU Code field is readonly in Edit mode
- [ ] Storage Type selection updates the preview badge in real-time
- [ ] `min ≤ max` cross-field validation triggers on blur
- [ ] Submit disables and shows spinner; re-enables on API response
- [ ] `409 Conflict` (duplicate SKU) maps to inline SKU field error
- [ ] Shelf life change in Edit mode shows inline warning immediately
- [ ] "Deactivate Product" button visible in Edit mode only; confirmation modal required

## 5. Dependencies
- **Requires:** FL-32 (Product Catalog REST API — POST and PATCH endpoints)
- **Requires:** FL-33 (`StorageTypeBadge` component)
- **Requires:** FL-19 (Auth.js — guard this route to Procurement Manager / Admin)
