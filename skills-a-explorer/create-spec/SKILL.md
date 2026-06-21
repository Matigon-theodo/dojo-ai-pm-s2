---
name: create-spec
description: Analyze a legacy touchpoint and produce a functional spec + a technical analysis
argument-hint: "<touchpoint node ID> (e.g. page:/logistic/parcel, command:rake_orders:cleanup)"
---

## Role

You are an expert software analyst. You produce two documents:
- A **functional spec** (spec.md) for PO validation — no technical details
- A **technical analysis** (analysis.md) for developers — one section per in-scope touchpoint

## Task

The user provides a touchpoint from `analysis/touchpoints.txt`:
<userinput>
$ARGUMENTS
</userinput>

Infer a **feature name** from the touchpoint (e.g., `page:/annexe/integration` → `annexe-integration`, `command:rake_orders:cleanup` → `orders-cleanup`).

### Migration slice definition

A migration slice is the smallest set of touchpoints that can be migrated and tested independently by the end user:

- **page:** → the page + every `endpoint:` triggered from its UI (AJAX loads, form POSTs, file downloads, modal content fetches). The page is untestable if any of these return 404.
- **endpoint: / command: / listener: / database:** → group of 1.

A POST that redirects to another page does NOT make the redirect target part of this slice — the client can verify the redirect lands on the old (legacy) page until that page gets its own migration.

### Output

Output under `docs/features/<feature>/`:

```
docs/features/<feature>/
  spec.md           ← functional spec, using template-spec.md
  analysis.md       ← technical analysis, using template-analysis.md
  browser/          ← screenshots and snapshots (PAGE only)
```

---

## Phase 1: Discover the migration slice

Identify the touchpoint in the graph:
```bash
bin/tracer touchpoints | grep <slug>
```

- **page:** → follow the discovery guide in @.claude/skills/create-spec/page-discovery.md
- **endpoint: / command: / listener: / database:** → the migration slice is just this single touchpoint. Read its source code and proceed to Phase 2.

---

## Phase 2: Analyze in-scope touchpoints

Start `analysis.md` with the top-level header from the template (title + spec link). Then, for each **in-scope** touchpoint (starting with the primary touchpoint), append a `## Touchpoint: <node-id>` section.

### Step 1: Get the call graph

Dump the full dependency tree directly into `analysis.md` using shell redirect, under the touchpoint's `### Call graph` section. You MUST run this command and do NOT use the Write tool to put the tree in the file.

```bash
bin/tracer tree '<node-id>' >> docs/features/<feature>/analysis.md
```

### Step 2: Read ALL source code yourself — NO subagent

Walk the tree top-down. For **every** `func:` node, read the full function source using the appropriate extractor script:

```bash
php bin/extract-php-function.php path/to/file methodName  # PHP
bun bin/extract-cs-function.ts path/to/file MethodName      # C#
ruby bin/extract-ruby-function.rb path/to/file method_name  # Ruby
```

- Read **every** function in the tree — do not skip any
- Also read ALL view/template files referenced by the controller using the Read tool
- Cross-check with the snapshot (PAGE): every UI element must be traceable to a template

### Step 3: Fill in the touchpoint section

The call graph is already in `analysis.md` from Step 1. Use the Edit tool to insert the other sub-sections (Résumé métier, Comportement frontend, Logique, etc.) around the call graph, following template-analysis.md.

Repeat Steps 1–3 for each in-scope touchpoint, appending sections to the same `analysis.md`.

---

## Phase 3: Grill-me with the Product Owner

The user is the **Product Owner** — non-technical. Only ask about functional behavior, never code (no classes, SQL, file paths, status codes, frameworks).

Interview them relentlessly about every aspect of this feature until you reach a shared understanding. Walk down each branch of the decision tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time.

If a question can be answered by exploring the codebase, explore the codebase instead.

Track every resolved decision — all must land in `spec.md` in Phase 4.

---

## Phase 4: Write the spec and verify

1. Write `spec.md` per template-spec.md.
2. Launch the `spec-verifier` agent on the feature folder and fix all issues it reports.

---

## Rules

- **Strip all template instructions from output.** The templates contain bracketed instructions like `[2-3 phrases expliquant...]` — these are guidance for YOU, not content. The final files must contain zero bracketed instruction placeholders.
- Do NOT connect to databases, run SQL queries, or access live systems (except agent-browser for PAGE UI capture).
- For all research: ONLY search and read from the legacy codebase. Do NOT read from target code or previously generated specs.
- All commands must run from the project root (where `tracer.yml` is).
- For maximum efficiency, invoke multiple independent tools simultaneously rather than sequentially.
