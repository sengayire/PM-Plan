# Task 1.2.2: Implement Global NestJS Audit Interceptor

## 1. Meta Information
- **Parent Story:** [User Story 1.2: Immutable Audit Logging](./user-story-1.2.md)
- **Epic:** [Epic 1: Identity & Access Management (IAM)](../../epics/EPICS.md)
- **Jira:** FL-23
- **Assignee:** Unassigned
- **Status:** To Do
- **Estimation:** 6h

## 2. Objective
Build a global NestJS interceptor that automatically captures all state-mutating HTTP requests (`POST`, `PATCH`, `DELETE`) across every controller and writes a structured audit record to the `Audit_Logs` table. This must be transparent to individual controllers — no manual logging calls should be required in business logic.

## 3. Implementation Requirements

### 3.1. Interceptor Logic
- Implement as a NestJS `@Injectable()` class implementing `NestInterceptor`.
- Apply globally via `APP_INTERCEPTOR` in the main `AppModule`.
- Intercept only `POST`, `PATCH`, and `DELETE` methods (skip `GET`).
- Capture and write:
  - `user_id` from the decoded JWT in the request context
  - `action` derived from HTTP method (`POST`→`CREATE`, `PATCH`→`UPDATE`, `DELETE`→`DELETE`)
  - `entity_type` from a decorator (`@AuditEntity('Batch')`) placed on each controller
  - `entity_id` from request params (`:id`) or response body (`id`)
  - `old_value`: For `PATCH`/`DELETE`, perform a pre-request DB read to capture the current state
  - `new_value`: Captured from the outgoing response body

### 3.2. Async Queue Pattern
- Write audit log entries **asynchronously** via BullMQ (or NestJS EventEmitter as a simpler fallback) to prevent audit logging from adding latency to API response times.
- Ensure queue failures are retried with exponential backoff — a failed audit write must NOT silently drop the record.

### 3.3. Sensitive Data Handling
- **Strip password fields** from `old_value`/`new_value` before writing to `Audit_Logs`.
- Mask sensitive financial values if required by compliance rules (configurable via env flag).

## 4. Acceptance Criteria (Developer Checklist)
- [ ] A `POST /products` request automatically creates an `Audit_Logs` entry with correct `user_id`, `action: CREATE`, `entity_type: Product`.
- [ ] A `PATCH /products/:id` request captures the pre-mutation state in `old_value`.
- [ ] A `GET` request does NOT generate any audit log entry.
- [ ] API response latency is not significantly increased by async audit writing (< 10ms overhead on p95).
- [ ] Password fields are never written to `old_value` or `new_value`.

## 5. Security & Dependencies
- Depends on Task 1.1.2 (JWT Auth Service) — user identity is extracted from the request JWT.
- Depends on Task 1.2.1 (Audit_Logs table schema).
