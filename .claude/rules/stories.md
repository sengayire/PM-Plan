# User Story Format — _pm-plan

## Rule: User stories follow AgriFlow story template with acceptance criteria and tasks

Every user story in `docs/stories/story.N.M/` must include structured acceptance criteria and implementation tasks that guide engineering work.

### File structure

```
docs/stories/story.3.2/
├── user-story-3.2.md           # Main story document
├── tasks/
│   ├── task-3.2.1.md           # Backend task
│   ├── task-3.2.2.md           # Frontend task
│   └── task-3.2.3.md           # Database/schema task
└── acceptance-criteria.md       # Detailed AC (optional, can live in main)
```

### User story template

```markdown
# Story 3.2 — Add SKU to Product Catalog

**Epic:** E3 (Unified Product Catalog)
**Status:** In Progress / Done
**Jira:** FL-42

## Overview

As a **catalog manager**, I want to **add new SKUs to the product catalog** so that **retailers can order against our growing portfolio**.

## Description

The platform must support adding product SKUs with rich metadata: name, category, unit pricing, storage requirements, and shelf-life tracking. This is the foundation for E5 (Inventory Ledger).

## Acceptance Criteria

### Backend API
- [ ] `POST /api/v1/products/skus` accepts `CreateSkuInput` (name, category, unit, price, storageType, shelfLife)
- [ ] Response includes generated `sku_id` (UUID), `created_at`, `created_by`
- [ ] Duplicates rejected: error if SKU name already exists for this category
- [ ] Write audit log entry: `sku.created` with full payload

### Frontend
- [ ] "Add SKU" form appears in Products → SKU List → [Create] button
- [ ] Form validates: name (required), category (required), shelfLife (number >= 1)
- [ ] On submit, calls `createSku` provider → invalidates `getSKUs` query
- [ ] Success toast: "SKU created: ${name}"
- [ ] Error toast shows backend message (e.g., "SKU name already exists")

### Database
- [ ] Prisma schema includes `SKU` model with fields: `id`, `name`, `category`, `unit`, `price`, `storageType`, `shelfLife`, `createdAt`, `createdBy`, `updatedAt`, `deletedAt`
- [ ] Unique constraint: `(name, category, deletedAt: null)`
- [ ] `createdBy` references `User.id`

### QC / Compliance
- [ ] All price values use Decimal type (no floating point)
- [ ] Soft delete enforced (no hard deletes)
- [ ] Audit log retained 5 years

## Implementation Tasks

See `tasks/task-3.2.*.md` for detailed breakdown.

- [ ] **Task 3.2.1** — Backend: Scaffold `products` module, `CreateSkuUseCase`, DTO, controller
- [ ] **Task 3.2.2** — Database: Add `SKU` model to Prisma, run migration
- [ ] **Task 3.2.3** — Frontend: Add `createSku` provider, form component, integration

## Dependencies

- **Blocks:** Story 5.1 (Inventory Ledger requires SKUs to exist)
- **Blocked by:** Story 3.1 (Category management must come first)
- **Related:** Story 6.2 (Order picking will reference SKU IDs)

## Notes

- Consider implementing SKU search/filter in follow-up story
- Photo/image attachment for SKUs deferred to E3.4
- Pricing strategy (bulk discount tiers) deferred to E6 (Orders)
```

### Acceptance Criteria guidelines

- **Be testable** — each AC must be verifiable (no vague language like "works correctly")
- **Include API specifics** — endpoint paths, HTTP methods, expected status codes, response shape
- **Include UI specifics** — which page, which component, which button, validation rules
- **Include data specifics** — Prisma schema additions, database constraints, indices
- **Include compliance** — audit logging, soft deletes, data retention, encryption
- **Separate by layer** — Backend, Frontend, Database, QC / Compliance sections

### Task structure

Each task is a concrete unit of implementation work:

```markdown
# Task 3.2.1 — Backend: Products Module Scaffolding

**Story:** 3.2
**Estimated effort:** 3 hours
**Owner:** Backend team
**Status:** Not started / In review

## Objective

Create the `products` module scaffold with `CreateSkuUseCase` and POST endpoint.

## Deliverables

- [ ] `src/modules/products/` folder structure (contracts, drivers, services, use-cases, dto, controller)
- [ ] `IProductsDriver` contract + `PrismaProductsDriver` implementation
- [ ] `CreateSkuUseCase` class with `execute(input: CreateSkuInput)` method
- [ ] `CreateSkuDto` with Zod schema validation
- [ ] `ProductsController` with `POST /api/v1/products/skus` endpoint
- [ ] Swagger documentation with `@ApiTags`, `@ApiOperation`, `@ApiResponse`
- [ ] Unit tests: `CreateSkuUseCase.test.ts`, `ProductsController.test.ts`
- [ ] Module registration in `AppModule`

## Validation Rules

- SKU name: required, 1–255 characters
- Category: required, must match existing category enum or reference
- Unit: required (enum: `KG`, `CRATE`, `BOX`, `UNIT`)
- Price: required, positive Decimal
- Storage type: required (enum: `AMBIENT`, `COLD_CHAIN`, `FROZEN`)
- Shelf-life: required, integer >= 1 (days)

## Test Cases

1. **Valid SKU creation** — all fields provided → returns 201 with sku_id
2. **Duplicate name** — SKU name exists in same category → returns 409 Conflict
3. **Missing required field** — no category → returns 400 Bad Request
4. **Invalid price** — negative or non-Decimal → returns 400 Bad Request
5. **Audit log entry** — successful creation writes to `audit_logs` table

## Implementation Details

Follow `flow-be/CLAUDE.md` conventions:
- DTOs use Zod schemas (no class-validator)
- Use-cases are one class per action
- Services are thin facades (cross-module exports)
- Contracts/drivers follow Ports & Adapters pattern
- All endpoints documented with Swagger

## Acceptance Checklist

- [ ] Code compiles: `yarn build`
- [ ] Tests pass: `yarn test`
- [ ] No TypeScript errors: `yarn lint:check`
- [ ] Swagger doc visible at `/docs`
- [ ] Endpoint tested via Swagger UI or curl
```

### Story status values

- **Pending** — Not yet started (dependencies blocked)
- **In Progress** — Currently being worked on
- **In Review** — Code written, awaiting QA/review
- **Done** — Merged, shipped, verified in production

### Linking to Jira

Every story must reference a Jira FL ticket:
```markdown
**Jira:** [FL-42](https://jira.example.com/browse/FL-42)
```

This ensures PM/Jira stays in sync. Automation in `story-pipeline` agent creates tickets if missing.

### Dependency syntax

Use arrow notation to express dependency chains:
```markdown
- **Blocks:** Story 5.1 (Inventory Ledger requires SKUs)
- **Blocked by:** Story 3.1 (Category mgmt must come first)
- **Related:** Story 6.2 (Orders will reference SKU IDs)
```

The ceo-agent validates these chains against the epic dependency gate before allowing generation.

### Version control

Commit story changes with atomic messages:
```bash
git add docs/stories/story.3.2/
git commit -m "feat(E3): add story 3.2 — Add SKU to catalog"
```

Stories are the source of truth. Code branches follow story numbers.
