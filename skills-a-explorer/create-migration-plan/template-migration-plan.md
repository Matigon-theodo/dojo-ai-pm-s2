[INSTRUCTIONS: Density over verbosity. Sacrifice grammar for concision.]
[No: call graph ASCII art, full translation tables, response JSON examples, error code tables, DB field inventories, trivial queries, performance section (flag risks inline), framework-specific UI prop types.]

# Migration Plan: [Feature Name]

**Source**: [spec.md](spec.md)
**Scope**: [UC1, UC2, UC3 — or "all" if no filter]
[If scoped: **Not in scope**: UC4, UC5 (planned for later)]
**Analysis**: [analysis.md](analysis.md)

[1-2 sentences: what this feature does, key migration challenge if any.]

## Touchpoint Mapping

[Table mapping legacy touchpoints to new ones. One row per in-scope touchpoint.]

| Legacy | New | Notes |
|--------|-----|-------|
| `GET /module/action` | `GET /api/resources` | [Only non-obvious notes] |
| `POST /module/action/delete` | `DELETE /api/resources/{id}` | |

## Changes from Legacy

- [List only non-obvious changes. Don't list standard migration patterns.]
- [Include: renamed/dropped fields, new validation, changed semantics, pagination differences]
- [Permissions: only mention if non-standard (authenticated-only is the default, don't repeat it)]

## Components

[Present as a tree. Legend:
- 🟢 New (needs to be created)
- 🟡 Existing, needs modification
- ⚪ Existing, reuse as-is

Shared components appear once; note which touchpoints use them in parentheses.]

```
path/to/target/
├── models/
│   ├── 🟢 Scanlog                         — new entity for scan records
│   └── ⚪ Order                            — reuse as-is
├── repositories/
│   └── 🟢 ScanlogRepository               — custom queries for orphan detection
├── handlers/
│   ├── 🟢 GetScanlogs                     — serves GET /scanlogs (UC1, UC8)
│   └── 🟢 PurgeScanlogs                   — handles POST purge (UC5)
├── controllers/
│   └── 🟢 ScanImageController             — binary image response (UC6, UC7)
└── services/
    └── 🟡 ScanFileService                 — add rename + detach methods
```

[After the tree, add notes ONLY for components that need non-obvious guidance — gotchas, injections, patterns to follow. One line per component. Skip components where the tree annotation is self-explanatory.]

- `GetScanlogs`: inject `ScanlogRepository`, follow pattern in existing list handler
- `ScanImageController`: return binary response, not streamed — file fits in memory

## Configuration

[List any new config keys required and which config files need updating. Skip this section if no config changes.]

| Key | Files | Description |
|-----|-------|-------------|
| `FEATURE_SOME_KEY` | `path/to/config` | Description |

## Clarifications (from grill-me)

[Any decision resolved with the user during grill-me that doesn't fit naturally in another section.

Skip this section if everything is already covered by other sections.]

- **[Topic]**: [Resolved decision]
- **[Topic]**: [...]

## Decisions & Gotchas

[Most important section. One bullet per decision/gotcha. Focus on:]
- [Choices that could reasonably go another way — explain why this way]
- [Constraints: when stating "can't use X", always include the technical reason WHY]
- [Traps: field name mismatches, encoding quirks, legacy write-time transforms]
- [Differences from the reference implementation the dev will follow]
- [Missing dependencies (constructor injections, new imports)]
- [Anything that would cause a 30min debugging session if missed]
- [Performance: N+1 risks, missing indexes, lazy-loaded relations — only if actually risky]

## Tests

[File name + pattern to follow. Then compact list of test names grouped by touchpoint. Only describe setup if non-standard.]

## References

[Similar implementations to follow. One line each.]

<!-- The plan must read as a decided document — no trace of the Q&A process. No "Open Questions" section — all questions must be resolved with the user before writing this plan. -->
