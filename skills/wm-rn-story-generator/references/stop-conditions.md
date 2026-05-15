# Stop Conditions

Stop and ask the user (do **not** invent values) if any of the following hold. In each case: surface the exact blocker, list what was found, and wait for user direction before writing any file.

| Condition | What to surface |
|---|---|
| `componentSourceDir` does not exist (the resolved path from Phase 1.1 Path Classification) | The path tried, its parent directory listing, the caller-supplied input, and the detected path tier. Primary WM fixes: wrong category, wrong slug, missing `npm install`. Secondary fixes: wrong relative path, wrong package name, missing install. |
| Secondary path provided but `componentImportPath` not supplied by caller | The source path supplied, the resolved `componentSourceDir`, and a prompt asking for the import path to use in the story `import` statement. |
| `componentExportName` cannot be determined: no default export found in `{slug}.component.{tsx\|ts\|js}` | The file path read, the export shapes found, and a prompt asking for the export name to use in the story. |
| `{componentSlug}.props.js` is missing inside `componentSourceDir` | The directory contents and the file expected. There is no source of truth for prop defaults — never invent. |
| Component is exported in an unrecognised shape (no default export, factory wrapper, abstract class) | The export shape found in `{componentSlug}.component.js`. |
| A prop's default value is non-trivial (function call, imported constant whose value can't be resolved) **AND** the prop must appear in `args` | The prop name, the literal default expression, and the file it came from. Ask whether to use `""` / `0` / `undefined`. |
| `sourceCategory` does not match any row in the Phase 1.2 category mapping | The folder name found and the list of recognised mappings. Ask the user which `storyCategoryTitle` to use. |
| Category is `basic` and disambiguation between `Form/...` and `Basic/...` is unclear (no sibling story to mirror) | The slug, the existing `title:` prefixes found via grep, and ask which to use. |
| `{storyFile}` already exists and the user did not say "regenerate" / "update" | The existing file path. Ask whether to overwrite or stop. |
| `existingStoryDir` was supplied but `{existingStoryDir}/{componentName}.stories.tsx` is not found | The path tried and the directory listing. |
| A required service provider (e.g. `NavigationService`) cannot be located in `@wavemaker/app-rn-runtime/core/` | The import path tried. |
| Sub-pattern A applies but a referenced child sub-component folder cannot be located inside `componentSourceDir` | The parent folder listing and the child the parent's `*.component.js` references. Do **not** invent a `defaultChildren` block from memory. |
| Sub-pattern E required array/object prop has no default in source and no `sampleData`-derivable shape | The prop name, the source line, and the `sampleData` keys available. |
| `iconclass` / `classname` enumeration cannot be confidently derived from `{slug}.styles.js` or an existing repo story | The Phase 2 evidence found. Ask whether to leave the prop as a free-form text control or supply an options list. |

> The Phase 2.6 Prop Inventory checkpoint surfaces most of these blockers. If a stop condition is hit during the inventory, do **not** print a partial inventory — surface the blocker and stop.
