# Story Template — Complex Component

Use when the component:

- Requires a service-provider decorator (Navigation / Gesture / Modal / Asset), **or**
- Has any non-primitive required prop (Sub-pattern A–E from Phase 2.7).

Pick exactly one sub-pattern body from § 4 below. The shell (imports, meta, three exports) is the same for all sub-patterns.

Replace every `{placeholder}` before saving. Strip the comment markers (`// TODO`, `// e.g.`) — they exist only in this template.

| Placeholder | Source |
|---|---|
| `{ComponentName}` | Phase 1 |
| `{componentSlug}` | Phase 1 |
| `{metaTitle}` | Phase 1 |
| `{componentImportPath}` | Phase 1 |
| `{relativePathToComponentDocumentation}` | Phase 1 |
| `{relativePathToConstants}` | Phase 1 |
| `{relativePathToServices}` | Phase 1 (e.g. `../../../../services/`) |

---

## 1. Shell — imports + meta + three exports

```tsx
import type { Meta, StoryObj } from "@storybook/react";
import React from "react";
import { View, Text } from "react-native";
import { action } from "storybook/actions";

import {ComponentName} from "{componentImportPath}";

// Service providers — include only those Phase 2.4 flagged:
// import { NavigationServiceProvider } from "@wavemaker/app-rn-runtime/core/navigation.service";
// import { GestureHandlerRootView } from "react-native-gesture-handler";
// import { ModalServiceComponent } from "{relativePathToServices}ModalService";
// import { AssetProvider } from "@wavemaker/app-rn-runtime/core/asset.provider";
// import { handleAsset } from "{relativePathToServices}Assethandler";

// Sub-pattern A child sub-components — from Phase 2.8 Child Component Inventory:
// import {ChildComponent} from "@wavemaker/app-rn-runtime/components/{cat}/{slug}/{child-folder}/{child-slug}.component";
// import WmLabel from "@wavemaker/app-rn-runtime/components/basic/label/label.component";

// Shared option arrays / sample data — include only those actually used:
// import { Users, salesData, carouselImages, quarterResults } from "{relativePathToConstants}";

import { ComponentDocumentation } from "{relativePathToComponentDocumentation}";
import overview from "./docs/overview.md?raw";
import props from "./docs/props.md?raw";
import events from "./docs/events.md?raw";
import methods from "./docs/methods.md?raw";
import styling from "./docs/styling.md?raw";

// ─── Mock service / shared data ──────────────────────────────────────────────

// Navigation service mock — only if NavigationServiceProvider is used:
// const navigationService = {
//   openUrl: (url: string, options?: { target?: string }) => {
//     action("openUrl")(url, options);
//   },
// };

// Sample data — shape MUST match what {ComponentName} consumes:
// const sampleData = [ /* … */ ];

// Sub-pattern A helper component — mirrors AccordionContent pattern from WmAccordion:
// const {ChildContent} = ({ num }: { num: number }) => (
//   <WmLabel
//     name={`content${num}`}
//     caption={`This is the content for {Label} ${num}.`}
//   />
// );

// ─── Docs page (injected via meta.parameters.docs.page) ──────────────────────
// NOT a Story export. Using parameters.docs.page avoids the collision with
// Storybook's global `autodocs` tag that causes a named `Docs` Story export
// to disappear from the sidebar.

const DocsPage = () => (
  <ComponentDocumentation
    overview={overview}
    props={props}
    events={events}
    methods={methods}
    styling={styling}
  />
);

// ─── Meta ────────────────────────────────────────────────────────────────────

const meta = {
  title: "{metaTitle}",
  component: {ComponentName},
  decorators: [
    (Story) => (
      // Compose the required provider stack (Navigation / Gesture / Modal / Asset)
      // around the padded View. Keep <View> as the innermost wrapper.
      //
      // Example:
      // <NavigationServiceProvider value={navigationService}>
      //   <View style={{ padding: 16 }}>
      //     <Story />
      //   </View>
      // </NavigationServiceProvider>
      <View style={{ padding: 16 }}>
        <Story />
      </View>
    ),
  ],
  parameters: {
    layout: "centered",   // "fullscreen" for full-page components (e.g. WmCarousel)
    docs: {
      page: DocsPage,
      canvas: { sourceState: "none" },
    },
  },
  argTypes: {
    // Declare argTypes that need non-default control types.
    // Do NOT re-disable globally-disabled props (name, styles, children, dataset).
  },
} satisfies Meta<typeof {ComponentName}>;

export default meta;

type Story = StoryObj<typeof meta>;

// ─── Standard ────────────────────────────────────────────────────────────────
// Pick ONE Standard body from § 4 below based on the detected sub-pattern.
// Defined FIRST so it becomes the Docs Canvas primary (Storybook's <Primary />
// block renders the first story export).

const Template = (args: any) => <{ComponentName} {...args} />;

// export const Standard: Story = { /* see § 4 */ };

// ─── Showcase ────────────────────────────────────────────────────────────────
// Hand-laid gallery. No args, no argTypes. See ../../references/showcase-guide.md.
// Layout MUST be responsive — section rows use flexWrap and instance wrappers
// use flexBasis + flexGrow so they reflow on narrow viewports.

export const Showcase: Story = {
  render: () => (
    <View style={{ gap: 32 }}>
      {/*
        Sections driven by Phase 2 visual axes — see showcase-guide.md.
        Each row:
          <View style={{ flexDirection: "row", flexWrap: "wrap", gap: 16, rowGap: 16 }}>
        Each instance wrapper:
          <View style={{ gap: 4, flexBasis: 240, flexGrow: 1, maxWidth: "100%" }}>
      */}
    </View>
  ),
};

// ─── Carry-over variant exports (update flow only) ───────────────────────────
// Appended after Showcase in original order. Body is `args: { ...Standard.args, … }`.
//
// export const WithIcon: Story = {
//   args: {
//     ...Standard.args,
//     name: "iconExample",
//     iconclass: "wi wi-bell",
//   },
// };
```

---

## 2. Required-prop floor

`Standard.args` MUST contain the rows required by the detected sub-pattern. This is the contract from [../../references/story-structure.md](../../references/story-structure.md) § 3.6.B.

| Sub-pattern | Floor |
|---|---|
| **A** | `children` rendered inline via `render` function (always). `dataset` in `args` when the parent declares it. |
| **A — column-kind** | `render` contains one `<{ChildComponent} />` per `sampleData` field (each with `binding` / `caption` / unique `name`). |
| **A — pane-kind** | `render` contains ≥ 2 `<{ChildComponent} />` instances via `.map()`, each with `name`, `title`, `show`, and a non-empty inner content component. |
| **A — form-kind** | `render` contains one `<WmComposite>` per `sampleData` field, wrapping a `<WmLabel caption="…" />` and a real input widget matching the field's inferred type. |
| **B** | `dataset` in `args` AND `renderItem` prop on the component in `render`, returning JSX referencing ≥ 2 distinct `sampleData` keys. |
| **C** | `dataset`, `datafield`, `displayfield`, `searchkey` (when declared). All values match keys on every `sampleData` row. |
| **D** | `dataset`, `nodelabel`, `nodeid`, `nodechildren`. Include `levels` / `treeicons` / `expandLevel` if declared. |
| **E** | `dataset` (or `data`) plus every required array/object (e.g. `dataKeys`, `xDataKeyArr`, `chartColors`). |

---

## 3. argTypes guidance

Only declare argTypes for visible props that need a non-default control type. Do **not** redeclare the globally-disabled props (`name`, `styles`, `children`, `dataset`).

Callbacks never become controls — they live in `args` wrapped in `action("…")`.

---

## 4. Sub-pattern bodies for `Standard`

### 4.A — Sub-pattern A, pane-kind (e.g. WmTabs, WmAccordion)

Children are rendered inline in the `render` function using `.map()`. Use a helper component
above `meta` for content (mirrors `AccordionContent` from WmAccordion.stories.tsx).

```tsx
// Optional helper component above meta — keeps render clean:
const {ChildContent} = ({ num }: { num: number }) => (
  <WmLabel
    name={`content${num}`}
    caption={`This is the content for {Label} ${num}.`}
  />
);

export const Standard: Story = {
  render: (args) => (
    <{ComponentName} {...args}>
      {[1, 2, 3].map((num) => (
        <{ChildComponent}
          key={num}
          name={`pane${num}`}
          title={`{Label} ${num}`}
          show={true}
          // onSelect={action("onSelect")}
        >
          <{ChildContent} num={num} />
        </{ChildComponent}>
      ))}
    </{ComponentName}>
  ),
  args: {
    name: "{componentSlug}Standard",
    // other primitive props from Phase 2 Prop Inventory
    // onTap: action("onTap"),
  },
  argTypes: {
    // visible-prop control overrides
  },
};
```

Variant stories for pane-kind also carry a matching `render` override:

```tsx
export const WithIcon: Story = {
  args: {
    ...Standard.args,
    name: "{componentSlug}WithIcon",
  },
  render: (args) => (
    <{ComponentName} {...args}>
      {[1, 2, 3].map((num) => (
        <{ChildComponent}
          key={num}
          name={`pane${num}`}
          title={`{Label} ${num}`}
          show={true}
          iconclass={/* icon per num, e.g. from an array */}
        >
          <{ChildContent} num={num} />
        </{ChildComponent}>
      ))}
    </{ComponentName}>
  ),
};
```

---

### 4.A — Sub-pattern A, column-kind (e.g. WmDataTable columns)

```tsx
const defaultChildren = (
  <>
    <{ChildComponent} name="col1" binding="{primaryField}" caption="{Primary Label}" />
    <{ChildComponent} name="col2" binding="{secondaryField}" caption="{Secondary Label}" />
    {/* one per sampleData field from Phase 2.8 inventory */}
  </>
);

export const Standard: Story = {
  render: (args) => (
    <{ComponentName} {...args}>
      {defaultChildren}
    </{ComponentName}>
  ),
  args: {
    name: "{componentSlug}Standard",
    dataset: sampleData,
    // other primitive props
  },
};
```

---

### 4.B — Sub-pattern B (renderItem, e.g. WmList)

The `renderItem` prop is passed directly on the component inside `render`, not through `args`.
For multi-template scenarios, define a `handleTemplate` function above `meta`.

```tsx
// Optional multi-template handler above meta:
// const handleTemplate = (templateName: string, $item: any, $index: number, list: any) => {
//   switch (templateName) {
//     case "{Template A}": return <{TemplateA} $item={$item} $index={$index} list={list} />;
//     case "{Template B}": return <{TemplateB} $item={$item} $index={$index} list={list} />;
//     default:             return <{TemplateA} $item={$item} $index={$index} list={list} />;
//   }
// };

export const Standard: Story = {
  render: (args) => (
    <{ComponentName}
      {...args}
      renderItem={($item: any, $index: number, list: any) => (
        <View style={{ padding: 8 }}>
          <Text>{$item.{primaryField}}</Text>
          <Text style={{ color: "#666" }}>{$item.{secondaryField}}</Text>
        </View>
        // For multi-template: handleTemplate(args.templatename, $item, $index, list)
      )}
    />
  ),
  args: {
    name: "{componentSlug}Standard",
    dataset: sampleData,
    // datafield / displayfield / other props from Phase 2 Prop Inventory
    onSelect: action("onSelect"),
  },
  argTypes: {
    // e.g. direction: { control: "radio", options: ["vertical", "horizontal"] },
  },
};
```

---

### 4.C — Sub-pattern C (field-mapping, e.g. WmSelect, WmSearch)

```tsx
export const Standard: Story = {
  render: Template,
  args: {
    name: "{componentSlug}Standard",
    dataset: sampleData,
    datafield: "{idKey}",        // matches every sampleData row — see Phase 2.9 alignment
    displayfield: "{labelKey}",
    searchkey: "{labelKey}",     // when the prop is declared in source
    onChange: action("onChange"),
  },
};
```

---

### 4.D — Sub-pattern D (hierarchical dataset)

```tsx
export const Standard: Story = {
  render: Template,
  args: {
    name: "{componentSlug}Standard",
    dataset: sampleData,
    nodelabel: "{labelKey}",
    nodeid: "{idKey}",
    nodechildren: "children",    // or whatever array key holds child nodes
    // expandLevel: 1,           // include when declared in source
    onSelect: action("onSelect"),
  },
};
```

---

### 4.E — Sub-pattern E (config-object / chart family, e.g. WmBarChart, WmLineChart)

Pure `args` — no custom `render` needed. Add commented variant examples that extend `Standard.args`.

```tsx
export const Standard: Story = {
  render: Template,
  args: {
    name: "{componentSlug}Standard",
    dataset: sampleData,           // e.g. quarterResults from constants/constant.ts
    xaxisdatakey: "{xKey}",        // e.g. "Quarter"
    yaxisdatakey: "{yKey}",        // e.g. "Revenue"
    // dataKeys / xDataKeyArr / chartColors — seed from Phase 2.7 Chart Config Inventory
    onSelect: action("onSelect"),
  },
  argTypes: {
    xaxisdatakey: {
      control: { type: "select" },
      options: [...Object.keys(sampleData[0])],
    },
    // showlegend: { control: { type: "select" }, options: ["hide", "top", "bottom"] },
  },
};

// Variant: multiple data series — spreads Standard.args (mirrors WmBarChart.MultipleDataPlots):
// export const MultipleDataPlots: Story = {
//   args: {
//     ...Standard.args,
//     yaxisdatakey: "{yKey1},{yKey2}",
//     showlegend: "top",
//   },
// };
```
