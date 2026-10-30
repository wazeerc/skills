# Visual design

Visual requirements for low-vision, color-blind, motion-sensitive, and motor-impaired users. Ratios below were computed with the WCAG relative-luminance formula.

## Contents

- [Contrast](#contrast)
- [Use of color](#use-of-color)
- [Focus indicators](#focus-indicators)
- [Target size](#target-size)
- [Zoom, reflow, and text spacing](#zoom-reflow-and-text-spacing)
- [Content on hover or focus](#content-on-hover-or-focus)
- [Motion and animation](#motion-and-animation)
- [Forced colors and themes](#forced-colors-and-themes)

## Contrast

| Content | AA | AAA |
|---|---|---|
| Text below 24 CSS px (or below 18.66 CSS px bold) | 4.5:1 (1.4.3) | 7:1 (1.4.6) |
| Large text: at least 24 CSS px, or 18.66 CSS px bold | 3:1 | 4.5:1 |
| Component boundaries and states, focus indicators, meaningful graphics | 3:1 (1.4.11) | No separate AAA level |
| Disabled controls, decoration, logos, incidental text in photos | None | None |

Gray text on white (`#fff`):

| Color | Ratio | Result |
|---|---|---|
| `#595959` | 7.00:1 | Passes AAA for all text |
| `#767676` | 4.54:1 | Lightest gray that passes AA for body text |
| `#949494` | 3.03:1 | Large text and UI components only |
| `#999999` | 2.85:1 | Fails for all text and UI components |

- Measure the rendered colors, including opacity and blending, against the actual background behind them.
- For text over images or gradients, check the lowest-contrast area, or add a solid or semi-opaque backdrop.
- Check every state (hover, focus, active, visited, selected, error), placeholder text, and every theme.
- Thin and light font weights look fainter than their ratio suggests; choose more contrast for them (best practice).
- APCA, the contrast method proposed for WCAG 3, isn't a WCAG 2.x conformance method; report WCAG 2 ratios.
- Tools: the browser DevTools color picker, the [WebAIM contrast checker](https://webaim.org/resources/contrastchecker/), or the TPGi Colour Contrast Analyser.

Compute ratios in scripts or tests instead of estimating them (6-digit hex):

```js
export function contrastRatio(hexA, hexB) {
  const luminance = (hex) => {
    const [r, g, b] = hex.replace('#', '').match(/../g).map((pair) => {
      const channel = parseInt(pair, 16) / 255;
      return channel <= 0.04045 ? channel / 12.92 : ((channel + 0.055) / 1.055) ** 2.4;
    });
    return 0.2126 * r + 0.7152 * g + 0.0722 * b;
  };
  const [lighter, darker] = [luminance(hexA), luminance(hexB)].sort((a, b) => b - a);
  return (lighter + 0.05) / (darker + 0.05);
}
```

## Use of color

- Underline links in blocks of text. If they aren't underlined, they need at least 3:1 contrast with the surrounding text plus a non-color cue, such as an underline, on hover and focus (WCAG technique G183).
- Mark errors, required fields, and statuses with text or an icon as well as color.
- In charts, label data directly or add patterns and shapes; don't rely on a color legend alone. Adjacent segments need 3:1 contrast with each other or a visible separator.

## Focus indicators

```css
:focus-visible {
  outline: 3px solid #0b57d0;
  outline-offset: 2px;
}
```

`#0b57d0` contrasts 6.39:1 with white and 3.29:1 with black. Check the indicator against every background it can appear on.

- Style `:focus-visible` and leave the browser default in place until you do. Never remove outlines globally (`*:focus { outline: none }`).
- Prefer `outline`: it doesn't shift layout and stays visible in forced-colors mode. If the design uses `box-shadow`, also set `outline: 2px solid transparent`, because forced-colors mode drops shadows but shows the outline.
- On pages with varied backgrounds, use a two-color indicator (W3C technique [C40](https://www.w3.org/WAI/WCAG22/Techniques/css/C40)). When the two colors contrast at least 9:1 with each other, one of them reaches 3:1 against any solid background. These two contrast 12.71:1:

```css
:focus-visible {
  outline: 2px solid #f9f9f9;
  outline-offset: 0;
  box-shadow: 0 0 0 4px #193146;
}
```

- Ancestors with `overflow: hidden` can clip the indicator; add padding or use a negative `outline-offset`.
- Keep focused elements clear of sticky headers and footers (2.4.11) with W3C technique [C43](https://www.w3.org/WAI/WCAG22/Techniques/css/C43):

```css
html {
  scroll-padding-block: var(--sticky-header-height, 0px) var(--sticky-footer-height, 0px);
}
```

- For AAA 2.4.13, a solid outline at least 2 CSS px thick around the whole component, with 3:1 contrast against the unfocused pixels it changes, meets the size and contrast requirements.

## Target size

- AA (2.5.8): at least 24 × 24 CSS px, or enough spacing that a 24 px circle centered on each undersized target doesn't intersect another target or its circle. Inline links in text, unmodified browser controls, targets with an equivalent control elsewhere, and targets whose size is essential (such as map pins) are exempt.
- Aim for 44 × 44 CSS px for primary actions and touch interfaces (2.5.5 at AAA).
- Enlarge the hit area with padding, not by enlarging the icon:

```css
.icon-button {
  display: inline-grid;
  place-items: center;
  min-inline-size: 24px;
  min-block-size: 24px;
}

@media (pointer: coarse) {
  .icon-button {
    min-inline-size: 44px;
    min-block-size: 44px;
  }
}
```

- Leave space between adjacent small targets, such as close buttons, pagination links, and table row actions.

## Zoom, reflow, and text spacing

- Build layouts that reflow at 320 CSS px (1280 px at 400% zoom) using flexible grids, `flex-wrap`, `max-inline-size`, and media queries (1.4.10).
- Size text in `rem` or `em` so it follows the user's browser font size, and never set fixed heights on containers of text.
- Break long strings such as URLs and email addresses with `overflow-wrap: anywhere`.
- Release `position: sticky` and `position: fixed` on short viewports; at high zoom, sticky bars can cover most of the screen.
- Truncation (ellipsis, line clamps) hides content at high zoom; provide the full text another way.
- Don't apply `!important` text spacing in inline styles; it blocks user style sheets (1.4.12).

Test 1.4.12 by applying these overrides and checking that nothing is cut off or overlaps:

```css
* {
  line-height: 1.5 !important;
  letter-spacing: 0.12em !important;
  word-spacing: 0.16em !important;
}

p {
  margin-bottom: 2em !important;
}
```

## Content on hover or focus

Tooltips, popovers, and hover menus must meet 1.4.13:

- **Dismissible**: Escape closes the content without moving the pointer or focus.
- **Hoverable**: the pointer can move from the trigger onto the content; avoid gaps, or delay hiding briefly.
- **Persistent**: the content stays until it's dismissed, the trigger loses hover or focus, or it's no longer relevant; no timeouts.

Pure CSS `:hover` popups can't be dismissed with Escape; add script.

## Motion and animation

```css
@media (prefers-reduced-motion: no-preference) {
  .panel {
    transition: transform 200ms ease-out;
  }

  html {
    scroll-behavior: smooth;
  }
}
```

```js
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
```

- Opt in to motion for users without the reduced-motion preference. Reduced motion means replacing large movement (parallax, zooming, sliding panels, auto-scrolling) with fades or instant changes, not removing every transition.
- Anything that moves, blinks, or scrolls automatically for more than 5 seconds needs a pause, stop, or hide control (2.2.2): carousels, background video, animated GIFs, and Lottie animations.
- Nothing may flash more than three times per second (2.3.1). Avoid large, saturated red flashes entirely.
- A global reset that shortens every animation is a reasonable safety net, but design a reduced alternative for each significant motion.

## Forced colors and themes

Windows contrast themes (forced-colors mode) replace author colors with a small system palette and usually drop `box-shadow`. Transparent borders and outlines get a visible system color; only backgrounds keep their transparency. Supporting this mode is a best practice beyond WCAG AA.

- Give components a real border, even `1px solid transparent`; forced-colors mode makes it visible, so boundaries survive.
- Show focus with `outline` (see above).
- Use `fill="currentColor"` or `fill: currentColor` for SVG icons so they follow the text color.
- States shown only by background color (selected, checked, current) disappear; add a border, underline, or icon. System color keywords are kept in forced-colors mode, so restyle with them where needed:

```css
@media (forced-colors: active) {
  .switch[aria-checked="true"] .thumb {
    background: Highlight;
  }
}
```

- Check contrast in every theme, including dark mode (`prefers-color-scheme`), and declare `color-scheme: light dark` so native controls match.
- Optionally strengthen borders and text for `prefers-contrast: more`.
- Don't lock orientation (1.3.4), and use real text instead of images of text (1.4.5).
