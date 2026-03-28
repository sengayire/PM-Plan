# Task 2.1.4: "Create / Edit Product" Form UI (Next.js)

## 1. Meta Information
- **Parent Story:** [User Story 2.1: Master Product Catalog Management](./user-story-2.1.md)
- **Epic:** Epic 2: Master Product & Supplier Catalog
- **Assignee:** Unassigned
- **Status:** To Do

## 2. Objective
Build a dynamic React form using Next.js for Procurement Managers to provision new catalogue SKUs or edit the parameters of existing active items. Client-side form validation is paramount here to prevent invalid data from frustrating the user post-submission.

## 3. High-Level Requirements

### 3.1. Form Interface
- Design a clean, responsive form (in a full route or side-drawer/modal, depending on UI patterns) featuring inputs for all `Product_Master` schema attributes.
- Reuse the same core form component for both **Create** and **Update** modes. 
  - If placed in "Update" mode, pre-hydrate the form `defaultValues` via a `GET /products/:id` fetch.

### 3.2. Validation & State Management
- Utilize a robust form engine (like `react-hook-form`) combined with schema validation (like `Zod` or `Yup`) to duplicate the backend restrictions on the frontend.
- **Rules to explicitly visually enforce:**
  - `shelf_life_days` must be an integer >= 0.
  - `storage_temp_min` must conceptually be <= `storage_temp_max`.
  - Required fields cannot be submitted empty.

### 3.3. Submission & UX Handling
- Upon submission, disable the primary action button and show a loading spinner to prevent accidental double-clicks creating duplicate SKUs.
- On a successful `201`/`200` HTTP response, trigger an optimistic success Toast notification and automatically redirect the user back to the Product Catalog list route.
- On a `400` or `500` error, catch the response payload and display a clear, human-readable error Toast or inline helper text.

## 4. Acceptance Criteria (Developer Checklist)
- [ ] Client-side validation accurately prevents submission and turns input fields red on invalid entries.
- [ ] Submitting standard data successfully hits `POST /products` or `PATCH /products/:id`.
- [ ] UI effectively locks/spins during asynchronous network transit.
- [ ] Success/Error Toast notifications trigger properly on API resolution.
- [ ] Forms appropriately reset and clear state upon unmounting.
