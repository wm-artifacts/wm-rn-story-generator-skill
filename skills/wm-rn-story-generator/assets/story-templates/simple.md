# Story Template — Simple Component

Use when **all props are primitives** (string, boolean, number) AND no service-provider decorator is needed beyond the default `View` wrapper.

Replace every `{placeholder}` before saving. Strip the comment markers (`// TODO`, `// e.g.`) — they exist only in this template.

| Placeholder | Source |
|---|---|
| `{ComponentName}` | Phase 1 |
| `{componentSlug}` | Phase 1 |
| `{metaTitle}` | Phase 1 (e.g. `Basic/Anchor`) |
| `{componentImportPath}` | Phase 1 |
| `{relativePathToComponentDocumentation}` | Phase 1 (e.g. `../../../../.storybook/components/ComponentDocumentation`) |
| `{relativePathToConstants}` | Phase 1 (only if option arrays are used) |

---

```tsx
import type { Meta, StoryObj } from "@storybook/react";
import React from "react";
import { View, Text } from "react-native";
import { action } from "storybook/actions";

import {ComponentName} from "{componentImportPath}";

// Optional — only if used by Showcase / Standard:
// import { animationNames, glyphMap } from "{relativePathToConstants}";

import { ComponentDocumentation } from "{relativePathToComponentDocumentation}";
import overview from "./docs/overview.md?raw";
import props from "./docs/props.md?raw";
import events from "./docs/events.md?raw";
import methods from "./docs/methods.md?raw";
import styling from "./docs/styling.md?raw";

// ─── Module-level shared values ──────────────────────────────────────────────
// Hoist anything reused by ≥2 stories. Showcase spreads these instead of
// referencing a non-existent `Basic.args`.

const baseArgs = {
  // {prop}: {default-from-Phase-2},
  // onTap: action("onTap"),
};

// const withIconArgs = {
//   ...baseArgs,
//   iconclass: "wi wi-dashboard",
// };

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
      <View style={{ padding: 16 }}>
        <Story />
      </View>
    ),
  ],
  parameters: {
    layout: "centered",
    docs: {
      page: DocsPage,
      canvas: { sourceState: "none" },
    },
  },
} satisfies Meta<typeof {ComponentName}>;

export default meta;

type Story = StoryObj<typeof meta>;

// ─── Standard ────────────────────────────────────────────────────────────────
// The interactive, controls-driven story. Populated from the Phase 2 Prop Inventory.
// Defined FIRST so it becomes the Docs Canvas primary (Storybook's <Primary />
// block renders the first story export).

const Template = (args: any) => <{ComponentName} {...args} />;

export const Standard: Story = {
  render: Template,
  args: {
    // {prop}: {default-from-Phase-2},
    // onTap: action("onTap"),
  },
  argTypes: {
    // Only declare argTypes that need an explicit control type override.
    // Do NOT redeclare name / styles / children / dataset — they are globally disabled.
    //
    // {prop}: { control: "text" },
    // iconposition: { control: "radio", options: ["left", "right"] },
    // animation: { control: { type: "select" }, options: animationNames },
  },
};

// ─── Carry-over variant exports (update flow only) ───────────────────────────
// Inserted BEFORE Showcase (after Standard). Body is `args: { ...Standard.args, … }`.
// Only props from the Phase 2 Prop-diff audit's keep/add lists appear here.
//
// export const WithIcon: Story = {
//   args: {
//     ...Standard.args,
//     name: "iconExample",
//     iconclass: "wi wi-bell",
//   },
// };

// ─── Showcase ────────────────────────────────────────────────────────────────
// Hand-laid gallery. No args, no argTypes. See ../../references/showcase-guide.md.
// ALWAYS the last story export in the file.
// parameters.layout "padded" overrides the meta-level "centered" so the canvas
// fills the full viewport width and the flex gallery can reflow properly.

export const Showcase: Story = {
  parameters: { layout: "padded" },
  render: () => (
    <View style={{ gap: 24, width: "100%" }}>
      {/*
        Section: {real visual axis from Phase 2}
        <View style={{ gap: 12 }}>
          <Text style={{ fontSize: 16, fontWeight: "bold" }}>{Section title}</Text>
          <View style={{ flexDirection: "row", flexWrap: "wrap", gap: 16, rowGap: 16 }}>
            <View style={{ gap: 4, flexBasis: 240, flexGrow: 1, maxWidth: "100%" }}>
              <Text style={{ fontSize: 12, color: "#666" }}>{concrete value}</Text>
              <{ComponentName} {...baseArgs} name="showcase1" />
            </View>
          </View>
        </View>
      */}
    </View>
  ),
};
```
