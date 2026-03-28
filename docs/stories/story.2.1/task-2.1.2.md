# Task 2.1.2: Product Catalog REST API (NestJS)

## 1. Meta Information
- **Parent Story:** [User Story 2.1: Master Product Catalog Management](./user-story-2.1.md)
- **Epic:** Epic 2: Master Product & Supplier Catalog
- **Assignee:** Unassigned
- **Status:** To Do

## 2. Objective
Build the robust, secure RESTful API endpoints required to Create, Read, and Update the Product Catalog within the NestJS backend. This API provides the interface for the Procurement frontend and external mobile clients to access SKU definitions.

## 3. High-Level Requirements

### 3.1. Controller & Endpoints
Develop a `ProductsController` with the following routes:
- `POST /products`: Accept a JSON payload to provision a new SKU.
- `GET /products`: Retrieve a paginated array of products. Must accept query parameters for pagination (`limit`, `offset`), keyword searching (`?q=tomato`), and filtering (`?category=Vegetables&is_active=true`).
- `GET /products/:id`: Fetch a specific product's full details by UUID.
- `PATCH /products/:id`: Update existing attributes (e.g., changing shelf life or toggling `is_active: false` for soft deletion). 

### 3.2. Payload Validation & DTOs
- Utilize `class-validator` and `class-transformer` within NestJS Data Transfer Objects (DTOs).
- Enforce strict validation rules (e.g., `@IsInt()`, `@Min(0)` for `shelf_life_days`, `@IsEnum()` for `unit_of_measure`) before the request reaches the service layer to prevent bad data from polluting the Master Catalog.

### 3.3. Security & RBAC Guards
- Globally protect the entire controller using the JWT Authentication Guard.
- Apply the specific RBAC Middleware/Guards (built in Task 1.1.3) to restrict all `POST` and `PATCH` mutations exclusively to users holding the `Procurement Manager` or `Admin` roles. (Read-only `GET` requests may be open to operational roles like Receivers).

## 4. Acceptance Criteria (Developer Checklist)
- [ ] The `GET /products` route successfully implements server-side pagination and returns matched results against a search query.
- [ ] `POST /products` correctly captures data, enforces DTO validation rules, and saves to the PostgreSQL DB, returning `201 Created`.
- [ ] Invalid DTO payloads return a descriptive `400 Bad Request`.
- [ ] Modifying data (`POST` / `PATCH`) as an unauthorized role (e.g., `Driver`) correctly triggers a `403 Forbidden` rejection.
- [ ] Unit tests are written for the `ProductsService` covering pagination and soft-delete business logic.
