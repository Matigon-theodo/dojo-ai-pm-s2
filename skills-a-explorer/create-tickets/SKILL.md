---
name: create-tickets
description: Split a migration plan into independently-grabbable tickets as local Markdown files, using vertical slices
argument-hint: "path/to/features/<feature>/migration-plan.md"
---

## Role

You break a migration plan into **tracer-bullet vertical slices** — independently grabbable tickets, each delivering a thin but complete path through every layer (schema, API, UI, tests). Each slice is demoable on its own. Prefer many thin slices over few thick ones.

## Task

Here is the user input:
<userinput>
$ARGUMENTS
</userinput>

Parse the input: a single path to `migration-plan.md`. Tickets are written next to it as Markdown files under `tickets/`.

Output:
```
docs/features/<feature>/
  spec.md
  analysis.md
  migration-plan.md
  tickets/
    01-<slug>.md
    02-<slug>.md
    …
```

---

## Phase 1: Read the inputs

Read all three sources for the feature: `migration-plan.md`, `spec.md` (linked from the plan) and `analysis.md` (linked from the plan).

---

## Phase 2: Draft vertical slices

Default rule: **1 UC = 1 slice**. Split a UC into multiple slices only if it's manifestly too thick (e.g., independent sub-flows that can ship separately). Never merge UCs into a single slice.

For each slice, prepare the fields required by template-ticket.md. Every Gherkin scenario, every component, and every touchpoint of the plan must end up in **exactly one** slice.

---

## Phase 3: Quiz the user

Present the breakdown to the user as a numbered list. For each slice, show: title, UC covered, touchpoints, components, blocked-by.

Ask the user:
- Does the granularity feel right? (too coarse / too fine)
- Are the dependency relationships correct?
- Should any slices be merged or split further?

Iterate until the user approves the breakdown. Do NOT write any files before approval.

---

## Phase 4: Write the tickets

Number tickets in **dependency order** (blockers first). The number is the local ticket ID used in `Blocked by` (e.g., `Blocked by: 03-introduce-scanlog-entity`).

For each approved slice, write `tickets/<NN>-<slug>.md` following template-ticket.md. The slug is derived from the title (kebab-case, ≤ 5 words).

---

## Phase 5: Verify

Spawn a subagent to independently verify the tickets folder. Pass it the paths of `spec.md`, `migration-plan.md`, and the `tickets/` folder. The subagent must check:

- Every Gherkin scenario in `spec.md` (for in-scope UCs) is covered by exactly one ticket's acceptance criteria — no missing, no duplicates.
- Every component listed in the plan's Components tree is assigned to at least one ticket's **Scope > Components**.
- Every touchpoint in the plan's Touchpoint Mapping is covered by at least one ticket's **Scope > Touchpoints**.
- Every `Blocked by` reference points to a ticket file that exists in the folder.
- Ticket numbering is consistent with dependencies: a ticket never lists a higher-numbered ticket in `Blocked by`.

Read the subagent's report and fix all issues yourself before returning to the user.

---

## Rules

- **Write tickets in English** (audience: devs / AFK agents).
- AC are reformulated from Gherkin scenarios into short English checkboxes — one scenario → one checkbox. Do not paste raw Gherkin.
- Tickets reference the plan/spec/analysis via relative links; they do NOT duplicate technical details from the plan.
- Do NOT add scope beyond what is in the migration plan. Feature parity, not expansion.
- If a UC has no Gherkin scenarios in the spec, stop and tell the user — the spec is incomplete.
