# Product Requirements Document: AgriFlow Rwanda

**Project Code Name:** _Operation Harvest_
**Version:** 2.1
**Status:** Approved for Execution
**Author:** Project Management Team

---

## Changelog

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-Q3 | PM Team | Initial draft |
| 2.0 | 2025-Q4 | PM Team | Technical & business alignment; Epic breakdown added |
| 2.1 | 2026-03-28 | PM Team | Fixed project name (AgriFlow); locked in NestJS stack; corrected Epic order (6→7→8); added NFR section; added RICA/RSB compliance scope; added product KPIs; added Epic dependency chain; added MVP launch criteria; added supplier journey; fixed GEMINI.md reference paths |

---

## 1. Executive Summary & Strategic Intent

AgriFlow is a technology-driven fresh fruits & vegetables distribution platform leveraging a **Service-Based Retail (SBR)** model. Instead of supermarkets managing fresh produce, AgriFlow operates the category end-to-end, removing operational burden from retail partners while solving Rwanda's ~40% post-harvest food loss.

**Core Objective:** Transform highly perishable inventory into a predictable, fast-moving asset using strict ledger-based data controls, real-time logistics, and an eventual AI-driven predictability moat.

---

## 2. Market Opportunity & Business Model

### Value Proposition
- **For Retailers:** Eliminates spoilage risk and ensures consistent just-in-time supply.
- **For Farmers:** Provides predictable demand and transparent, performance-based grading.
- **For AgriFlow:** Retains stock ownership until the final sale, capturing higher margins (18–25%) via shared risk.

### Revenue Streams
1. **Service Retail:** Commission-based supermarket operations.
2. **Wholesale:** Bulk supply to B2B channels (hotels, restaurants).
3. **Packaging:** Margin on branded, packed produce.
4. **Logistics:** Distribution margins.

---

## 3. Product Principles, Engineering Constraints & Compliance

To ensure data integrity and food safety compliance, the system architecture is bound by strict constraints aligned with **Rwanda FDA**, **RICA (Rwanda Inspectorate Council of Agriculture)**, and **RSB (Rwanda Standards Board)** requirements.

### 3.1 Engineering Constraints

| Principle | Technical Constraint |
| :--- | :--- |
| **Absolute Traceability** | Every product movement must be logged. **No hard deletes allowed** in the database. All entities use soft-delete (`is_active: false`). |
| **Ledger-Based Inventory** | Double-entry tracking: every movement must have an explicit `source_location` and `destination_location`. |
| **Strict Quality Control** | Any batch that does not receive a `PASSED` QC status must be programmatically routed to `QUARANTINE`. |
| **Offline-Tolerance** | Mobile apps (warehouse pickers, drivers) must function in low/no-connectivity areas and sync when online. Minimum offline operation: 8 hours without connectivity loss. |
| **Compliance Readiness** | Built-in "Audit Mode" for instant retrieval of temperature logs, batch origins, and RBAC audit trails. |

### 3.2 Rwanda FDA Compliance Requirements
- **Temperature monitoring:** Products with `storage_type IN ('COLD_CHAIN', 'FROZEN')` must have temperature logs retrievable for the past 12 months within 24 hours of an inspection request.
- **Batch traceability:** Every inventory batch must be traceable back to its originating supplier and receiving QC event, with a complete chain-of-custody log.
- **Rejection documentation:** All QC-rejected batches require a documented reason code and photographic evidence stored in S3.

### 3.3 RICA (Rwanda Inspectorate Council of Agriculture) Requirements
- **Supplier certification tracking:** Registered suppliers must have their RICA certification status recorded in the system. Expired certifications must trigger a warning on the receiving workflow — the system must not automatically block receiving (RICA does not require automated blocking), but must flag the expiry and log it to the audit trail.
- **Produce grade records:** QC grades assigned at receiving must align with RICA produce grading standards (Grade A / Grade B / Reject). The system's QC grading options must include these classifications.
- **Inspection readiness:** The audit log and QC records must be exportable in a format suitable for RICA annual inspection requests (CSV minimum; PDF preferred).

### 3.4 RSB (Rwanda Standards Board) Requirements
- **Labelling traceability:** Each batch's Traceability ID (`[Date]-[SupplierID]-[SKU]`) must be printable as a label that includes the supplier name, product name, QC grade, and expiry date. This satisfies RSB traceability labelling requirements for produce sold at retail.
- **Weight accuracy:** All receiving records must store weight in KG with two decimal precision (e.g., 124.75 KG) to satisfy RSB measurement standards.
- **Standards compliance flag:** The Product Master can record an optional `rsb_certified: boolean` field for products that carry RSB quality certification marks.

---

## 4. System Architecture & Tech Stack

AgriFlow is built as a **Service-Modular Monolith** for the MVP — a single deployable service with clearly separated modules (Inventory, Orders, Suppliers, etc.) that can be extracted into independent microservices in Phase 2 without a rewrite.

> **Architecture Decision:** Microservices decoupling (Inventory and Orders as independent services) is a **Phase 2 scaling ambition**, not the MVP architecture. All MVP development targets the Modular Monolith pattern.

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| **Frontend** | Next.js, Tailwind CSS, React Query, Zustand | Mobile-responsive; server components for SSR; React Query for caching |
| **Backend** | Node.js — **NestJS** | Chosen for its modular architecture, TypeORM integration, decorator-based validation, and strong TypeScript support |
| **Mobile Apps** | Flutter or React Native | Offline-first architecture for warehouse pickers and drivers |
| **Database** | PostgreSQL | Relational integrity mandatory for financial and inventory ledgers |
| **Infrastructure** | AWS (RDS, S3, Lambda), Docker | S3 for QC photos and supplier contracts; Lambda for expiry alerts; Docker for deployment portability |

---

## 5. MVP Scope & Epic Breakdown (Phase 1)

Year 1 focus: **Operational Discipline** — building a "Digital Twin" of the physical supply chain.

### A. Operations Core

**Epic 1: Identity & Access Management** (FL-4 · Sprint 1)
- **Goal:** Establish a secure perimeter and internal accountability.
- **Scope:** JWT-based login, role-based access control (RBAC), and user profile management.
- **Key Deliverable:** A centralized admin panel to manage permissions and a searchable, exportable audit log for every system action.

**Epic 2: Partner & Supplier Ecosystem** (FL-5 · Sprint 2)
- **Goal:** Digitalize the supply chain entry point.
- **Scope:** Supplier onboarding, contract digital storage (S3), price-book management, and performance scorecards.
- **Key Deliverable:** A Supplier Scorecard that tracks performance based on QC rejection rates and delivery timelines, with risk flagging for underperforming suppliers.

**Epic 3: Unified Product Catalog** (FL-6 · Sprint 2)
- **Goal:** Standardize how items are defined and classified across the entire system.
- **Scope:** SKU definitions, storage type classification (COLD_CHAIN / DRY / FROZEN / AMBIENT), shelf-life logic, units of measure, and product categories.
- **Key Deliverable:** A master SKU list with storage type and shelf-life parameters that powers all inventory, receiving, and compliance modules downstream.

**Epic 4: Smart Receiving & Batch Traceability** (FL-7 · Sprint 3)
- **Goal:** Ensure food safety and eliminate "blind" stock entry at the warehouse gate.
- **Scope:** QC workflow with RICA grade assignment, automated batch code generation (`[Date]-[SupplierID]-[SKU]`), expiry date calculation from product shelf-life, and mandatory photo uploads for rejections.
- **Key Deliverable:** A mobile-friendly inspection interface that produces a Traceability ID for every accepted batch and routes all failed batches to QUARANTINE automatically.

**Epic 5: Ledger-Based Inventory Engine** (FL-8 · Sprint 4)
- **Goal:** Provide real-time stock visibility and prevent spoilage through disciplined rotation.
- **Scope:** Multi-warehouse stock tracking, FIFO enforcement (expiry-based), and automated low-stock and expiry alerts via AWS Lambda.
- **Key Deliverable:** A real-time Inventory Ledger where every gram of produce is accounted for with a full double-entry movement history (source → destination for every transaction).

### B. Revenue & Distribution

**Epic 6: B2B Order & Fulfillment** (FL-9 · Sprint 5)
- **Goal:** Streamline the revenue workflow from wholesale buyer to warehouse dispatch.
- **Scope:** Wholesale order portal, picking list generation with automated FIFO batch allocation, and stock validation.
- **Key Deliverable:** Digital invoicing and delivery assignment system that confirms stock availability before order confirmation.

**Epic 7: Logistics & Last-Mile Delivery** (FL-10 · Sprint 5)
- **Goal:** End-to-end accountability from warehouse to retail shelf.
- **Scope:** Driver assignments, mobile Proof of Delivery (PoD) with GPS timestamping, signature capture, and offline-first operation.
- **Key Deliverable:** A driver interface for acknowledging deliveries and reporting discrepancies, with geofenced drop-off confirmation.

**Epic 8: Consignment & Retail Module** (FL-11 · Sprint 6)
- **Goal:** Manage stock not yet "sold" sitting at retail locations under the SBR model.
- **Scope:** Consignment stock tracking per retail location, daily sales reporting from retail partners, and automated commission calculations.
- **Key Deliverable:** Settlement invoices generated automatically based on actual sales vs. remaining stock, closing the financial loop for each consignment cycle.

### C. Compliance & Guardrails

**Epic 9: Loss & Compliance Guardrail** (FL-12 · Sprint 6)
- **Goal:** Protect margins and ensure instant regulatory readiness for Rwanda FDA, RICA, and RSB inspections.
- **Scope:** Spoilage recording with Reason Codes, temperature log monitoring for COLD_CHAIN and FROZEN products, and QC audit dashboards.
- **Key Deliverable:** A Loss Dashboard identifying exactly where and why produce is being wasted, plus an Audit Mode for on-demand FDA/RICA inspection exports covering any requested date range.

### 5.1 Epic Dependency Chain

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
- Epic 4 cannot begin until Epic 2 (Suppliers table) and Epic 3 (Product_Master with `storage_type`) are deployed to staging.
- Epic 5 cannot begin until Epic 4's `Receiving_Logs` and `Inventory_Batches` schema exists.
- Epic 9's `storage_type` dependency contract (FL-50) must be reviewed and signed off before Epic 4 and Epic 9 sprint kickoffs.

---

## 6. Non-Functional Requirements (NFRs)

### 6.1 Performance
| Requirement | Target | Measurement |
|-------------|--------|-------------|
| API response time | p95 < 300ms for read endpoints; p95 < 500ms for write endpoints | Load test with k6 at 100 concurrent users |
| Barcode scan latency | < 200ms from scan to UI feedback | Measured on the receiving mobile flow |
| Page initial load | < 2s LCP on 4G mobile connection | Lighthouse CI on critical paths |
| Offline sync | Sync completes in < 30 seconds after connectivity restores | Tested with 8 hours of offline operations |

### 6.2 Reliability & Availability
| Requirement | Target |
|-------------|--------|
| Ledger database uptime | 99.9% (< 8.7h downtime/year) |
| Data integrity post-offline sync | 100% — no data loss for operations recorded offline |
| AWS RDS automated backups | Daily snapshots with 7-day retention |
| Disaster recovery RTO | < 4 hours for full service restoration from backup |

### 6.3 Security
- All API endpoints require JWT authentication (Epic 1 RBAC).
- `bank_details` JSONB on Suppliers must never be written to Audit_Logs in raw form — masked before any audit capture.
- S3 objects (QC photos, supplier contracts) must use private ACL; access via pre-signed URLs only (max 1-hour expiry for read URLs).
- Passwords must be hashed with bcrypt (min cost factor 12). No plaintext credential storage.
- Rate limiting: password reset endpoint max 3 requests/hour per IP.

### 6.4 Scalability
- The Modular Monolith must be designed so that the Inventory and Order modules can be extracted into independent services (separate DB schemas, no cross-module DB joins in service layer) without a rewrite.
- Target user load for Phase 1 pilot: 50 concurrent internal users; 200 registered accounts.

### 6.5 Compliance & Data Retention
- `Audit_Logs` table must be append-only and retained for minimum **5 years** (Rwanda FDA and RICA inspection lookback requirement).
- Temperature logs must be retained for minimum **12 months** per Rwanda FDA requirements.
- QC rejection photos stored in S3 must not be deleted for minimum **2 years**.

---

## 7. Project Delivery & Execution Framework

### 7.1 Delivery Performance KPIs
1. **Sprint Delivery Reliability:** Percentage of committed sprint tasks completed on time.
2. **Timeline Adherence:** Progress against the Phase 1 MVP rollout plan.
3. **Scope Stability Index:** Percentage of sprint tasks that complete without mid-sprint scope change.
4. **Defect Escape Rate:** Volume of critical bugs found via operations vs. caught by QA.

### 7.2 Product Outcome KPIs (MVP Success Metrics)
These metrics define whether Phase 1 has achieved its operational objective. Measured 60 days post-pilot launch:

| Metric | Target | Rationale |
|--------|--------|-----------|
| Post-harvest food loss (pilot SKUs) | ≤ 20% (from baseline ~40%) | Core problem being solved |
| Inventory accuracy rate | ≥ 98% (system vs. physical count) | Validates ledger-based tracking |
| Receiving cycle time | < 30 min per delivery from arrival to batch-posted | Validates QC workflow efficiency |
| Supplier onboarding (pilot) | 10+ active suppliers registered in system | Validates Epic 2 adoption |
| QC rejection documentation rate | 100% of rejections have photo + reason code | Validates FDA compliance readiness |
| On-time delivery rate (logistics) | ≥ 90% of orders delivered within agreed window | Validates Epic 7 reliability |
| Order fulfillment cycle time | < 24h from order placement to dispatch | Validates Epic 6 throughput |

### 7.3 Technical Quality KPIs
- **System Reliability:** 99.9% uptime for ledger database; 100% data integrity post-offline sync.
- **Performance Metrics:** API response times, barcode scanning latency, load times (see NFRs Section 6.1).

---

## 8. Customer & Operational Journeys

### 8.1 Supplier/Farmer Journey
1. Supplier registered in system by Procurement Manager (Epic 2) — RICA certification status recorded.
2. Commercial agreement uploaded as contract document (S3).
3. Supplier delivers produce to warehouse — receiving clerk selects supplier from verified registry.
4. QC inspection conducted: weight recorded, RICA grade assigned, photos taken for any rejects.
5. Batch Traceability ID generated (`[Date]-[SupplierID]-[SKU]`) and RSB-compliant label printed.
6. Supplier's Performance Scorecard updated with delivery result (Epic 2 scoring engine).
7. Supplier notified of QC result and any rejection with documented evidence.

### 8.2 Retail Service (SBR) Journey
1. Store onboarded (Epic 2) — shelf capacity and consignment terms set.
2. Auto-replenishment triggered via FIFO logic (Epic 5).
3. Logistics delivers with PoD (Epic 7) — GPS timestamped, signature captured.
4. Daily sales logged at retailer (Epic 8).
5. Commission calculated and settlement invoice generated (Epic 8).

### 8.3 Wholesale Journey
1. Buyer places order via portal (Epic 6).
2. Stock confirmed and allocated using FIFO (Epic 5).
3. Delivery with PoD (Epic 7).
4. Invoice sent and payment collected.

---

## 9. MVP Launch Criteria

Phase 1 is considered ready for pilot launch when all of the following gates are met:

### Gate 1: Core Operations (Hard Required)
- [ ] Epic 1 (IAM): All roles provisioned; audit log operational
- [ ] Epic 2 (Suppliers): At least 10 pilot suppliers registered with contracts attached
- [ ] Epic 3 (Product Catalog): All pilot SKUs catalogued with `storage_type` and shelf-life data
- [ ] Epic 4 (Receiving): QC workflow live; all receiving events generating Traceability IDs
- [ ] Epic 5 (Inventory): FIFO enforcement active; stock ledger accurate to ± 0.5% vs physical count

### Gate 2: Revenue Workflows (Required for Commercial Launch)
- [ ] Epic 6 (Orders): At least 1 wholesale order processed end-to-end
- [ ] Epic 7 (Logistics): PoD capture working on driver mobile app (online + offline)
- [ ] Epic 8 (Consignment): At least 1 settlement invoice generated from real sales data

### Gate 3: Compliance Readiness
- [ ] Epic 9 (Compliance): Temperature logs captured for all COLD_CHAIN/FROZEN SKUs
- [ ] FDA Audit Mode: Temperature log export working for any 30-day period
- [ ] RICA: Supplier certification status visible in system; QC grades aligned to RICA standards
- [ ] RSB: Traceability label printable for any batch

**Pilot scope:** Minimum 1 supermarket location, 1 warehouse, 10 suppliers, 20 active SKUs.
**Pilot duration:** 60 days before full commercial rollout decision.

---

## 10. Future Phases

### Phase 2 — Scaling & Expansion (Years 2–3)

**Epic 10: Multi-Hub & Regional Logistics**
- **Goal:** Transition from a single warehouse to a national distribution network.
- **Scope:** Inter-warehouse transfers, regional stock balancing, multi-hub synchronization.

**Epic 11: Dynamic Pricing & Margin Engine**
- **Goal:** Optimize revenue based on real-time market conditions and inventory age.
- **Scope:** Volume discounts, time-based promotions to clear near-expiry stock, and regional price adjustments.

**Epic 12: Private Label & CRM**
- **Goal:** Build brand equity and manage long-term corporate retail relationships.
- **Scope:** Private label batch tracking, SLA monitoring for supermarkets, and automated contract renewal reminders.

> **Architecture evolution (Phase 2):** Extract the Inventory and Order modules from the Modular Monolith into independent microservices with separate databases as load demands.

### Phase 3 — Intelligence (Competitive Moat)

**Epic 13: Predictive Supply Chain AI**
- **Goal:** Use historical data to out-compete traditional distributors on supply reliability.
- **Scope:** Demand forecasting per outlet, AI-driven spoilage prediction, and automated procurement suggestions.
- **Key Deliverable:** A Smart Reorder system that predicts what a supermarket needs before they ask for it.

**Epic 14: Computer Vision QC**
- **Goal:** Standardize produce quality grading at scale without human bias.
- **Scope:** Image-based grading of produce and automated visual defect detection during receiving.

### Phase 4 — Infrastructure

**Epic 15: Platform Resilience & Integration**
- **Goal:** Ensure 99.9% uptime and seamless third-party connectivity.
- **Scope:** Dockerized deployment pipeline, API Gateway for banking/ERP integrations, and automated disaster recovery.
