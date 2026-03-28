# Task 2.2.2: Build Supplier & Price Book REST API (NestJS)

## 1. Meta Information
- **Parent Story:** [User Story 2.2: Supplier Registration & Product Price Book](./user-story-2.2.md)
- **Epic:** Epic 2: Partner & Supplier Ecosystem (FL-5)
- **Jira:** FL-36
- **Assignee:** Unassigned
- **Status:** To Do
- **Estimation:** 8h

## 2. Objective
Build the complete NestJS API layer for Supplier management and Supplier-Product price book operations. This is the backend foundation for both the Supplier Directory and Price Book UIs.

## 3. High-Level Requirements

### 3.1. Supplier CRUD Endpoints
Implement a `SuppliersController`:
- `POST /suppliers` — Register a new supplier.
- `GET /suppliers` — Paginated list with search (`?q=name`) and status filter (`?is_active=true`).
- `GET /suppliers/:id` — Fetch full supplier profile.
- `PATCH /suppliers/:id` — Update supplier details or soft-delete (`is_active: false`).

### 3.2. Price Book (Nested Resource) Endpoints
- `GET /suppliers/:id/products` — List all product mappings for a supplier (paginated).
- `POST /suppliers/:id/products` — Create a new product-price mapping.
  - Validate: `product_id` exists in `Product_Master`, `agreed_price` > 0.
  - On duplicate `(supplier_id, product_id)`: return `409 Conflict` with a message directing the user to update the existing mapping.
- `PATCH /suppliers/mappings/:mappingId` — Update `agreed_price` for an existing mapping.

### 3.3. RBAC & Validation
- Apply JWT guard globally on the controller.
- Restrict `POST` / `PATCH` mutations to `Buyer`, `Procurement Manager`, and `Admin` roles.
- Use `class-validator` DTOs for all request payloads.

## 4. Acceptance Criteria (Developer Checklist)
- [ ] `POST /suppliers` creates a supplier and returns `201 Created`.
- [ ] `POST /suppliers/:id/products` with a duplicate mapping returns `409 Conflict`.
- [ ] `PATCH /suppliers/mappings/:id` updates only the `agreed_price` and sets `updated_at`.
- [ ] All mutations trigger an `Audit_Logs` entry via the global interceptor.
- [ ] A Driver or Picker role attempting mutation receives `403 Forbidden`.
