# Story Structure (Phase 3)

Goal: write `{storyDir}/{componentSlug}.stories.tsx` with **three canonical exports** — `Docs`, `Showcase`, `Standard` — and any carry-over variant exports from the update flow.

> Every rule in [forbidden-output.md](forbidden-output.md) applies here. Read it first if you haven't this session.

---

## 3.1 Complexity Gate

| Condition | Type | Template |
|---|---|---|
| All props are primitives (string, boolean, number) AND no service-provider decorator is needed | **Simple** | [../assets/story-templates/simple.md](../assets/story-templates/simple.md) |
| Component requires a service-provider decorator (Navigation / Gesture / Modal / Asset) **OR** has required object/array props (Sub-pattern A–E) | **Complex** | [../assets/story-templates/complex.md](../assets/story-templates/complex.md) |

Pick the matching template, then lazy-load it. Do not load both.

> Callbacks (`on*`) alone do **not** make a component Complex — they are always wired with `action()` in `args`, never as controls.

---

## 3.2 Canonical Import Order

```tsx
import type { Meta, StoryObj } from "@storybook/react";
import React from "react";
import { View, Text } from "react-native";
import { action } from "storybook/actions";

import {ComponentName} from "{componentImportPath}";

// Service providers — only when Phase 2.4 flagged them:
// import { NavigationServiceProvider } from "@wavemaker/app-rn-runtime/core/navigation.service";
// import { GestureHandlerRootView } from "react-native-gesture-handler";
// import { ModalServiceComponent } from "{relativePathToServices}ModalService";
// import { AssetProvider } from "@wavemaker/app-rn-runtime/core/asset.provider";
// import { handleAsset } from "{relativePathToServices}Assethandler";

// Sub-pattern A only — child sub-components from Phase 2.8 inventory:
// Primary WM: import {ChildComponent} from "@wavemaker/app-rn-runtime/components/{cat}/{slug}/{child-folder}/{child-slug}.component";
// Secondary: derive the import path from componentSourceDir's package root + the child folder path found in Phase 2.8 inventory.

// Shared option arrays — import only those actually used:
// import { animationNames, glyphMap, Users, salesData, carouselImages, quarterResults } from "{relativePathToConstants}";

import { ComponentDocumentation } from "{relativePathToComponentDocumentation}";
import overview from "./docs/overview.md?raw";
import props from "./docs/props.md?raw";
import events from "./docs/events.md?raw";
import methods from "./docs/methods.md?raw";
import styling from "./docs/styling.md?raw";
```

Rules:

- `Text` is included only when the file uses it (typically inside Showcase or pane-kind defaultChildren).
- `token.md` is emitted to disk but **not imported** here — `ComponentDocumentation` does not currently accept a `token` prop.
- The filename is `styling.md` and the import variable is `styling`. Both match the renderer's prop name.
- Add imports only when used — no unused imports.

---

## 3.3 Meta Block — `satisfies` form

Declare a `DocsPage` function **above `meta`** that renders `<ComponentDocumentation />`. Then inject it into `meta.parameters.docs.page`. This is the repo-correct pattern — it works with the global `autodocs` tag in `preview.tsx` and matches the legacy stories in `components/Wm*/`.

```tsx
// Declared above meta — not exported as a Story:
const DocsPage = () => (
  <ComponentDocumentation
    overview={overview}
    props={props}
    events={events}
    methods={methods}
    styling={styling}
  />
);

const meta = {
  title: "{metaTitle}",
  component: {ComponentName},
  decorators: [
    (Story) => (
      <View style={{ padding: 16 }}>
        <Story />
      </View>
    ),
  ],
  parameters: {
    layout: "centered",  // "fullscreen" for full-page components (WmCarousel etc.)
    docs: {
      page: DocsPage,
      canvas: { sourceState: "none" },
    },
  },
} satisfies Meta<typeof {ComponentName}>;

export default meta;

type Story = StoryObj<typeof meta>;
```

**Hard rules:**

- The `meta` declaration MUST end with `} satisfies Meta<typeof {ComponentName}>;` — repo convention. Do **not** use `const meta: Meta<typeof X> = { ... };`.
- `meta.title` MUST equal `{metaTitle}` from Phase 1 (`{storyCategoryTitle}/{Pascal(componentSlug)}`).
- `DocsPage` is a plain function, **not** a `Story` export. Do not emit `export const Docs: Story`.
- `parameters.docs.page` replaces the auto-generated autodocs page content — this is how `ComponentDocumentation` becomes visible in the sidebar's Docs entry.
- Do **not** add `parameters.docs.description.component` — docs live in `docs/*.md` surfaced via `DocsPage`.

---

## 3.4 Decorators — when to upgrade

| Component signal (from Phase 2.4) | Decorator stack |
|---|---|
| Default | `<View style={{ padding: 16 }}><Story /></View>` |
| Component uses `NavigationService` | Wrap with `<NavigationServiceProvider value={navigationService}>` — declare a small `navigationService` mock above `meta` |
| Component uses gestures (e.g. `advanced/carousel`) | Wrap with `<GestureHandlerRootView style={{ flex: 1 }}>` |
| Component opens dialogs/modals (e.g. `data/form`) | Wrap with `<ModalServiceComponent>` |
| Component renders images/lottie via app assets | Wrap with `<AssetProvider value={handleAsset}>` (import `AssetProvider` from `@wavemaker/app-rn-runtime/core/asset.provider`; import `handleAsset` from `services/Assethandler`) |

Always keep `<View style={{ padding: 16 }}><Story /></View>` as the **innermost** wrapper.

### Navigation service mock (example)

```tsx
const navigationService = {
  openUrl: (url: string, options?: { target?: string }) => {
    action("openUrl")(url, options);
  },
};
```

Declare above `meta` only when the decorator needs it.

---

## 3.5 Module-level shared values

Hoist these above `meta` when used by ≥2 stories:

- **`sampleData`** — shared sample dataset (lists, tables, charts). Prefer constants from `constants/constant.ts` (`Users`, `salesData`, `carouselImages`, `quarterResults`) — import them rather than redeclare.
- **`defaultChildren`** (Sub-pattern A only) — the JSX fragment passed as children to the parent. Body comes from the Phase 2.8 Child Component Inventory.
- **`baseArgs`, `withIconArgs`, …** — module-level arg constants the Showcase uses instead of spreading `Basic.args`.

```tsx
const baseArgs = {
  caption: "Click me",
  hyperlink: "https://www.wavemaker.com",
  classname: "link-primary",
  onTap: action("onTap"),
};

const withIconArgs = {
  ...baseArgs,
  iconclass: "wi wi-dashboard",
};
```

---

## 3.6 Story export order

**`Standard`** first. **`Showcase`** last. Carry-over variants (update flow) go in between.

Final export order: `Standard` → carry-overs (if any) → `Showcase`.

> **Why `Standard` first.** Storybook's `<Primary />` block (rendered by `ComponentDocumentation` in the Docs page) shows the *first defined story*. Putting `Standard` first makes the Docs Canvas show a single component instance instead of the multi-variant `Showcase` gallery.

> **Why `Showcase` last.** It appears at the bottom of the sidebar so developers see interactive `Standard` first. It also gives carry-over variant stories a natural location between the two.

> **No `export const Docs: Story`** — the Docs entry in the sidebar is powered by `parameters.docs.page: DocsPage` in `meta` (§ 3.3). Emitting a named `Docs` story export collides with Storybook's `autodocs` tag (set globally in `preview.tsx`) and causes the Docs entry to disappear from the sidebar.

### 3.6.A `Standard: Story`

The interactive, controls-driven story. Defined **first** so it becomes the Docs Canvas primary.

```tsx
const Template = (args: any) => <{ComponentName} {...args} />;

export const Standard: Story = {
  render: Template,
  args: {
    // every prop from Phase 2 Prop Inventory marked Include in args? = yes
    // callbacks wired with action(…)
  },
  argTypes: {
    // one entry per arg key that needs a control type override
    // do NOT redeclare globally-disabled props (name, styles, children, dataset)
  },
};
```

**Hard rules:**

- `args` keys MUST come **directly** from the Phase 2 Prop Inventory. No invented props.
- Default values MUST be the literal defaults from `{slug}.props.js`. When the literal default is `null` and the prop is a string, use `""`. Otherwise use `undefined`.
- Callbacks (`on*`) appear in `args` with `action("<event-name>")` — never as `argTypes` controls.
- For Sub-pattern A, include `children: defaultChildren` in `args` so the Template renders the resolved children.

#### Required-prop floor (Complex components)

`Standard.args` must contain — at minimum — every prop required for visible content, even non-interactive ones:

| Sub-pattern | Floor — these MUST appear in args |
|---|---|
| **A** | `children: defaultChildren` (always). `dataset` when the parent declares it. |
| **A — column-kind** | `defaultChildren` is a non-empty fragment with one `<{ChildComponent} />` per `sampleData` field (each with `binding` / `caption` / unique `name`). |
| **A — pane-kind** | `defaultChildren` contains ≥ 2 `<{ChildComponent} />` instances, each with `name`, `title`, `index`, and a non-empty inner `<View>` containing `<Text>` content. |
| **A — form-kind** | `defaultChildren` contains one `<WmComposite>` per `sampleData` field, wrapping a `<WmLabel caption="…" />` and a real input widget matching the field's inferred type. |
| **B** | `dataset` and `renderItem` returning JSX that references ≥ 2 distinct `sampleData` keys. |
| **C** | `dataset`, `datafield`, `displayfield`, and `searchkey` when declared. All values match keys on `sampleData` rows. |
| **D** | `dataset`, `nodelabel`, `nodeid`, `nodechildren`. When `levels`/`treeicons`/`expandLevel` are declared, include them. |
| **E** | `dataset` / `data` plus every required array/object (e.g. `dataKeys`, `xDataKeyArr`, `chartColors`). |

If a Complex story is saved without its sub-pattern's floor, regeneration is invalid (the component renders empty).

### 3.6.B `Showcase: Story`

RN `View` + `Text` gallery driven by Phase 2 visual axes. Uses module-level constants (`baseArgs`, `withIconArgs`) — never spreads `Basic.args` (there is no `Basic` export). **Always the last story export in the file.**

```tsx
export const Showcase: Story = {
  parameters: { layout: "padded" },  // override meta's "centered" so the gallery fills the viewport
  render: () => (
    <View style={{ gap: 24, width: "100%" }}>
      {/* sections — see showcase-guide.md */}
    </View>
  ),
};
```

No `args`. No `argTypes`. See [showcase-guide.md](showcase-guide.md). Showcase must be **responsive** — sections use `flexWrap` with `flexBasis`-sized instance wrappers so they reflow on narrow viewports.

---

## 3.7 Carry-over variant exports (update flow only)

> **Update-flow contract.** When the caller supplies an `existingStoryDir`, every additional named Story export from that file (e.g. `WithIcon`, `WithBadge`) is preserved and **re-emitted in the new file**, in its original relative order. This is non-optional — silently dropping carry-overs is a regression.

When Phase 1 detected an existing story directory and Phase 2 produced a Prop-diff audit, **insert** the carry-overs **after `Standard` and before `Showcase`** (Showcase must remain the last story export):

```tsx
// After Standard, before Showcase:
export const WithIcon: Story = {
  args: {
    ...Standard.args,
    name: "iconAnchor",
    caption: "Anchor with icon",
    iconclass: "wi wi-dashboard",
    // any keep-list props that override Standard.args
  },
};

// ... more carry-overs ...

// Showcase is always last:
export const Showcase: Story = { ... };
```

Rules:

- Order matches the original file (e.g. `WithIcon`, then `WithBadge`).
- Body is `args: { ...Standard.args, /* overrides */ }`. No `argTypes`.
- Only props in the Prop-diff audit's `keep` and `add` lists appear here. Stripped props are removed.
- Callbacks preserved verbatim (e.g. `onTap: action("onTap")` stays as-is from the legacy file).

---

## 3.8 Callback wiring — `action()`

Every prop whose name starts with `on` and whose Phase 2 default is `null` is a callback. In each story's `args`, wire it:

```tsx
args: {
  onTap: action("onTap"),
  onChange: action("onChange"),
}
```

Never expose a callback as an `argType` control.

---

## 3.9 Final assembly checklist

- [ ] Imports include only what's used (no unused imports).
- [ ] `DocsPage` function declared above `meta` — renders `<ComponentDocumentation />` with five markdown imports.
- [ ] `meta` ends with `} satisfies Meta<typeof {ComponentName}>;`.
- [ ] `meta.title` = `{metaTitle}` from Phase 1.
- [ ] `meta.decorators` matches Phase 2.4 (default `View`, or extended with required providers).
- [ ] `meta.parameters.layout` set (`"centered"` for most; `"fullscreen"` for full-page components).
- [ ] `meta.parameters.docs.page` = `DocsPage` and `canvas.sourceState` = `"none"`.
- [ ] **No** `export const Docs: Story` — docs page is wired via `meta.parameters.docs.page`.
- [ ] Story export order: `Standard` first → carry-over variants (if any) in original order → `Showcase` **last**.
- [ ] `Standard` is populated from Phase 2 Prop Inventory; callbacks use `action()`.
- [ ] Required-prop floor met for the detected sub-pattern (Complex only).
- [ ] `Showcase` has no `args`, no `argTypes`; declares `parameters: { layout: "padded" }`; outer gallery `View` has `width: "100%"`; uses module-level constants for shared props; uses responsive `flexWrap` + `flexBasis` wrappers.
- [ ] Carry-over variant exports (if any) inserted **between `Standard` and `Showcase`**, with `args: { ...Standard.args, ... }`.
- [ ] No `token` import (until `ComponentDocumentation` adds the prop).
- [ ] No `mockListener`, no `?raw` markdown beyond the five `docs/*.md`, no `token` import, no design-token wiring, no globally-disabled props re-declared.
