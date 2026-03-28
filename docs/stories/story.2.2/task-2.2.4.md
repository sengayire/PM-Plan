# Task 2.2.4: Build Supplier Price Book UI (Nested Product Mapping)

## 1. Meta Information
- **Parent Story:** [User Story 2.2: Supplier Registration & Product Price Book](./user-story-2.2.md)
- **Epic:** Epic 2: Partner & Supplier Ecosystem (FL-5)
- **Jira:** FL-38
- **Assignee:** Unassigned
- **Status:** To Do
- **Estimation:** 6h

## 2. Objective
Build the "Price Book" tab within the Supplier Profile page where Buyers can manage the complete list of products a supplier is approved to deliver along with their agreed prices.

## 3. High-Level Requirements

### 3.1. Supplier Profile Page (`/suppliers/:id`)
- Display supplier details (name, contact, location, status) in a header card.
- Tabbed interface: "Details" and "Price Book".

### 3.2. Price Book Tab
- Data table: Product Name, SKU Code, Agreed Price (RWF), Currency, Last Updated.
- "Add Product Mapping" CTA button.
- **Add Mapping Flow:**
  - Searchable dropdown of `Product_Master` SKUs (calls `GET /products?q=`).
  - Price input (numeric, required, > 0).
  - Submit calls `POST /suppliers/:id/products`.
  - On `409 Conflict`: show error toast "This product is already mapped. Edit the existing price instead."
  - On success: add row to table, show success toast.
- **Edit Price:** Inline "Edit" action on each row — opens a small inline form to update the price. Calls `PATCH /suppliers/mappings/:id`.
- Confirm before marking a supplier `is_active: false` (soft-delete button on Details tab).

## 4. Acceptance Criteria (Developer Checklist)
- [ ] Price Book tab correctly loads all product mappings for the supplier.
- [ ] The product search dropdown returns live filtered results from `GET /products`.
- [ ] Submitting a duplicate mapping shows the `409` error toast.
- [ ] "Edit Price" inline form updates the price without a full page reload.
- [ ] Price changes are immediately reflected in the table after a successful PATCH.
