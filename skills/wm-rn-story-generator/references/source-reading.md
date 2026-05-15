# Source Reading (Phase 2)

**Mandatory. Never skip.** A story generated without reading the source will have wrong props, wrong defaults, and a hollow Showcase.

The runtime ships only **compiled JavaScript** — there are no `.d.ts` files for components in `@wavemaker/app-rn-runtime`. Phase 2 reads the compiled `.js` and infers types from default values + naming conventions.

---

## 2.1 Files to Read (in order)

In `componentSourceDir` (e.g. `node_modules/@wavemaker/app-rn-runtime/components/{cat}/{slug}/` for WM primary paths):

### Source format priority

For each file, try extensions in this order — use the first that exists:

| Priority | Extensions to try | When present |
|---|---|---|
| 1 (preferred) | `.tsx`, `.ts` | TypeScript source — read class properties directly; types and defaults are explicit |
| 2 (fallback) | `.js` | Compiled JS — use `_defineProperty` extraction (§ 2.2) |

Same-repo secondary sources commonly have TypeScript source files. Always prefer them — they are richer than compiled JS.

1. `{componentSlug}.props.{tsx|ts|js}` — primary prop list with defaults. Try TypeScript extensions first.
2. `{componentSlug}.component.{tsx|ts|js}` — to inspect conditional rendering branches, required service providers, child sub-components, sample-data shape, public methods, and any extra defaults.
3. (Optional) `{componentSlug}.styles.{tsx|ts|js}` — only if you need to confirm class families for a `className` argType.

If none of `{componentSlug}.props.tsx`, `{componentSlug}.props.ts`, `{componentSlug}.props.js` exist, see [stop-conditions.md](stop-conditions.md).

---

## 2.2 Extract From `{slug}.props.{tsx|ts|js}`

**TypeScript source (`.tsx`/`.ts`):** Read class property declarations directly — types and defaults are explicit, no `_defineProperty` unwrapping needed. Example:

```ts
export default class WmButtonProps extends BaseProps {
  animation: string = null;
  caption: string = null;
  iconposition: 'left' | 'right' = 'left';
  onTap: () => void = null;
}
```

**Compiled JS (`.js` fallback):** The compiled props class extends `BaseProps` and defines each prop via `_defineProperty(this, "<name>", <default>)` inside the constructor. Example (`button.props.js`):

```js
import { BaseProps } from '@wavemaker/app-rn-runtime/core/base.component';
export default class WmButtonProps extends BaseProps {
  constructor(...args) {
    super(...args);
    _defineProperty(this, "animation", null);
    _defineProperty(this, "caption", null);
    _defineProperty(this, "iconposition", 'left');
    _defineProperty(this, "onTap", null);
    _defineProperty(this, "iconsize", 0);
    _defineProperty(this, "accessibilityrole", "button");
    // ...
  }
}
```

For each `_defineProperty(this, "<name>", <default>)`, record: prop name, default value (literal from source), and an inferred type (see § 2.3).

The component-specific `Props` class adds **only** the props listed here; inherited `BaseProps` contributes shared infrastructure (`name`, `styles`, `dataset`, `children`, `listener`) — those are handled globally and **must not** be re-disabled per story (see § 2.5).

---

## 2.3 Type & Control Inference

Compiled JS doesn't preserve TS types. Infer from default value + naming convention:

| Signal | Inferred type | Storybook control |
|---|---|---|
| Default is `true` / `false` | `boolean` | `{ control: "boolean" }` |
| Default is a number | `number` | `{ control: "number" }` |
| Default is a string literal | `string` | `{ control: "text" }` |
| Default is `null` and name suggests text (`caption`, `hint`, `placeholder`, `badgevalue`) | `string` | `{ control: "text" }` |
| Default is `null` and name ends in `class` (`iconclass`, `classname`) | `string` (CSS class) | `{ control: { type: "select" }, options: [...] }` — populate from confirmed class family (`btn-*`, `link-*`) or `glyphMap`; otherwise leave the prop in `args` only |
| Name ends in `position` (`iconposition`) | enum string | `{ control: "radio", options: [...] }` |
| Name ends in `color` or matches `/(background\|color)$/i` | color | global color matcher in `.storybook/preview.tsx` already maps it; explicit `{ control: "color" }` is fine |
| Name ends in `Date` / `date` | date | global date matcher in `.storybook/preview.tsx` already maps it; explicit `{ control: "date" }` is fine |
| Name is `animation` | enum string | `{ control: { type: "select" }, options: animationNames }` (import from `constants/constant.ts`) |
| Name starts with `on` and default is `null` | callback | **omit** from `argTypes`; in `args` use `action("<name>")` |

If a prop's type genuinely cannot be inferred and it would otherwise need a control, surface the ambiguity and ask — never invent enum options.

---

## 2.4 Extract From `{slug}.component.js`

Read the compiled component to capture:

- **Conditional rendering branches** — reveal which props matter visually (e.g. `iconposition` rendering left vs right).
- **Required service contexts** — if the component imports one of these, the story's `decorators` must wrap a matching provider:
  | Imported / used | Wrap with |
  |---|---|
  | `NavigationService`, `NavigationServiceConsumer` | `<NavigationServiceProvider value={navigationService}>` (mock the service locally) |
  | Gesture handler primitives | `<GestureHandlerRootView style={{ flex: 1 }}>` |
  | Modal opens / dialogs | `<ModalServiceComponent>` from `services/ModalService` |
  | `AssetProvider` / image / lottie helpers | `<AssetProvider>` from `services/Assethandler` |
- **Sample-data shape** needed for `Showcase` (e.g. carousel slides, list items). Prefer existing constants in `constants/constant.ts` (`Users`, `salesData`, `carouselImages`, `quarterResults`).
- **Public methods** attached to the component instance (ref / forwardRef handle, class methods on the default export). These populate `methods.md`.
- **Child sub-components** consumed structurally (Sub-pattern A) — recorded in § 2.7 inventory.

---

## 2.5 Globally-disabled props (do not redeclare)

`.storybook/preview.tsx` already disables these in `argTypes` for every story:

```ts
argTypes: {
  styles:   { table: { disable: true } },
  children: { table: { disable: true } },
  dataset:  { table: { disable: true } },
  name:     { table: { disable: true } },
}
```

Do **not** add these `table.disable` rules per story — they are redundant. If a specific story needs `dataset` to be visible (e.g. WmSwitch), **un-disable** it with `dataset: { table: { disable: false } }` and explain why.

---

## 2.6 Prop Inventory — MANDATORY PRE-WRITE CHECKPOINT

After reading the source and **before writing any file**, print a Prop Inventory table in chat. Hard checkpoint — no `.stories.tsx` and no `docs/*.md` until the inventory is emitted.

### Required columns

| Prop | Default (props.js) | Inferred type | Include in args? | Control / Notes |
|---|---|---|---|---|

### Rules

- Every prop must come **directly** from `{slug}.props.js`. No invented rows. No props from memory.
- Cite the literal default exactly as found in source (e.g. `null`, `'left'`, `0`, `"button"`).
- Set `Include in args?` to `yes` or `no` with a short reason:
  - `yes — primary content / interactive variation / boolean toggle`
  - `yes — required by sub-pattern X to render visible content` (Complex floor)
  - `no — callback (use action() in args, no argType)`
  - `no — single sensible value, not interactive`
  - `no — already handled by global preview argTypes`
- Required object/array props (Complex only) get `yes` with a sample-data note.

### Sample row

| Prop | Default | Inferred type | Include in args? | Control / Notes |
|---|---|---|---|---|
| `caption` | `null` | `string` | yes — primary content | `{ control: "text" }`, default `""` |
| `iconposition` | `'left'` | enum string | yes — visual axis | `{ control: "radio", options: ["left","right"] }` |
| `iconclass` | `null` | string (CSS class) | yes — variant axis | `{ control: { type: "select" }, options: [iconOptions...] }` |
| `onTap` | `null` | callback | no — wired with `action("onTap")` in `args` | — |
| `accessibilityrole` | `"button"` | string | no — single sensible value | — |

### Service provider note

If § 2.4 surfaced a required provider, append a "Decorator" line below the table:

```
Decorator: NavigationServiceProvider → wrap <Story /> inside provider with mock service.
```

If the inventory cannot be completed (source unreadable, file missing), **stop** — see [stop-conditions.md](stop-conditions.md).

---

## 2.7 Complex components — Sub-pattern detection

When the Prop Inventory contains any non-primitive required prop, **also print a Sub-pattern Detection sub-table**. This drives which body to copy from `assets/story-templates/complex.md` and is a hard checkpoint — no file may be written until printed.

| Sub-pattern (A–E) | Applies? | Triggering prop(s) — verbatim from `{slug}.props.js` / `{slug}.component.js` |
|---|---|---|

Allowed sub-patterns:

- **A** — Children-as-columns/fields. Trigger: `children` consumed structurally inside `{slug}.component.js` (mapped over to derive columns / fields / panes).
- **B** — RenderItem. Trigger: a prop typed as a render callback (commonly `renderItem`, `renderRow`).
- **C** — Field-mapping dataset. Trigger: any of `datafield`, `displayfield`, `searchkey` declared in props.
- **D** — Hierarchical dataset. Trigger: any of `nodelabel`, `nodeid`, `nodechildren` declared in props.
- **E** — Config-object / multi-array. Trigger: required arrays/objects beyond `dataset` (e.g. `chartColors`, `dataKeys`, `xDataKeyArr`).

A component may match more than one row.

### Sub-pattern A — pick the kind

When **A** applies, also state which kind of child layout the parent uses. Decide by **source signal in the parent**, not component name:

| Kind | Detection signal in `{slug}.component.js` | Body to copy (complex.md) |
|---|---|---|
| **column-kind** | Children iterated to read `child.props.binding` / `child.props.field` | § 4.2.A |
| **pane-kind** | Children read `child.props.title` / `child.props.index`, no `binding` | § 4.2.B |
| **form-kind** | Parent declares `formdata` / `datasource` / `onSubmit` OR renders `WmComposite` directly | § 4.2.C |

If signals don't match cleanly, re-read `{slug}.component.js` before guessing.

### Sub-pattern E — Chart Config Inventory

When **E** applies, also print a Chart Config Inventory listing every required object/array prop (those without a default in source or marked required):

| Config prop | Type | Default (from source) | Baseline value to seed |
|---|---|---|---|

- Use the literal default whenever present.
- When no default exists, derive the baseline from the shape of `sampleData` — e.g. `dataKeys` from `Object.keys(sampleData[0])` minus the x-axis key, `xDataKeyArr` from `sampleData.map(d => d.x)`. Never invent.

---

## 2.8 Child Component Inventory (Sub-pattern A only)

When Sub-pattern A applies, **also** print a Child Component Inventory. Resolve by listing sibling folders under the parent's `componentSourceDir` (e.g. `advanced/carousel/carousel-content/`, `advanced/carousel/carousel-template/`) and identifying which child the parent reads off `React.Children`.

| Child component | Folder path | Runtime import path | Required props (verbatim from child's `props.js`) | sampleData field bound (form-kind only) |
|---|---|---|---|---|

Rules:

- Required-prop list MUST come from the child's own `{slug}.props.js` — never invented.
- If the child folder cannot be located, **stop** — see [stop-conditions.md](stop-conditions.md). Do not invent a `defaultChildren` block from memory.
- Every child instance written into `defaultChildren` needs a unique `name`. The runtime infrastructure props (`listener`, `styles`) are handled globally — do not add a `listener` mock per child.
- For form-kind, every `sampleData` field must map to exactly one input widget. If a field type doesn't match a widget cleanly, redesign `sampleData` first.

---

## 2.9 Field-mapping Alignment (Sub-patterns C and D)

When C or D applies, **also** print a Field-mapping Alignment sub-table:

| Mapping prop | Value to use | Sample-data row key it must match |
|---|---|---|

Rules:

- The matched key MUST exist on **every** `sampleData` row. If no consistent key exists, redesign `sampleData` first.
- For Sub-pattern D, all three mapping props (`nodelabel`, `nodeid`, `nodechildren`) appear here — `nodechildren` aligns with the array-typed key (e.g. `"children"`).

---

## 2.10 Prop-diff audit (update flow only)

When Phase 1 detected an existing story directory, also print a Prop-diff audit for each carry-over variant export:

```
Carry-over: WithIcon
  keep:    caption, iconclass, hyperlink, classname, onTap
  strip:   <props no longer present in {slug}.props.js>
  add:     <new required props from Phase 2 with safe default>
```

The audit is the contract: only props that survive `keep` + `add` end up in the carry-over export's `args`.

---

> See [forbidden-output.md](forbidden-output.md) for output rules that apply to every prop you decide to include.
