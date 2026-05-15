<!-- FORMAT REFERENCE — NOT a literal copy-paste template. -->
<!--
This file is the structural shape `{docsDir}/props.md` MUST produce. The actual
content is auto-generated per component from the Phase 2 Prop Inventory using
the categorisation rules in ../../references/docs-generation.md.

Always start the GENERATED file with the AUTO-GENERATED marker as the first
line. Always render groups in this order, omitting empty groups. The first
non-empty group uses <details open>; all others stay collapsed.

Format rules inside <details>:
  - <summary>, <div>, </div> are indented 2 spaces under <details>.
  - Pipe-table rows are flush-left (column 0) — indented rows are NOT parsed.
  - No blank line between <div> and the first table row, or between the last
    table row and </div>.
  - One blank line between sibling <details> blocks.
-->

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
  <summary>Accessibility</summary>
  <div>
| Property | Type | Default | Description |
| --- | --- | --- | --- |
| `accessibilitylabel` | string | - | Label announced by assistive technology. |
  </div>
</details>

<details>
  <summary>Layout</summary>
  <div>
| Property | Type | Default | Description |
| --- | --- | --- | --- |
| `class` | string | - | Layout class applied to the root element. |
  </div>
</details>

<details>
  <summary>Dataset</summary>
  <div>
| Property | Type | Default | Description |
| --- | --- | --- | --- |
| `dataset` | array | - | Data source consumed by the component. |
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

<details>
  <summary>Graphics</summary>
  <div>
| Property | Type | Default | Description |
| --- | --- | --- | --- |
| `iconclass` | string | - | Icon class rendered next to the caption. |
  </div>
</details>
