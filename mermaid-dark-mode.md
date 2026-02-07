# Mermaid dark mode theming in Quartz

## Problem

Mermaid diagrams have poor contrast in dark mode: light/washed-out text on light pastel node backgrounds, making them unreadable.

## Root cause

Mermaid's theming engine is fundamentally broken for dark mode:

1. **`theme: "dark"` silently ignores all custom `themeVariables`**. Only `theme: "base"` respects them. This is a known bug: https://github.com/mermaid-js/mermaid/issues/4157 and https://github.com/mermaid-js/mermaid/issues/4264

2. **`theme: "base"` with `darkMode: true`** still produces unreliable colors. Mermaid internally derives secondary/tertiary colors from `primaryColor` using the khroma library, and the derivation produces low-contrast results that can't be fully overridden via `themeVariables`.

3. **CSS `text { fill: var(--darkgray) }` in Quartz's `base.scss`** forces SVG text fills globally, overriding whatever Mermaid sets. In dark mode `--darkgray` is `#d4d4d4` (light gray), causing light text.

4. **Mermaid v11 uses `<foreignObject>` with HTML for node labels**, not SVG `<text>` elements. This means SVG `fill` doesn't affect node text; CSS `color` is needed instead.

## What was tried (and failed)

- Setting `themeVariables` with `theme: "dark"` -- silently ignored
- Using `theme: "base"` with `darkMode: true` and Quartz CSS variables -- Mermaid's color derivation still produced bad contrast
- Targeted CSS overrides on `.node rect`, `.nodeLabel`, `.label` -- wrong class names for Mermaid v11
- CSS `text { fill: ... !important }` inside mermaid scope -- only affects SVG text, not foreignObject HTML labels

## What worked

Always render Mermaid in light mode and use CSS to make it work in dark mode:

### `quartz/components/scripts/mermaid.inline.ts`

Removed all dark mode logic. Always use `theme: "base"` with hardcoded light theme hex colors (not CSS variables, since Mermaid only accepts hex). This produces a clean, readable light-mode diagram regardless of page theme.

### `quartz/styles/custom.scss`

```scss
[saved-theme="dark"] pre:has(> code.mermaid) {
  background-color: #faf8f8;    // light background for the container
  border-radius: 8px;
  padding: 1rem;

  text {
    fill: #4e4e4e !important;   // SVG text elements (edge labels)
  }

  * {
    color: #4e4e4e !important;  // HTML text in foreignObject (node labels)
  }
}
```

Key insight: `color` affects HTML text (inside foreignObject), `fill` affects SVG `<text>` elements. Both are needed. Using `!important` to override Mermaid's baked-in styles. The `*` selector for `color` is safe because CSS `color` doesn't affect SVG shapes (which use `fill`/`stroke`).

## References

- Mermaid themeVariables ignored with non-base theme: https://github.com/mermaid-js/mermaid/issues/4264
- Mermaid themeVariables incorrectly overridden: https://github.com/mermaid-js/mermaid/issues/4157
- Quartz issue for mermaid dark mode: https://github.com/jackyzha0/quartz/issues/2202
- GitHub community discussion (unresolved): https://github.com/orgs/community/discussions/35733
- Mermaid CSS variables feature request (not implemented): https://github.com/mermaid-js/mermaid/issues/6677
- Mermaid base theme source: https://github.com/mermaid-js/mermaid/blob/develop/packages/mermaid/src/themes/theme-base.js
