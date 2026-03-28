# Task 3.1.2: Build Product Catalog REST API (NestJS CRUD)

## 1. Meta Information
- **Parent Story:** [User Story 3.1: Master Product (SKU) Management](./user-story-3.1.md)
- **Epic:** [Epic 3: Unified Product Catalog (FL-6)](../../epics/EPICS.md)
- **Jira:** FL-32
- **Estimate:** 8h
- **Assignee:** Unassigned
- **Status:** To Do

## 2. Objective
Build the `ProductsController` in NestJS exposing full CRUD for `Product_Master`. This API is consumed by the catalog UI, the receiving flow (Epic 4), and the supplier price mapping workflows (Epic 2).

## 3. Implementation Requirements

### 3.1. Endpoints

| Method | Route | Description | RBAC |
|--------|-------|-------------|------|
| POST | `/products` | Create new SKU | Procurement Manager, Admin |
| GET | `/products` | Paginated, filterable list | All authenticated |
| GET | `/products/:id` | Full product detail | All authenticated |
| PATCH | `/products/:id` | Update fields (including soft-delete) | Procurement Manager, Admin |

### 3.2. List Endpoint (GET /products)
Query parameters:
- `page`, `limit` — pagination (default: `page=1`, `limit=20`)
- `q` — search across `name` and `sku_code` (ILIKE)
- `storage_type` — filter by ENUM value (`COLD_CHAIN`, `DRY`, `FROZEN`, `AMBIENT`)
- `category_id` — filter by category UUID (Story 3.2 adds categories)
- `is_active` — default `true`; `false` returns archived products (Admin/Procurement only)

Response format:
```json
{
  "data": [...],
  "total": 42,
  "page": 1,
  "limit": 20
}
```

### 3.3. DTOs & Validation
`CreateProductDto`:
- `sku_code`: `@IsString()`, `@IsNotEmpty()`, max 50 chars
- `name`: `@IsString()`, `@IsNotEmpty()`
- `shelf_life_days`: `@IsInt()`, `@Min(0)`
- `storage_type`: `@IsEnum(StorageTypeEnum)`, **required**
- `unit_of_measure`: `@IsEnum(UnitOfMeasureEnum)`, **required**
- `storage_temp_min`, `storage_temp_max`: `@IsNumber()`, both optional; cross-field validation: min ≤ max when both provided
- `category_id`: `@IsUUID()`, optional

`UpdateProductDto`: `PartialType(CreateProductDto)` — all fields optional.

### 3.4. Business Rules
- **SKU uniqueness:** Check case-insensitively before insert; return `409 Conflict` with message `"SKU code already exists"` if duplicate.
- **Soft delete only:** `PATCH /products/:id` with `{ is_active: false }` is the only allowed "delete" operation. Hard delete endpoint must not exist.
- **Shelf-life change warning:** When `shelf_life_days` is changed via PATCH, service must check if any active `Inventory_Batches` reference this product. If yes, include a warning in the response body: `{ "warning": "Shelf life changed. Existing batches retain their original expiry dates." }`.

### 3.5. Security & Audit
- All `POST` and `PATCH` requests guarded by RBAC middleware (FL-18).
- Global audit interceptor (FL-23) captures all mutations automatically.
- `GET` endpoints open to all authenticated roles (Receivers, QC staff need read access).

## 4. Acceptance Criteria (Developer Checklist)
- [ ] `POST /products` returns `201` on success; `409` on duplicate `sku_code`; `400` on validation failure
- [ ] `GET /products` supports all query params and returns paginated response
- [ ] `GET /products?storage_type=COLD_CHAIN` returns only cold-chain products
- [ ] `PATCH /products/:id` with `{ is_active: false }` soft-deletes (product still in DB)
- [ ] Shelf-life change on product with active batches returns `200` with warning field
- [ ] Non-Procurement/Admin role attempting `POST` or `PATCH` receives `403 Forbidden`
- [ ] Unit tests cover: SKU uniqueness check, soft-delete, shelf-life warning logic

## 5. Dependencies
- **Requires:** FL-31 (Product_Master schema)
- **Requires:** FL-18 (RBAC middleware), FL-23 (Audit interceptor)
- **Unblocks:** FL-33 (Catalog UI), FL-34 (Form UI)
- **Consumed by:** Epic 2 Story 2.2 (price mapping), Epic 4 (receiving zone validation)
