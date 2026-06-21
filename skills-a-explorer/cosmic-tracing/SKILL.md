---
name: cosmic-tracing
description: Reference guide for COSMIC call-graph tracing and annotation
disable-model-invocation: false
---

# COSMIC Tracing Reference

This document describes how to trace a legacy call graph from any touchpoint, annotate each function with COSMIC data movements, and record everything in the graph store. It is loaded automatically into `cosmic-tracer` agents. Language-specific supplements (e.g. `cosmic-tracing-php`) provide source-reading and dependency-resolution details.

## Graph Store Commands

Run from the project directory (where `tracer.yml` is):

```bash
bin/tracer claim <node_id>
# → "ok" (proceed) or "taken" (skip)

bin/tracer done <node_id> --lines START:END --cosmic E:n,R:n,W:n,X:n dep1 dep2 ...

# For touchpoint/subprocess nodes — no claim, just done
bin/tracer done <node_id> dep1 dep2 ...
```

Call `done` ONLY after you have:
1. Read the **entire** function source
2. Counted COSMIC from actual source lines (not estimated)
3. Resolved all callee node IDs (file path + qualified name)

## Recipes

### Recipe: process a single function node

```
process(node_id):
  1. claim node_id → if "taken", skip immediately.
  2. Read the full source (use the appropriate language-specific source extractor).
  3. Count COSMIC data movements from actual source lines.
  4. List ALL internal function/method calls (be exhaustive — up to 100 callees).
  5. Resolve each callee to its node ID:
     - Find the file (see language-specific supplement for conventions)
     - Construct: func:{relative_path}:{ClassName.method} (ALWAYS include the class/receiver type — never use bare method names)
  6. done node_id --lines START:END --cosmic E:n,R:n,W:n,X:n dep1 dep2 ...
     (done automatically creates pending nodes for deps not yet in the graph)
```

**Exhaustiveness is critical.** The rule is: if it's defined in the codebase, it's a dependency — even trivial helpers like param whitelisting methods. Don't filter by "interesting business logic." List every non-builtin call in the function body. If there are more than 100, pick the top 100 by business relevance. If fewer, list them ALL even if it takes a long time. Missing a dependency is worse than being slow.

### Recipe: trace a touchpoint

This recipe applies to all touchpoint types: `page:`, `endpoint:`, `command:`, `listener:`, `database:`.

```
1. Locate the entry point code:
   - page:/path or endpoint:METHOD:/path → find the controller action or script file
   - command:{invocation} → find the CLI script or command class
   - listener:{type}:{name} → find the consumer/handler class
   - database:{type}:{name} → find the stored procedure/function/view in SQL files
2. Read the entry point code. Identify subprocesses (see "Subprocesses").
   Determine all node IDs (controller variants, subprocesses) — these are just strings.
   Before constructing a func: node ID, run the extractor on the target function to see
   its actual declaration. Use the function name exactly as declared (e.g. `cry_ENCRYPT`
   not `cry_encrypt`). The `--lines` validator will reject a casing mismatch.
   **Exception:** Language-specific supplements may override casing rules (e.g. PL/SQL uppercases all identifiers).
3a. Single-process (no branching):
   a. done touchpoint node with the function node as its only dep.
   b. process(entry_point_node)           ← e.g. func:file:Class.method
3b. Multi-process (mutually exclusive branches):
   a. done touchpoint node with all subprocess nodes as deps.
   b. For each subprocess:
      i.  done subprocess node with its variant as dep.
      ii. process(entry_point_variant_node)    ← e.g. func:file:Class.method:nettoyage
4. Continue with DFS (see below).
```

**Rule: never `claim` a dep before its parent's `done`.** The `done` call creates the dep node (with an edge from parent). The subsequent `claim` just locks it for processing. This ensures func nodes are never orphans in the graph. Touchpoint/subprocess nodes may be temporary orphans — that's acceptable.

### Recipe: DFS — continue into the dependency tree

After processing any node, **DFS into its discovered deps** instead of stopping. `claim` prevents duplicates; `done` is the checkpoint — if you stop mid-DFS, completed nodes are safe.

```
stack = deps from the node you just processed
while stack is not empty:
  node_id = stack.pop()
  claim node_id → if "taken", skip
  process(node_id)
  push newly discovered deps onto stack
```

Keep going until the subtree is fully traced.

## Node IDs

Format: `{type}:{identity}`. File paths are **relative to `code_path`** (set in `tracer.yml`). Node IDs must not contain spaces — replace spaces with `_`.

| Type | Format | Example |
|------|--------|---------|
| Page | `page:{path}` | `page:/dashboard.php` |
| Endpoint | `endpoint:{METHOD}:{path}` | `endpoint:GET:/logistic/delete` |
| Command | `command:{invocation}` | `command:php_bin/console_app:cleanup` |
| Listener | `listener:{type}:{name}` | `listener:queue:order-events` |
| Database | `database:{type}:{name}` | `database:proc:sp_get_orders` |
| Subprocess | `subprocess:{METHOD}:{path}:{name}` | `subprocess:POST:/order/scancontrol:nettoyage` |
| Function | `func:{file}:{qualified_name}` | `func:www/lib/db.php:db_query` |
| Entry point variant | `func:{file}:{qualified_name}:{subprocess_name}` | `func:path/BackController.php:Order_backController.scancontrolAction:nettoyage` |

- Single-process touchpoint = no variant suffix. Touchpoint points directly to the base function node (e.g., `func:file:ClassName.method`).
- Multi-process variant suffix = `:{subprocess_name}` (e.g. `:nettoyage`). Each variant belongs to a subprocess node.
- Class/receiver method: `func:path/File.php:ClassName.methodName` (bare class name, no namespace). Always include the class/receiver type — multiple classes in the same file can define methods with the same name, so bare method names are ambiguous.
- Overloaded functions (same name, different params): see your language-specific supplement for the disambiguation convention. A bare name must always be unambiguous.
- File top-level scope: `func:www/dashboard.php` (no function name)
- Use exact class name spelling/casing from source.

## Subprocesses

When a single touchpoint dispatches **mutually exclusive behaviors** via parameters (e.g., `?nettoyage` vs `?detach` vs default), each is a separate COSMIC functional process. Variations on the same behavior (pagination, sorting) are not.

Each subprocess becomes a **first-class node** in the graph:
- **Subprocess node** (`subprocess:METHOD:/path:name`) — groups the call tree for one business operation
- **Entry point variant node** (`func:file:Class.method:subprocess_name`) — the entry point split per subprocess, with its own COSMIC and deps

For **single-process touchpoints** (no branching): skip the subprocess layer entirely. The touchpoint node points directly to the base function node with no suffix. No subprocess node is created.

Shared callees deeper in the tree keep their standard `func:file:Class.method` node ID — they are NOT split. A shared function has exactly one COSMIC annotation regardless of which subprocess calls it.

## COSMIC Counting

COSMIC (ISO/IEC 19761) counts **data movements** across the boundary of a functional process:
- **Entry (E)** — data entering from a user or external system
- **Exit (X)** — data leaving to a user or external system
- **Read (R)** — data read from persistent storage
- **Write (W)** — data written to persistent storage

Each data movement = **1 CFP**.

### Attribution

Attribute each data movement to the function that **directly performs** it, not to parent callers:

- **E:** The function that first reads from `$_GET`, `$_POST`, `$request->input()`, CLI arguments (`$argv`), queue message payload, etc.
- **X:** The function that decides to send data out (render, redirect, email, background job, API call, CLI output).
- **R:** The function that constructs or triggers the read (builds SQL, calls ORM finder, accesses lazy-loaded property).
- **W:** The function that constructs or triggers the write.

Generic wrappers (`redirect()`, `db_query()`, `header.php`/`footer.php`) don't own the data movement — their caller does. X for the rendered HTTP response belongs on the controller/entry point.

### Internal branches — worst-case union

Each function node has **one** COSMIC annotation. Sum all distinct data movements across **all** internal branches (`if/else`, `switch`, loops). Same table read in two branches = `R:1`; different tables = `R:2`. Only touchpoint-level subprocess dispatch creates separate nodes — internal branches do not.

### What to count

**Entry (E)** — 1 per distinct data group:
- All scalar request parameters = **1 E** regardless of how many fields
- File upload = 1 separate E
- User identity from auth = 1 E (only if explicitly used for business logic)
- CLI arguments = 1 E
- Queue message payload = 1 E

**Read (R)** — 1 per distinct data group from persistent storage:
- Single-table query = 1 R. JOINs = 1 R per table that contributes data (filter-only JOINs don't count).
- Each distinct file type read from filesystem = 1 R
- Multiple queries on same entity for different purposes = count each separately
- Loops reading N rows of same entity = 1 R (not N)

**Write (W)** — 1 per distinct data group to persistent storage:
- Each distinct table written (insert/update/delete) = 1 W
- Each distinct file type written = 1 W. Log writes = 1 W total.
- Loops writing N rows of same entity = 1 W (not N)

**Exit (X)** — 1 per distinct data group leaving:
- Rendered page / JSON = 1 X. Redirect = 1 X. Email = 1 X. Background task via `system()`/`exec()` = 1 X.
- CLI output (stdout) = 1 X
- Queue message published = 1 X

### What NOT to count

- Session reads/writes (`$_SESSION`, `get_flash()`, `set_flash()`) — transient, not persistent
- Cache operations
- Pure in-memory transformations

### Edge cases

- **Lazy loading:** `$order->customer` triggers a hidden SELECT → R on the accessing function.
- **Eager loading:** R on the function issuing `::with('customer')`, not on later accessors.
- **Template queries:** Lazy-loaded property in template = R on the template node.
- **External API calls:** Sending = X, receiving response = E, both on the calling function.
- **Queue/job dispatch:** Dispatching = X. The job itself is a separate functional process — don't trace into it.
- **Upserts:** W:1 per table regardless of which path executes.
- **DB triggers:** INSERT on A triggers write to B → W:1 for A + W:1 for B, both on the inserting function.
- **Stored procedures in repo:** Trace into them, create a node. Outside repo: attribute R/W to the caller.
- **Views (DB):** SELECT on a view = R per underlying table, same as a JOIN.

**IMPORTANT:** Count actual data movements by reading the source code line-by-line. Never estimate from patterns. Pattern-based estimation systematically under-counts.

## Dependencies

### What IS a dependency (type: `func:`)

- Direct function/method calls in the source
- Callbacks passed by name
- Calls inside ALL branches of the active verb: if/else, switch, try/catch, loops
- Included/required files with top-level code
- Factory-resolved polymorphism: list **all** concrete implementations as deps of the factory method that resolves them

### What is NOT a dependency

- Language builtins and standard library functions
- Framework ORM/DB abstraction methods (the caller owns the R/W, not the framework)
- Inherited constructors with no custom implementation in the source file
- Static constants and class constants

See your language-specific supplement for the full exclusion list.

### Table accesses

Extract from SQL in string literals. Tables are reflected in the COSMIC annotation of the calling function, NOT listed as tree nodes.

## General Source-Reading Rules

- **IMPORTANT:** All commands must run from the project directory (where `tracer.yml` is).
- **Path resolution:** Node IDs use paths relative to `code_path` (e.g. `src/foo.ts`), but file system tools and extractors need paths relative to the project root. Read `code_path` from `tracer.yml` at the start and prepend it to all file paths: e.g. if `code_path: legacy`, use `legacy/src/foo.ts` for the extractor and Read/Glob/Grep.
- **Never use Read to extract functions.** Use the language-specific extractor. No extractor for the language? **Stop and report it.**
- Only trace the code path for the requested verb/method (for HTTP touchpoints).
- A shared function (non-variant) should have the **same COSMIC annotation** everywhere. If `claim` returned "taken", skip it.
- **Never give up due to size.** Important business logic is often in the latter half. "Too complex" means read MORE carefully, not give up.
- **One function at a time.** Fully finish (claim → read → COSMIC → resolve deps → done) before starting the next. Never leave a claimed node without calling `done`.
