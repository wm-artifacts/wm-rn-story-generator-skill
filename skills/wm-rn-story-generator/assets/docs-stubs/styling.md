# Styling

<!--
LITERAL TEMPLATE. Copy verbatim into {docsDir}/styling.md and replace the
{ComponentName} / {componentSlug} placeholders. Do NOT include the
AUTO-GENERATED marker — styling.md is the manual stub.

When the update flow detected {existingStoryDir}/docs/styling.md (or style.md),
port its prose below before saving.
-->

The `{ComponentName}` component is styled via the WaveMaker theming pipeline. Styles defined on the component's `styles` prop are merged with the active theme's `{componentSlug}` style object.

## Style props

Styles are composed of named entries (`root`, `text`, `icon`, …) — refer to `{componentSlug}.styles.js` in the runtime package for the full list applicable to this component.

## Class families

If the component supports a `classname` prop, the supported families are documented in the runtime styles file. Only families that are confirmed in source belong here.
