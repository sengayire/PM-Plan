# AgriFlow Rwanda — Operation Harvest

> **PM Planning Repository.** This repo contains no application code — only PRDs, epics, user stories, and engineering task specs that drive implementation. The operative file for Claude Code is [CLAUDE.md](./CLAUDE.md).

---

## What Is This?

**AgriFlow** is a tech-enabled B2B agri-distribution platform solving systemic fresh produce supply chain inefficiencies in Rwanda through a data-centric, ledger-based operational model using a **Service-Based Retail (SBR)** model — AgriFlow retains stock ownership until the final sale, removing operational burden from retail partners.

**Goal:** Transform highly perishable farm goods into a predictable, fast-moving financial asset.

---

## Market Context (Rwanda)

- **The Problem:** Rwanda faces ~40% post-harvest food loss, particularly in highly perishable goods (tomatoes 33–60%, potatoes ~25%).
- **Root Causes:** Lack of cold chain infrastructure (only 9% of dealers have cold rooms, 5% use refrigerated trucks), poor transport, and reliance on informal middlemen.
- **The Solution:** Just-In-Time (JIT) delivery, strict expiry-based FIFO stock rotation, and removing informal friction through a digitally-operated supply chain.

---

## Key Personas

| Persona | Core Need |
|---------|-----------|
| **Retailer (Supermarket Manager)** | Consistent JIT supply, zero spoilage risk, digital ordering away from WhatsApp |
| **Farmer/Supplier** | Predictable demand, fair RICA-aligned QC grading, performance-based ratings |
| **Warehouse QC Clerk & Picker** | Structured tools enforcing rules — mandatory photos for rejections, forced FIFO picking |
| **Logistics/Delivery Driver** | Offline-first routing, geofenced drop-off validation, PoD capture (8-hour offline minimum) |

---

## Strategic Phases

| Phase | Timeline | Theme | Epics |
|-------|----------|-------|-------|
| **Phase 1 — MVP** | Year 1 | Operational Discipline: "Digital Twin" of the supply chain | 1–9 |
| **Phase 2 — Scaling** | Years 2–3 | Multi-hub logistics, dynamic pricing, private label & CRM | 10–12 |
| **Phase 3 — Intelligence** | Years 3–4 | Predictive demand forecasting, Computer Vision QC | 13–14 |
| **Phase 4 — Infrastructure** | Ongoing | Platform resilience, banking/ERP API integrations | 15 |

---

## Document Index

| Document | Purpose |
|----------|---------|
| [CLAUDE.md](./CLAUDE.md) | **Start here for Claude Code** — constraints, writing standards, git workflow, agent boot sequence |
| [docs/prd/PRD.md](./docs/prd/PRD.md) | Full PRD: tech stack, NFRs, MVP scope, compliance (FDA/RICA/RSB), KPIs, launch criteria |
| [docs/epics/EPICS.md](./docs/epics/EPICS.md) | Jira-ready backlog: epics → stories → tasks with sprint assignments |
| [docs/epics/EPICS-FULL.md](./docs/epics/EPICS-FULL.md) | Extended epic descriptions with compliance notes and dependency chain |
| [docs/context/agriflow-agent-context.md](./docs/context/agriflow-agent-context.md) | Sub-agent shared context: constraints, Jira keys, output paths, sprint map |
| [docs/stories/](./docs/stories/) | All user stories and tasks, organised by `story.{epic}.{story}/` |

---

## Phase 1 Epic Overview

| Epic | Jira | Sprint | Goal |
|------|------|--------|------|
| Epic 1: Identity & Access Management | FL-4 | 1 | Secure perimeter + immutable audit log |
| Epic 2: Partner & Supplier Ecosystem | FL-5 | 2 | Verified supplier registry + scorecards |
| Epic 3: Unified Product Catalog | FL-6 | 2 | Master SKU list with storage_type + shelf-life |
| Epic 4: Smart Receiving & Traceability | FL-7 | 3 | QC workflow, Traceability IDs, QUARANTINE routing |
| Epic 5: Ledger-Based Inventory Engine | FL-8 | 4 | Double-entry stock ledger, expiry-based FIFO |
| Epic 6: B2B Order & Fulfillment | FL-9 | 5 | Order portal, FIFO picking lists, digital invoicing |
| Epic 7: Logistics & Last-Mile Delivery | FL-10 | 5 | Driver app, offline PoD, geofenced drop-off |
| Epic 8: Consignment & Retail Module | FL-11 | 6 | SBR consignment tracking, settlement invoices |
| Epic 9: Loss & Compliance Guardrail | FL-12 | 6 | Loss dashboard, temperature logs, FDA Audit Mode |
