# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is the **product management planning repository** for **AgriFlow Rwanda** (code name: "Operation Harvest") — a B2B fresh produce distribution platform solving post-harvest food loss in Rwanda through a Service-Based Retail (SBR) model. This repo contains no application code; it holds PRDs, epics, user stories, and engineering task specs that drive implementation.

## Repository Structure

```
GEMINI.md              — AI PM context file (core vision, personas, constraints, glossary, output guidelines)
READMED.md             — High-level product roadmap organized by phase and epic
docs/
  prd/PRD.md           — Product Requirements Document (tech stack, constraints, MVP scope)
  epics/EPICS.md       — Jira-ready backlog: epics -> user stories -> technical tasks
  epics/EPICS-FULL.md  — Extended epic descriptions
  stories/             — Detailed user stories and their task breakdowns
    story.{E}.{S}/     — Folder per story (E=epic number, S=story number)
      user-story-{E}.{S}.md
      task-{E}.{S}.{T}.md
```

## Document Hierarchy

**Epic > User Story > Task** — each level links to its parent and children via relative markdown links. Tasks are the smallest unit, scoped to hours of developer work.

## Key Constraints to Enforce When Generating Documents

These constraints are defined in the PRD and GEMINI.md and must be reflected in all generated planning artifacts:

- **No hard deletes** — all entities use soft delete (`is_active: false`). Past audit trails must never become orphaned.
- **Ledger-based inventory** — every stock movement requires explicit `source_location` and `destination_location` (double-entry).
- **Strict QC gating** — any batch without `PASSED` QC status must route to `QUARANTINE` programmatically.
- **Offline-first mobile** — warehouse picker and driver apps must work without connectivity and sync when online.
- **Compliance/audit mode** — temperature logs, batch origins, and RBAC audit trails must be instantly retrievable for Rwanda FDA inspections.

## Tech Stack (for task specs)

- **Frontend:** Next.js, Tailwind, React Query, Zustand
- **Backend:** Node.js (NestJS)
- **Database:** PostgreSQL (relational integrity mandatory for financial/inventory ledgers)
- **Mobile:** Flutter or React Native
- **Infrastructure:** AWS (RDS, S3, Lambda), Docker

## Writing Standards for Generated Artifacts

When creating or editing planning documents, follow GEMINI.md section 6 strictly:

- **User Stories** use `AS A [Persona], I WANT TO [Action] SO THAT [Value]` format
- **Acceptance Criteria** use BDD format (`Given / When / Then`)
- **Every story must include at least 2 edge cases** with mitigations
- **Tasks** must be specific, actionable, and estimable in hours
- **Technical context** in stories must mention data implications, audit trail requirements, and RBAC constraints

## Naming Conventions

- Story folders: `story.{epic}.{story}` (e.g., `story.2.1`)
- Story files: `user-story-{epic}.{story}.md`
- Task files: `task-{epic}.{story}.{task}.md`

## Glossary

- **SBR:** Service-Based Retail (AgriFlow retains stock ownership on consignment)
- **FIFO:** First-In, First-Out (expiry-based, not receipt-date-based)
- **PoD:** Proof of Delivery
- **RQC:** Receiving & Quality Control
- **SRM:** Supplier Relationship Management
- **Traceability ID:** System-generated tag linking Supplier + Date + QC Grade
