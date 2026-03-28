---
name: jira-sync
description: Syncs AgriFlow local story and task markdown files to the FL Jira project. Creates new issues for stories and subtasks for tasks that don't exist yet. Updates existing issues if acceptance criteria or estimates have changed. Sprint-aware — assigns issues to the correct sprint based on the epic-sprint mapping. Run after story-validator passes.
allowed-tools: Read, Glob, Grep, mcp__atlassian__createJiraIssue, mcp__atlassian__editJiraIssue, mcp__atlassian__searchJiraIssuesUsingJql, mcp__atlassian__getJiraIssue, mcp__atlassian__addCommentToJiraIssue, mcp__atlassian__getVisibleJiraProjects, mcp__atlassian__getJiraProjectIssueTypesMetadata, mcp__atlassian__getTransitionsForJiraIssue, mcp__atlassian__transitionJiraIssue, mcp__atlassian__createIssueLink, TodoWrite
disable-model-invocation: true
argument-hint: "[story path, e.g. story.4.1 or sprint-3]"
context: fork
---

# Jira Sync Skill — AgriFlow Rwanda

**Role:** Bidirectional sync between local `docs/stories/` markdown files and the FL Jira project. Runs after `story-validator` passes a full PASS for the target stories.

---

## Jira Configuration

```
Project key:  FL
Cloud ID:     db1c0436-5f1d-4b4a-a439-db5e89d44eed
Board URL:    https://psengayire.atlassian.net/jira/software/projects/FL/list
Issue types:  Epic, Story (or Task), Sub-task
```

---

## Boot Sequence

```
1. Read docs/context/agriflow-agent-context.md   ← epic→sprint mapping (Section 4)
2. Confirm story-validator has passed for target stories
   Check: docs/context/validation-report.md — must show PASS
3. Discover story files to sync (see Input section)
4. Run sync workflow (see §A–D below)
5. Write sync report to docs/context/jira-sync-report.md
```

**Hard rule:** Never sync stories that have not passed story-validator. If no validation report exists or it shows FAIL, stop and report the blocker.

---

## Epic → Sprint → Jira Key Reference

| Epic | Jira Epic Key | Sprint | Sprint Status |
|------|---------------|--------|---------------|
| Epic 1: IAM | FL-4 | Sprint 1 | Fully synced (FL-13 → FL-28). Do not re-create. |
| Epic 2: Supplier Ecosystem | FL-5 | Sprint 2 | Fully synced (FL-30 → FL-47). Do not re-create. |
| Epic 3: Product Catalog | FL-6 | Sprint 2 | Fully synced (FL-29, FL-31–34, FL-42, FL-48–50). Do not re-create. |
| Epic 4: Smart Receiving | FL-7 | Sprint 3 | Epic exists; stories need sync. |
| Epic 5: Inventory Engine | FL-8 | Sprint 4 | Epic exists; stories need sync. |
| Epic 6: B2B Order & Fulfillment | FL-9 | Sprint 5 | Epic exists; stories need sync. |
| Epic 7: Logistics & Delivery | FL-10 | Sprint 5 | Epic exists; stories need sync. |
| Epic 8: Consignment & Retail | FL-11 | Sprint 6 | Epic exists; stories need sync. |
| Epic 9: Loss & Compliance | FL-12 | Sprint 6 | Epic exists; stories need sync. |

**Critical:** Epics 1–3 are fully populated in Jira. Running sync on story.1.*, story.2.*, or story.3.* must check for existing issues before creating — never duplicate.

---

## §A — Discover Stories to Sync

```
Input (explicit):   story.{E}.{S} (single story)
Input (sprint):     story.{E}.* (all stories for an epic)
Input (bulk):       docs/stories/ (all stories — use with caution)
```

For each target folder:
1. Glob `user-story-{E}.{S}.md` — parse story metadata
2. Glob `task-{E}.{S}.*.md` — parse task files

---

## §B — Parse Story Metadata

From `user-story-{E}.{S}.md`, extract:

```
title:    First H1 heading after "# Story {E}.{S}: "
epic_key: From "**Epic:**" line — e.g., FL-7
sprint:   From "**Sprint:**" line — e.g., Sprint 3
priority: From "**Priority:**" line
estimate: From "**Estimate:**" line
story:    The AS A / I WANT TO / SO THAT block
criteria: All Given/When/Then blocks
```

Build the Jira description from the story content — include:
- User story sentence
- Acceptance criteria (formatted as a numbered list)
- Edge cases (formatted as a numbered list)
- Link to local file: `docs/stories/story.{E}.{S}/user-story-{E}.{S}.md`

---

## §C — Check for Existing Issue

Before creating, search Jira to prevent duplicates:

```jql
project = FL
AND issuetype in (Story, Task)
AND summary ~ "Story {E}.{S}"
AND "Epic Link" = {epic_key}
```

Or search by a unique label (see §E — Labelling).

**Decision logic:**
- **Issue not found** → create new (§D.1)
- **Issue found, description unchanged** → skip, log "already in sync"
- **Issue found, acceptance criteria changed** → update description (§D.2) and add comment noting the change

---

## §D — Create or Update Issues

### §D.1 — Create Story Issue

```
createJiraIssue:
  project:     FL
  issuetype:   Story
  summary:     "Story {E}.{S}: [title]"
  description: [formatted story content — story sentence + BDD criteria + edge cases]
  priority:    [High | Medium | Low]
  labels:      ["agriflow-story", "story-{E}-{S}", "sprint-{N}", "epic-{E}"]
  parent:      [epic Jira key, e.g., FL-7]   ← links story to its epic
```

After creating, record the new issue key (e.g., FL-51) for subtask creation.

### §D.2 — Update Existing Story Issue

```
editJiraIssue:
  issueKey:    [existing key]
  description: [updated content]

addCommentToJiraIssue:
  issueKey:    [existing key]
  comment:     "Acceptance criteria updated from local docs/stories/story.{E}.{S}/user-story-{E}.{S}.md on [date]"
```

### §D.3 — Create Subtasks (Tasks)

For each `task-{E}.{S}.{T}.md`:

Parse:
```
title:    First H1: "# Task {E}.{S}.{T}: [verb-led description]"
estimate: From "**Estimate:**" line (Xh)
role:     From "**Role:**" line
description: Full task description
```

Create subtask:
```
createJiraIssue:
  project:     FL
  issuetype:   Sub-task
  parent:      [story issue key from §D.1]
  summary:     "Task {E}.{S}.{T}: [verb-led description]"
  description: [task description + acceptance checkboxes]
  labels:      ["agriflow-task", "task-{E}-{S}-{T}", "role-{backend|frontend|mobile|infra}"]
  estimate:    [Xh — store in description since story points field is not on screen]
```

**Note on estimates:** The FL project does not have Story Points on the screen (customfield_10016 is inactive). Store hour estimates in the description as `**Estimate:** Xh` — do not try to set the story points field.

---

## §E — Labelling Convention

All issues created by this skill must include structured labels for traceability:

| Label | Applied to | Purpose |
|-------|-----------|---------|
| `agriflow-story` | Story issues | Identifies all stories created by this workflow |
| `agriflow-task` | Subtask issues | Identifies all tasks created by this workflow |
| `story-{E}-{S}` | Story issues | e.g., `story-4-2` — enables JQL filtering by story |
| `task-{E}-{S}-{T}` | Subtask issues | e.g., `task-4-2-1` — enables exact task lookup |
| `sprint-{N}` | Story + task issues | e.g., `sprint-3` — sprint membership |
| `epic-{E}` | Story + task issues | e.g., `epic-4` — epic membership |
| `role-{role}` | Subtask issues | e.g., `role-backend`, `role-mobile` |

---

## §F — Sprint Assignment

Jira sprint assignment via the API requires the sprint ID (an integer), not the sprint name. To get the sprint ID:

1. Use `fetchAtlassian` with the board's sprints endpoint to list sprint IDs
2. Map sprint name ("Sprint 3") to its ID
3. Set `customfield_10020` on issue creation/update to the sprint ID

If sprint IDs are not retrievable, set the sprint in the issue description as a fallback: `**Sprint:** Sprint N`. A team member can bulk-assign in the Jira board UI.

---

## §G — Sync Report

Write to `docs/context/jira-sync-report.md`:

```markdown
# Jira Sync Report

**Date:** YYYY-MM-DD
**Scope:** [story.E.S | sprint-N | all]
**Synced by:** jira-sync skill

## Summary

| Story | Story Issue | Tasks Created | Result |
|-------|------------|---------------|--------|
| story.4.1 | FL-51 (created) | FL-52, FL-53, FL-54 | Synced |
| story.4.2 | FL-55 (created) | FL-56, FL-57, FL-58 | Synced |
| story.1.1 | FL-13 (already in sync) | — | Skipped |

## New Issues Created

- FL-51: Story 4.1 — QC Inspection Workflow
- FL-52: Task 4.1.1 — Create Receiving_Events schema
- ...

## Updated Issues

- FL-XX: Updated acceptance criteria (story.4.2 revised)

## Skipped (already in sync)

- FL-13, FL-14, FL-15 (Sprint 1 stories — no changes detected)

## Errors

- [Any failures with Jira API calls — include error message and affected issue]
```

---

## Safety Rules

1. **Never sync stories with FAIL validation status** — check `docs/context/validation-report.md` first
2. **Never create duplicate issues** — always search before creating (§C)
3. **Never modify Sprint 1 issues** (FL-13 → FL-28) or Sprint 2 issues (FL-29 → FL-50) unless acceptance criteria have explicitly changed in the local markdown files
4. **Never set `bank_details` content** in any Jira issue description — this data is masked in audit logs and must not appear in Jira
5. **If createJiraIssue fails**, log the error in the sync report and continue with remaining issues — do not abort the entire sync for a single failure

---

## Trigger Conditions

Run this skill when:
- A sprint's stories have been generated by `product-manager` AND validated by `story-validator`
- A story's acceptance criteria have been updated locally and Jira needs to reflect the change
- Onboarding a new sprint for planning (populate Jira with the sprint's stories/tasks 1–2 days before sprint kickoff)

Do **not** run on:
- Epics 1–3 stories unless local files have changed (already fully synced)
- Stories that have not passed story-validator
- Epic (FL-4 to FL-12) issue updates — epic-level changes are made manually in Jira
