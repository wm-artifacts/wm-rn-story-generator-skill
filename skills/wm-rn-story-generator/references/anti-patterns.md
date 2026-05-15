# Anti-Patterns — Debug Reference

Load only when verification fails or the rendered story misbehaves. The positive rules live in the per-phase references; this file shows bad/good pairs for the two rules in [forbidden-output.md](forbidden-output.md) plus a handful of common slip-ups.

---

## 1. Controls on `Docs` or `Showcase`

```tsx
// Bad
export const Docs: Story = {
  args: { caption: "Click me" },
  argTypes: { caption: { control: "text" } },
  render: () => <ComponentDocumentation /* … */ />,
};

export const Showcase: Story = {
  args: { ...Basic.args },   // Basic doesn't exist in this skill
  render: () => (/* gallery */),
};
```

```tsx
// Good
const baseArgs = { caption: "Click me", onTap: action("onTap") };

export const Docs: Story = {
  render: () => <ComponentDocumentation /* … */ />,
  parameters: { layout: "fullscreen" },
};

export const Showcase: Story = {
  render: () => (
    <View>
      <WmAnchor {...baseArgs} name="s1" />
    </View>
  ),
};
```

---

## 2. Invented prop names

```tsx
// Bad — `tooltip` is not in anchor.props.js
export const Standard: Story = {
  args: { caption: "Help", tooltip: "Click for help" },
};
```

```tsx
// Good — every key comes from a _defineProperty(...) row in anchor.props.js
export const Standard: Story = {
  args: { caption: "Help", hint: "Click for help" },
};
```

---

## 3. Common slip-ups

```tsx
// Bad — CSF2 template binding
const Template = (args) => <WmButton {...args} />;
export const Basic = Template.bind({});
Basic.args = { caption: "Click me" };
```

```tsx
// Good — CSF3
export const Standard: Story = {
  render: (args: any) => <WmButton {...args} />,
  args: { caption: "Click me" },
};
```

---

```tsx
// Bad — `const meta: Meta<typeof X> = { ... };` form
const meta: Meta<typeof WmButton> = { title: "Form/Button", component: WmButton };
```

```tsx
// Good — `satisfies` form (repo convention)
const meta = { title: "Form/Button", component: WmButton } satisfies Meta<typeof WmButton>;
```

---

```tsx
// Bad — relative import into node_modules
import WmButton from "../../../node_modules/@wavemaker/app-rn-runtime/components/basic/button/button.component";
```

```tsx
// Good — package import
import WmButton from "@wavemaker/app-rn-runtime/components/basic/button/button.component";
```

---

```tsx
// Bad — re-disabling globally-disabled argTypes
argTypes: {
  name:    { table: { disable: true } },
  styles:  { table: { disable: true } },
  dataset: { table: { disable: true } },
}
```

```tsx
// Good — preview.tsx already disables them. Only flip when you specifically need one visible:
argTypes: {
  dataset: { table: { disable: false } },
}
```

---

```tsx
// Bad — callback as a control
argTypes: { onTap: { action: "onTap" } }
```

```tsx
// Good — callback wired in args
args: { onTap: action("onTap") }
```
