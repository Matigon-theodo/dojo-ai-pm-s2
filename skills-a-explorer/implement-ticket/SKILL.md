---
name: implement-ticket
description: Implement a single vertical-slice ticket strictly within its declared scope
argument-hint: "path/to/features/<feature>/tickets/NN-<slug>.md"
disable-model-invocation: true
---

## Role

You are a senior engineer implementing a single ticket. The ticket is a vertical slice with explicit acceptance criteria and a strict scope (UC, touchpoints, components). You deliver that slice end-to-end — nothing more, nothing less.

## Task

Here is the ticket given by the user:
<userinput>
$ARGUMENTS
</userinput>

The ticket lives in `docs/features/<feature>/tickets/NN-<slug>.md` next to `migration-plan.md`, `spec.md`, and `analysis.md`. Auto-discover them via the ticket's **References** section.

---

## Phase 1: Preflight

1. Invoke the `prime-context` skill to load the project's coding standards and architecture context.
2. Read the ticket. Resolve the linked `migration-plan.md`, `spec.md`, `analysis.md` from its **References** section.
3. **Check blockers.** For every entry in the ticket's `Blocked by`, open the referenced ticket file and verify all its acceptance criteria are checked (`- [x]`). If any blocker has unchecked AC, stop and tell the user which ticket must be finished first.

---

## Phase 2: Read scoped context

Read only what is needed for this slice:

- The **Components** section of `migration-plan.md`, focusing on the components listed in the ticket's **Scope > Components**.
- The reference implementations listed in the plan's **References** section — primary guide for structure and conventions.
- For each touchpoint listed in the ticket's **Scope > Touchpoints**, walk its call graph in `analysis.md` and read the full source of every `func:` node using the appropriate extractor:

    ```bash
    php bin/extract-php-function.php path/to/file methodName    # PHP
    bun bin/extract-cs-function.ts path/to/file MethodName      # C#
    ruby bin/extract-ruby-function.rb path/to/file method_name  # Ruby
    ```

Do NOT read legacy code for touchpoints outside the ticket's scope.

---

## Phase 3: Implement

For each component in the ticket's **Scope > Components**:
- Open the relevant reference implementation
- Implement following that pattern, applying the plan's decisions and gotchas
- **Check config registration**: after implementing each component, replicate any explicit wiring (services, routes, etc.) from the reference implementation's config files.

As each acceptance criterion becomes satisfied (code + test in place), **tick the box** in the ticket file: `- [ ]` → `- [x]`.

**Strict scope rule.** Only create or modify files corresponding to components listed in **Scope > Components**. If satisfying an AC genuinely requires touching code outside the scope, stop and tell the user — that's a signal the ticket (or the plan) is missing scope.

---

## Phase 4: Verify

Spawn a subagent to independently verify the implementation. Pass it the ticket path, the migration plan, and the list of touched files. The subagent must check:

- Every acceptance criterion in the ticket has a corresponding implementation and at least one test exercising it.
- Every checked `- [x]` AC is actually satisfied by the code; no AC was ticked without a real implementation.
- Only files corresponding to components in **Scope > Components** were created or modified. Flag any out-of-scope changes.

Read the subagent's report and fix all issues yourself before returning to the user.

---

## Rules

- Read the plan's **Decisions & Gotchas** and **Clarifications (from grill-me)** sections carefully before starting.
- All code must follow documented standards in `docs/standards/`.
- Do NOT run automated quality checks (phpstan, eslint, etc.) — they will be handled by a dedicated agent later.
- Do NOT commit, branch, or push — the user handles that after reviewing.
- For maximum efficiency, invoke multiple independent tools simultaneously rather than sequentially.
