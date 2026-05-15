<!-- FORMAT REFERENCE — NOT a literal copy-paste template. -->
<!--
This file is the structural shape `{docsDir}/events.md` MUST produce. The
actual content is auto-generated per component from the Phase 2 on* callback
props using the categorisation rules in ../../references/docs-generation.md.

Always start the GENERATED file with the AUTO-GENERATED marker as the first
line. Render groups in order Basic / Mouse / Touch / Component-specific,
omitting empty groups. First non-empty group uses <details open>.

Use the verbatim descriptions in docs-generation.md for common events; write a
neutral one-sentence description for component-specific events.

Format rules inside <details>:
  - <summary>, <div>, </div> are indented 2 spaces under <details>.
  - Pipe-table rows are flush-left (column 0) — indented rows are NOT parsed.
  - No blank line between <div> and the first table row, or between the last
    table row and </div>.
  - One blank line between sibling <details> blocks.
-->

<!-- AUTO-GENERATED FILE. Do not edit manually. -->

# Callback Events

<details open>
  <summary>Basic Events</summary>
  <div>
| Event | Description |
| --- | --- |
| `onFocus` | This event handler is called each time the component is focused. |
| `onBlur` | This event handler is called each time focus leaves the component. |
| `onChange` | This event handler is called whenever the component's value changes. |
  </div>
</details>

<details>
  <summary>Touch Events</summary>
  <div>
| Event | Description |
| --- | --- |
| `onTap` | This event handler is called whenever the component is tapped. |
| `onDoubleTap` | This event handler is called whenever the component is double-tapped. |
| `onLongTap` | This event handler is called whenever the component is long-pressed. |
  </div>
</details>
