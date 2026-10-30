---
name: web-accessibility
description: 'Build, review, fix, and test web user interfaces for accessibility against WCAG 2.2 Level A and AA, using semantic HTML, the WAI-ARIA Authoring Practices, keyboard and focus management, and automated plus manual checks. Use when: accessibility or a11y work; WCAG 2.2 conformance; accessibility audit or code review; screen reader or keyboard support; focus management; color contrast; accessible forms, error messages, or login; ARIA roles, names, and states; dialogs, menus, tabs, or other custom widgets; axe, jest-axe, cypress-axe, Playwright axe, Storybook a11y, or Lighthouse accessibility findings.'
argument-hint: 'Files, component, URL, or user flow; optional mode (build, review, fix, test) and target (default WCAG 2.2 AA)'
---

# Web Accessibility

Make web interfaces usable by people with disabilities, and show the evidence. The default target is **WCAG 2.2 Level AA**; content that meets it also meets WCAG 2.1 and 2.0 AA.

This is a general baseline. Repository instructions (`AGENTS.md`, `.github/copilot-instructions.md`), design-system documentation, and project-specific accessibility skills take precedence for conventions and tooling. They may raise the target; lower it only on an explicit project decision.

## Start

1. **Mode**: Build (new UI), Review (audit code or a running page), Fix (remediate findings), or Test (add automated checks). When a request combines modes, review before fixing and test after fixing.
2. **Scope**: files, components, routes, or user flows, plus the states to cover: open and closed overlays, errors, loading, empty, and narrow viewports.
3. **Stack and tooling**: framework, component library or design system, and existing accessibility tooling (`eslint-plugin-jsx-a11y`, `jest-axe` or `vitest-axe`, `cypress-axe`, `@axe-core/playwright`, Storybook a11y addon, Lighthouse CI). Reuse what exists. Add new tools only when the user asks or agrees.
4. **References**: load only what the task needs.

| Reference | Read when |
|---|---|
| [WCAG 2.2 checklist](references/wcag-22-checklist.md) | Auditing, mapping a finding to a success criterion (SC), conformance questions |
| [Structure and content](references/structure-and-content.md) | Page title, language, landmarks, headings, lists, tables, links and buttons, images, SVG, media |
| [ARIA and focus](references/aria-and-focus.md) | Accessible names, roles, states, hiding content, live regions, keyboard and focus management, SPA routing |
| [Component patterns](references/component-patterns.md) | Dialogs, disclosures, accordions, tabs, menus, comboboxes, tooltips, toasts, carousels, sortable tables, drag and drop |
| [Forms and authentication](references/forms-and-auth.md) | Labels, hints, validation, error messages, autocomplete, login, multi-step flows, timeouts |
| [Visual design](references/visual-design.md) | Contrast, use of color, focus indicators, target size, zoom, reflow, text spacing, motion, forced colors |
| [Testing](references/testing.md) | Automated tools and recipes, reading axe and Lighthouse output, browser checks, manual and screen reader testing |

## Definition of done

Apply to every UI change. The [checklist](references/wcag-22-checklist.md) maps each item to its success criteria. Apply each criterion's scope and exceptions; this is a review aid, not a blanket rule for every component. Items marked *(best practice)* go beyond WCAG 2.2 AA; report gaps in them as best practice, not as failures.

1. **Semantics**: native elements for their purpose (`<button>` for actions, `<a href>` for navigation, form controls, lists, `<table>`), landmarks, and a logical heading outline. DOM order matches the visual and reading order.
2. **Names**: every control, image, frame, and landmark that needs a name has one, and it contains the visible label text. Informative images have concise `alt`; decorative images have `alt=""`.
3. **Keyboard**: everything works with a keyboard alone. Tab reaches each control in a logical order; Enter, Space, arrow keys, and Escape behave as the pattern defines; there are no traps. No positive `tabindex` *(best practice)*.
4. **Focus**: keyboard focus is visible (2.4.7); visual information needed to identify the focused state meets the 3:1 contrast requirement where 1.4.11 applies; and focused components are not entirely hidden by author-created content (2.4.11). Focus moves deliberately when UI opens, closes, or disappears, and returns to a logical place.
5. **States**: expanded, selected, checked, pressed, current, invalid, busy, and disabled states are exposed programmatically, not only visually.
6. **Feedback**: errors are identified in text, associated with their fields, and summarized on submit. Status messages that don't move focus are announced by a live region that exists before the update.
7. **Visual**: text contrast is at least 4.5:1, or 3:1 for large text (at least 24 CSS px, or 18.66 CSS px bold). Visual information needed to identify components and their states, and meaningful parts of graphics, meets 3:1 where 1.4.11 applies. Color is never the only cue.
8. **Adaptable**: usable at 200% text size, at 320 CSS px wide without two-dimensional scrolling except for content that requires it, with increased text spacing, and in either orientation unless orientation is essential. Zoom is never disabled. Works in forced-colors mode *(best practice)*.
9. **Pointer**: targets meet the 24 × 24 CSS px minimum or spacing alternative, subject to the exceptions in 2.5.8. Dragging and path-based or multi-touch gestures have single-pointer alternatives where required. Meet pointer cancellation (2.5.2); prefer click/up activation, and use down-event activation only where a criterion exception applies.
10. **Motion and time**: content that moves, blinks, or scrolls for more than 5 seconds, or auto-updates, can be paused, stopped, or hidden; nothing flashes more than three times per second; time limits can be turned off, adjusted, or extended. Large motion respects `prefers-reduced-motion` *(best practice)*.
11. **Forms and authentication**: visible labels and instructions, `autocomplete` on personal-data fields, no paste blocking, no memory or puzzle test without an alternative, and no re-entering information already provided in the same process.
12. **Consistency and language**: a descriptive `<title>` for every page and view; `lang` on `<html>` and on passages in another language; navigation, help, and labels consistent across pages.

## Choose the right element

| Need | Use | Avoid |
|---|---|---|
| Go to a page or location | `<a href="...">` | Button that sets `location`; `<a>` without `href`; `href="#"` |
| Perform an action | `<button type="button">` | `<div onclick>`, `<span role="button">`, `<a href="#">` |
| Submit a form | `<button type="submit">` inside `<form>` | Click handler on a non-button |
| On/off setting | `<input type="checkbox">`, or `<button aria-pressed>` for a toggle button | Visual-only "active" class |
| Show or hide a section | `<details>` and `<summary>`, or `<button aria-expanded aria-controls>` | Clickable heading `<div>` |
| One choice from a few | Radio group in `<fieldset>` with `<legend>` | Custom listbox |
| One choice from many | `<select>` | Custom dropdown, unless native can't meet the need |
| Modal | `<dialog>` opened with `showModal()` | `<div>` overlay without focus management or inert background |
| Tabular data | `<table>` with `<caption>` and `<th scope>` | CSS grid of `<div>` elements |
| Status message | `role="status"` (polite) or `role="alert"` (urgent) container rendered before updates | Live region inserted together with its message |

## Red flags to scan for

Search the code in scope for these leads (use multiline search, because JSX attributes often span lines), then confirm each one in context before reporting it.

- `<(div|span|li|td|img|svg|p)\b[^>]*\s(onClick|@click|v-on:click|\(click\))`: non-interactive element used as a control (2.1.1, 4.1.2).
- `href=["'](#|javascript:[^"']*)["']`: link used as a button (2.1.1, 4.1.2).
- `outline-none|outline["']?\s*:\s*["']?(none|0)\s*[;"'},!]`: focus indicator removed; confirm an equally visible replacement (2.4.7).
- `tab[iI]ndex=\{?["']?[1-9]`: positive `tabindex` breaks focus order (2.4.3).
- `user-scalable|maximum-scale`: confirm zoom isn't disabled (1.4.4).
- `aria-hidden`: confirm the hidden subtree has no focusable or needed content (1.3.1, 4.1.2).
- `<img\b`: confirm each image has an appropriate `alt` (1.1.1).
- `placeholder=`: confirm a visible label exists (3.3.2, 4.1.2).
- `<(div|span)\b[^>]*aria-label`: naming is prohibited on generic elements and often ignored; confirm the element has a role that supports a name (4.1.2).
- `role=["'](button|link|checkbox|tab|menu|menuitem|switch|option)`: confirm a native element can't be used, and that keyboard support and states are complete; `menu` must not be used for site navigation (4.1.2).
- `paste`: confirm paste isn't blocked in credential or code fields (3.3.8).
- `autoFocus|autofocus`: confirm the focus move is expected (2.4.3, 3.2.1).
- `setTimeout|setInterval` in toasts, carousels, or sessions: confirm users can pause, extend, or dismiss (2.2.1, 2.2.2).
- `:hover`: confirm the same information or styling is available on `:focus-visible`, and that hover content meets 1.4.13.
- `<(video|audio|iframe)\b`: confirm captions, transcripts, or audio description, no autoplaying sound, and a `title` on each iframe (1.2.x, 1.4.2, 4.1.2).
- `@keyframes|animation:|scroll-behavior:\s*smooth`: confirm auto-playing motion can be paused and large motion respects `prefers-reduced-motion` (2.2.2; 2.3.3 at AAA).

## Workflows

### Build

1. Use an existing accessible component from the design system or library when one fits. Follow its accessibility API (labels, `aria-*` props, focus handling) and don't override its semantics.
2. Otherwise start from native HTML. For a custom widget, implement the complete APG pattern from [component patterns](references/component-patterns.md): roles, states, keyboard, and focus.
3. Build names, states, keyboard support, focus management, error handling, and responsive behavior with the feature, not after it.
4. Add automated checks with the repository's existing tooling (see Test).
5. Self-review against the definition of done and the red flags, and list what still needs manual or screen reader verification.

### Review

1. Confirm the scope, target level, and states to cover.
2. **Static review**: read the components and the markup they render, walk the definition of done and the relevant checklist sections, and scan for red flags. Trace repeated issues to the shared component, token, or utility.
3. **Runtime review** (only when a running app is available and the user asked for or agreed to browser checks): automated scan for leads, then the accessibility tree, a keyboard walk-through, and checks at 320 CSS px, 200% zoom, increased text spacing, forced colors, and reduced motion. See [testing](references/testing.md).
4. Record each finding with evidence, location, success criterion, severity, and a concrete fix. Classify it as a **WCAG failure**, a **best practice**, or **needs verification**.
5. Report in the format below. Never call a page "compliant" or "accessible" based on automated results or code review alone; state what was tested and how.

### Fix

1. Locate the source (rule ID, selector, component, success criterion) and fix the root cause, in the shared component, token, or utility when the pattern repeats.
2. Prefer restoring native semantics and removing incorrect ARIA over adding more ARIA.
3. Keep visible labels and accessible names in sync, avoid duplicate announcements, and preserve keyboard behavior and role-based test selectors.
4. Verify with the check that found the issue plus a keyboard pass, and add a regression test where the repository supports it.
5. Report what changed, what was verified, and what still needs manual or assistive-technology testing.

### Test

1. Use the repository's test runner, accessibility tooling, and testing conventions.
2. **Component tests**: run axe on each meaningful state, query by role and accessible name, and assert keyboard behavior and focus.
3. **End-to-end tests**: scan key pages after each state change and drive critical journeys with the keyboard only.
4. **Lint**: use the framework's accessibility plugin when the repository has one.
5. Never disable rules, exclude elements, or weaken assertions to get a pass. Turning off page-level rules such as `region` in isolated component tests is the one routine exception (see [testing](references/testing.md#component-tests)). If another scoped exclusion is unavoidable, such as for a third-party widget, document the reason with a tracked TODO.

## Severity

Severity is user impact in context, not WCAG level. The names match axe impact levels so automated and manual findings sort together.

| Severity | Meaning | Typical examples |
|---|---|---|
| Critical | Blocks a task for some users or risks harm | Keyboard trap; control unreachable or unnamed in a core flow; focus left behind an open modal; flashing above the threshold |
| Serious | Major barrier with a hard or unreliable workaround | Body text below 4.5:1; no visible focus; errors shown only by color; errors or results not announced; content lost at 320 CSS px |
| Moderate | Real difficulty with a reasonable workaround | Vague link text in context; inconsistent labels for the same function; small, crowded targets |
| Minor | Friction or a best-practice gap, not a WCAG failure on its own | Skipped heading level; redundant `title`; link opens a new tab without warning |

## Report format

```markdown
## Accessibility report: <scope>
- Target: WCAG 2.2 AA
- Methods: <code review | tool and version | keyboard | screen reader and browser | 320 CSS px | 200% zoom | forced colors>
- Summary: <n> critical, <n> serious, <n> moderate, <n> minor; <n> need verification

| # | Severity | Type | WCAG SC | Location | Issue and impact | Fix | Evidence |
|---|---|---|---|---|---|---|---|
| 1 | Critical | Failure | 2.1.1 Keyboard (A) | `src/Card.tsx:42` | Details open only on mouse click; keyboard users can't reach them | Make the trigger a `<button>` | Code review; Tab skips the card |

### Needs verification
- <Check, how to perform it, and why it couldn't be decided>

### Not tested
- <Pages, states, browsers, or assistive technologies outside this review>
```

For Build and Fix work, report the changes made, the checks run, and the remaining manual checks.

## Sources

- [WCAG 2.2](https://www.w3.org/TR/WCAG22/) (W3C Recommendation, also ISO/IEC 40500:2025) and [Understanding WCAG 2.2](https://www.w3.org/WAI/WCAG22/Understanding/)
- [How to Meet WCAG 2.2 (Quick Reference)](https://www.w3.org/WAI/WCAG22/quickref/)
- [WAI-ARIA Authoring Practices Guide (APG)](https://www.w3.org/WAI/ARIA/apg/) and [ARIA in HTML](https://www.w3.org/TR/html-aria/)
- [W3C WAI tutorials](https://www.w3.org/WAI/tutorials/) and [MDN accessibility](https://developer.mozilla.org/en-US/docs/Web/Accessibility)
- WCAG 3 is an early draft; don't use it for conformance claims.
