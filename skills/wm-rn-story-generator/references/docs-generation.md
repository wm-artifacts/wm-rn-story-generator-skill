# Docs Generation (Phase 4)

Goal: create `{docsDir}` containing exactly six markdown files that feed the `ComponentDocumentation` panel rendered by the `Docs` story.

> **Two kinds of files.** `overview.md` / `styling.md` / `token.md` are **literal templates** copied from [`../assets/docs-stubs/`](../assets/docs-stubs/) with placeholder substitution only. `props.md` / `events.md` / `methods.md` are **per-component auto-generated** from the Phase 2 inventory using the format examples in this file.

---

## File overview

| File | Source | Marker line | Tab it feeds |
|---|---|---|---|
| `overview.md` | literal template ([`../assets/docs-stubs/overview.md`](../assets/docs-stubs/overview.md)) | — *(no marker)* | Overview |
| `props.md` | auto-generated from Phase 2 props (format below) | `<!-- AUTO-GENERATED FILE. Do not edit manually. -->` | Properties |
| `events.md` | auto-generated from Phase 2 `on*` callbacks (format below) | `<!-- AUTO-GENERATED FILE. Do not edit manually. -->` | Events |
| `methods.md` | auto-generated from Phase 2 public methods (format below) | `<!-- AUTO-GENERATED FILE. Do not edit manually. -->` | Methods |
| `styling.md` | literal template ([`../assets/docs-stubs/styling.md`](../assets/docs-stubs/styling.md)) | — *(no marker)* | Styling |
| `token.md` | literal template ([`../assets/docs-stubs/token.md`](../assets/docs-stubs/token.md)) | `<!-- AUTO-GENERATED FILE. Do not edit manually. -->` | *(not yet wired — emitted for parity)* |

Filename rules:

- The styling file is **always `styling.md`** (never `style.md`). The story imports it as `styling`.
- Every auto-generated file starts with `<!-- AUTO-GENERATED FILE. Do not edit manually. -->` as the **first line**.
- `overview.md` and `styling.md` MUST NOT carry that marker — they are manual stubs.
- `token.md` carries the marker but is **not** imported by the story until `ComponentDocumentation` adds a token prop.

---

## `props.md` — categorisation rules

Generate from the props extracted in Phase 2. Sort each prop into the **first matching group**.

| Group | Props that belong here |
|---|---|
| **Basic** | `name`, `caption`, `label`, `value`, `placeholder`, `badgevalue`, `hint`, and any other primary content/identity props |
| **Accessibility** | `tabindex`, `shortcutkey`, `accessibilityrole`, `accessibilitylabel`, `arialabel`, `hint` (if not already in Basic) |
| **Layout** | `width`, `height`, `margin`, `padding`, `class`, `classname`, `xsclass`, `smclass`, `mdclass`, `lgclass` |
| **Dataset** | `dataset`, `datafield`, `displayfield`, `displayexpression`, `searchkey`, `hyperlink`, `type` (when data-related) |
| **Behavior** | `show`, `disabled`, `readonly`, `loadOnDemand`, `target`, `animation`, `encodeurl`, `autofocus`, `required` |
| **Graphics** | `iconclass`, `iconurl`, `iconwidth`, `iconheight`, `iconmargin`, `iconposition`, `imageurl` |

- Group `<details>` in the order above; **omit empty groups**.
- The first non-empty group uses `<details open>` (expanded); all others are collapsed.
- Description text is inferred from the prop's name/purpose and existing WaveMaker conventions. When uncertain, write a one-sentence neutral description. **Do not invent.**
- If the component has no props at all, write the no-props fallback.

### Format example

```markdown
<!-- AUTO-GENERATED FILE. Do not edit manually. -->

# Properties

<details open>
  <summary>Basic</summary>
  <div>
| Property | Type | Default | Description |
| --- | --- | --- | --- |
| `caption` | string | - | Primary caption text shown by the component. |
  </div>
</details>

<details>
  <summary>Behavior</summary>
  <div>
| Property | Type | Default | Description |
| --- | --- | --- | --- |
| `disabled` | boolean | false | Disables interaction when true. |
  </div>
</details>
```

### No-props fallback

```markdown
<!-- AUTO-GENERATED FILE. Do not edit manually. -->

# Properties

The {ComponentName} component does not expose any specific properties.
```

---

## `events.md` — categorisation rules

Generate from `on*` callback props extracted in Phase 2.

| Group | Events |
|---|---|
| **Basic Events** | `onFocus`, `onBlur`, `onChange`, `onSelect`, `onLoad`, `onReady` |
| **Mouse Events** | `onClick`, `onDoubleClick`, `onMouseEnter`, `onMouseLeave` |
| **Touch Events** | `onTap`, `onDoubleTap`, `onLongTap`, `onSwipeUp`, `onSwipeDown`, `onSwipeLeft`, `onSwipeRight` |
| **Component-specific Events** | Any `on*` not covered above (group label = component name + `" Events"`) |

- Group `<details>` in the order above; omit empty groups.
- The first non-empty group uses `<details open>`; others collapsed.
- Standard descriptions for common events (verbatim):

| Event | Description |
|---|---|
| `onFocus` | This event handler is called each time the component is focused. |
| `onBlur` | This event handler is called each time focus leaves the component. |
| `onTap` | This event handler is called whenever the component is tapped. |
| `onDoubleTap` | This event handler is called whenever the component is double-tapped. |
| `onLongTap` | This event handler is called whenever the component is long-pressed. |
| `onChange` | This event handler is called whenever the component's value changes. |

- For component-specific events, write a one-sentence neutral description. Do not invent semantics not present in source.

### Format example

```markdown
<!-- AUTO-GENERATED FILE. Do not edit manually. -->

# Callback Events

<details open>
  <summary>Touch Events</summary>
  <div>
| Event | Description |
| --- | --- |
| `onTap` | This event handler is called whenever the component is tapped. |
  </div>
</details>
```

### No-events fallback

```markdown
<!-- AUTO-GENERATED FILE. Do not edit manually. -->

# Callback Events

The {ComponentName} component does not expose any specific events.
```

---

## `methods.md` — generation rules

Generate from public methods on the compiled component class in `{slug}.component.js` (instance methods, not lifecycle methods).

- If one or more methods exist, render them in a single `<details open>` block titled `Methods`.
- Parameter and return-type cells are inferred from the compiled signature. When parameters are unknown, write `—`.
- Description: write a one-sentence neutral description based on the method name. Do not invent semantics.
- If the component exposes no methods, write the no-methods fallback.

### Format example

```markdown
<!-- AUTO-GENERATED FILE. Do not edit manually. -->

# Methods

<details open>
  <summary>Methods</summary>
  <div>
| Method | Parameters | Return Type | Description |
| --- | --- | --- | --- |
| `focus` | — | void | Programmatically focuses the component. |
  </div>
</details>
```

### No-methods fallback

```markdown
<!-- AUTO-GENERATED FILE. Do not edit manually. -->

# Methods

The {ComponentName} component does not expose any specific methods.
```

---

## Markdown table format inside `<details>`

Tables MUST follow this exact shape — the renderer (`@storybook/addon-docs/blocks` `<Markdown>`) is strict:

- `<details>` / `</details>` and `<summary>` are HTML; **`<summary>`, `<div>`, and `</div>` use a 2-space indent under `<details>`** (prettier-style nesting).
- **No blank line** between `<div>` and the first table row, or between the last table row and `</div>` — blank lines inside the block break the surrounding accordion styling.
- **Table rows MUST be flush-left (column 0)**. Indented pipe rows are not recognised as a table by the CommonMark parser and render as raw `| … |` text.
- Between sibling `<details>` blocks, keep one blank line (standard markdown block separator).

### WRONG — blank line after `<div>` breaks the table

```markdown
<details>
  <summary>Behavior</summary>
  <div>
                           ← ❌ this blank line breaks the renderer
| Property | Type | Default | Description |
| --- | --- | --- | --- |
| `disabled` | boolean | false | Disables interaction when true. |
  </div>
</details>
```

### RIGHT — `<div>` immediately followed by the first table row

```markdown
<details>
  <summary>Behavior</summary>
  <div>
| Property | Type | Default | Description |
| --- | --- | --- | --- |
| `disabled` | boolean | false | Disables interaction when true. |
  </div>
</details>
```

This rule applies to **every** `<details>` block in every docs file. The first `<details open>` block and every subsequent collapsed `<details>` must all follow the same format — no exceptions.

---

## Literal templates

`overview.md`, `styling.md`, `token.md` are copied verbatim from `../assets/docs-stubs/` with placeholder substitution (`{ComponentName}`, `{componentSlug}`). They do not auto-generate from source.

---

## Update flow — porting legacy prose

When Phase 1 detected an existing story directory and that directory contains a `docs/` folder, port prose only into the three literal templates:

| Legacy file → new file |
|---|
| `{existingStoryDir}/docs/overview.md` → `{docsDir}/overview.md` |
| `{existingStoryDir}/docs/styling.md` (or `style.md`) → `{docsDir}/styling.md` |
| `{existingStoryDir}/docs/token.md` → `{docsDir}/token.md` (only if present) |

The three auto-generated files (`props.md`, `events.md`, `methods.md`) always regenerate from Phase 2 source — never port them from the legacy folder, as their data may be out of sync with the runtime.

Legacy files like `studio-props-and-events.md` or `script-props-methods.md` (from the old `ComponentDocumentation` shape) do **not** map directly; copy their introductory prose into `overview.md` if useful, then discard the rest.

---

> Forbidden patterns are listed in [forbidden-output.md](forbidden-output.md). Never wire design-token data inside any docs file.
