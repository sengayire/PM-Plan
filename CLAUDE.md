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

## Git Workflow Standards

These rules apply to every branch, commit, and PR Claude creates in this repository.

### Branch Naming

```
{type}/{scope}
```

| Type | When to use |
|------|-------------|
| `docs` | New or updated planning artifacts (stories, tasks, epics, PRD) |
| `feat` | Net-new feature scoping work |
| `fix` | Corrections to existing documents |
| `chore` | Housekeeping — restructuring, renaming, config, standards |
| `epic` | Epic-level work that spans multiple stories |

**Rules:**
- Always branch from `main` (`git checkout main && git pull` first)
- `{scope}` is kebab-case, max 40 characters
- Include the Jira key when the work is ticket-scoped (e.g., `FL-42`)
- Never use an auto-generated worktree name as the branch name

**Examples:**
```
docs/epic-2-story-3-rqc-gate
feat/FL-40-supplier-onboarding
fix/story-1.2-edge-cases
chore/branching-standards
```

---

### Commit Message Template

```
{type}({scope}): {short description}          ← max 72 chars

{body}                                         ← what changed and why; omit if obvious
                                               ← wrap at 80 chars

Jira: {KEY}                                    ← omit if no ticket
Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

**Types:** `docs` | `feat` | `fix` | `chore` | `refactor`

**Scope:** use the epic/story ref or area — e.g., `epic-2`, `story-3.2`, `prd`, `epics`

**Example:**
```
docs(story-2.3): add BDD acceptance criteria for RQC QC gate

Added Given/When/Then scenarios for quarantine routing when a batch
does not have PASSED QC status. Includes two edge cases: partial
batch failure and re-inspection workflow.

Jira: FL-42
Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

---

### PR Naming

```
[TYPE] {scope}: {short description}            ← max 72 chars total
```

**Examples:**
```
[DOCS] story-2.3: RQC batch QC gate acceptance criteria
[FEAT] epic-4: offline-first driver delivery flow planning
[CHORE] standards: branch naming and commit templates
[FIX] story-1.2: correct edge case mitigation wording
```

---

### PR Description Template

```markdown
## Summary
- {bullet: what changed}
- {bullet: why — motivation or Jira context}

## Scope
- **Epics affected:** E{n} — {name}
- **Stories / tasks:** story.{E}.{S}, task.{E}.{S}.{T}
- **Jira:** {KEY}

## Checklist
- [ ] Follows GEMINI.md §6 writing standards
- [ ] User stories use AS A / I WANT TO / SO THAT format
- [ ] Acceptance criteria in BDD (Given / When / Then)
- [ ] At least 2 edge cases documented per story
- [ ] Tasks are specific, actionable, and hour-estimable
- [ ] Audit trail, RBAC, and data implications noted where relevant
- [ ] No orphaned doc cross-references

🤖 Generated with [Claude Code](https://claude.ai/code)
```

---

## Glossary

- **SBR:** Service-Based Retail (AgriFlow retains stock ownership on consignment)
- **FIFO:** First-In, First-Out (expiry-based, not receipt-date-based)
- **PoD:** Proof of Delivery
- **RQC:** Receiving & Quality Control
- **SRM:** Supplier Relationship Management
- **Traceability ID:** System-generated tag linking Supplier + Date + QC Grade
