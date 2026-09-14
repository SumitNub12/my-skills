---
name: designer-ui-style
description: Enforce Designer UI layout, design-token, and styling constraints. Use when generating React components, using Box/Flex/Stack/Grid, touching design tokens, CSS modules, theme variables, or files under src/App/components and src/App/tokens.
---

# Designer UI Style

Single source of truth for Designer UI layout and styling. Every layout and style decision must go through this skill.

Applies to ALL code generation. Do not create new layout wrappers, generic containers, or ad-hoc styling. Always reuse the existing primitives and design tokens below.

## 1. Layout Components — Never Write Raw Layout Divs

Import from `src/App/components/new/layout/LayoutComponents`:

| Component  | Purpose                                  | Key Props                                                                                                       |
| ---------- | ---------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| **Box**    | Generic container with spacing/border/bg | `p`, `px/py/pt/pr/pb/pl`, `m`, `mx/my/...`, `background`*, `border`, `borderRadius`*, `width`, `height`, `as` |
| **Flex**   | Row-oriented flexbox layout              | `direction`, `align`, `justify`, `wrap`, `gap`                                                                  |
| **Stack**  | Vertical flex layout (column)            | `gap`, `align`, `justify`                                                                                       |
| **Inline** | Horizontal flex with wrap                | `gap`, `align`, `justify`                                                                                       |
| **Grid**   | CSS Grid layout                          | `columns` (number or string), `gap`, `align`                                                                    |
| **Bleed**  | Negative-margin breakout                 | `amount`, `horizontal`, `vertical`                                                                              |

* `background` and `borderRadius` on Box accept token strings like `"color.background.brand"` and `"radius.075"`.

Always use these when you need a container, a row, a column, a grid, or spacing. Never write `<div style={{ display: 'flex', ... }}>` or raw `<div className={...}>` for layout.

## 2. Design Token System

All spacing, color, typography, and border-radius values come from the design token system at `src/App/tokens/`. Never use raw numbers like `gap={4}`.

### Token Scale Quick Reference

| Token               | Value | Replaces  |
| ------------------- | ----- | --------- |
| `token.space.s050`  | 4px   | `gap={1}` |
| `token.space.s100`  | 8px   | `gap={2}` |
| `token.space.s150`  | 12px  | `gap={3}` |
| `token.space.s200`  | 16px  | `gap={4}` |
| `token.space.s250`  | 20px  | `gap={5}` |
| `token.space.s300`  | 24px  | —         |
| `token.space.s400`  | 32px  | —         |
| `token.space.s500`  | 40px  | —         |
| `token.space.s600`  | 48px  | —         |
| `token.space.s800`  | 64px  | —         |
| `token.space.s1000` | 80px  | —         |

### How to Use Tokens

In JSX — import the `token` object (autocompletion friendly):

```jsx
import { token } from 'path/to/tokens';

// Good — token.space dot notation
<Stack gap={token.space.s200}>
  <Flex gap={token.space.s100} align="center">
    <Box p={token.space.s050} background={token.color.backgroundBrand}>...</Box>
  </Flex>
</Stack>

// Bad — magic numbers
<Stack gap={4}>
  <Flex gap={2}>...</Flex>
</Stack>
```

In CSS Modules — use `--ds-*` custom properties:

```css
/* Good */
.myClass {
  gap: var(--ds-space-100);
  border-radius: var(--ds-radius-075);
  font-size: var(--ds-font-size-085);
}

/* Bad */
.myClass {
  gap: 0.5rem;
  border-radius: 6px;
  font-size: 0.85rem;
}
```

### Available Token Categories

| Category    | JS Access Pattern                                  | CSS Custom Property Pattern    |
| ----------- | -------------------------------------------------- | ------------------------------ |
| Spacing     | `token.space.s100`, `token.space.s200`             | `var(--ds-space-100)`          |
| Color       | `token.color.background`, `token.color.text`       | `var(--ds-color-background)`   |
| Font size   | `token.fontSize.s085`, `token.fontSize.s100`       | `var(--ds-font-size-085)`      |
| Font weight | `token.fontWeight.medium`, `token.fontWeight.bold` | `var(--ds-font-weight-medium)` |
| Radius      | `token.radius.s075`, `token.radius.round`          | `var(--ds-radius-075)`         |

## 3. UI Components — Use Existing Building Blocks

Import these instead of raw `<button>`, `<input>`, `<select>` etc.:

| Component       | File Path                                |
| --------------- | ---------------------------------------- |
| **Button**      | `components/new/button/Button`           |
| **Input**       | `components/new/input/Input`             |
| **ButtonGroup** | `components/new/buttonGroup/ButtonGroup` |
| **Switch**      | `components/new/switch/Switch`           |

If you need a component that doesn't exist yet inside `components/new/`:

1. First check whether one of the existing App components covers the need: `Card`, `Dialog`, `Modal`, `Tabs`, `Divider`, `Badge`, `CheckBox`, `RadioButton`, `MultiSelect`, `SelectDropDown`, `ToolbarButton`, `PrimaryButton`, `SecondaryButton`, `IconButton`, `TextButton`, `ClosableBox`, etc.
2. Only if nothing matches, create a new component — but export it from `src/App/components/new/<name>/<Name>.js` so it joins the shared set.

## 4. Styling Rules

### 4a. Theme Variables — Only Through Tokens or `--app-*` Vars

Theme variables are defined in `src/App/styles.css` `:root`. Reference them through design tokens (preferred) or direct `--app-*` custom properties (acceptable in legacy code).

```css
/* Preferred — token variables */
.myClass {
  color: var(--ds-color-text);
}

/* Acceptable — direct --app-* usage */
.myClass {
  color: var(--app-text-color);
}

/* Never hardcode */
.myClass {
  color: #f0f0f0;
}
```

### 4b. CSS Modules for Component Styles

Every component gets a co-located `*.module.css` file. Use `Utils.getClasses()` to merge with parent-passed classNames.

```jsx
import classes from './MyComponent.module.css';
import Utils from '../../Utils';

const mergedClass = Utils.getClasses(classes.root, className);
```

### 4c. Inline Styles — Only for Dynamic Values

Static styles belong in the `.module.css` file. Only use inline `style={}` for truly dynamic values (e.g., computed width, runtime color).

### 4d. Disabled State Convention

```css
.item-disabled {
  opacity: 0.4;
  pointer-events: none;
}
```

## 5. Component Structure Convention

All new UI components under `src/App/components/new/` follow this shape:

```
new/<component-name>/
  <Name>.js        — component logic
  <Name>.module.css — component styles
```

Use default exports for consistency with the existing codebase.

## 6. Quick Reference — Import Paths

```js
// Design tokens (single import)
import { token } from 'path/to/tokens';
// Usage: token.space.s200, token.color.text, token.fontSize.s085, token.radius.s075

// Layout primitives
import {
  Box,
  Flex,
  Stack,
  Inline,
  Grid,
  Bleed
} from 'components/new/layout/LayoutComponents';

// UI components
import Button from 'components/new/button/Button';
import Input from 'components/new/input/Input';
import ButtonGroup from 'components/new/buttonGroup/ButtonGroup';
import Switch from 'components/new/switch/Switch';

// Utility
import Utils from 'path/to/Utils'; // Utils.getClasses(), etc.
```

## 7. Non-Negotiable

- Every layout must use `Box` / `Flex` / `Stack` / `Inline` / `Grid`.
- Every spacing value (`gap`, `p`, `m`) must use a design token — no raw numbers.
- Every styled element must use CSS Modules + theme variables.
- No raw `<div>` layout containers with inline flexbox or hardcoded colors.
- No hardcoded hex colors — always reference `--ds-*` or `--app-*` variables.

(If you genuinely need a token value that doesn't exist yet, add it to `src/App/tokens/tokens.js` and `tokens.css`.)

## 8. Code Comments

- Keep comments minimal. Only comment non-obvious logic. Do not add redundant or verbose comments.
- Do not use special symbols in comments. No emoji, no box-drawing, no decorative characters. Plain ASCII text only.
