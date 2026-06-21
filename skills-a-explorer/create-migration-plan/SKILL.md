---
name: create-migration-plan
description: Create migration plan from spec + analyses, mapping legacy behavior to new architecture
argument-hint: "path/to/features/<feature>/spec.md [only UC1 and UC3]"
---

## Role

You are a senior engineer writing a concise migration plan for experienced devs. The plan validates key choices, flags gotchas and traps early — it does NOT micro-manage implementation details or spell out obvious patterns.

Your audience knows the codebase. They need to know WHAT's different, WHERE the traps are, and WHY you chose this approach over alternatives. They don't need step-by-step instructions.

## Task

Here is the user input:
<userinput>
$ARGUMENTS
</userinput>

Parse the input:
- **spec path**: the path to `spec.md` (always present)
- **scope filter** (optional): text after the spec path that narrows scope (e.g., "only UC1 and UC2", "only UC3", "sans UC5")

Write the migration plan in the same folder as the spec. Name it `migration-plan.md` (one plan per feature). Follow exactly the template template-migration-plan.md.

---

## Phase 1: Prime context and scope

1. Check that the user has loaded the project's coding standards and architecture context into the current conversation (e.g., via `/prime-context` or equivalent). If not, say it and stop.

2. Read the spec. It contains:
   - **Use cases** (UC1, UC2, …) — each describing a user-facing action
   - **Touchpoints table** — mapping each touchpoint to its function
   - **Link to analysis** (`analysis.md`) — one section per in-scope touchpoint

3. **Determine scope.** If the user provided a scope filter: identify which UCs are in scope, then from the touchpoints table identify which touchpoints serve those UCs. Only plan for those. List excluded UCs in the plan header as "Not in scope: UC4, UC5, …". If no filter, all UCs and all ✅ touchpoints are in scope.

---

## Phase 2: Read all source code

Read `analysis.md`. For every in-scope touchpoint section, walk its call graph and read the full source of **every** `func:` node using the appropriate extractor.

**Paths in the call graph are relative to the `code_path` in `tracer.yml`.** Prefix them with `code_path` when calling extractors:

```bash
php bin/extract-php-function.php <code_path>/path/to/file methodName    # PHP
bun bin/extract-cs-function.ts <code_path>/path/to/file MethodName      # C#
ruby bin/extract-ruby-function.rb <code_path>/path/to/file method_name  # Ruby
```

- Read **every** function in the tree — do not skip any
- Also read ALL view/template files referenced by the controller using the Read tool

Then search the **target codebase** for existing components before considering new ones:
- Before proposing a new component, verify no existing one already does what you need. Read the existing component's code, compare against legacy analysis, only propose new if nothing matches.
- Before writing code that calls an existing service, read **all its public methods** to avoid duplicating logic a helper already provides.
- Before referencing any API endpoint, verify it exists in the codebase. If it doesn't, flag as prerequisite.
- When proposing a new architectural pattern not used elsewhere, plan to flag it explicitly and explain why existing patterns don't apply.

---

## Phase 3: Grill-me with the engineer

Interview them relentlessly about every aspect of this migration until you reach a shared understanding. Walk down each branch of the decision tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time.

If a question can be answered by exploring the codebase, explore the codebase instead.

---

## Phase 4: Write the plan and verify

1. Write `migration-plan.md` following template-migration-plan.md.
2. Spawn a subagent to independently verify the plan. Pass it the paths of `spec.md`, `analysis.md`, and `migration-plan.md`. The subagent must check:

   - Every `func:` node in `analysis.md`'s call graph is referenced in the plan — migrated, explicitly dropped, or handled by an existing ⚪ component.
   - Every logic rule in `analysis.md`'s Logic section is mapped to a component in the plan.
   - Every legacy table/field in `analysis.md` is mapped to a target table or explicitly noted as "no change".
   - Every Gherkin scenario in `spec.md` (for in-scope UCs) has corresponding coverage in the plan.
   - All sections of `template-migration-plan.md` are filled; out-of-scope UCs are listed in the plan header.

   Read the subagent's report and fix all issues yourself before returning to the user.

---

## Rules

- **Write the migration plan in English** (the legacy analysis is in French, but the plan is for devs and must be in English).
- Every element in every in-scope touchpoint section of `analysis.md` must be accounted for. If something is intentionally NOT migrated, state it explicitly.
- Out-of-scope UCs (from user's scope filter) do NOT need to be accounted for — just listed as excluded.
- Do NOT add features, touchpoints, or behaviors not in the analyses. Feature parity, not expansion.
- Flag duplicated logic across files. If the same logic exists in 3+ places, propose factoring it out.
- When multiple touchpoints share entities, services, or patterns, consolidate them in the Components section — don't repeat the same component per touchpoint.
