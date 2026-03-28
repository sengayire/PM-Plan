# AgriFlow Rwanda — Phase 1 MVP Backlog

> **Jira-Ready Backlog:** Epics > User Stories > Technical Tasks
> **Naming:** `story.{E}.{S}` / `task-{E}.{S}.{T}` — E=epic, S=story, T=task
> **Authoritative Source:** [PRD](../prd/PRD.md) Section 5 | [GEMINI.md](../../GEMINI.md) Section 6

---

> **Migration Note:** This document adopts the PRD's canonical 9-epic numbering.
> Existing story folders require renaming:
>
> | Old Path | New Path | Reason |
> |---|---|---|
> | `story.2.1/` (Product Catalog) | `story.3.1/` | Product Catalog is now Epic 3 |
> | `story.2.2/` (Supplier-Product Map) | `story.2.2/` | No change — stays in Epic 2 |
>
> Story 1.1 (RBAC) is unaffected.

---

## A. Operations Core

---

### Epic 1: Identity & Access Management (IAM)

**Goal:** Establish a secure perimeter and internal accountability via RBAC.

**Success Metrics:**
- 100% of API endpoints protected by role-based middleware
- Zero unauthorized access incidents post-launch
- Audit log query response < 2s for any 90-day window

**In Scope:** JWT authentication, RBAC (Admin, Manager, Picker, Driver, Finance), user lifecycle (create, activate, deactivate), immutable audit logging, password reset.
**Out of Scope:** SSO/OAuth federation, biometric auth, multi-tenant IAM.

**Dependencies:** None (foundational — blocks all other epics).

#### Story 1.1: Role-Based Access Control Setup — MUST

**As an** Admin, **I want to** create user accounts and assign specific operational roles, **so that** users only access data and actions relevant to their function.

**Key Acceptance Criteria:**
- **Given** an Admin submits a new user form **When** role is "Warehouse Picker" **Then** account is created with `is_active: false` and activation link dispatched.
- **Given** a Driver attempts a `PATCH` on Supplier Price Book **When** JWT role != authorized **Then** API returns `403` and attempt is logged to `Audit_Logs`.

**Edge Cases:**
1. Role change mid-shift — invalidate active JWT, force re-authentication.
2. Accidental user deletion — hard deletes forbidden; soft delete via `is_active: false`.

**Tasks:**
- `task-1.1.1` — Create PostgreSQL `Users` table and `Roles` enum/table — 4h
- `task-1.1.2` — Implement JWT auth service (login, token generation, refresh) — 6h
- `task-1.1.3` — Develop RBAC middleware for API endpoint role-checking — 4h
- `task-1.1.4` — Integrate Auth.js on frontend with `CredentialsProvider` and session callbacks — 6h
- `task-1.1.5` — Build Admin UI for user management (create, read, update, deactivate) — 8h
- `task-1.1.6` — Implement automated credential dispatch (Email/SMS) via NestJS — 4h

#### Story 1.2: Immutable Audit Logging — MUST

**As a** System Auditor, **I want** a non-editable log of all critical data changes, **so that** I can trace discrepancies and satisfy FDA inspection requirements.

**Key Acceptance Criteria:**
- **Given** any user performs a POST/PATCH/DELETE on an inventory entity **When** the request completes **Then** an entry is written to `Audit_Logs` with user ID, timestamp, entity, action, old value, and new value.
- **Given** an Auditor searches logs by date range and entity type **When** results are returned **Then** response time is < 2 seconds for up to 90 days of data.

**Edge Cases:**
1. Bulk import triggers thousands of log entries — use batch inserts with async queue to prevent API slowdown.
2. Audit log table grows beyond query performance — implement date-based partitioning.

**Tasks:**
- `task-1.2.1` — Create `Audit_Logs` table with indexed columns (user_id, entity, timestamp) — 3h
- `task-1.2.2` — Implement global NestJS interceptor to log POST/PATCH/DELETE on inventory entities — 6h
- `task-1.2.3` — Build Audit Log search UI with filters (date range, user, entity, action) — 6h
- `task-1.2.4` — Add CSV/PDF export for audit log query results — 4h

#### Story 1.3: Password Reset & Session Management — MUST

**As a** platform user, **I want to** reset my password securely, **so that** I can regain access without admin intervention.

**Key Acceptance Criteria:**
- **Given** a user requests password reset **When** they provide a registered email **Then** a time-limited (15 min) reset link is sent; no indication of whether the email exists (prevent enumeration).
- **Given** a user's session JWT expires **When** they make an API request **Then** the system returns `401` and the frontend redirects to login.

**Edge Cases:**
1. Multiple reset requests in quick succession — rate-limit to 3 per hour per email.
2. Reset link used after expiry — reject with clear message and offer to resend.

**Tasks:**
- `task-1.3.1` — Implement password reset endpoint with secure token generation and expiry — 4h
- `task-1.3.2` — Build password reset frontend flow (request, token validation, new password) — 4h
- `task-1.3.3` — Add JWT refresh token rotation and session expiry handling — 4h

---

### Epic 2: Partner & Supplier Ecosystem

**Goal:** Digitalize the supply chain entry point — supplier onboarding, contracts, and performance visibility.

**Success Metrics:**
- 100% of active suppliers have digital profiles with contract documents
- Supplier scorecard updated automatically after every receiving event
- Average supplier onboarding time < 1 business day

**In Scope:** Supplier registration, contract/document storage (S3), price-book management, supplier performance scorecard (rejection rate, on-time delivery).
**Out of Scope:** Supplier self-service portal, automated procurement/PO generation, payment processing.

**Dependencies:** Blocked by Epic 1 (IAM — role-based access required for Procurement Manager).

#### Story 2.1: Supplier Onboarding & Profile Management — MUST

**As a** Procurement Manager, **I want to** register new suppliers with their business details and upload contracts, **so that** all supplier data is centralized and audit-ready.

**Key Acceptance Criteria:**
- **Given** a Procurement Manager submits the supplier registration form **When** all required fields are valid **Then** a new supplier profile is created with `is_active: true` and a system-generated `supplier_code`.
- **Given** a contract PDF is uploaded **When** storage succeeds **Then** the file is stored in S3 with a reference in `Supplier_Documents` linked to the supplier ID.

**Edge Cases:**
1. Duplicate supplier registration (same TIN/business ID) — reject with link to existing profile.
2. Contract file exceeds size limit — validate client-side (10MB max) and return `413` from API.

**Tasks:**
- `task-2.1.1` — Create `Suppliers` and `Supplier_Documents` PostgreSQL schemas — 4h
- `task-2.1.2` — Implement Supplier CRUD API endpoints with validation — 6h
- `task-2.1.3` — Configure S3 bucket and presigned URL upload flow for contract documents — 4h
- `task-2.1.4` — Build Supplier Registration and Profile UI — 8h

#### Story 2.2: Supplier-Product Price Mapping — MUST

**As a** Procurement Manager, **I want to** map products to specific suppliers with agreed pricing, **so that** the system validates costs during receiving and calculates accurate margins.

**Key Acceptance Criteria:**
- **Given** a Procurement Manager links a supplier to a product **When** they enter the agreed price and currency **Then** a `Supplier_Product_Map` record is created with effective date.
- **Given** a price mapping already exists for the same supplier-product pair **When** a new price is submitted **Then** the old record is soft-closed (end date set) and a new record created, preserving price history.

**Edge Cases:**
1. Currency mismatch between supplier agreements — store `currency` per mapping; MVP supports RWF only with validation.
2. Price set to zero — reject; minimum price must be > 0.

**Tasks:**
- `task-2.2.1` — Create `Supplier_Product_Map` table with price history support — 3h
- `task-2.2.2` — Implement Supplier-Product mapping API (create, update, list by supplier/product) — 5h
- `task-2.2.3` — Build Price Mapping UI within Supplier profile view — 6h

#### Story 2.3: Supplier Performance Scorecard — SHOULD

**As a** Procurement Manager, **I want to** view a supplier's scorecard showing rejection rates and delivery reliability, **so that** I can make data-driven sourcing decisions.

**Key Acceptance Criteria:**
- **Given** a supplier has completed receiving events **When** the Procurement Manager views their scorecard **Then** they see: total deliveries, on-time %, QC pass rate, average quality grade, and trend over last 90 days.
- **Given** a supplier's QC rejection rate exceeds 15% **When** the scorecard is rendered **Then** a warning indicator is displayed.

**Edge Cases:**
1. New supplier with zero deliveries — display "Insufficient data" instead of zeroes or misleading charts.
2. Scorecard data spans a period with system downtime — clearly mark data gaps.

**Tasks:**
- `task-2.3.1` — Create scorecard aggregation query (joins Receiving + QC_Logs by supplier) — 4h
- `task-2.3.2` — Implement Scorecard API endpoint with caching (refresh every 15 min) — 4h
- `task-2.3.3` — Build Scorecard dashboard component with trend charts — 6h

---

### Epic 3: Unified Product Catalog

**Goal:** Standardize how items are defined and handled across inventory, receiving, and sales.

**Success Metrics:**
- 100% of products have shelf-life and storage temperature parameters defined
- Zero stock entries without a valid `product_id` foreign key
- Product search returns results in < 500ms

**In Scope:** SKU definitions, shelf-life logic, storage temperature requirements, units of measure, product categories, product activation/deactivation.
**Out of Scope:** Product images/thumbnails for mobile UI, hierarchical category trees, barcode generation.

**Dependencies:** Blocked by Epic 1 (IAM). Blocks Epic 4 (Receiving needs product templates for expiry calculation).

#### Story 3.1: Master Product (SKU) Management — MUST

**As a** Procurement Manager, **I want to** define products with shelf-life and storage requirements, **so that** the system automates expiry calculations and enforces FIFO.

**Key Acceptance Criteria:**
- **Given** a Procurement Manager submits a new product with shelf-life "4 days", temp range "4-8C", UoM "KG" **When** validation passes **Then** the product is created and available in receiving/procurement workflows.
- **Given** a user enters a negative `shelf_life_days` or blank `unit_of_measure` **When** submitting **Then** UI prevents submission; API returns `400` if bypassed.

**Edge Cases:**
1. Shelf-life change after active inventory exists — new value applies only to future batches; existing batches retain original expiry.
2. Product discontinued — soft-deactivate (`is_active: false`); hide from dropdowns but preserve for historical records.

**Tasks:**
- `task-3.1.1` — Create `Product_Master` schema (shelf_life_days, storage_temp_min/max, unit_of_measure, sku_code, is_active) — 4h
- `task-3.1.2` — Implement Product CRUD API endpoints with validation — 5h
- `task-3.1.3` — Build Product Catalog data table with search, filter, and pagination — 6h
- `task-3.1.4` — Build Create/Edit Product modal with form validation — 5h

#### Story 3.2: Product Category & Storage Classification — SHOULD

**As a** Warehouse Manager, **I want to** classify products by category and storage type (cold vs. dry), **so that** warehouse zones can be assigned correctly.

**Key Acceptance Criteria:**
- **Given** a product is created **When** category and storage type are assigned **Then** the product is queryable by category and storage classification.
- **Given** a product is marked "Cold Storage" **When** a batch of this product is received **Then** the system suggests cold-zone warehouse locations only.

**Edge Cases:**
1. Product requires both cold and dry variants (e.g., fresh vs. dried herbs) — model as separate SKUs.
2. Category deleted while products reference it — prevent deletion; require reassignment first.

**Tasks:**
- `task-3.2.1` — Create `Product_Categories` table and add `category_id`, `storage_type` to Product_Master — 3h
- `task-3.2.2` — Implement Category CRUD API and product-category association — 4h
- `task-3.2.3` — Add category filter and storage type badge to Product Catalog UI — 3h

---

## B. Revenue & Distribution

---

### Epic 4: Smart Receiving & Batch Traceability

**Goal:** The "front gate" — ensure food safety, quality gating, and end-to-end traceability from farm to shelf.

**Success Metrics:**
- 100% of received batches have a system-generated Traceability ID
- Zero batches enter available inventory without `PASSED` QC status
- QC inspection completion time < 5 minutes per batch

**In Scope:** Receiving entry against PO, automated batch code generation (`[Date]-[SupplierID]-[SKU]`), QC inspection workflow, photo upload for rejects, quarantine routing, expiry date auto-calculation from product templates.
**Out of Scope:** Automated PO generation, computer vision QC (Phase 3), supplier self-reported QC.

**Dependencies:** Blocked by Epic 1 (IAM), Epic 2 (Suppliers), Epic 3 (Products — shelf-life needed for expiry calc). Blocks Epic 5 (inventory entries come from receiving).

#### Story 4.1: Receiving Entry Against Purchase Order — MUST

**As a** Warehouse Receiver, **I want to** log incoming shipments against a PO, **so that** I can verify what was delivered versus what was ordered.

**Key Acceptance Criteria:**
- **Given** a Receiver selects a PO and scans/enters delivered items **When** they submit the receiving form **Then** a `Receiving_Entry` record is created with a unique `Batch_ID` generated as `[YYYYMMDD]-[SupplierID]-[SKU]`.
- **Given** the delivered quantity differs from the PO quantity **When** the Receiver submits **Then** the variance is recorded and flagged for Procurement review.

**Edge Cases:**
1. PO not found in system (manual/phone order) — allow ad-hoc receiving with mandatory supplier + product selection; flag as "Unmatched PO" for reconciliation.
2. Same supplier delivers the same SKU twice in one day — append a sequence number to Batch_ID to ensure uniqueness.

**Tasks:**
- `task-4.1.1` — Create `Receiving_Entries` and `Receiving_Line_Items` schemas with Batch_ID generation logic — 5h
- `task-4.1.2` — Implement Receiving API (create entry, link to PO, calculate variance) — 6h
- `task-4.1.3` — Build Receiving Entry form UI (PO lookup, item entry, quantity confirmation) — 8h
- `task-4.1.4` — Implement auto-expiry calculation: `received_date + product.shelf_life_days` — 2h

#### Story 4.2: QC Inspection & Photo Upload — MUST

**As a** QC Inspector, **I want to** inspect a batch, upload photos, and pass or fail it, **so that** bad produce is blocked from entering available inventory.

**Key Acceptance Criteria:**
- **Given** a QC Inspector opens a pending batch **When** they record grade, upload photos, and submit **Then** a `QC_Logs` entry is created linked to the `Batch_ID`.
- **Given** a batch receives `QC_Status` != 'PASSED' **When** the inspection is submitted **Then** the batch `Stock_Status` is programmatically set to `QUARANTINE` — no manual override allowed without Manager approval.

**Edge Cases:**
1. Photo upload fails (network drop in warehouse) — queue locally on device and sync when connectivity resumes (offline-first).
2. Inspector disputes with supplier about grade — log both the inspector's grade and any supplier contestation notes; final status remains inspector's call until Manager override.

**Tasks:**
- `task-4.2.1` — Create `QC_Logs` table (batch_id, inspector_id, grade, status, notes, photos[]) — 3h
- `task-4.2.2` — Configure S3 photo upload with presigned URLs and link to QC record — 4h
- `task-4.2.3` — Implement QC status enforcement: non-PASSED → QUARANTINE (no bypass) — 3h
- `task-4.2.4` — Build mobile-friendly QC Inspection form with camera integration — 8h

#### Story 4.3: Quarantine Disposition Workflow — MUST

**As a** Warehouse Manager, **I want to** review quarantined batches and decide their disposition (return to supplier, discard, re-grade), **so that** quarantined stock doesn't sit indefinitely.

**Key Acceptance Criteria:**
- **Given** a batch is in QUARANTINE status **When** a Manager selects a disposition action **Then** the system logs the decision, updates stock status, and creates the corresponding ledger entry (return/discard).
- **Given** a Manager selects "Re-grade" **When** submitted **Then** the batch is re-queued for QC inspection by a different inspector.

**Edge Cases:**
1. Quarantine batch expires before disposition — auto-flag for discard; prevent re-grade of expired stock.
2. Supplier dispute on rejection — record the dispute but do not release stock until re-grade passes.

**Tasks:**
- `task-4.3.1` — Implement disposition API (return, discard, re-grade) with audit trail — 5h
- `task-4.3.2` — Build Quarantine Review dashboard (list, filter by age, action buttons) — 6h
- `task-4.3.3` — Add auto-expiry flagging for quarantined batches via background job — 3h

---

### Epic 5: Ledger-Based Inventory Engine

**Goal:** Real-time, high-integrity stock management using double-entry ledger principles.

**Success Metrics:**
- 100% of stock movements have explicit `source_location` and `destination_location` (zero orphan entries)
- Ledger balance matches physical count within 2% tolerance
- Expiry alerts triggered at 48h and 24h before expiry with zero misses

**In Scope:** Double-entry stock ledger, multi-location tracking, FIFO pick suggestion (expiry-based), automated expiry and low-stock alerts, stock adjustment/write-off recording.
**Out of Scope:** Multi-warehouse transfers (Phase 2 — Epic 10), demand forecasting, automated reordering.

**Dependencies:** Blocked by Epic 4 (inventory entries originate from receiving). Blocks Epic 6 (order fulfillment needs stock availability).

#### Story 5.1: Real-Time Stock Visibility by Batch & Location — MUST

**As an** Inventory Manager, **I want to** see stock levels broken down by batch, location, and expiry date, **so that** I can prevent spoilage and maintain accurate counts.

**Key Acceptance Criteria:**
- **Given** the Inventory Manager opens the stock dashboard **When** data loads **Then** they see aggregated quantities grouped by product, location, and batch with expiry dates.
- **Given** any stock movement occurs (receiving, picking, transfer, write-off) **When** recorded **Then** it creates a ledger entry with `source_location`, `destination_location`, `quantity`, `batch_id`, and `timestamp` — no field may be null.

**Edge Cases:**
1. Negative stock balance after a system error — prevent via DB constraint; alert ops team if constraint triggers.
2. Concurrent stock updates on same batch — use pessimistic locking (SELECT FOR UPDATE) on ledger writes.

**Tasks:**
- `task-5.1.1` — Create `Inventory_Ledger` table with double-entry schema (source_location, destination_location, batch_id, quantity, movement_type) — 5h
- `task-5.1.2` — Implement stock balance materialized view (aggregated from ledger entries) — 4h
- `task-5.1.3` — Build Stock Dashboard UI (grouped by product/location/batch, sortable by expiry) — 8h
- `task-5.1.4` — Add DB constraints: non-null locations, non-negative balance check trigger — 3h

#### Story 5.2: FIFO Pick Suggestion — MUST

**As a** Warehouse Picker, **I want** the system to suggest the oldest-expiry batch first when I start an order, **so that** we minimize spoilage and comply with FIFO policy.

**Key Acceptance Criteria:**
- **Given** a pick list is generated for an order **When** multiple batches of the same product exist **Then** the system allocates from the batch with the earliest `expiry_date` first.
- **Given** the nearest-expiry batch has insufficient quantity **When** pick suggestion runs **Then** it splits across multiple batches, still ordered by expiry ASC.

**Edge Cases:**
1. Two batches with identical expiry date — secondary sort by `received_date ASC` (true FIFO).
2. Nearest-expiry batch is in QUARANTINE — skip quarantined batches; only suggest AVAILABLE stock.

**Tasks:**
- `task-5.2.1` — Implement FIFO allocation algorithm: `ORDER BY expiry_date ASC, received_date ASC` with quantity splitting — 6h
- `task-5.2.2` — Build Pick Suggestion API endpoint (input: order line items; output: batch-level pick instructions) — 4h
- `task-5.2.3` — Build mobile-friendly Picker UI showing batch, location, and quantity to pick — 6h

#### Story 5.3: Stock Adjustment & Write-Off Recording — MUST

**As an** Inventory Manager, **I want to** record stock adjustments and write-offs with mandatory reason codes, **so that** every unit is accounted for and loss causes are trackable.

**Key Acceptance Criteria:**
- **Given** a Manager initiates a stock adjustment **When** they select a batch, enter quantity, and choose a reason code (e.g., Spoilage, Damage, Theft, Count Correction) **Then** a ledger entry is created moving stock to a virtual `WRITE_OFF` location.
- **Given** a write-off is recorded **When** the audit log is queried **Then** the adjustment shows the Manager's ID, timestamp, batch, quantity, and reason code.

**Edge Cases:**
1. Adjustment quantity exceeds available balance — reject; cannot write off more than exists.
2. Frequent adjustments on same batch — trigger alert to operations for physical count verification.

**Tasks:**
- `task-5.3.1` — Create `Reason_Codes` reference table and `WRITE_OFF` virtual location — 2h
- `task-5.3.2` — Implement stock adjustment API with balance validation and ledger entry creation — 5h
- `task-5.3.3` — Build Stock Adjustment UI with batch selector, reason code dropdown, and confirmation — 5h

#### Story 5.4: Automated Expiry & Low-Stock Alerts — SHOULD

**As a** Warehouse Manager, **I want** automated alerts when batches approach expiry or stock falls below threshold, **so that** I can take action before losses occur.

**Key Acceptance Criteria:**
- **Given** a batch is within 48 hours of expiry **When** the scheduled job runs **Then** an alert (in-app notification + email) is sent to the Warehouse Manager.
- **Given** a product's total available stock falls below its configured reorder threshold **When** checked **Then** a low-stock alert is generated.

**Edge Cases:**
1. Alert fatigue from too many near-expiry items — group alerts by product/location; allow snooze for 12h.
2. Background job fails silently — implement heartbeat monitoring; escalate if no job run in 2 hours.

**Tasks:**
- `task-5.4.1` — Implement scheduled background job (cron) for expiry scanning (48h and 24h thresholds) — 4h
- `task-5.4.2` — Implement low-stock threshold check on inventory balance changes — 3h
- `task-5.4.3` — Build in-app notification system and email dispatch for alerts — 6h
- `task-5.4.4` — Build Alert Management UI (view, acknowledge, snooze) — 4h

---

### Epic 6: B2B Order & Fulfillment

**Goal:** Streamline the revenue workflow from order placement through picking to invoicing.

**Success Metrics:**
- Order-to-picking-list generation < 30 seconds
- Zero orders confirmed against unavailable stock
- 100% of delivered orders produce an auto-generated invoice

**In Scope:** B2B order portal, real-time stock validation, picking list generation with FIFO allocation, auto-invoicing on delivery confirmation, order cancellation.
**Out of Scope:** B2C / consumer ordering, payment gateway integration, credit terms management, dynamic pricing.

**Dependencies:** Blocked by Epic 1 (IAM), Epic 5 (stock availability). Blocks Epic 7 (delivery needs confirmed orders).

#### Story 6.1: B2B Order Placement — MUST

**As a** Supermarket Manager (B2B Buyer), **I want to** place orders via a digital portal, **so that** I don't have to call the warehouse and can see real-time availability.

**Key Acceptance Criteria:**
- **Given** a Supermarket Manager browses the order portal **When** they add items to cart **Then** real-time available stock is displayed (excluding QUARANTINE and already-allocated quantities).
- **Given** a Supermarket Manager submits an order **When** requested quantity exceeds available stock **Then** the system rejects the line item with a clear message showing available quantity.

**Edge Cases:**
1. Stock depleted between cart and submission (race condition) — re-validate at submission time; reject stale allocations.
2. Large order spanning multiple batches — system auto-splits across batches; buyer sees a single line item.

**Tasks:**
- `task-6.1.1` — Create `Orders` and `Order_Line_Items` schemas with status workflow (Draft, Confirmed, Picking, Shipped, Delivered, Cancelled) — 4h
- `task-6.1.2` — Implement Order API with real-time stock validation and allocation — 6h
- `task-6.1.3` — Build B2B Order Portal UI (product browse, cart, checkout) — 10h

#### Story 6.2: Picking List Generation with FIFO Allocation — MUST

**As a** Warehouse Manager, **I want** confirmed orders to automatically generate picking lists with FIFO-allocated batches, **so that** pickers know exactly what to pick from where.

**Key Acceptance Criteria:**
- **Given** an order is confirmed **When** the picking list is generated **Then** each line item is broken into batch-level picks ordered by `expiry_date ASC`.
- **Given** a Picker marks a pick as complete **When** all picks for an order are done **Then** the order status advances to "Ready for Dispatch."

**Edge Cases:**
1. Batch runs out mid-pick (physical shortage vs. system count) — allow partial pick and flag discrepancy for stock adjustment.
2. Picker picks wrong batch — require batch barcode scan confirmation before marking complete.

**Tasks:**
- `task-6.2.1` — Implement picking list generation service using Epic 5's FIFO allocation — 5h
- `task-6.2.2` — Create `Pick_Lists` and `Pick_Items` schemas — 3h
- `task-6.2.3` — Build Picker mobile UI (pick instruction, batch scan confirm, complete/exception flow) — 8h

#### Story 6.3: Automated Invoice Generation — MUST

**As a** Finance Officer, **I want** invoices generated automatically upon delivery confirmation, **so that** billing is immediate and error-free.

**Key Acceptance Criteria:**
- **Given** a delivery's Proof of Delivery is uploaded (Epic 7) **When** the "Delivered" event fires **Then** the system auto-generates a PDF invoice and links it to the order.
- **Given** a partial delivery occurs **When** PoD is captured **Then** the invoice reflects only the delivered quantities.

**Edge Cases:**
1. Delivery with discrepancies (damaged items rejected at door) — invoice only accepted quantities; create a credit note for rejected items.
2. Invoice generation fails (PDF service down) — queue for retry; alert Finance team after 3 failures.

**Tasks:**
- `task-6.3.1` — Implement invoice generation service (PDF via Puppeteer/PDFKit) triggered by PoD event — 6h
- `task-6.3.2` — Create `Invoices` schema (order_id, amounts, status, pdf_url) — 3h
- `task-6.3.3` — Build Invoice list and detail view for Finance UI — 6h

#### Story 6.4: Order Cancellation & Partial Fulfillment — SHOULD

**As a** Supermarket Manager, **I want to** cancel or modify an order before it enters picking, **so that** I can respond to demand changes.

**Key Acceptance Criteria:**
- **Given** an order is in "Confirmed" status **When** the buyer requests cancellation **Then** allocated stock is released back to available inventory and order status is set to "Cancelled."
- **Given** an order is already in "Picking" status **When** cancellation is requested **Then** the system rejects the cancellation and advises contacting the Warehouse Manager.

**Edge Cases:**
1. Cancellation requested during stock allocation (race condition) — use DB transaction to ensure atomic allocate-or-cancel.
2. Partial order modification (reduce quantity) — allow quantity reduction on unpicked items only.

**Tasks:**
- `task-6.4.1` — Implement order cancellation API with stock de-allocation and status guards — 5h
- `task-6.4.2` — Implement partial order modification (quantity reduction on unpicked items) — 4h
- `task-6.4.3` — Add cancellation and modification UI to buyer's order management view — 4h

---

### Epic 7: Logistics & Last-Mile Delivery

**Goal:** Accountability from warehouse dock to retail shelf.

**Success Metrics:**
- 100% of deliveries have GPS-timestamped Proof of Delivery
- Delivery discrepancy rate < 3%
- Driver app functions fully offline and syncs within 30 seconds of connectivity

**In Scope:** Driver assignment, route display, mobile PoD (signature capture + GPS), delivery exception reporting, offline-first architecture.
**Out of Scope:** Automated route optimization (Phase 2), fleet management, real-time tracking dashboard for buyers.

**Dependencies:** Blocked by Epic 6 (needs confirmed, picked orders). Feeds into Epic 6 Story 6.3 (PoD triggers invoicing) and Epic 8 (consignment delivery).

#### Story 7.1: Driver Route & Delivery Assignment — MUST

**As a** Warehouse Manager, **I want to** assign deliveries to drivers with route information, **so that** drivers know their daily schedule and stops.

**Key Acceptance Criteria:**
- **Given** orders are "Ready for Dispatch" **When** a Manager assigns them to a Driver **Then** the Driver sees the delivery list on their mobile app with stop addresses and order details.
- **Given** multiple deliveries are assigned **When** the Driver views their route **Then** stops are listed in a logical sequence (manual ordering by Manager for MVP).

**Edge Cases:**
1. Driver reassignment mid-route (vehicle breakdown) — allow Manager to transfer remaining stops to another Driver; already-delivered stops stay with original Driver.
2. Address not found / incorrect — Driver can flag the stop with a "Location Issue" note and photo.

**Tasks:**
- `task-7.1.1` — Create `Delivery_Assignments` schema (driver_id, order_ids, sequence, status) — 3h
- `task-7.1.2` — Implement delivery assignment API (assign, resequence, reassign) — 5h
- `task-7.1.3` — Build Manager delivery assignment UI (drag-and-drop stop ordering) — 6h
- `task-7.1.4` — Build Driver mobile route view (stop list, order details, navigation link) — 6h

#### Story 7.2: Proof of Delivery with Signature & GPS — MUST

**As a** Driver, **I want to** capture customer signatures and GPS coordinates on delivery, **so that** there is tamper-proof evidence of successful delivery.

**Key Acceptance Criteria:**
- **Given** a Driver arrives at a stop **When** they capture the recipient's signature and tap "Confirm Delivery" **Then** the system records the signature image, GPS coordinates, and timestamp.
- **Given** the Driver has no network connectivity **When** they capture PoD **Then** data is stored locally and auto-syncs when connectivity resumes.

**Edge Cases:**
1. Customer refuses to sign (disputes quantity/quality) — Driver marks as "Delivery Exception" with reason and photos of disputed items.
2. GPS unavailable (indoor/basement) — allow delivery confirmation with a mandatory note explaining GPS absence; flag for Manager review.

**Tasks:**
- `task-7.2.1` — Implement PoD data model (signature_image, gps_lat, gps_lng, timestamp, exception_flag) — 3h
- `task-7.2.2` — Build signature capture component for mobile (canvas-based) — 4h
- `task-7.2.3` — Implement offline storage queue with background sync service — 6h
- `task-7.2.4` — Emit "Delivery Confirmed" event to trigger invoice generation (Epic 6.3) — 2h

#### Story 7.3: Delivery Exception & Discrepancy Reporting — SHOULD

**As a** Driver, **I want to** report delivery exceptions (rejected items, wrong quantities, damaged goods), **so that** discrepancies are captured at the point of delivery.

**Key Acceptance Criteria:**
- **Given** a delivery has discrepancies **When** the Driver reports items rejected by the customer **Then** the exception is logged with reason code, quantity, and photos.
- **Given** an exception is logged **When** the Manager reviews it **Then** the system suggests a credit note for the affected items.

**Edge Cases:**
1. Full delivery rejection (customer refuses everything) — mark as "Fully Rejected"; stock must return to warehouse and re-enter inventory via a return receiving flow.
2. Driver claims delivery but customer disputes — PoD GPS + signature serves as evidence; flag for dispute resolution.

**Tasks:**
- `task-7.3.1` — Create delivery exception schema (reason_code, quantities, photos, resolution_status) — 3h
- `task-7.3.2` — Implement exception reporting API with return-to-warehouse stock flow — 5h
- `task-7.3.3` — Build exception reporting UI in Driver mobile app — 5h
- `task-7.3.4` — Build Manager exception review and resolution dashboard — 5h

---

## C. Consignment & Compliance

---

### Epic 8: Consignment & Retail Module

**Goal:** The core SBR engine — manage stock on consignment at retail locations, track sell-through, and automate settlement.

**Success Metrics:**
- 100% of consignment dispatches create ledger entries (warehouse → retail location)
- Retail sales reconciliation completed within 24 hours of reporting
- Automated commission invoice accuracy > 99%

**In Scope:** Consignment stock dispatch and tracking, retail partner sales reporting, stock reconciliation (dispatched vs. sold vs. remaining), automated commission/settlement invoicing.
**Out of Scope:** POS integration (manual sales entry for MVP), real-time shelf monitoring, consignment-to-wholesale conversion.

**Dependencies:** Blocked by Epic 5 (inventory ledger), Epic 7 (delivery to retail). Connects to Epic 6 (invoicing infrastructure).

#### Story 8.1: Consignment Stock Dispatch & Tracking — MUST

**As an** Inventory Manager, **I want to** dispatch stock on consignment to retail locations and track it as "Consigned" (still AgriFlow-owned), **so that** I maintain visibility of stock outside the warehouse.

**Key Acceptance Criteria:**
- **Given** a consignment dispatch is created **When** submitted **Then** a ledger entry moves stock from `WAREHOUSE` to `RETAIL_LOCATION_X` with movement type `CONSIGNMENT_OUT`.
- **Given** stock is at a retail location **When** the Inventory Manager views stock dashboard **Then** consigned stock appears under the retail location with distinct "Consigned" status.

**Edge Cases:**
1. Retail location returns unsold stock — create a `CONSIGNMENT_RETURN` ledger entry moving stock back; trigger QC re-inspection if near expiry.
2. Consigned stock expires at retail — retail partner reports via sales reporting; system creates a write-off linked to the retail location.

**Tasks:**
- `task-8.1.1` — Create `Retail_Locations` table and extend inventory ledger with CONSIGNMENT movement types — 4h
- `task-8.1.2` — Implement consignment dispatch API (stock allocation, ledger entry, status update) — 5h
- `task-8.1.3` — Build Consignment Dispatch UI (select products, quantities, destination retail location) — 6h

#### Story 8.2: Retail Sales Reporting & Reconciliation — MUST

**As a** Finance Officer, **I want** retail partners to report daily sales, **so that** I can reconcile dispatched stock against sold and remaining quantities.

**Key Acceptance Criteria:**
- **Given** a retail partner submits a daily sales report **When** validated **Then** the system deducts sold quantities from the consigned balance and calculates AgriFlow revenue.
- **Given** reported sales + remaining stock != dispatched stock **When** reconciliation runs **Then** the discrepancy is flagged with a variance alert.

**Edge Cases:**
1. Retail partner misses a reporting day — flag as "Pending Report" and block new consignment dispatches until reconciled.
2. Reported sales exceed consigned stock — reject the report; likely a data entry error.

**Tasks:**
- `task-8.2.1` — Create `Consignment_Sales_Reports` schema (retail_location, date, product, qty_sold, qty_remaining) — 3h
- `task-8.2.2` — Implement sales reporting API with reconciliation logic and variance detection — 6h
- `task-8.2.3` — Build Retail Sales Entry UI (simple daily form for retail partners) — 5h
- `task-8.2.4` — Build Reconciliation Dashboard showing dispatched vs. sold vs. remaining with variance flags — 6h

#### Story 8.3: Automated Commission & Settlement Invoice — MUST

**As a** Finance Officer, **I want** the system to calculate commissions and generate settlement invoices automatically, **so that** payments to retail partners are timely and accurate.

**Key Acceptance Criteria:**
- **Given** a reconciliation period closes (weekly/monthly) **When** the settlement job runs **Then** commission is calculated per the retail partner's agreed rate and a settlement invoice PDF is generated.
- **Given** a settlement invoice is generated **When** the Finance Officer reviews it **Then** they see line-by-line breakdown: product, qty sold, unit price, commission rate, net amount.

**Edge Cases:**
1. Commission rate changes mid-period — apply old rate to sales before change date, new rate after.
2. Disputed settlement — allow Finance to put invoice in "Disputed" status with notes; prevent payment until resolved.

**Tasks:**
- `task-8.3.1` — Create `Commission_Rates` table (retail_location, product/category, rate, effective_date) — 3h
- `task-8.3.2` — Implement commission calculation engine with period-based rate application — 5h
- `task-8.3.3` — Implement settlement invoice PDF generation (reuse Epic 6 PDF service) — 4h
- `task-8.3.4` — Build Settlement Invoice review and approval UI — 5h

---

### Epic 9: Loss & Compliance Guardrail

**Goal:** Margin protection and regulatory readiness — track every gram of loss and be FDA-audit-ready at all times.

**Success Metrics:**
- 100% of spoilage/loss events have a documented reason code
- Temperature logs available for any batch within 30 seconds (FDA requirement)
- Loss dashboard updated in real-time; monthly loss review cadence established

**In Scope:** Spoilage recording with mandatory reason codes, temperature monitoring logs, FDA audit dashboard (batch origin, QC records, temperature history, RBAC audit trail), loss analytics.
**Out of Scope:** IoT temperature sensor integration (manual entry for MVP), automated regulatory report submission, insurance claim processing.

**Dependencies:** Blocked by Epic 4 (QC data), Epic 5 (inventory data), Epic 1 (audit logs).

#### Story 9.1: Spoilage Recording with Reason Codes — MUST

**As a** Warehouse Manager, **I want to** record spoilage events with mandatory reason codes, **so that** every unit of loss is documented and analyzable.

**Key Acceptance Criteria:**
- **Given** a Manager identifies spoiled stock **When** they submit a spoilage record **Then** the system requires: batch_id, quantity, reason code (Expired, Temperature Breach, Physical Damage, Pest, Other), and optional photo.
- **Given** a spoilage record is submitted **When** recorded **Then** a ledger write-off entry is created automatically and the audit log captures the event.

**Edge Cases:**
1. Spoilage discovered during picking (mid-order) — allow inline spoilage recording from Picker UI; adjust pick suggestion to next FIFO batch.
2. Reason code "Other" selected — require free-text explanation (minimum 10 characters).

**Tasks:**
- `task-9.1.1` — Create `Spoilage_Records` schema (batch_id, quantity, reason_code, notes, photo_url, recorded_by) — 3h
- `task-9.1.2` — Implement spoilage recording API with auto-ledger write-off — 4h
- `task-9.1.3` — Build Spoilage Recording form (batch lookup, reason code, photo upload) — 5h

#### Story 9.2: Temperature Monitoring & Logging — MUST

**As a** Compliance Officer, **I want** temperature readings logged for storage zones, **so that** we can prove cold-chain compliance during FDA audits.

**Key Acceptance Criteria:**
- **Given** a Warehouse staff member records a temperature reading **When** they submit zone, temperature, and timestamp **Then** a `Temperature_Logs` entry is created.
- **Given** a recorded temperature is outside the product's acceptable range **When** submitted **Then** the system flags a "Temperature Breach" alert and links affected batches.

**Edge Cases:**
1. Staff forgets to log temperature — scheduled reminder notifications at configured intervals (e.g., every 4 hours).
2. Temperature breach affects multiple batches in the same zone — bulk-flag all batches in that zone/time window for QC re-inspection.

**Tasks:**
- `task-9.2.1` — Create `Temperature_Logs` and `Storage_Zones` schemas — 3h
- `task-9.2.2` — Implement temperature logging API with breach detection logic — 4h
- `task-9.2.3` — Build Temperature Entry form and breach alert notification — 4h
- `task-9.2.4` — Implement scheduled temperature reminder notifications — 2h

#### Story 9.3: FDA Audit Dashboard — MUST

**As a** Compliance Officer, **I want** a single dashboard that retrieves batch origins, QC records, temperature logs, and RBAC audit trails instantly, **so that** I can respond to FDA inspections in minutes, not days.

**Key Acceptance Criteria:**
- **Given** an auditor requests the history of a specific batch **When** the Compliance Officer enters the Batch_ID **Then** the dashboard shows: supplier origin, receiving date, QC inspection results + photos, all stock movements, temperature readings for storage zones, and all user actions on that batch.
- **Given** an FDA audit requires data for a date range **When** queried **Then** results return in < 5 seconds for up to 6 months of data.

**Edge Cases:**
1. Batch has been through multiple movements (receiving → warehouse → retail → return) — display full chronological timeline.
2. Data spans a system migration or schema change — ensure backward compatibility of audit queries.

**Tasks:**
- `task-9.3.1` — Implement batch traceability aggregation API (joins Receiving, QC, Ledger, Temp, Audit tables) — 6h
- `task-9.3.2` — Build FDA Audit Dashboard UI with batch search and chronological timeline view — 8h
- `task-9.3.3` — Add PDF/CSV export for full batch audit report — 4h

#### Story 9.4: Loss Analytics Dashboard — SHOULD

**As a** Operations Manager, **I want** a dashboard showing loss trends by cause, product, location, and time period, **so that** I can identify systemic issues and reduce waste.

**Key Acceptance Criteria:**
- **Given** spoilage records exist **When** the Manager opens the Loss Dashboard **Then** they see: total loss by weight and value, breakdown by reason code, top-loss products, top-loss locations, and trend over selected period.
- **Given** a product's loss rate exceeds 10% of received volume in a month **When** the dashboard renders **Then** it is highlighted as a "High Loss" product.

**Edge Cases:**
1. No loss data for selected period — display "No losses recorded" rather than empty charts.
2. Loss percentage calculation when received volume is zero — handle division by zero; show "N/A."

**Tasks:**
- `task-9.4.1` — Implement loss analytics aggregation queries with caching — 5h
- `task-9.4.2` — Build Loss Analytics Dashboard with charts (by reason, product, location, trend) — 8h
- `task-9.4.3` — Add configurable "High Loss" threshold alerts — 3h

---

## Inter-Epic Dependency Map

```
Epic 1 (IAM) ──────────┬──→ Epic 2 (Suppliers)
                        ├──→ Epic 3 (Products) ──→ Epic 4 (Receiving) ──→ Epic 5 (Inventory)
                        │                                                       │
                        │                                                       ├──→ Epic 6 (Orders) ──→ Epic 7 (Delivery)
                        │                                                       │                            │
                        │                                                       ├──→ Epic 8 (Consignment) ←──┘
                        │                                                       │
                        └──→ Epic 9 (Compliance) ←──────────────────────────────┘
```

**Critical Path:** Epic 1 → Epic 3 → Epic 4 → Epic 5 → Epic 6 → Epic 7

**Parallel Tracks (after Epic 1):**
- Track A: Epic 2 (Suppliers) — can start immediately after Epic 1
- Track B: Epic 3 (Products) → Epic 4 → Epic 5 → Epic 6 → Epic 7 (critical path)
- Track C: Epic 8 (Consignment) — can start after Epic 5 + Epic 7
- Track D: Epic 9 (Compliance) — can start after Epic 4 + Epic 5

---

## Summary: Story Count & Effort Estimate

| Epic | Stories | Priority Mix | Est. Hours |
|---|---|---|---|
| 1. IAM | 3 | 3 MUST | ~62h |
| 2. Suppliers | 3 | 2 MUST, 1 SHOULD | ~54h |
| 3. Product Catalog | 2 | 1 MUST, 1 SHOULD | ~30h |
| 4. Receiving & QC | 3 | 3 MUST | ~63h |
| 5. Inventory Engine | 4 | 3 MUST, 1 SHOULD | ~69h |
| 6. B2B Orders | 4 | 3 MUST, 1 SHOULD | ~66h |
| 7. Logistics | 3 | 2 MUST, 1 SHOULD | ~53h |
| 8. Consignment | 3 | 3 MUST | ~47h |
| 9. Compliance | 4 | 3 MUST, 1 SHOULD | ~59h |
| **Total** | **29** | **23 MUST, 6 SHOULD** | **~503h** |
