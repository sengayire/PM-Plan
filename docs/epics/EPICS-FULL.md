# AgriFlow Rwanda — Full Epic Descriptions

> **Purpose:** Extended descriptions for all Epics across all phases. Each Epic is designed as an independent, functional module following AgriFlow's **Service-Modular Monolith** architecture for Phase 1 — a single deployable service with clearly separated modules that can be extracted into microservices in Phase 2 without a rewrite.
>
> **Authoritative Reference:** [PRD](../prd/PRD.md) · [CLAUDE.md](../../CLAUDE.md) · [Epics Backlog](./EPICS.md)

---

## Phase 1: MVP — Operational Core (Year 1)

> **Theme:** Operational Discipline — building a "Digital Twin" of the physical supply chain.
>
> **Architecture:** Service-Modular Monolith. Microservices extraction (Inventory, Orders) is a Phase 2 ambition, not an MVP target.

### A. Operations Core

---

### Epic 1: Identity & Access Management (IAM)

**Jira:** FL-4 · **Sprint:** 1

**Goal:** Establish a secure perimeter and internal accountability for every action in the system.

**Functional Scope:**
- JWT-based authentication (login, token refresh, revocation)
- Role-based access control (RBAC): Admin, Manager, Warehouse Picker, Driver, Finance
- User lifecycle management (create, activate, deactivate — no hard deletes; `is_active: false`)
- Immutable, append-only `Audit_Logs` table covering all POST/PATCH/DELETE events (entity, actor, timestamp, old/new values)
- Password reset with rate limiting (max 3 requests/hour per IP)

**Key Deliverable:** A centralized admin panel to manage permissions and a searchable, exportable audit log for every system action — exportable as CSV/PDF for Rwanda FDA and RICA inspections covering any requested date range.

**Out of Scope:** SSO/OAuth federation, biometric authentication, multi-tenant IAM.

**Dependencies:** None — this Epic is foundational and blocks all others.

**Compliance Notes:**
- `Audit_Logs` must be append-only and retained for minimum **5 years** (Rwanda FDA/RICA lookback requirement).
- `bank_details` on Suppliers must be masked before any audit log capture — never stored in raw form.
- All API endpoints must be protected by role-based middleware; unauthorized access attempts must be logged.

---

### Epic 2: Partner & Supplier Ecosystem

**Jira:** FL-5 · **Sprint:** 2

**Goal:** Digitalize the supply chain entry point and establish a verified, performance-tracked supplier registry.

**Functional Scope:**
- Supplier onboarding: company profile, contact details, `bank_details` (encrypted JSONB), RICA certification status and expiry tracking
- Contract digital storage: document upload and retrieval via AWS S3 (private ACL, pre-signed URLs)
- Price-book management: SKU-level pricing per supplier
- **Supplier Scorecard engine:** automated scoring based on QC rejection rates, delivery timelines, and variance

**Key Deliverable:** A Supplier Scorecard that tracks performance based on QC rejection rates and delivery timelines, with risk-flagging for underperforming suppliers. RICA certification expiry triggers a warning on the receiving workflow and is written to the audit trail (system must not auto-block receiving — flag only, per RICA requirement).

**Out of Scope:** Automated supplier payment disbursement, procurement tendering workflows.

**Dependencies:** Epic 1 (IAM — RBAC and audit logging must be live).

**Compliance Notes (RICA):**
- Registered suppliers must have their RICA certification status and expiry date recorded.
- Expired certifications must generate a visible warning during the receiving workflow (Epic 4) and be logged to `Audit_Logs`.
- Supplier data must be exportable for RICA annual inspection requests (CSV minimum).

---

### Epic 3: Unified Product Catalog

**Jira:** FL-6 · **Sprint:** 2

**Goal:** Standardize how items are defined, classified, and referenced across the entire system.

**Functional Scope:**
- SKU definitions: name, category, unit of measure (UoM), description
- `storage_type` classification: `COLD_CHAIN` / `DRY` / `FROZEN` / `AMBIENT` — drives downstream compliance (temperature logging in Epic 9) and QC routing (Epic 4)
- Shelf-life parameters per SKU: used by Epic 4 to calculate expiry dates at receiving
- Product categories (e.g., Leafy Greens, Root Vegetables, Citrus)
- Optional `rsb_certified: boolean` flag for RSB quality certification marks

**Key Deliverable:** A master SKU list with `storage_type` and shelf-life parameters that powers all inventory, receiving, and compliance modules downstream.

**Out of Scope:** Consumer-facing product pages, nutritional labelling, packaging design tooling.

**Dependencies:** Epic 1 (IAM).

> **Dependency Contract:** The `storage_type` field on `Product_Master` is a hard dependency for Epic 4 (receiving expiry calculation) and Epic 9 (temperature log scope). The `storage_type` Dependency Contract (FL-50) must be reviewed and signed off before Epic 4 and Epic 9 sprint kickoffs.

**Compliance Notes (RSB):**
- The Product Master's `storage_type` determines which batches require temperature logging (RSB and Rwanda FDA requirement).
- All products carrying RSB quality certification must have `rsb_certified: true` recorded.

---

### Epic 4: Smart Receiving & Batch Traceability

**Jira:** FL-7 · **Sprint:** 3

**Goal:** Ensure food safety and eliminate "blind" stock entry at the warehouse gate.

**Functional Scope:**
- QC inspection workflow with **RICA-aligned grade assignment** (Grade A / Grade B / Reject)
- Automated Traceability ID generation: `[Date]-[SupplierID]-[SKU]`
- Expiry date calculation: computed from product shelf-life parameters defined in Epic 3
- Mandatory photo upload (to AWS S3) for every rejected batch — reason code required
- Automatic programmatic routing: any batch without `PASSED` QC status → `QUARANTINE` stock status (no manual override without a Manager-level role action logged to audit trail)
- RSB-compliant batch label print: Traceability ID, supplier name, product name, QC grade, expiry date
- Weight recorded in KG with two decimal precision (RSB measurement standard)

**Key Deliverable:** A mobile-friendly inspection interface that produces a Traceability ID for every accepted batch and programmatically routes all failed batches to QUARANTINE, with photo evidence and reason codes mandatory for all rejections.

**Out of Scope:** Automated temperature sensor integration (covered in Epic 9), multi-location receiving (covered in Epic 10).

**Dependencies:** Epic 2 (Suppliers registry must exist), Epic 3 (`Product_Master` with `storage_type` and shelf-life params must be deployed to staging before Epic 4 sprint kickoff — hard blocker).

**Compliance Notes (Rwanda FDA / RICA / RSB):**
- Every batch must be traceable back to its originating supplier and receiving QC event.
- QC rejection photos stored in S3 must not be deleted for minimum **2 years**.
- Batch label must satisfy RSB traceability labelling requirements for produce sold at retail.
- QC grades must align with RICA produce grading standards.

---

### Epic 5: Ledger-Based Inventory Engine

**Jira:** FL-8 · **Sprint:** 4

**Goal:** Provide real-time stock visibility and prevent spoilage through disciplined, rule-enforced stock rotation.

**Functional Scope:**
- **Double-entry inventory ledger:** every stock movement requires explicit `source_location` AND `destination_location` — no "floating" stock adjustments
- FIFO enforcement: expiry-date-based (not receipt-date-based) — oldest expiry is always allocated first
- Multi-warehouse stock tracking (single warehouse in Phase 1; architecture must support multi-hub in Phase 2)
- Automated low-stock and near-expiry alerts via AWS Lambda
- Real-time movement log: full history of every transaction per batch (receive → pick → dispatch → return → write-off)

**Key Deliverable:** A real-time Inventory Ledger where every gram of produce is accounted for with a full double-entry movement history (`source → destination` for every transaction) — the authoritative record for stock counts, FIFO allocation, and compliance audits.

**Out of Scope:** Inter-warehouse transfers and stock balancing (Epic 10), demand forecasting (Epic 13).

**Dependencies:** Epic 4 (`Receiving_Logs` and `Inventory_Batches` schema must exist — hard blocker).

---

### B. Revenue & Distribution

---

### Epic 6: B2B Order & Fulfillment

**Jira:** FL-9 · **Sprint:** 5

**Goal:** Streamline the revenue workflow from wholesale buyer to warehouse dispatch.

**Functional Scope:**
- Wholesale order portal: buyers place orders against available inventory
- Stock validation at order submission: confirms stock availability before order confirmation
- Picking list generation with **automated FIFO batch allocation** (expiry-based, sourced from Epic 5 ledger)
- Digital invoicing: auto-generated on order confirmation
- Delivery assignment: links confirmed orders to logistics (Epic 7)

**Key Deliverable:** A digital invoicing and delivery assignment system that confirms stock availability before order confirmation and produces FIFO-allocated picking lists for warehouse staff.

**Out of Scope:** Consumer (B2C) orders, multi-currency invoicing, credit facility management (future Phase 3 CRM).

**Dependencies:** Epic 5 (Inventory Ledger — FIFO allocation requires live stock data).

---

### Epic 7: Logistics & Last-Mile Delivery

**Jira:** FL-10 · **Sprint:** 5

**Goal:** End-to-end accountability from warehouse dispatch to retail shelf.

**Functional Scope:**
- Driver assignment: link delivery runs to registered drivers (from Epic 1 user registry)
- Mobile Proof of Delivery (PoD): GPS timestamp, signature capture, and item confirmation
- Offline-first operation: full functionality without connectivity; sync on reconnect (minimum 8-hour offline tolerance)
- Discrepancy reporting: driver can flag short delivery, damaged goods, or refused delivery at point of handoff
- Geofenced drop-off confirmation: GPS validates driver is at the correct delivery location

**Key Deliverable:** A driver mobile interface for acknowledging deliveries and reporting discrepancies, with geofenced drop-off confirmation — all PoD records synced to the system and linked to the originating order.

**Out of Scope:** Route optimization algorithms (Phase 2), fleet telematics integration, third-party logistics API.

**Dependencies:** Epic 6 (B2B Orders — delivery assignments are generated from confirmed orders).

---

### Epic 8: Consignment & Retail Module

**Jira:** FL-11 · **Sprint:** 6

**Goal:** Manage stock sitting at retail locations under AgriFlow's Service-Based Retail (SBR) model — stock remains AgriFlow's asset until sold.

**Functional Scope:**
- Consignment stock tracking: per retail location, linked to the originating batch (Traceability ID from Epic 4)
- Daily sales reporting from retail partners: actual units sold per SKU per day
- Automated commission calculation: based on actual sales vs. consignment quantity delivered
- Settlement invoice generation: closes the financial loop per consignment cycle

**Key Deliverable:** Settlement invoices generated automatically based on actual sales vs. remaining consignment stock, providing a clean financial close for each SBR cycle.

**Out of Scope:** Retail POS system integration (Phase 2), consumer loyalty programs.

**Dependencies:** Epic 7 (Logistics PoD — consignment stock at location is created when PoD is confirmed).

---

### C. Compliance & Guardrails

---

### Epic 9: Loss & Compliance Guardrail

**Jira:** FL-12 · **Sprint:** 6

**Goal:** Protect margins and ensure instant regulatory readiness for Rwanda FDA, RICA, and RSB inspections.

**Functional Scope:**
- Spoilage recording with structured Reason Codes (e.g., `COLD_CHAIN_FAILURE`, `TRANSIT_DAMAGE`, `EXPIRY_REACHED`)
- Temperature log monitoring for products with `storage_type IN ('COLD_CHAIN', 'FROZEN')` — continuous capture
- QC cleaning checklists and compliance workflow records
- **Loss Dashboard:** aggregates spoilage by location, SKU, supplier, and reason code
- **Audit Mode:** on-demand export of temperature logs, batch origins, and RBAC audit trails for any requested date range — formats: CSV (minimum), PDF (preferred)

**Key Deliverable:** A Loss Dashboard identifying exactly where and why produce is being wasted, plus an Audit Mode for on-demand FDA/RICA inspection exports that can satisfy any inspection request within 24 hours.

**Out of Scope:** Predictive spoilage modeling (Epic 13), physical sensor hardware integration (third-party contract scope).

**Dependencies:** Epic 4 (Receiving/batch data), Epic 5 (Inventory Ledger — spoilage is a ledger write-off event). `storage_type` Dependency Contract (FL-50) must be reviewed before sprint kickoff.

**Compliance Notes (Rwanda FDA / RICA / RSB):**
- Temperature logs must be retrievable for the past **12 months** within 24 hours of an inspection request (Rwanda FDA).
- All QC-rejected batches must have a documented reason code and photographic evidence (Rwanda FDA).
- Audit log and QC records must be exportable in CSV/PDF for RICA annual inspection requests.

---

### Epic Dependency Chain

```
Epic 1 (IAM)
  └─► Epic 2 (Suppliers) ──────┐
  └─► Epic 3 (Product Catalog) ┤
                               ▼
                         Epic 4 (Receiving)
                               │
                               ▼
                         Epic 5 (Inventory)
                               │
                    ┌──────────┴──────────┐
                    ▼                     ▼
              Epic 6 (Orders)      Epic 7 (Logistics)
                    │                     │
                    └──────────┬──────────┘
                               ▼
                         Epic 8 (Consignment)
                               │
                               ▼
                         Epic 9 (Compliance)
                         (also reads from Epics 4 & 5)
```

**Hard blockers:**
- Epic 4 cannot begin until Epic 2 (`Suppliers` table) and Epic 3 (`Product_Master` with `storage_type` and shelf-life params) are deployed to staging.
- Epic 5 cannot begin until Epic 4's `Receiving_Logs` and `Inventory_Batches` schema exists.
- Epic 9's `storage_type` dependency contract (FL-50) must be reviewed and signed off before Epic 4 and Epic 9 sprint kickoffs.

---

## Phase 2: Scaling & Expansion (Years 2–3)

> **Architecture evolution:** Extract the Inventory and Order modules from the Modular Monolith into independent microservices with separate databases as load demands increase.

### Epic 10: Multi-Hub & Regional Logistics

**Goal:** Transition from a single warehouse to a national distribution network.

**Functional Scope:** Inter-warehouse transfers with full ledger entries (source → destination), regional stock balancing, and multi-hub synchronization.

**Key Deliverable:** A hub management dashboard allowing stock rebalancing across regional warehouses, with full traceability maintained across transfer events.

---

### Epic 11: Dynamic Pricing & Margin Engine

**Goal:** Optimize revenue based on real-time market conditions and inventory age.

**Functional Scope:** Volume discounts, time-based promotions to clear near-expiry stock (days-to-expiry pricing), and regional price adjustments per buyer segment.

**Key Deliverable:** A pricing rule engine that automatically applies discounts as batches approach expiry, maximizing margin recovery over straight write-off.

---

### Epic 12: Private Label & CRM

**Goal:** Build brand equity and manage long-term corporate retail relationships.

**Functional Scope:** Private label batch tracking, SLA monitoring for supermarket partners, and automated contract renewal reminders.

**Key Deliverable:** A CRM view per supermarket partner showing SLA adherence, consignment settlement history, and upcoming contract renewals.

---

## Phase 3: Intelligence (Competitive Moat)

### Epic 13: Predictive Supply Chain AI

**Goal:** Use historical data to out-compete traditional distributors on supply reliability.

**Functional Scope:** Demand forecasting per outlet, AI-driven spoilage prediction based on historical loss patterns, and automated procurement suggestions.

**Key Deliverable:** A Smart Reorder system that predicts what a supermarket needs before they request it — reducing both stockouts and over-ordering.

---

### Epic 14: Computer Vision QC

**Goal:** Standardize produce quality grading at scale without human bias.

**Functional Scope:** Image-based grading of produce at receiving, automated visual defect detection, and integration with the existing QC workflow (Epic 4) as a decision-assist layer.

**Key Deliverable:** A CV-assisted grading interface that flags quality issues in real time during receiving, with the final grade still confirmed by a QC operator (human-in-the-loop for Phase 3).

---

## Phase 4: Infrastructure

### Epic 15: Platform Resilience & Integration

**Goal:** Ensure 99.9% uptime and seamless third-party connectivity as the platform scales.

**Functional Scope:** Dockerized deployment pipeline (CI/CD), API Gateway for banking and ERP integrations (e.g., accounting systems for settlement invoices), and automated disaster recovery with RTO < 4 hours.

**Key Deliverable:** A fully containerized, monitored deployment with automated failover and documented integration contracts for third-party financial and ERP systems.
