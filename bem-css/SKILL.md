---
name: bem-css
description: "Implement and review CSS with the BEM methodology. Use when writing CSS, styling a component, refactoring selectors, naming blocks, elements, or modifiers, or reviewing a stylesheet for BEM violations. Not for general visual design, HTML architecture, JavaScript behavior, or framework-specific decisions unless they directly affect CSS class names."
user-invocable: false
argument-hint: "CSS or component files, the target block, and mode (implement, refactor, or review)"
metadata:
  generated-by: "create-better-skill (https://www.skills.sh/wazeerc/skills/create-better-skill)"
---

# BEM CSS

Implement and review CSS as independent, reusable BEM blocks. Use the [BEM CSS methodology](https://bem.info/en/methodology/css/) as the authority when project conventions do not specify a compatible rule.

## Start

1. Determine the mode: **implement**, **refactor**, or **review**. For a combined request, review the existing selectors before changing them.
2. Identify the target files, the block or component being changed, its elements, its modifiers, and any existing naming convention.
3. Read repository instructions and inspect nearby CSS before choosing names. Preserve an established project prefix or syntax when it remains BEM-compatible.
4. State the proposed selectors and affected files. Ask for confirmation before editing files.
5. After editing, run the narrowest available formatter, linter, typecheck, or test for the touched CSS. If no executable check exists, inspect the diff and report that limitation.

## Model the component

Describe the component in BEM entities before writing selectors:

- **Block**: an independent component or reusable concept, such as `.card` or `.menu`.
- **Element**: a meaningful part of its block, written `.block__element`, such as `.card__title`.
- **Modifier**: a boolean or key-value variation of a block or element, written `.block_modifier` or `.block_key_value`, such as `.card_featured` or `.button_size_large`.
- **Mix**: multiple independent BEM entities on one DOM node when each owns a separate responsibility, such as `button card__action`.

Use names that describe the entity's role, not its appearance or DOM tag. Prefer `.card__title` over `.card__blue-text`, and `.button_size_large` over `.button_24px`.

Do not create a selector for every nested DOM node. Create an element only when the part has a meaningful role within the block. If a nested part is independently reusable, make it a block and use a mix where needed.

## Write selectors

Follow these rules unless the project explicitly documents a compatible exception:

1. Use class selectors for blocks, elements, and modifiers.
2. Do not use IDs or tag selectors to style BEM entities.
3. Prefer one simple class per rule: `.card`, `.card__title`, `.card_featured`.
4. Avoid combining classes such as `.button.button_active`; a modifier should remain independently overridable.
5. Avoid descendant selectors such as `.page .card__title` and `.card .card__title`. Put the relationship in the element class or use a state/theme rule only when the relationship is essential.
6. Keep nesting minimal. Nest only when a block state, theme, or modifier must change a related element, for example `.button_hovered .button__text`.
7. Do not combine a tag and class, such as `button.button`; this increases specificity and makes modifiers harder to override.
8. Do not style a block based on where it happens to be rendered. Put external geometry and positioning on the parent element or a mix, for example `.header__button`, not `.header .button`.

Keep specificity low and predictable. When a variation is likely to change, add or adjust a modifier instead of changing the base block or adding a contextual override.

## Use modifiers and mixes

Use modifiers to express appearance, state, or behavior that varies while the entity remains the same. Keep the base block responsible for stable internal styling and put the variation in the modifier:

```css
.button {
  font-family: inherit;
  border: 1px solid currentColor;
}

.button_size_large {
  min-block-size: 3rem;
}
```

Use a mix when one node participates in more than one independent block or when a parent must supply external geometry. Keep responsibilities separate so mixed blocks do not define the same property and depend on source order.

Do not merge unrelated blocks merely because they share a color, size, or isolated declaration. Share behavior through a meaningful block or mix only when the components are functionally related.

## Organize files

When the project already organizes styles by component, keep each block's base styles, elements, and modifiers together or in the project's established BEM structure. Do not introduce a new file layout for a small change.

For a new structure, prefer a block directory with focused files:

```text
button/
  button.css
  _size/button_size_large.css
```

Follow the project's build system and naming syntax when it differs. A flat stylesheet is acceptable when that is the repository convention; preserve the same entity boundaries in the selectors.

## Review checklist

Check every touched selector and report the exact selector when a rule fails:

- The name identifies a block, element, modifier, or mix clearly.
- Elements belong to the correct block and are not chained (`.a__b__c`).
- Modifiers extend an entity without replacing its base class.
- Blocks remain reusable outside their current parent.
- External margin, positioning, and layout belong to the parent element or a mix.
- IDs, tag selectors, unnecessary combined selectors, and deep contextual nesting are absent.
- Specificity is low enough that modifiers can override base styles without escalation.
- Mixed blocks do not overlap in responsibility for the same property.
- New variation is represented by a modifier or a new block instead of a contextual exception.
- The change preserves the repository's formatter, linter, cascade, and build conventions.

## Boundaries

Handle CSS and CSS-adjacent class naming only when it is needed to implement or review the BEM change. Do not redesign the visual system, rewrite HTML structure, change JavaScript behavior, migrate a framework, or impose a new file architecture unless the user explicitly asks for that work. If the existing markup prevents a correct BEM implementation, explain the smallest required markup change and ask before making it.

When a project convention conflicts with the methodology, preserve the project convention if it is intentional, call out the tradeoff, and avoid silently rewriting unrelated selectors.
