# ARIA and focus

ARIA changes what assistive technology announces. It never adds behavior: `role="button"` doesn't make an element focusable or keyboard-operable. Keyboard and focus handling are always your code's job.

## Contents

- [Rules of ARIA use](#rules-of-aria-use)
- [Accessible names and descriptions](#accessible-names-and-descriptions)
- [Common states and properties](#common-states-and-properties)
- [Hiding content](#hiding-content)
- [Live regions](#live-regions)
- [Keyboard interaction](#keyboard-interaction)
- [Focus management](#focus-management)
- [Single-page apps and React](#single-page-apps-and-react)

## Rules of ARIA use

From the W3C note [Using ARIA](https://www.w3.org/TR/using-aria/):

1. If a native HTML element or attribute has the semantics and behavior you need, use it.
2. Don't change native semantics unless you must. For example, use `<div role="tab"><h2>Billing</h2></div>`, not `<h2 role="tab">`.
3. Make every interactive ARIA control keyboard-operable.
4. Never put `role="presentation"` or `aria-hidden="true"` on a visible, focusable element.
5. Give every interactive element an accessible name.

- Roles carry obligations: required states (`aria-checked` on `checkbox` and `switch`), required children (`tablist` owns `tab`, `listbox` owns `option`, `menu` owns `menuitem`), and required parents. Missing pieces break the widget.
- Check that each attribute is allowed on its role and each value is valid; [ARIA in HTML](https://www.w3.org/TR/html-aria/) lists what's allowed on each HTML element.
- Avoid `role="application"`; it turns off the screen reader's reading and navigation keys.
- Target [WAI-ARIA 1.2](https://www.w3.org/TR/wai-aria-1.2/), the current Recommendation. Check assistive-technology support before relying on draft ARIA 1.3 features such as `aria-description`.

## Accessible names and descriptions

Browsers compute the name in this order ([accname 1.2](https://www.w3.org/TR/accname-1.2/), simplified); the first match wins:

1. `aria-labelledby`: text of the referenced elements, in the order listed.
2. `aria-label`.
3. Native labeling: `<label>`, `alt`, `<caption>`, `<legend>`, `<figcaption>`.
4. Content, for roles that take their name from content (button, link, heading, tab, option, cell, menuitem).
5. `title`, then `placeholder`, as a last resort that many users never see.

- Prefer visible text, from the content or a `<label>`, so everyone gets the same name.
- `aria-label` and `aria-labelledby` replace the content. Keep the visible label text in the name, at the start (2.5.3).
- Disambiguate repeated controls by combining visible text: `<button id="del-104" aria-labelledby="del-104 inv-104">Delete</button>` is named "Delete INV-104" when `inv-104` is the row header.
- Keep names short. Don't include the role ("Close", not "Close button") or the state; assistive technology announces both separately.
- Don't name generic elements (`<div>` and `<span>` without a role), `presentation`, or `none`; the name is prohibited and usually ignored.
- Use `aria-describedby` for hints, formats, and errors. Descriptions are read after the name and role, and users can skip them, so put essential information in the name or visible text. Don't rely on `title`.
- Every ID referenced by `for`, `aria-labelledby`, `aria-describedby`, or `aria-controls` must exist exactly once. In reusable components, generate IDs (for example with React `useId()`).

## Common states and properties

| Attribute | Put it on | Notes |
|---|---|---|
| `aria-expanded` | The button that toggles content, not the content | `true` or `false`; omit it on controls that don't toggle anything |
| `aria-controls` | Toggles, tabs, comboboxes | ID of the controlled element; screen reader support is limited, so never rely on it alone |
| `aria-pressed` | Toggle buttons | Keep the label constant ("Mute") while the state changes |
| `aria-checked` | `checkbox`, `radio`, `switch`, `menuitemcheckbox`, `menuitemradio` roles | `mixed` for a tri-state checkbox |
| `aria-selected` | `tab`, `option`, and rows or cells in selectable grids | Not for navigation links; use `aria-current` |
| `aria-current` | The current item in a set: `page`, `step`, `location`, `date`, `time`, or `true` | Only one per set |
| `aria-disabled="true"` | Controls that stay focusable but inactive | Block activation in script; native `disabled` also removes the control from the Tab order |
| `aria-invalid="true"` | Fields that failed validation | Set only after validation, and link the message with `aria-describedby` |
| `required` or `aria-required="true"` | Required fields | Both expose "required"; native `required` also triggers browser validation |
| `aria-haspopup` | Buttons that open a `menu`, `listbox`, `tree`, `grid`, or `dialog` | `true` means `menu`; don't use it on disclosures |
| `aria-busy="true"` | A region being updated | Support varies; announce completion through a live region as well |
| `aria-modal="true"` | `dialog` or `alertdialog` | Only when the rest of the page is truly inert |
| `aria-sort` | The header cell of the sorted column | `ascending` or `descending`; only on the column currently sorted |
| `aria-activedescendant` | The focused container or input of a composite widget | ID of the visually active option; see [keyboard interaction](#keyboard-interaction) |
| `aria-owns` | Rarely needed | Fix the DOM order instead |

## Hiding content

| Technique | Visible | Exposed to assistive technology | Focusable | Use for |
|---|---|---|---|---|
| `hidden` attribute, `display: none`, `visibility: hidden` | No | No | No | Content hidden from everyone: closed panels, inactive views |
| `.visually-hidden` class | No | Yes | Yes, if focusable | Extra context for screen readers; skip links |
| `aria-hidden="true"` | Yes | No | Still focusable | Decorative icons and duplicated content; never on focusable elements or their ancestors |
| `inert` attribute | Yes | No | No | Background content behind a custom modal; off-screen drawers |
| `opacity: 0`, off-screen positioning, `height: 0` | No | Yes | Yes | Avoid for closed UI; keyboard users can tab into invisible controls |

```css
.visually-hidden:not(:focus):not(:active) {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip-path: inset(50%);
  white-space: nowrap;
}
```

## Live regions

- `role="status"` (polite) for success messages, progress, and result counts.
- `role="alert"` (assertive; interrupts) only for urgent, time-sensitive errors.
- `role="log"` for chat and activity feeds where new entries are appended.
- `aria-live="polite"` on a custom container, with `aria-atomic="true"` if the whole region should be read.

Rules:

- The region must be in the DOM, and not hidden with `display: none`, **before** its content changes. Content present at page load, or a region inserted together with its message, often isn't announced.
- To repeat the same message, clear the region and set the text again after a short delay.
- Keep messages short. Debounce updates such as search counts instead of announcing every keystroke.
- Don't wrap large areas such as forms or lists in a live region; announce a summary.
- Don't combine `role="alert"` with a wrapping `aria-live` region; that announces twice.
- When focus moves to the new content (an error summary, a dialog), a live region isn't needed.

```html
<div id="announcer" role="status" class="visually-hidden"></div>
```

```js
let announceTimer;

export function announce(message) {
  const region = document.getElementById('announcer');
  region.textContent = '';
  clearTimeout(announceTimer);
  announceTimer = setTimeout(() => {
    region.textContent = message;
  }, 100);
}
```

In React, render the region once in the app shell and keep its text in state or a store; never mount it together with its message.

## Keyboard interaction

| Key | Expected behavior |
|---|---|
| Tab, Shift+Tab | Move between interactive elements and into or out of composite widgets |
| Enter | Activate links and buttons; submit a form from a text field |
| Space | Activate buttons; toggle checkboxes and switches |
| Arrow keys | Move within composite widgets: radio groups, tabs, menus, listboxes, grids, sliders, toolbars |
| Home, End | First or last item in a composite widget |
| Escape | Close dialogs, menus, popovers, and tooltips; cancel a drag |
| Page Up, Page Down | Larger steps in sliders and long lists |

- `tabindex="0"` adds a custom control to the Tab order; prefer a native element instead.
- `tabindex="-1"` makes an element focusable by script only: dialog headings, error summaries, inactive items in a roving group.
- Never use a positive `tabindex`.
- Don't add `tabindex="0"` to static text "so screen readers can read it"; they read it with their own commands.

Composite widgets take one Tab stop and use arrow keys inside:

- **Roving tabindex**: the active item has `tabindex="0"` and the rest `-1`; arrow keys move `tabindex="0"` and call `focus()`. Use for tabs, toolbars, menus, and radio-like groups.
- **`aria-activedescendant`**: DOM focus stays on the container or text input, which references the active option's ID; style that option as active. Use for comboboxes and listboxes where focus must stay in a text input.

Single-character shortcuts must be possible to turn off or remap, or work only while their component has focus (2.1.4). Don't override browser or screen reader shortcuts.

## Focus management

- Move focus only in response to a user action, when the context changes: opening, closing, deleting, or navigating.
- Give non-interactive focus targets `tabindex="-1"`, and choose meaningful targets such as a heading or a named container.
- Never let focus fall back to `<body>` because the focused element was removed or hidden.
- Return focus when temporary UI closes.

| Situation | Move focus to | Notes |
|---|---|---|
| Modal dialog opens | The first focusable element; a static element at the start for long content; the least destructive button for destructive confirmations | `showModal()` focuses the first focusable element; set `autofocus` to choose another |
| Modal dialog closes | The element that opened it | If that element is gone, the next logical element |
| Menu opens from its button | The first menu item | Escape closes the menu and returns focus to the button |
| Disclosure or accordion opens | Keep focus on the button | The revealed content follows the button in the DOM |
| Item deleted from a list | The next item, else the previous one, else the list heading | Announce "Deleted <item name>" |
| "Load more" | The first new item, or keep focus and announce the count | |
| Form submitted with errors | The error summary (`tabindex="-1"`) or the first invalid field | |
| Single-page-app route change | The new page's `<h1>` (`tabindex="-1"`) or `<main>` | Update `document.title` first |
| Filter or sort in place | Keep focus on the control | Announce the new result count |
| Toast or notification | Don't move focus | Announce through a live region; make its actions reachable without time pressure |

## Single-page apps and React

- **Route changes**: update `document.title`, then move focus to the new `<h1>` or announce the page, and restore scroll position. Check what the router already does: Next.js announces route changes, while many routers do nothing; avoid double announcements.

```js
export function focusPageHeading() {
  const heading = document.querySelector('main h1');
  if (!heading) return;
  heading.setAttribute('tabindex', '-1');
  heading.focus();
}
```

- **JSX attributes**: `htmlFor`, `className`, and `tabIndex`; `role` and `aria-*` keep their HTML spelling. `aria-expanded={isOpen}` renders `"true"` or `"false"`.
- **IDs**: generate label and description IDs with `useId()`; never hard-code IDs in reusable components.
- **Valid structure**: use fragments instead of wrapper `<div>` elements that break `<ul>`/`<li>` and table structure.
- **Focus timing**: move focus with refs in an effect after the target renders, never during render.
- **Portals**: a modal rendered into `document.body` needs focus moved in and back, and the app root made `inert` while it's open. React 19 treats `inert` as a boolean (`inert={true}`); the React 18 workaround `inert=""` now means false. Prefer `<dialog>` with `showModal()`.
- **Hiding**: unmounted content can't be focused; CSS-only hiding of focusable content can.
- **`autoFocus`**: fine for the first field of a dialog or a dedicated search page; avoid on general page load.
- **Component APIs**: forward refs, `id`, and `aria-*` props so consumers can label and describe components.
- **Other frameworks**: Vue projects can lint with `eslint-plugin-vuejs-accessibility`; Angular CDK's `a11y` module provides `FocusTrap`, `LiveAnnouncer`, and `FocusMonitor`.
