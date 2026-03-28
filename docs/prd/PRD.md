# 📝 Product Requirements Document: GreenFlow Rwanda

**Project Code Name:** _GreenFlow Rwanda_ 
**Version:** 2.0 (Technical & Business Alignment)

**Status:** Approved for Execution

**Author:** Project Management Team

---

## 1. Executive Summary & Strategic Intent

GreenFlow is a technology-driven fresh fruits & vegetables distribution platform leveraging a **Service-Based Retail (SBR)** model. Instead of supermarkets managing fresh produce, GreenFlow operates the category end-to-end, removing operational burden from retail partners while solving Rwanda's ~40% post-harvest food loss.

**Core Objective:** Transform highly perishable inventory into a predictable, fast-moving asset using strict ledger-based data controls, real-time logistics, and an eventual AI-driven predictability moat.

---

## 2. Market Opportunity & Business Model

### Value Proposition
- **For Retailers:** Eliminates spoilage risk and ensures consistent just-in-time supply.
- **For Farmers:** Provides predictable demand and transparent, performance-based grading.
- **For GreenFlow:** Retains stock ownership until the final sale, capturing higher margins (18-25%) via shared risk.

### Revenue Streams
1. **Service Retail:** Commission-based supermarket operations.
2. **Wholesale:** Bulk supply to B2B channels (hotels, restaurants).
3. **Packaging:** Margin on branded, packed produce.
4. **Logistics:** Distribution margins.

---

## 3. Product Principles & Engineering Constraints

To ensure data integrity and food safety compliance (Rwanda FDA, RICA, RSB), the system architecture is bound by strict constraints:

| Principle | Technical Constraint |
| :--- | :--- |
| **Absolute Traceability** | Every product movement must be logged. **No "Hard Deletes" allowed** in the database. |
| **Ledger-Based Inventory** | Double-entry tracking: Every movement must have an explicit `source_location` and `destination_location`. |
| **Strict Quality Control** | Any batch that does not receive a `PASSED` QC status must be programmatically routed to `QUARANTINE`. |
| **Offline-Tolerance** | Mobile apps (Warehouse pickers, Drivers) MUST function in low/no-connectivity areas and sync when online. |
| **Compliance Readiness** | Built-in "Audit Mode" for instant retrieval of temperature logs, batch origins, and IRB-compliant scorecards. |

---

## 4. System Architecture & Tech Stack

We are building a **Service-Modular Monolith** for the MVP, structured to easily decouple into Microservices (Inventory, Orders) as scale demands.

- **Frontend:** Next.js, Tailwind, React Query, Zustand. (Mobile-responsive web).
- **Backend:** Node.js (NestJS) or Python (FastAPI).
- **Mobile Apps:** Flutter or React Native (Offline-first architecture).
- **Database:** PostgreSQL (Relational integrity is mandatory for financial/inventory ledgers).
- **Infrastructure:** AWS (RDS, S3 for QC photos/contracts, Lambda for expiry alerts). Dockerized for platform resilience.

---

## 5. MVP Scope & Epic Breakdown (Phase 1)

The Year 1 focus is **Operational Discipline** (Building a "Digital Twin" of the physical supply chain).

### A. Operations Core 
- **Epic 1: Identity & Access Management:** JWT auth, RBAC, and immutable audit logs.
- **Epic 2: Partner & Supplier Ecosystem:** Supplier onboarding, price-books, and performance scorecards.
- **Epic 3: Unified Product Catalog:** SKU definitions, shelf-life logic, storage parameters.
- **Epic 4: Smart Receiving & Batch Traceability:** QC workflow, automated batch generation (`[Date]-[SupplierID]-[SKU]`), and mandatory photo uploads for rejects.
- **Epic 5: Ledger-Based Inventory Engine:** Multi-warehouse stock tracking, FIFO enforcement, and automated expiry alerts (Lambda logic).

### B. Revenue & Distribution
- **Epic 6: B2B Order & Fulfillment:** Wholesale portal, automated picking lists, and stock validation.
- **Epic 8: Consignment & Retail Module:** The core SBR engine. Tracking consignment stock, calculating sales vs. remaining stock, and automated commission invoices.
- **Epic 7: Logistics & Last-Mile Delivery:** Driver app with GPS timestamping and signature capture (Proof of Delivery).

### C. Compliance & Guardrails
- **Epic 9: Loss & Compliance Guardrail:** Spoilage recording with "Reason Codes," temperature logs, and FDA Audit dashboards.

---

## 6. Future Phases (The Scaling & Intelligence Moat)

- **Phase 2 (Expansion):** Epic 10 (Multi-Hub Logistics), Epic 11 (Dynamic Pricing & Margin Engine to clear near-expiry stock), Epic 12 (Private Label CRM).
- **Phase 3 (Intelligence):** Epic 13 (Predictive Supply Chain AI for smart reordering), Epic 14 (Computer Vision QC for bias-free grading).
- **Phase 4 (Foundation):** Epic 15 (Platform Resilience & API Gateways).

---

## 7. Project Delivery & Execution Framework

As the Project Management Team, we own **delivery and technical quality**.

### A. Delivery Performance KPIs
1. **Sprint Delivery Reliability:** Percentage of committed sprint tasks completed.
2. **Timeline Adherence:** Progress against the Phase 1 MVP rollout.
3. **Scope Stability Index:** Management of requirement changes during active sprints.

### B. Product Quality KPIs
4. **Defect Escape Rate:** Volume of critical bugs found via operations vs. caught by QA.
5. **System Reliability:** 99.9% uptime for the ledger database; 100% data integrity post-offline sync.
6. **Performance Metrics:** Application load times, rapid barcode scanning latency, and API response under concurrent load.

---

## 8. Customer / Operational Journey

**The Retail Service (SBR) Journey:**
1. Store Onboarded (Epic 2) & Shelf Capacity Set.
2. Auto Replenishment Triggered via FIFO logic (Epic 5).
3. Logistics Delivers with PoD (Epic 7).
4. Daily Sales Logged at Retailer.
5. Commission Calculated & Settlement Invoice Generated (Epic 8).

**The Wholesale Journey:**
1. Buyer Places Order via Portal (Epic 6).
2. Stock Confirmed & Allocated (Epic 5).
3. Delivery with PoD (Epic 7).
4. Invoice Sent & Payment Collected.

