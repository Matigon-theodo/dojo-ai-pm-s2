# [Short action-oriented title in English]

## What to build

[1–3 sentences describing the end-to-end behavior delivered by this slice. No file paths, no code snippets. Link to the plan for technical detail.]

## Acceptance criteria

[One checkbox per relevant Gherkin scenario from spec.md, reformulated as a short English statement. Do not paste raw Gherkin.]

- [ ] [User-visible behavior 1]
- [ ] [User-visible behavior 2]
- [ ] [Error / edge case 1]

## Scope

- **UC**: UC1 — [title from spec.md]
- **Touchpoints**: `GET /module/action`, `POST /module/action/delete`
- **Components**: 🟢 GetFoo, 🟢 FooRepository, 🟡 FooService

## References

- [Migration plan](../migration-plan.md)
- [Spec](../spec.md)
- [Legacy analysis](../analysis.md)

## Blocked by

- `03-introduce-foo-entity` — [why this slice depends on it, 1 line]

[Or: "None — can start immediately"]
