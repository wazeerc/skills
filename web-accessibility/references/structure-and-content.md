# Structure and content

Semantic structure lets assistive-technology users understand and navigate a page the way sighted users scan it.

## Contents

- [Document basics](#document-basics)
- [Landmarks and skip link](#landmarks-and-skip-link)
- [Headings](#headings)
- [Lists and tables](#lists-and-tables)
- [Links and buttons](#links-and-buttons)
- [Images, icons, and SVG](#images-icons-and-svg)
- [Media](#media)
- [Reading order and layout](#reading-order-and-layout)
- [Readable content](#readable-content)

## Document basics

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Edit profile - Settings - Acme</title>
</head>
<body>
  ...
</body>
</html>
```

- `lang` holds a valid language tag that matches the content (3.1.1). Mark passages in other languages with their own `lang` (3.1.2), and set `dir="rtl"` for right-to-left text.
- Never add `user-scalable=no` or `maximum-scale=1` to the viewport (1.4.4).
- Put the specific part of the title first and the site name last. Make titles unique per page and update them on every single-page-app route change (2.4.2).
- Give every `<iframe>` a `title` that describes its content (4.1.2).

## Landmarks and skip link

| HTML | Exposed role | Notes |
|---|---|---|
| `<header>` at the top level | `banner` | Not a landmark when nested in `<article>`, `<aside>`, `<main>`, `<nav>`, or `<section>` |
| `<nav>` | `navigation` | Name it when there's more than one, for example `aria-label="Main"` or `"Breadcrumb"` |
| `<main>` | `main` | Exactly one visible per page; the skip-link target |
| `<aside>` | `complementary` | Not a landmark when nested in `<article>`, `<aside>`, `<nav>`, or `<section>` without an accessible name |
| `<footer>` at the top level | `contentinfo` | Not a landmark when nested in `<article>`, `<aside>`, `<main>`, `<nav>`, or `<section>` |
| `<search>`, or `<form role="search">` | `search` | Site or page search |
| `<form>` with an accessible name | `form` | Only named forms are landmarks |
| `<section>` with an accessible name | `region` | Only for areas worth jumping to |

- Keep all visible content inside landmarks (best practice).
- Don't repeat the role in a label: screen readers already say "navigation", so use `aria-label="Main"`, not `"Main navigation"`.

```html
<body>
  <a class="skip-link" href="#main">Skip to main content</a>
  <header>
    <a href="/">Acme home</a>
    <nav aria-label="Main">...</nav>
  </header>
  <main id="main" tabindex="-1">
    <h1>Edit profile</h1>
  </main>
  <footer>...</footer>
</body>
```

```css
.skip-link {
  position: absolute;
  inset-block-start: 0.5rem;
  inset-inline-start: 0.5rem;
  z-index: 1000;
  padding: 0.75rem 1rem;
  background: #fff;
  color: #1a1a1a;
}

.skip-link:not(:focus) {
  clip-path: inset(50%);
  width: 1px;
  height: 1px;
  overflow: hidden;
  white-space: nowrap;
}
```

`tabindex="-1"` on the target lets scripts and older browsers move focus to it without adding a Tab stop.

## Headings

- Use one `<h1>` that describes the page (a convention, not a WCAG rule).
- Choose levels by structure and style them with CSS. Skipped levels aren't a WCAG failure, but they confuse navigation (best practice).
- Give each major section a heading. A visually hidden heading is fine when the section's purpose is visually obvious, such as "Filters".
- Never use headings only for visual emphasis, and never leave them empty.
- In reusable components, let consumers set the heading level (for example a `level` or `as` prop) so the outline fits each page.

## Lists and tables

- Mark up groups of related items, including navigation links, as `<ul>`, `<ol>`, or `<dl>`.
- Safari with VoiceOver drops list semantics for lists styled with `list-style: none` outside `<nav>`. Add `role="list"` when the item count matters to users.

```html
<table>
  <caption>Invoices, March 2026</caption>
  <thead>
    <tr>
      <th scope="col">Invoice</th>
      <th scope="col">Date</th>
      <th scope="col">Amount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">INV-104</th>
      <td>3 March 2026</td>
      <td>€120.00</td>
    </tr>
  </tbody>
</table>
```

- Name tables with `<caption>` or `aria-labelledby`. Use `scope` for simple headers, and `id` with `headers` for multi-level headers, or split the data into simpler tables.
- Don't use tables for layout. If one can't be avoided, add `role="presentation"`.
- Wrap wide tables in a focusable, named scroll container so keyboard users can scroll them: `<div role="region" aria-labelledby="invoices-caption" tabindex="0">`.
- Changing `display` on table elements can remove table semantics in some browsers. Verify in the accessibility tree and restore with ARIA table roles if needed.
- Use the APG [grid pattern](https://www.w3.org/WAI/ARIA/apg/patterns/grid/) only for interactive, spreadsheet-like widgets, never for static data. Sortable columns are covered in [component patterns](component-patterns.md#sortable-table).

## Links and buttons

- Links go somewhere (`<a href>`); buttons do something (`<button>`). Styling doesn't change semantics, so a link styled as a button is still a link.
- Link text describes the destination and is distinct for different destinations on the same page. Avoid bare "Click here" and "Read more". If the design needs short visible text, extend the name with [visually hidden](aria-and-focus.md#hiding-content) text and keep the visible words first (2.5.3):

```html
<a href="/reports/2026">Read more<span class="visually-hidden"> about the 2026 annual report</span></a>
```

- Say when a link opens a new tab, downloads a file, or leaves the site, for example "Annual report (PDF, 2 MB)" (best practice).
- Never nest interactive elements, such as a button inside a link (4.1.2).
- For clickable cards, make the heading the link and stretch its hit area with a pseudo-element, instead of wrapping the whole card in `<a>` and creating a long, verbose name:

```css
.card { position: relative; }
.card h3 a::after { content: ""; position: absolute; inset: 0; }
```

- Mark the current item in navigation, breadcrumbs, and pagination with `aria-current="page"`.
- Give buttons `type="button"` unless they submit a form. Name icon-only buttons with visually hidden text or `aria-label`.

## Images, icons, and SVG

Choose the text alternative by the image's role (see the W3C [alt decision tree](https://www.w3.org/WAI/tutorials/images/decision-tree/)).

| Image | Text alternative |
|---|---|
| Informative (photo or illustration that conveys information) | Concise `alt` with the information that matters in context; no "image of" |
| Decorative, or redundant with adjacent text | `alt=""` (never omit `alt`), a CSS background, or `aria-hidden="true"` on inline SVG |
| Functional (the only content of a link or button) | The action or destination, such as "Search" or "Acme home" |
| Image of text | The same text, and prefer real text (1.4.5) |
| Complex (chart, diagram, map) | Short `alt` plus a long description nearby: visible text, a data table, or `<details>` |
| Group conveying one idea (such as a star rating) | `alt` on one image ("4 out of 5 stars") and `alt=""` on the rest |

```html
<img src="team.jpg" alt="Five engineers reviewing a diagram on a whiteboard">
<img src="divider.svg" alt="">
<a href="/"><img src="logo.svg" alt="Acme home"></a>

<figure>
  <img src="q3-revenue.png" alt="Bar chart of Q3 revenue by region, detailed in the table below">
  <figcaption>Q3 revenue by region</figcaption>
</figure>
```

Informative inline SVG:

```html
<svg role="img" aria-labelledby="signups-title" viewBox="0 0 120 60">
  <title id="signups-title">Sign-ups rose 40% from January to March</title>
</svg>
```

Decorative icon next to visible text, then an icon-only button:

```html
<button type="button">
  <svg aria-hidden="true" viewBox="0 0 24 24">...</svg>
  Delete
</button>

<button type="button" aria-label="Delete invoice INV-104">
  <svg aria-hidden="true" viewBox="0 0 24 24">...</svg>
</button>
```

- Hide icon-font glyphs and decorative emoji with `aria-hidden="true"` and provide real text.
- Use CSS background images only for decoration; informative images need markup with a text alternative.
- Review AI-generated `alt` text; it often misses context or states details that don't matter.

## Media

- **Captions**: `<track kind="captions" srclang="en" label="English" src="captions.vtt" default>`, accurate and synchronized, with speaker changes and meaningful sounds (1.2.2, 1.2.4).
- **Transcripts**: next to the player for audio and video. They also serve deafblind and low-bandwidth users (1.2.1, 1.2.3).
- **Audio description**: describe essential visuals in the main soundtrack or offer a described version (1.2.5). Browser support for `<track kind="descriptions">` is limited, so don't rely on it alone.
- **Player**: keyboard-operable, named controls with visible focus and a captions toggle.
- **Autoplay**: no autoplaying sound longer than 3 seconds (1.4.2). Silent background video longer than 5 seconds needs a pause control (2.2.2) and should respect `prefers-reduced-motion` (best practice).

## Reading order and layout

- DOM order sets the reading and focus order. Reordering with `order`, `row-reverse` or `column-reverse`, `grid-auto-flow: dense`, explicit grid placement, or absolute positioning can make focus jump around (1.3.2, 2.4.3).
- Place content revealed by a control right after the control in the DOM, or move focus to it.
- Content rendered elsewhere in the DOM, such as portals for modals and popovers, needs explicit focus management (see [ARIA and focus](aria-and-focus.md#focus-management)).

## Readable content

- Use plain language and short sentences, put the key information first, and explain jargon and abbreviations (3.1.3 and 3.1.4 at AAA; good practice everywhere).
- Keep instructions independent of sensory cues, for example "Select **Continue**" rather than "Select the green button on the right" (1.3.3).
- For cognitive accessibility, see W3C [Making Content Usable for People with Cognitive and Learning Disabilities](https://www.w3.org/TR/coga-usable/).
