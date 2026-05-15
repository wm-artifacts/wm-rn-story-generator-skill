# wm-rn-story-generator

An AI skill for WaveMaker React Native Storybook projects. Give it a component source path and it produces a complete CSF3 TypeScript story file with three canonical exports (`Standard`, `Showcase`, `Docs`) and a sibling `docs/` folder with six markdown files — ready to commit.

---

## Who This Is For

Developers maintaining the WaveMaker React Native Storybook. If you're adding or updating a story for a component under `@wavemaker/app-rn-runtime`, this skill does the heavy lifting so you don't have to think about meta structure, argTypes wiring, Showcase layout, or docs file conventions.

---

## Installation

```sh
npx skills add wm-artifacts/wm-rn-story-generator-skill
```

The skill is picked up automatically by any AI assistant that supports npm-based skills.

---

## Usage

### 1. Invoke the skill with a component path

Pass the component source path — short form or full package form both work:

```
/wm-rn-story-generator generate a story for basic/anchor
```

```
/wm-rn-story-generator generate a story for @wavemaker/app-rn-runtime/components/chart/area-chart
```

### 2. Review the Prop Inventory

Before writing any file, the skill prints a **Prop Inventory** table in chat listing every prop, its type, default value, and control type. Review it and correct anything that looks wrong.

If the skill hits a blocker it cannot resolve from source (missing directory, ambiguous export, non-trivial default), it pauses and asks — respond with the missing detail and it continues.

### 3. Commit the output

The skill produces:

1. A story file at `storybook/stories/wm-{category}/wm-{slug}/{slug}.stories.tsx`
2. Six docs files under `storybook/stories/wm-{category}/wm-{slug}/docs/`
3. A one-time entry appended to `.storybook/main.ts` (idempotent)

Review, adjust if needed, then commit to your feature branch.

---

## What Gets Generated

For a component at `basic/anchor` the skill writes:

```
storybook/stories/wm-basic/wm-anchor/
├── anchor.stories.tsx
└── docs/
    ├── overview.md
    ├── props.md
    ├── events.md
    ├── methods.md
    ├── styling.md
    └── token.md
```

### Story exports

| Export | Notes |
|---|---|
| `Standard` | Controls-driven story; first export so it becomes the Docs Canvas primary |
| Carry-over variants | Update flow only — preserved between `Standard` and `Showcase` |
| `Showcase` | Visual gallery across all key prop axes; always the last export |
| `Docs` | Rendered via `meta.parameters.docs.page` using `ComponentDocumentation` |

### Docs files

| File | How it's produced |
|---|---|
| `overview.md` | Manual stub — fill in component purpose and key behaviours |
| `props.md` | Auto-generated from Phase 2 prop scan |
| `events.md` | Auto-generated from Phase 2 prop scan |
| `methods.md` | Auto-generated from Phase 2 prop scan |
| `styling.md` | Manual stub — fill in theming and style override notes |
| `token.md` | Auto-generated stub — reserved for design-token documentation |

---

## Advanced usage

### Update flow — preserve existing variants

Supply the existing story directory as a second argument to trigger the update flow. All named Story exports are carried over and docs prose is ported from the legacy file.

```
/wm-rn-story-generator generate a story for basic/anchor components/WmAnchor
```

### Secondary source (non-WM package)

For components not in `@wavemaker/app-rn-runtime`, also supply the import path:

```
/wm-rn-story-generator generate a story for @my-org/components/input/button --importPath @my-org/components/input/button
```

---

## Project Files

| File | Purpose |
|---|---|
| [skills/wm-rn-story-generator/SKILL.md](skills/wm-rn-story-generator/SKILL.md) | Skill definition — instructions the AI follows |
| [skills/wm-rn-story-generator/references/](skills/wm-rn-story-generator/references/) | Phase-by-phase rules loaded on demand |
| [skills/wm-rn-story-generator/assets/story-templates/](skills/wm-rn-story-generator/assets/story-templates/) | Simple and Complex story scaffolds |
| [skills/wm-rn-story-generator/assets/docs-stubs/](skills/wm-rn-story-generator/assets/docs-stubs/) | Canonical bodies for the six docs files |

---

## Repo wiring assumptions

The skill assumes the consuming Storybook project has:

- **`.storybook/preview.tsx`** — globally disables `styles`, `children`, `dataset`, `name` in `argTypes` and applies `autodocs` tag
- **`.storybook/components/ComponentDocumentation.tsx`** — accepts `overview`, `props`, `events`, `methods`, `styling` props
- **`constants/constant.ts`** — exports `animationNames`, `glyphMap`, `Users`, `salesData`, `carouselImages`, `quarterResults`
- **`services/`** — exports `handleAsset`, `ModalServiceComponent`, `RefreshWrapper`, `SearchService`, `WmTimeService`

Legacy stories under `components/Wm{Name}/Wm{Name}.stories.tsx` are **never** modified or deleted — two Storybook entries coexist until you remove the legacy file.
