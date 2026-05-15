# Forbidden Output

Two rules. Keep them in working memory for every phase. Everything else is covered positively in the per-phase references.

---

## 1. No `args` / `argTypes` on `Docs` or `Showcase`

- `Docs` renders `<ComponentDocumentation />` and nothing else. No `args`, no `argTypes`.
- `Showcase` is a hand-laid `View` gallery. No `args`, no `argTypes`. Shared props live in **module-level constants** (`baseArgs`, `withIconArgs`, …) declared above `meta`.
- Only `Standard` (and any carried-over variant exports from the update flow) may declare `args`. Only `Standard` may declare `argTypes`.

The legacy `{...Basic.args}` spread pattern does not apply — there is no `Basic` export. Rewrite to spread the module-level constant instead.

---

## 2. No invented prop names

Every key inside `Standard.args` (and inside any carry-over variant's `args`) must trace **directly** to a row in the **Phase 2 Prop Inventory** (sourced from `{slug}.props.{tsx|ts|js}`). If a key is not in the Phase 2 Prop Inventory, it does not belong in the story.

Same rule for `argTypes`: a control for a prop that the runtime does not accept will silently do nothing and mislead consumers of the story.

---

> Everything else (RN primitives, `satisfies` form, `action()` callback wiring, single source of truth for globally-disabled props, docs in `docs/*.md`) is captured positively in the relevant phase reference — not restated here.
