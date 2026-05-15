# Showcase Story Guide

The `Showcase` story is a **visual gallery of real usage examples** — not a controls playground. Its purpose is to show developers and designers *how the component can be displayed* across its most meaningful configurations.

Sections are driven entirely by what makes sense for the component. There is no fixed number of sections required.

---

## Layout Rules

- Use **React Native primitives**: `View` for layout, `Text` for section/instance labels. Storybook runs in a browser via `addon-react-native-web`, so `flexDirection`, `gap`, `flexWrap`, `padding` all work.
- Every variant instance has a `Text` label above it. The label is the **concrete value being demonstrated** (e.g. `"primary"`, `"left"`, `"disabled"`, `"With Icon"`), not a generic placeholder.
- Use real, meaningful sample data — prefer constants from `constants/constant.ts` (`Users`, `salesData`, `carouselImages`, `quarterResults`).
- Avoid disabled variants unless the section is specifically about disabled states.
- Each instance must have a unique `name` prop.
- Callbacks must remain wired — spread a module-level constant (`baseArgs`, `withIconArgs`) that includes `action()` wrappers.
- **Showcase is always the last story export in the file.** Declare it after `Standard` and any carry-over variants.

---

## Responsive layout (mandatory)

Showcase must fill the full canvas width and reflow on narrow viewports.

**Story-level override** — always declare `parameters: { layout: "padded" }` on the `Showcase` export. This overrides the meta-level `"centered"` so the canvas gives the gallery full available width instead of shrinking it to content size.

Use this exact wrapper recipe — no fixed `width` on instance wrappers.

| Container | Style |
|---|---------|
| Outer (whole gallery) | `<View style={{ gap: 24, width: "100%" }}>` |
| Section block | `<View style={{ gap: 12 }}>` |
| Section row of instances | `<View style={{ flexDirection: "row", flexWrap: "wrap", gap: 16, rowGap: 16 }}>` |
| Instance wrapper (label + component) | `<View style={{ gap: 4, flexBasis: 240, flexGrow: 1, maxWidth: "100%" }}>` |

Rules:

- `flexWrap: "wrap"` on row containers is **mandatory** — without it, narrow viewports clip overflow.
- Use `flexBasis` to set the desired per-card width (240 is a good default for most form controls; raise to 320 for cards/tiles, lower to 180 for icons/chips). `flexGrow: 1` lets cards stretch to fill leftover row space.
- Never set a fixed `width` on the instance wrapper — it defeats the wrap.
- For components that need to render full-width regardless of viewport (e.g. `WmLinearLayout`, `WmList`), set the instance wrapper to `flexBasis: "100%"` and skip `flexGrow`.

---

## No `args`, no `argTypes`

Showcase declares neither. Shared props live in module-level constants above `meta`:

```tsx
const baseArgs = {
  caption: "Click me",
  classname: "link-primary",
  onTap: action("onTap"),
};
```

Then inside Showcase: `<{ComponentName} {...baseArgs} name="..." /* overrides */ />`. This replaces the legacy `{...Basic.args}` pattern (there is no `Basic` export in this skill).

---

## When to use `.map()`

`.map()` is **allowed** in Showcase renders for compactness — existing `WmAnchor.Showcase` in the legacy `components/WmAnchor/` is the reference.

| Situation | Approach |
|---|---|
| 1–3 instances per section | Write each `<{ComponentName} />` explicitly. |
| ≥4 instances of the same shape (same prop set, only values differ) | Build a `variants` array of `{ title, items: [...] }` and `.map()` over it. |

---

## How to Determine Sections

Read the component source (Phase 2) and ask each question. Add a section only when the answer is yes.

| Question | If yes → add a section |
|---|---|
| Does the component have a `type` / `variant` / `classname` family with multiple confirmed values? | One section showing each value side by side |
| Does the component have an `iconposition` prop or icon-related props? | One section: Basic / With Icon / Icon Right / Icon Left as applicable |
| Does the component show meaningfully different layouts? | One section per layout mode |
| Does the component render data (list, table, chart)? | One section with realistic data; optionally one with empty / loading state |
| Does the component have state variants (read-only, disabled, error)? | One section for state variants |
| Does the component accept `classname` with confirmed WaveMaker class families (`btn-*`, `link-*`)? | One section per confirmed family |

> No required minimum number of sections. Let the component drive the structure.

---

## Skeleton — explicit instances (≤3 per section)

```tsx
export const Showcase: Story = {
  parameters: { layout: "padded" },
  render: () => (
    <View style={{ gap: 24, width: "100%" }}>

      {/* Section: {Real visual axis from Phase 2} */}
      <View style={{ gap: 12 }}>
        <Text style={{ fontSize: 16, fontWeight: "bold" }}>{Section title}</Text>
        <View style={{ flexDirection: "row", flexWrap: "wrap", gap: 16, rowGap: 16 }}>
          <View style={{ gap: 4, flexBasis: 240, flexGrow: 1, maxWidth: "100%" }}>
            <Text style={{ fontSize: 12, color: "#666" }}>{concrete value 1}</Text>
            <{ComponentName} {...baseArgs} name="showcase1" /* overrides */ />
          </View>
          <View style={{ gap: 4, flexBasis: 240, flexGrow: 1, maxWidth: "100%" }}>
            <Text style={{ fontSize: 12, color: "#666" }}>{concrete value 2}</Text>
            <{ComponentName} {...baseArgs} name="showcase2" /* overrides */ />
          </View>
        </View>
      </View>

    </View>
  ),
};
```

---

## Skeleton — `.map()` over variants array (≥4 per section)

Mirrors the legacy `WmAnchor.Showcase`:

```tsx
export const Showcase: Story = {
  parameters: { layout: "padded" },
  render: () => {
    const variants = [
      {
        title: "{Section title 1}",
        items: [
          { ...baseArgs, name: "v1a", caption: "..." },
          { ...baseArgs, name: "v1b", caption: "..." },
          { ...baseArgs, name: "v1c", caption: "..." },
          { ...baseArgs, name: "v1d", caption: "..." },
        ],
      },
      {
        title: "{Section title 2}",
        items: [
          { ...withIconArgs, name: "v2a", iconclass: "wi wi-bell" },
          { ...withIconArgs, name: "v2b", iconclass: "wi wi-message" },
        ],
      },
    ];

    return (
      <View style={{ gap: 24, width: "100%" }}>
        {variants.map((section, i) => (
          <View key={i} style={{ gap: 12 }}>
            <Text style={{ fontSize: 16, fontWeight: "bold" }}>{section.title}</Text>
            <View style={{ flexDirection: "row", flexWrap: "wrap", gap: 16, rowGap: 16 }}>
              {section.items.map((item, j) => (
                <View key={j} style={{ flexBasis: 240, flexGrow: 1, maxWidth: "100%" }}>
                  <{ComponentName} {...item} />
                </View>
              ))}
            </View>
          </View>
        ))}
      </View>
    );
  },
};
```

---

## CSS Class-Variant Section

Only add when class variants are **confirmed** in the component's `{slug}.styles.js` or in an existing repo story. Don't invent class families.

| Signal | Action |
|---|---|
| `classname` confirmed with `btn-*` / `link-*` / `text-*` family | One section per family |
| 1–2 class variants only | Inline into an existing section; no dedicated section needed |
| No `classname`-based styling | Skip entirely |

---

## Update-flow note

When carrying over a Showcase block from `{existingStoryDir}`, rewrite any `{...Basic.args}` references to use the module-level constants (`{...baseArgs}`, `{...withIconArgs}`). The `Basic` export does not exist in the new file.
