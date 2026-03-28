# AgriFlow Agent Context

> **READ THIS FIRST.** Every Claude sub-agent working on this repository must read this file before writing any planning artifact. This is the single authoritative source of truth for constraints, conventions, and project state.

---

## 1. Project Identity

- **Product:** AgriFlow Rwanda (code name: Operation Harvest)
- **Type:** B2B fresh produce distribution platform, Service-Based Retail (SBR) model
- **Problem:** ~40% post-harvest food loss in Rwanda
- **Jira Project:** FL — https://psengayire.atlassian.net/jira/software/projects/FL/list
- **Cloud ID:** `db1c0436-5f1d-4b4a-a439-db5e89d44eed`
- **Architecture:** Service-Modular Monolith (Phase 1). No microservices until Phase 2.

---

## 2. Hard Constraints — NEVER Violate

These are non-negotiable. Every story, task, and acceptance criterion must honour them.

| # | Constraint | What it means in practice |
|---|---|---|
| C1 | **No hard deletes** | All entities use `is_active: false` for removal. Audit trails must never become orphaned. Never write `DELETE FROM` or permanent removal in task specs. |
| C2 | **Ledger-based inventory (double-entry)** | Every stock movement requires both `source_location` AND `destination_location`. No "floating" adjustments. |
| C3 | **Strict QC gating** | Any batch without `PASSED` QC status must be programmatically routed to `QUARANTINE`. No manual override without a Manager-role audit log entry. |
| C4 | **Offline-first mobile** | Warehouse picker and driver apps must function for ≥8 hours without connectivity and sync on reconnect. No online-only mobile workflows. |
| C5 | **Compliance/audit mode** | Temperature logs (`COLD_CHAIN` / `FROZEN` products), batch origins, and RBAC audit trails must be instantly retrievable for Rwanda FDA inspections. `Audit_Logs` is append-only; retain for 5 years. |

**Additional data rules:**
- `bank_details` on Suppliers must **never** be written to `Audit_Logs` in raw form — mask before capture.
- S3 objects (QC photos, supplier contracts) must use private ACL; access via pre-signed URLs only (max 1-hour expiry).
- `storage_type` values: `COLD_CHAIN` | `DRY` | `FROZEN` | `AMBIENT` — this field drives temperature logging scope and QC routing downstream.
- FIFO = **expiry-date-based**, not receipt-date-based.
- Traceability ID format: `[Date]-[SupplierID]-[SKU]`

---

## 3. Output Path Conventions

```
docs/stories/
  story.{E}.{S}/
    user-story-{E}.{S}.md     ← one user story per folder
    task-{E}.{S}.{T}.md       ← one task per file, T starting at 1
```

**Examples:**
- Story folder: `docs/stories/story.4.2/`
- Story file: `docs/stories/story.4.2/user-story-4.2.md`
- Task file: `docs/stories/story.4.2/task-4.2.1.md`

**Never** write to `bmad/outputs/` or `bmad/context/` — those paths do not exist in this repo.

---

## 4. Jira Key Mapping & Sprint Assignments

### Epic → Jira Key → Sprint

| Epic | Jira Key | Sprint | Status |
|------|----------|--------|--------|
| Epic 1: IAM | FL-4 | Sprint 1 | Fully populated in Jira (FL-13 → FL-28) |
| Epic 2: Partner & Supplier Ecosystem | FL-5 | Sprint 2 | Fully populated in Jira (FL-30 → FL-47) |
| Epic 3: Unified Product Catalog | FL-6 | Sprint 2 | Fully populated in Jira (FL-29, FL-31–34, FL-42, FL-48–50) |
| Epic 4: Smart Receiving & Traceability | FL-7 | Sprint 3 | Epic created; stories not yet in Jira |
| Epic 5: Ledger-Based Inventory Engine | FL-8 | Sprint 4 | Epic created; stories not yet in Jira |
| Epic 6: B2B Order & Fulfillment | FL-9 | Sprint 5 | Epic created; stories not yet in Jira |
| Epic 7: Logistics & Last-Mile Delivery | FL-10 | Sprint 5 | Epic created; stories not yet in Jira |
| Epic 8: Consignment & Retail Module | FL-11 | Sprint 6 | Epic created; stories not yet in Jira |
| Epic 9: Loss & Compliance Guardrail | FL-12 | Sprint 6 | Epic created; stories not yet in Jira |

### Sprint Capacity Reference

| Sprint | Weeks | Epics | Target Stories | Est. Capacity |
|--------|-------|-------|----------------|---------------|
| Sprint 1 | 1–2 | Epic 1 | 3 | ~63h |
| Sprint 2 | 3–4 | Epic 2 + Epic 3 | 5 | ~58h |
| Sprint 3 | 5–6 | Epic 4 | 3–4 | ~52h |
| Sprint 4 | 7–8 | Epic 5 | 4 | ~57h |
| Sprint 5 | 9–10 | Epic 6 + Epic 7 | 7 | ~102h |
| Sprint 6 | 11–12 | Epic 8 + Epic 9 | 7 | ~87h |

---

## 5. Epic Dependency Chain (Hard Blockers)

```
Epic 1 (IAM)
  └─► Epic 2 (Suppliers) ──────┐
  └─► Epic 3 (Product Catalog) ┤
                               ▼
                         Epic 4 (Receiving)   ← requires Suppliers table + Product_Master with storage_type
                               │
                               ▼
                         Epic 5 (Inventory)   ← requires Receiving_Logs + Inventory_Batches schema
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
                         (also reads Epics 4 & 5)
```

**Gate rule:** Do not generate stories for an epic whose hard-blocker dependencies are not yet deployed to staging. Check this before generating story files.

**FL-50 gate:** The `storage_type` dependency contract (task-3.2.3) must be reviewed and signed off before Epic 4 and Epic 9 sprint kickoffs.

---

## 6. Writing Standards (CLAUDE.md §Writing Standards)

Every story and task generated must meet these standards — the story validator will check them.

### User Story Format
```
AS A [Persona], I WANT TO [Action] SO THAT [Value/Reason].
```

Valid personas: Admin, Procurement Manager, Warehouse QC Clerk, Warehouse Picker, Logistics Manager, Driver, Finance Manager, System Auditor, Supermarket Manager, Supplier.

### Acceptance Criteria Format (BDD)
```
Given [initial state / precondition]
When [action / event]
Then [expected outcome]
```
- Every story must have **≥3 acceptance criteria** in BDD format.
- Must include at least one **negative / error path** criterion (e.g., what happens when QC fails, network is offline, permission is denied).

### Edge Cases
- Every story must have **≥2 edge cases** with explicit mitigations.
- Edge cases go in a dedicated `## Edge Cases` section.

### Technical Context
Every story must include a `## Technical Context` section covering:
- **Data implications** (which tables are written/read, schema fields involved)
- **RBAC constraints** (which role can perform the action; what happens on unauthorized access → 403 + audit log)
- **Audit trail** (which events are logged to `Audit_Logs`)
- **Constraint flags** (mention any of C1–C5 that apply)

### Task Format
```
task-{E}.{S}.{T}.md

# Task {E}.{S}.{T}: [Verb-led description]

**Story:** [user-story-{E}.{S}.md](../user-story-{E}.{S}.md)
**Estimate:** Xh
**Role:** [Backend | Frontend | Mobile | Infra | Full-Stack]

## Description
[Specific, actionable description. What needs to be built/done.]

## Acceptance
- [ ] [Specific, verifiable completion criterion]
```

- Tasks must be **≤8h** each. If a task exceeds 8h, split it.
- Task descriptions must start with a **verb** (Create, Implement, Build, Write, Add, Configure).
- Hour estimate is **mandatory** — no estimate = incomplete task.

---

## 7. Compliance Scope by Epic

| Epic | Rwanda FDA | RICA | RSB |
|------|-----------|------|-----|
| Epic 1 (IAM) | Audit_Logs 5-year retention | Audit log export for inspections | — |
| Epic 2 (Suppliers) | — | RICA certification status + expiry flag | — |
| Epic 3 (Product Catalog) | storage_type drives temp log scope | RICA grade alignment | rsb_certified field; storage_type |
| Epic 4 (Receiving) | Rejection photos ≥2yr retention; batch traceability | RICA grade assignment (A/B/Reject) | Traceability label; 2-decimal weight (KG) |
| Epic 5 (Inventory) | Full movement history | — | — |
| Epic 6 (Orders) | — | — | — |
| Epic 7 (Logistics) | PoD records | — | — |
| Epic 8 (Consignment) | — | — | — |
| Epic 9 (Compliance) | Temp logs 12-month retrieval within 24h; Audit Mode export | QC records exportable CSV/PDF | — |

---

## 8. Tech Stack (for task specs)

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js, Tailwind CSS, React Query, Zustand |
| Backend | Node.js — NestJS (TypeORM, decorator-based validation) |
| Mobile | Flutter or React Native (offline-first) |
| Database | PostgreSQL (relational integrity mandatory) |
| Infrastructure | AWS RDS, S3 (QC photos / contracts), Lambda (expiry alerts), Docker |

---

## 9. Agent Pre-Flight Checklist

Before writing any story or task file, confirm:

- [ ] I have read this file (agriflow-agent-context.md) in full
- [ ] The target epic's hard-blocker dependencies are resolved (Section 5)
- [ ] I know the correct output path (`docs/stories/story.{E}.{S}/`)
- [ ] I know the Jira key for the parent epic (Section 4)
- [ ] My story format matches Section 6 (AS A / BDD / edge cases / technical context)
- [ ] My tasks have verb-first descriptions and hour estimates ≤8h
- [ ] I have applied all relevant constraints from C1–C5 (Section 2)
- [ ] I have noted the compliance scope for this epic (Section 7)
