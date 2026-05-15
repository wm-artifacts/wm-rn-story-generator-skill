---
name: wm-rn-story-generator
description: Storybook story generator for WaveMaker React Native components. Produces a CSF3 scaffold under `storybook/stories/wm-{category}/wm-{slug}/{slug}.stories.tsx` with three canonical exports — `Docs`, `Showcase`, `Standard` — plus optional preserved variant exports when updating an existing story. Reads compiled runtime JS, emits a 6-file `docs/` folder, and registers the new path with `.storybook/main.ts` on first run.
---

# WaveMaker React Native Storybook Story Generator

Generate one Storybook story per WaveMaker React Native component. The story is a **scaffold** in CSF3 TypeScript:

- File location: `storybook/stories/wm-{sourceCategory}/wm-{componentSlug}/{componentSlug}.stories.tsx`.
- Sibling `docs/` folder with six markdown files (`overview.md`, `props.md`, `events.md`, `methods.md`, `style.md`, `token.md`).
- Three canonical exports — `Docs` (via `meta.parameters.docs.page`), `Standard`, `Showcase` — with `Standard` being the only controls-driven story. `Standard` is defined **first** so it becomes the Docs Canvas primary.
- `meta` uses `satisfies Meta<typeof {ComponentName}>` (repo convention).
- Component import: **primary** source (`@wavemaker/app-rn-runtime`) — import path auto-derived as `@wavemaker/app-rn-runtime/components/{cat}/{slug}/{slug}.component`. **Secondary** sources (other npm packages, relative paths, direct node_modules paths) — caller supplies the import path explicitly.
- Callbacks (`on*` props) wired with `action("...")` from `storybook/actions`.
- RN primitives (`View`, `Text`) for every layout — never `<div>`.

This file is the **orchestrator**. Detailed rules live in `references/`; copyable templates and docs stubs live in `assets/`. Load each file only when you enter its phase.

---

## Folder layout

```
.
├── SKILL.md                              ← orchestrator (this file)
├── references/                           ← rules and decision guides
│   ├── forbidden-output.md
│   ├── stop-conditions.md
│   ├── inputs-resolution.md
│   ├── source-reading.md
│   ├── story-structure.md
│   ├── showcase-guide.md
│   ├── docs-generation.md
│   ├── verification.md
│   └── anti-patterns.md
└── assets/
    ├── story-templates/
    │   ├── simple.md
    │   └── complex.md
    └── docs-stubs/
        ├── overview.md                   ← literal template (manual stub, no marker)
        ├── props.md                      ← format reference (auto-generated per component)
        ├── events.md                     ← format reference (auto-generated per component)
        ├── methods.md                    ← format reference (auto-generated per component)
        ├── styling.md                    ← literal template (manual stub, no marker)
        └── token.md                      ← literal template (auto-generated stub, has marker)
```

`references/` = "what the model reads to think correctly". `assets/` = "what gets copied or substituted into the project".

---

## Repo wiring (honor these — do not duplicate them in stories)

Four facts about the surrounding repo that every generated story relies on:

1. **`.storybook/preview.tsx`** globally disables `styles`, `children`, `dataset`, `name` in `argTypes`, applies `autodocs` tag, sets `color`/`date` control matchers, and pins `Basic`-first sort. Do **not** re-disable those four props per story. Un-disable only when the component intentionally needs one of them visible (e.g. `dataset: { table: { disable: false } }`).
2. **`.storybook/main.ts`** discovers `../components/**/*.stories.@(js|jsx|ts|tsx)`. The skill appends `../storybook/stories/**/*.stories.@(js|jsx|ts|tsx)` to that array on first run, idempotently — skip the append if the entry already exists.
3. **`.storybook/components/ComponentDocumentation.tsx`** accepts five props: `overview`, `props`, `events`, `methods`, `styling`. The `Docs` story renders this component with the five markdown imports. `token.md` is emitted to `docs/` for parity with the React skill but is not imported by the story until the renderer adds a token tab.
4. **`constants/constant.ts`** exports `animationNames`, `glyphMap`, `Users`, `salesData`, `carouselImages`, `quarterResults`. **`services/`** exports `handleAsset`, `ModalServiceComponent`, `RefreshWrapper`, `SearchService`, `WmTimeService`. `AssetProvider` (the React context component) comes from `@wavemaker/app-rn-runtime/core/asset.provider` — not from `services/`. Decorator usage: `<AssetProvider value={handleAsset}>`. Reuse these — do not invent parallel option arrays.

Legacy stories under `components/Wm{Name}/Wm{Name}.stories.tsx` are **never** modified or deleted by this skill. Two Storybook entries will coexist until the user removes the legacy file.

---

## Reference Map

| Phase / concern | Load |
|---|---|
| Bans applicable everywhere | [references/forbidden-output.md](references/forbidden-output.md) — keep in working memory across all phases |
| Blockers that require stopping | [references/stop-conditions.md](references/stop-conditions.md) |
| Phase 1 — Resolve inputs (and detect update flow) | [references/inputs-resolution.md](references/inputs-resolution.md) |
| Phase 2 — Read component source, build Prop Inventory | [references/source-reading.md](references/source-reading.md) |
| Phase 3 — Generate `.stories.tsx` (meta, decorators, three exports + optional carry-overs) | [references/story-structure.md](references/story-structure.md), [references/showcase-guide.md](references/showcase-guide.md) |
| Phase 3 — Story scaffold (Simple) | [assets/story-templates/simple.md](assets/story-templates/simple.md) |
| Phase 3 — Story scaffold (Complex) | [assets/story-templates/complex.md](assets/story-templates/complex.md) |
| Phase 4 — Generate `docs/*.md` files | [references/docs-generation.md](references/docs-generation.md) (rules) + [assets/docs-stubs/](assets/docs-stubs/) (canonical bodies) |
| Phase 5 — Verification checklist + grep self-checks | [references/verification.md](references/verification.md) |
| Review / debugging | [references/anti-patterns.md](references/anti-patterns.md) |

---

## Workflow at a glance

### Phase 1 — Resolve Inputs

Caller supplies a component source path. **Primary** (WM runtime): `basic/anchor` or `@wavemaker/app-rn-runtime/components/advanced/carousel` — import path auto-derived. **Secondary** (any other resolvable path): other npm packages (e.g. `@my-org/components/input/button`), relative paths (e.g. `../../packages/my-lib/components/basic/button`), or direct node_modules paths — caller must also supply the import path. Optional second input: an existing story directory (`components/WmAnchor`) — triggers the **update flow**, which catalogues existing exports for carry-over and ports docs prose.

Derive `componentSlug`, `componentName` (`Wm{Pascal(slug)}`), `sourceCategory`, `storyCategoryGroup` (`wm-{sourceCategory}`), `storyCategoryTitle`, `componentImportPath`, `componentSourceDir`, `storyDir`, `storyFile`, `docsDir`, relative paths back to `.storybook/`, `constants/`, `services/`.

→ Full rules: [references/inputs-resolution.md](references/inputs-resolution.md)

### Phase 2 — Read Component Source

Mandatory. Read the source files in `componentSourceDir` (e.g. `node_modules/@wavemaker/app-rn-runtime/components/{cat}/{slug}/` for WM primary paths). For same-repo secondary sources, TypeScript source files (`.tsx`/`.ts`) are **preferred** over compiled `.js` — types and defaults are explicit.

1. `{slug}.props.{tsx|ts|js}` — extract props and defaults. TypeScript: read class properties directly. Compiled JS: extract every `_defineProperty(this, "<name>", <default>)` row.
2. `{slug}.component.{tsx|ts|js}` — spot conditional rendering, required service providers, child sub-components, sample-data shape.
3. (Optional) `{slug}.styles.{tsx|ts|js}` — confirm `className` family options when applicable.

> **Hard checkpoint:** print the **Prop Inventory** table in chat *before any file is written*. No `.stories.tsx`, no `docs/*.md` until the inventory has been emitted.

For **Complex** components, also emit (before any file is written):

- **Sub-pattern Detection** table (which of A–E applies, citing trigger props verbatim).
- Sub-pattern A **kind** (column-kind / pane-kind / form-kind) — driven by parent source signals, never component name.
- **Child Component Inventory** (Sub-pattern A) — every sibling sub-component folder under `componentSourceDir`, with required props from each child's `props.js`.
- **Field-mapping Alignment** (Sub-patterns C and D).
- **Chart Config Inventory** (Sub-pattern E).
- **Prop-diff audit** (update flow only) — for each carry-over export, which props to keep / strip / add.

If any inventory cannot be filled from source, **stop** — see [references/stop-conditions.md](references/stop-conditions.md).

→ Full rules: [references/source-reading.md](references/source-reading.md)

### Phase 3 — Generate Story File

#### 3.1 Complexity gate

- **Simple** — all props are primitives AND no service-provider decorator is required. Use [assets/story-templates/simple.md](assets/story-templates/simple.md).
- **Complex** — any service provider needed OR any required non-primitive prop (Sub-pattern A–E). Use [assets/story-templates/complex.md](assets/story-templates/complex.md).

#### 3.2 Assemble the file

Imports → optional service mocks / sample data hoisted at module scope → `DocsPage` function (above `meta`, not exported) → `meta` (always `... satisfies Meta<typeof {ComponentName}>`, including `parameters.docs.page: DocsPage`) → `Standard` export → optional carry-over variant exports (update flow only) → `Showcase` export **last**.

- **`DocsPage`** — plain function (not a Story export) declared above `meta`. Renders `<ComponentDocumentation />` with the five markdown imports. Injected via `meta.parameters.docs.page`. This is the correct pattern for this repo: it works with the global `autodocs` tag in `preview.tsx` and avoids the sidebar collision that causes a named `export const Docs: Story` to disappear.
- **`Standard`** — defined **first** so it becomes the Docs Canvas primary (Storybook's `<Primary />` block renders the first story export). `render: Template`, full `args` + `argTypes` from the Prop Inventory. Required-prop floor enforced per sub-pattern. Callbacks wired with `action(…)` in `args`.
- **`Showcase`** — **Always the last story export.** RN `View` + `Text` gallery driven by Phase 2 visual axes. Uses module-level constants (`baseArgs`, `withIconArgs`, …) for shared props. No `args`, no `argTypes`. Declare `parameters: { layout: "padded" }` so the canvas fills the full viewport width. Layout MUST be responsive — outer `View` has `width: "100%"`, section rows use `flexDirection: "row"` + `flexWrap: "wrap"` with `flexBasis` + `flexGrow` instance wrappers.
- **Carry-over variants** (update flow only) — inserted **between `Standard` and `Showcase`** (before Showcase) in their original order. Each has `args: { ...Standard.args, /* overrides */ }`. No `argTypes`. Callbacks preserved verbatim. **Update flow:** when an `existingStoryDir` is supplied, every extra named Story export from that file is preserved and re-emitted here.

→ Full rules: [references/story-structure.md](references/story-structure.md)
→ Showcase section design: [references/showcase-guide.md](references/showcase-guide.md)

### Phase 4 — Generate Docs Files

Create `{docsDir}` with exactly six files: `overview.md`, `props.md`, `events.md`, `methods.md`, `styling.md`, `token.md`.

- `overview.md`, `styling.md` — literal templates from [assets/docs-stubs/](assets/docs-stubs/). No AUTO-GENERATED marker. Update flow ports prose from any matching legacy `{existingStoryDir}/docs/*.md`.
- `props.md`, `events.md`, `methods.md` — auto-generated from Phase 2. Each starts with `<!-- AUTO-GENERATED FILE. Do not edit manually. -->`.
- `token.md` — literal AUTO-GENERATED stub. Written to disk but not imported by the story.

→ Full rules: [references/docs-generation.md](references/docs-generation.md)

### Phase 5 — Verification

Run the full checklist before declaring done.

- `DocsPage` function declared above `meta` (not a Story export); `meta.parameters.docs.page` = `DocsPage`.
- **No** `export const Docs: Story` — causes sidebar collision with `autodocs` tag.
- `Standard` is the first story export; `Showcase` is the **last** story export; carry-overs (if any) appear between them.
- `argTypes:` appears only inside `Standard` and (when present) the carry-overs.
- `docsDir` contains exactly the six expected files.
- `.storybook/main.ts` lists the new glob.
- Every key in `Standard.args` matches a row in the Prop Inventory.

→ Full checklist: [references/verification.md](references/verification.md)

---

## Performance / loading order

1. Read this `SKILL.md` once at the start. It is the only orchestration file.
2. Keep [references/forbidden-output.md](references/forbidden-output.md) in working memory — short and applies everywhere.
3. Load each phase reference only when you enter that phase. **Do not pre-load all references.**
4. Load only one of `assets/story-templates/simple.md` or `assets/story-templates/complex.md` — pick after Phase 3.1 decides complexity.
5. `references/showcase-guide.md` is loaded only when assembling Showcase. `references/anti-patterns.md` is for review/debugging only.
6. If a stop condition fires, load [references/stop-conditions.md](references/stop-conditions.md) and surface the blocker — never invent.

---

## Forbidden output

See [references/forbidden-output.md](references/forbidden-output.md) for the single source of truth (three short positive rules).

## Stop conditions

See [references/stop-conditions.md](references/stop-conditions.md). When any condition fires: surface the blocker, list what was found, wait for user direction.
