---
name: open-pr
description: Open a PR with plan-linked description and blueprint review comments
argument-hint: "[feature-directory-path]"
allowed-tools: Bash(gh *), Bash(git diff*), Bash(git log*), Bash(git rev-parse*), Bash(git branch*), AskUserQuestion
disable-model-invocation: false
---

Open a PR whose description and review comments are scoped to exactly what was implemented in this branch, linked to the migration plan.

## Arguments

Feature directory path: `$ARGUMENTS`

Example: `docs/features/activity-lifecycle`

## Instructions

### Step 1: Read the plan

1. Read `$ARGUMENTS/server.md` and/or `$ARGUMENTS/webapp.md` (either or both may exist)
2. Parse plan sections at the high level. A "plan section" is a named unit of work, typically:
   - Server: each flow diagram (e.g. "UF1: POST /segments/upsert", "UF2: GET /segments/tree"), "Database Schema Migration", "Component Inventory" subsections, "Testing Strategy"
   - Webapp: each user flow diagram, route config, component hierarchy entries, i18n, catalog index integration
3. For each section, extract:
   - **Section name** (e.g. "UF1: POST /segments/upsert", "SegmentCatalogTree organism", "Route configuration")
   - **Flow diagrams** (code blocks under "Flow Diagram" / "User Flow Diagram" headings)
   - **`⚠️ No blueprint` entries** with component name, closest blueprint ID (if any), and reason

### Step 2: Scan git changes and map to plan sections

1. Run `git diff main --name-only` to list all changed files
2. For each plan section, determine if it was implemented by matching changed files:
   - Convert component names to likely file paths (e.g. `SegmentService` → `segment.service.ts`, `SegmentCatalogTree` → `SegmentCatalogTree.tsx`)
   - Search in `path/to/server/src/` and `path/to/webapp/src/`
   - A section is "implemented" if at least one of its key components has a matching changed file
3. Flag changed files that don't map to any plan section (potential unplanned work)
4. Flag plan sections with zero matching changed files (potential missing work)

### Step 3: Confirm scope with the user

Present the analysis using AskUserQuestion. Show:

1. **Sections that appear implemented in this PR** — list with matched files. Ask the user to confirm.
2. **Sections with no matching changes** — ask: "Were these done in a previous PR (exclude from description), planned for a later PR (exclude), or should they be in this PR (missing)?"
3. **Changed files not matching any plan section** — ask: "Are these supporting changes for the implemented sections, or unrelated?" (e.g. test factories, schema updates, type generation)
4. **Anything that seems missing** — if an implemented section's plan mentions dependencies (e.g. "Run `generate-types`", "Add i18n keys") but no matching files appear in the diff, flag them.

After this step you must know:
- **`included_sections`**: plan sections whose work is in this PR
- **`excluded_sections`**: sections done before or deferred (DO NOT mention in PR body)
- **`blueprint_gaps`**: `⚠️ No blueprint` entries from `included_sections` only

### Step 4: Build the PR body

Use this template:

```
## Summary
<Feature name from the migration plan title>

<1–3 sentence description of what this PR implements, scoped to included_sections only. Do NOT mention excluded sections.>

### Plan Reference
📋 [`server.md`]($ARGUMENTS/server.md) | [`webapp.md`]($ARGUMENTS/webapp.md)

**Implemented in this PR:**
<Bulleted list of included_sections with one-line description each>

### Server Call Graph
<Paste flow diagrams ONLY from included server sections, each in a code block>

### Webapp User Flow
<Paste flow diagrams ONLY from included webapp sections, each in a code block>

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

**Rules:**
- If server.md or webapp.md doesn't exist, omit that section.
- Only include call graphs for `included_sections`. Never paste diagrams for excluded sections.
- If there are no `⚠️ No blueprint` entries in the included sections, write "All components follow existing blueprints. ✅" instead of the review table.
- Keep the summary focused: a reviewer should understand what this PR does without knowing the full plan.

### Step 5: Open the PR

Ask the user to confirm the PR title before creating.

```bash
gh pr create --title "feat: <feature-name>" --body "$(cat <<'EOF'
<body content>
EOF
)"
```

### Step 6: Post inline review comments for blueprint gaps

If no `⚠️ No blueprint` entries exist in `included_sections`, skip this step entirely.

For each `⚠️ No blueprint` entry from `included_sections` matched to a changed file:

#### 6a. Extract flow context

From the migration plan, extract the flow diagram block containing the flagged component. Capture:
- The call chain from HTTP entry point down to the flagged node
- `📘 blueprint-ref` markers on sibling/parent nodes (what IS standard)
- `❌ Error path` entries within/below the flagged component
- `── Transaction ──` boundary markers
- Cross-domain service calls (calls to services outside the component's module)

If the annotation includes `(closest: \`{id}\`)`, read `path/to/blueprint-index.md` to get the blueprint's one-line description. If no closest blueprint is specified, the component is genuinely novel.

#### 6b. Identify the single highest-risk concern

From the flow context, pick the **one** most important risk. Categories to consider: transaction safety, error handling, permissions, cross-domain coupling, state management, data integrity. Only flag something with a concrete finding.

#### 6c. Build review checklist

Write 1–3 yes/no verification questions referencing specific methods from the flow diagram.

#### 6d. Find the target line in the diff

For each flagged file, locate the component's definition line to anchor the comment:

1. Run `git diff main -- {file_path}` and find the line where the flagged class/function/component is defined (e.g., `export class ActivityService`, `export function ActivityStatusBadge`, `async update(`)
2. Extract the **absolute line number in the new file** from the diff hunk headers (`@@ -a,b +c,d @@` → count from `+c`) for that definition line
3. If the definition isn't in the diff (file existed before), use `subject_type: "file"` as fallback

#### 6e. Compose and post

Build a **short** comment per file (~150 words max):

**When closest blueprint exists:**

```markdown
⚠️ **No blueprint — {reason}** (closest: `{id}`)
{One sentence: what's different from the blueprint and why it matters}
- [ ] {checklist item}
- [ ] {checklist item}
```

**When no closest blueprint:**

```markdown
⚠️ **No blueprint — {reason}**
{One sentence: the key risk to verify}
- [ ] {checklist item}
- [ ] {checklist item}
```

Post all comments as a single PR review using the reviews endpoint with `--input` JSON:

```bash
OWNER_REPO=$(gh repo view --json nameWithOwner -q '.nameWithOwner')
COMMIT_SHA=$(git rev-parse HEAD)

cat <<'REVIEWJSON' > /tmp/review-payload.json
{
  "commit_id": "{COMMIT_SHA}",
  "body": "Blueprint coverage report — files below have no matching blueprint and require manual review.",
  "event": "COMMENT",
  "comments": [
    {
      "path": "{relative_file_path}",
      "line": {line_number},
      "side": "RIGHT",
      "body": "{comment_body_with_escaped_newlines}"
    }
  ]
}
REVIEWJSON

gh api "repos/${OWNER_REPO}/pulls/{pr_number}/reviews" \
  --method POST \
  --input /tmp/review-payload.json
```

Build the JSON payload programmatically — one entry per flagged file in the `comments` array. Escape newlines and quotes in the body for valid JSON.

**Line targeting rules:**
- `line`: absolute line number in the new file where the component is defined — the comment appears on that line in the PR diff
- `side`: always `"RIGHT"` (commenting on new code)
- For multi-line components (e.g., a method spanning lines 42–58), use `start_line` + `line` to highlight the range
- If the definition line is NOT part of the diff (unchanged code), fall back to a file-level comment: omit `line`/`side` and add `"subject_type": "file"` instead

### Step 7: Report

Output the PR URL and a summary:
- Plan sections included in this PR
- Number of call graph diagrams included
- Number of files flagged for manual review
- List of flagged files with reasons
