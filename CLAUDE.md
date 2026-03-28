# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is the **product management planning repository** for **AgriFlow Rwanda** (code name: "Operation Harvest") — a B2B fresh produce distribution platform solving post-harvest food loss in Rwanda through a Service-Based Retail (SBR) model. This repo contains no application code; it holds PRDs, epics, user stories, and engineering task specs that drive implementation.

## Repository Structure

```
README.md              — Product reference: market context, personas, strategic phases
CLAUDE.md              — Operative file: constraints, writing standards, git workflow, agent boot sequence
PRODUCT-CONTEXT.md     — Redirect to README.md (preserved for git history)
docs/
  prd/PRD.md           — Product Requirements Document (tech stack, constraints, MVP scope)
  epics/EPICS.md       — Jira-ready backlog: epics -> user stories -> technical tasks
  epics/EPICS-FULL.md  — Extended epic descriptions with compliance notes and dependency chain
  context/
    agriflow-agent-context.md — Shared context read by all sub-agents at boot
  stories/             — Detailed user stories and their task breakdowns
    story.{E}.{S}/     — Folder per story (E=epic number, S=story number)
      user-story-{E}.{S}.md
      task-{E}.{S}.{T}.md
.claude/
  agents/
    story-pipeline.md  — Orchestrates product-manager → story-validator → jira-sync
  skills/
    product-manager/   — Story + task generation, PRD work, epic breakdown
    story-validator/   — Validates stories against writing standards (context: fork)
    jira-sync/         — Syncs story/task files to Jira FL project (manual-only)
```

## Document Hierarchy

**Epic > User Story > Task** — each level links to its parent and children via relative markdown links. Tasks are the smallest unit, scoped to hours of developer work.

---

## Key Personas

1. **Retailer (Supermarket Manager):** Consistent JIT supply, zero spoilage risk, digital ordering away from WhatsApp.
2. **Farmer/Supplier:** Predictable demand, fair RICA-aligned QC grading, performance-based ratings.
3. **Warehouse QC Clerk & Picker:** Structured tools that enforce rules — mandatory photos for rejections, forced FIFO picking, no silent overrides.
4. **Logistics/Delivery Driver:** Offline-first routing, geofenced drop-off validation, PoD capture without connectivity (8-hour minimum).

> Full market context and strategic phases in [README.md](./README.md).

## Agent Boot Sequence

**Every sub-agent spawned for this repository must execute these steps before writing any artifact:**

```
1. Read docs/context/agriflow-agent-context.md       ← constraints, paths, Jira keys, writing standards
2. Read docs/epics/EPICS-FULL.md (target epic only)  ← scope, out-of-scope, compliance notes
3. Check epic dependency chain (context §5)          ← verify hard-blocker dependencies are resolved
4. Confirm output path: docs/stories/story.{E}.{S}/  ← NEVER write to bmad/ paths
```

Agents that skip step 1 will silently violate hard constraints (no hard deletes, ledger model, QC gating, offline-first, compliance scope). This is the primary cause of non-compliant artifacts.

**Agent and skill files:**
- `.claude/agents/story-pipeline` — full pipeline: generate → validate → sync in one command
- `.claude/skills/product-manager` — story + task generation, PRD work (`effort: high`)
- `.claude/skills/story-validator` — validates stories (`context: fork`, `effort: low`)
- `.claude/skills/jira-sync` — syncs to Jira FL project (`disable-model-invocation: true`)

---

## Key Constraints to Enforce When Generating Documents

These constraints are defined in the PRD and README.md and must be reflected in all generated planning artifacts:

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

When creating or editing planning documents, follow README.md section 6 strictly:

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
- [ ] Follows README.md §6 writing standards
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
