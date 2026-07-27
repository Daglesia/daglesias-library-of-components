# AI_GUIDE.md — instructions for AI assistants working in this repo

This file is context for an AI coding assistant (Claude, Copilot, etc.), not
end-user documentation — component docs live in Penpot. Read this before
adding, editing, or reviewing anything in `src/`.

## What this package is

`dlc-ui` is a CSS component library, distributed as an npm package, written
in SASS and compiled to plain CSS (`dist/dlc-ui.css`). Consumers install it,
import the compiled stylesheet, and apply classes (`dlc-button`,
`dlc-card`, etc.) directly in their markup. There is no JS runtime — any
interactive behavior (opening a modal, toggling `.is-open`) is the
consumer's responsibility; this library only owns the CSS.

## Naming rules (non-negotiable)

- **Prefix:** every class starts with `dlc-`. No exceptions, including
  utility classes.
- **BEM:** `dlc-block`, `dlc-block__element`, `dlc-block--modifier`.
  - Block = the component root, e.g. `dlc-card`.
  - Element = `dlc-card__header`. Elements are always written as a
    direct child selector of the block using SASS `&__name` nesting —
    never nest an element inside a modifier or another element
    (`dlc-card__header__title` is wrong; flatten to `dlc-card__title`).
  - Modifier = `dlc-button--outline`. Modifiers change appearance/layout,
    never structure. A modifier class is always combined with its block
    class in markup (`class="dlc-button dlc-button--outline"`), never
    used alone.
- **State classes are not BEM modifiers.** JS-toggled states use `is-*`
  (`is-open`, `is-loading`, `is-disabled`), unprefixed, added alongside the
  block class. Reserve `--modifier` for variants a developer chooses at
  render time; reserve `is-*` for states that change at runtime.
- Native pseudo-classes (`:disabled`, `:checked`) are preferred over `is-*`
  wherever the DOM already exposes that state; only add an `is-*` class
  when there's no native equivalent (e.g. `is-loading`, `is-open`).

## File layout

```
src/
  index.scss              entry point — @use every partial here, in order
  abstracts/
    _index.scss            barrel: @forward variables + mixins
    _variables.scss         design tokens ($dlc-* Sass vars + --dlc-* CSS vars)
    _mixins.scss             shared mixins (focus ring, transitions, variants…)
  base/
    _reset.scss             minimal, scoped normalization for [class*="dlc-"] only
  components/
    _button.scss, _input.scss, _card.scss, …   one file per block
```

- **One component = one file = one block.** File name matches the class
  name minus the prefix: `dlc-button` lives in `_button.scss`.
- Every component partial must start with `@use "../abstracts" as *;` to
  get `$dlc-*` variables and `dlc-*` mixins unprefixed.
- Use the modern Sass module system (`@use`/`@forward`). Do not reintroduce
  `@import` — it's deprecated and was previously the source of a
  hard-to-find mixin bug in this repo (see git history / commit that added
  variadic args to `dlc-transition`).

## Design tokens — always use them, never hardcode

- Colors, spacing, radii, shadows, font sizes, transitions, breakpoints,
  and z-index all come from `abstracts/_variables.scss`.
- Two layers exist on purpose:
  - `$dlc-*` Sass variables — compile-time defaults, `!default` so a
    consumer can override before importing the source SCSS.
  - `--dlc-*` CSS custom properties on `:root` — runtime-overridable
    without a rebuild (theming, dark mode, per-instance overrides).
- **Rule of thumb:** colors, radii, shadows, and transitions that a
  consumer might reasonably want to theme at runtime should reference the
  `var(--dlc-*)` CSS custom property inside component files. Purely
  structural values (spacing, font-size, font-weight, breakpoints,
  z-index) that aren't meant to be themed use the `$dlc-*` Sass variable
  directly. Follow the existing components as precedent — check how
  `_button.scss` and `_card.scss` split this before adding a new one.
- If a token you need doesn't exist yet, add it to `_variables.scss`
  (both the `$dlc-*` var and, if it should be themeable, the `--dlc-*`
  custom property) — don't inline a one-off hex code or px value in a
  component file.
- Never use `!important`. If a component needs to win a specificity fight,
  fix the selector, not the declaration.

## Adding a new component — checklist

1. Create `src/components/_<name>.scss`.
2. Header comment block listing the Block, its Elements, Modifiers, and
   States, mirroring the format at the top of `_button.scss`.
3. `@use "../abstracts" as *;` right after the header comment.
4. Build the block with `&__element` / `&--modifier` nesting under a
   single root `.dlc-<name> { }` selector.
5. Use `@include dlc-focus-ring;` on any focusable/interactive element,
   `@include dlc-transition(...)` on anything with hover/active states,
   and `@include dlc-disabled-state;` for disabled handling.
6. Add `@use "components/<name>";` to `src/index.scss`, alphabetically
   grouped with the other component `@use` lines is not required — match
   the existing informal ordering (form-adjacent components first, then
   layout/feedback components).
7. Run `npm run build` and confirm no Sass warnings or errors, then check
   the diff in `dist/dlc-ui.css` looks like what you intended — the
   compiled CSS is committed and versioned, not gitignored.

## Things to check before finishing any change

- [ ] Every new class starts with `dlc-`.
- [ ] No element selector is nested more than one level deep (no
      `__element__subelement`).
- [ ] No hardcoded color, spacing, or radius that duplicates an existing
      token — reuse or extend `_variables.scss` instead.
- [ ] No `!important`.
- [ ] Interactive elements have a visible focus state
      (`dlc-focus-ring` mixin) and a `:disabled` / `.is-disabled` state.
- [ ] `npm run build` runs clean (no deprecation warnings, no errors) and
      both `dist/dlc-ui.css` and `dist/dlc-ui.min.css` are regenerated and
      included in the commit.
- [ ] New component's markup structure is compared against the Penpot
      spec for that component before considering it done — this repo
      does not duplicate that documentation, so check Penpot directly
      rather than guessing at structure.

## Build commands

```
npm run build   # one-off build: dist/dlc-ui.css + dist/dlc-ui.min.css
npm run watch   # rebuild dist/dlc-ui.css on save while developing
```
