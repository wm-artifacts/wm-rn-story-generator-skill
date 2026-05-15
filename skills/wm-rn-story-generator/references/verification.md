# Verification Checklist (Phase 5)

Run before declaring the run complete. Any unchecked item is a hard failure — fix and re-verify. The full forbidden list lives in [forbidden-output.md](forbidden-output.md).

---

## A. Workflow checkpoints

- [ ] Phase 1 derived-value sanity summary printed in chat.
- [ ] Phase 2 Prop Inventory table printed in chat **before** any file was written.
- [ ] Phase 2 Sub-pattern Detection sub-table printed if the component is Complex.
- [ ] Phase 2 Child Component Inventory printed if Sub-pattern A applies.
- [ ] Phase 2 Field-mapping Alignment printed if Sub-pattern C or D applies.
- [ ] Phase 2 Chart Config Inventory printed if Sub-pattern E applies.
- [ ] Phase 2 Prop-diff audit printed for each carry-over variant (update flow only).
- [ ] `.storybook/main.ts` glob append performed (or confirmed already present) — surfaced in chat.

---

## B. Story file — required structure

- [ ] Path is `{storyDir}/{componentSlug}.stories.tsx` (e.g. `storybook/stories/wm-basic/wm-anchor/anchor.stories.tsx`).
- [ ] Imports include `@storybook/react` types, `react`, `react-native` (`View`, optionally `Text`), and the component via `{componentImportPath}`.
- [ ] Service-provider imports present iff Phase 2.4 required them.
- [ ] Five docs imports present: `overview`, `props`, `events`, `methods`, `styling` from `./docs/*.md?raw`. **No** `token` import.
- [ ] `DocsPage` function declared above `meta` (not exported as a Story); renders `<ComponentDocumentation />` with the five markdown props.
- [ ] `meta.parameters.docs.page` = `DocsPage` and `meta.parameters.docs.canvas.sourceState` = `"none"`.
- [ ] **No** `export const Docs: Story` — a named Docs export collides with the global `autodocs` tag and disappears from the sidebar.
- [ ] `meta` ends with `} satisfies Meta<typeof {ComponentName}>;`.
- [ ] `meta.title` equals `{metaTitle}` from Phase 1.
- [ ] Innermost decorator is `<View style={{ padding: 16 }}><Story /></View>`.
- [ ] `meta.parameters.layout` set (`"centered"` typical, `"fullscreen"` for full-page components).
- [ ] No inline `parameters.docs.description.component` block — docs live in `docs/*.md`.
- [ ] Story export order: `Standard` is first; `Showcase` is **last**; any carry-over variants sit between them. `Standard` MUST be first so Storybook's `<Primary />` block renders a single instance on the Docs Canvas.
- [ ] `Standard` has `args` populated from Phase 2 Prop Inventory, and (when needed) `argTypes` only for non-globally-disabled prop overrides.
- [ ] `Showcase` has no `args`, no `argTypes`; declares `parameters: { layout: "padded" }`; outer gallery `View` has `width: "100%"`; responsive flex wrappers: section rows have `flexWrap: "wrap"` + `rowGap`; instance wrappers use `flexBasis` + `flexGrow: 1` + `maxWidth: "100%"` (no fixed `width`).
- [ ] Every `on*` callback in `args` uses `action("<event-name>")` — never an `argType` control.
- [ ] Every key in any `args` block traces to a `_defineProperty(...)` row in `{slug}.props.js`.
- [ ] Default values match source literals (or `""` when source default is `null` for a string prop; `undefined` otherwise).
- [ ] Required-prop floor met for the detected sub-pattern (Complex only).
- [ ] Carry-over variant exports (if any) appear **between `Standard` and `Showcase`** (before Showcase), body is `args: { ...Standard.args, /* overrides */ }`, no `argTypes`.

---

## C. Docs directory — required structure

- [ ] `{docsDir}/` contains exactly six files: `overview.md`, `props.md`, `events.md`, `methods.md`, `styling.md`, `token.md`.
- [ ] `props.md`, `events.md`, `methods.md`, `token.md` each start with `<!-- AUTO-GENERATED FILE. Do not edit manually. -->`.
- [ ] `overview.md` and `styling.md` do **not** carry the AUTO-GENERATED marker.
- [ ] `props.md` groups (`Basic` / `Accessibility` / `Layout` / `Dataset` / `Behavior` / `Graphics`) appear in that order; empty groups omitted; first non-empty group uses `<details open>`.
- [ ] `events.md` groups follow Basic / Mouse / Touch / Component-specific order; standard event descriptions used verbatim.
- [ ] Tables inside `<details><div>` have **no** blank-line separators: the first table row sits on the line immediately after `<div>`, and `</div>` sits on the line immediately after the last row.
- [ ] Table rows are **flush-left** (start at column 0 — no leading spaces). `<summary>`, `<div>`, and `</div>` use a 2-space indent under `<details>`.

---

## D. Quick grep self-checks

```bash
# 1. Forbidden tokens / legacy patterns — should return NOTHING
grep -nE "data-design-token|parameters\.designTokens|tokenData|mockListener" {storyFile}
grep -nE "import token from \"./docs/token" {storyFile}
grep -nE "\\{\\.\\.\\.Basic\\.args\\}" {storyFile}

# 2. Wrong meta form / wrong primitives — should return NOTHING
grep -nE "Meta<typeof [A-Za-z]+> = \{|<div" {storyFile}

# 3. Showcase is last story export — manual check
#    Run: grep -nE "^export const .+: Story" {storyFile}
#    The last line returned must be the Showcase export.

# 4. Required patterns — should each return at least one match
grep -nE "satisfies Meta<typeof"                                   {storyFile}
grep -nE "from \"{componentImportPath}\""                          {storyFile}   # substitute actual resolved value
grep -nE "^export const (Standard|Showcase): Story"               {storyFile}   # Standard MUST appear first
grep -nE "from \"./docs/(overview|props|events|methods|styling)\\.md\\?raw\"" {storyFile}
grep -nE "parameters: \{ layout: \"padded\""                      {storyFile}   # Showcase layout override

# 5. Blank line after <div> in docs — should return NOTHING
#    A blank line immediately after <div> breaks the Storybook markdown renderer.
#    grep -A1 "  <div>" {docsDir}/props.md — every line after <div> must be a table header, not blank.
grep -A1 "^  <div>$" {docsDir}/props.md | grep "^$" && echo "BLANK LINE AFTER <div> DETECTED — FIX REQUIRED" || echo "props.md OK"
```

---

## E. Smoke test in Storybook

After saving, run `npm run storybook` and confirm:

- The story loads under `{storyCategoryTitle}/{Pascal(componentSlug)}` in the sidebar.
- `Docs` renders the five-tab `ComponentDocumentation` panel; the Canvas inside Docs shows a **single instance** (Standard), not the multi-variant Showcase.
- `Standard` renders with Controls populated; callbacks log to the **Actions** panel.
- `Showcase` renders the hand-laid gallery and **no Controls panel**, and reflows on narrow viewports.
- Carry-over variant stories (if any) render with the expected overrides.

If the story fails to render, consult [anti-patterns.md](anti-patterns.md).
