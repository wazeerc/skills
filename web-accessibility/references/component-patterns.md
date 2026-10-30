# Component patterns

Patterns based on the [WAI-ARIA Authoring Practices Guide (APG)](https://www.w3.org/WAI/ARIA/apg/patterns/). Before building a custom widget, check whether a native element or a design-system component already does the job. APG examples demonstrate the patterns; test your implementation with a keyboard and at least one screen reader.

## Contents

- [Buttons, toggles, and switches](#buttons-toggles-and-switches)
- [Disclosure](#disclosure)
- [Accordion](#accordion)
- [Modal dialog](#modal-dialog)
- [Tabs](#tabs)
- [Menu button](#menu-button)
- [Navigation with submenus](#navigation-with-submenus)
- [Combobox and custom select](#combobox-and-custom-select)
- [Tooltip and toggletip](#tooltip-and-toggletip)
- [Toasts and notifications](#toasts-and-notifications)
- [Carousel](#carousel)
- [Sortable table](#sortable-table)
- [Loading and progress](#loading-and-progress)
- [Drag and drop alternatives](#drag-and-drop-alternatives)
- [Breadcrumbs and pagination](#breadcrumbs-and-pagination)

## Buttons, toggles, and switches

```html
<button type="button">Save</button>
<button type="button" aria-pressed="false">Mute</button>
<button type="button" role="switch" aria-checked="true">Email notifications</button>
```

- Native buttons handle Enter, Space, focus, and the role. A custom `role="button"` needs `tabindex="0"`, Enter and Space handlers, and disabled handling; avoid it.
- Toggle buttons flip `aria-pressed` and keep the same label. If the label changes ("Play" to "Pause"), don't also use `aria-pressed`.
- Use a switch for on/off settings that apply immediately; inside forms that are submitted later, use a checkbox.
- Native `disabled` removes the button from the Tab order, so users can't discover it or learn why it's unavailable. When that matters, use `aria-disabled="true"`, ignore activation, and explain the reason.
- Don't set `disabled` on the focused button while a request runs; some browsers then drop focus to the page. Use `aria-disabled="true"` and ignore repeat clicks.
- APG: [Button](https://www.w3.org/WAI/ARIA/apg/patterns/button/), [Switch](https://www.w3.org/WAI/ARIA/apg/patterns/switch/).

## Disclosure

Native, with no script:

```html
<details>
  <summary>Shipping options</summary>
  <p>Standard delivery takes 3 to 5 working days.</p>
</details>
```

Custom:

```html
<button type="button" aria-expanded="false" aria-controls="shipping-panel">Shipping options</button>
<div id="shipping-panel" hidden>
  <p>Standard delivery takes 3 to 5 working days.</p>
</div>
```

```js
toggle.addEventListener('click', () => {
  const expanded = toggle.getAttribute('aria-expanded') === 'true';
  toggle.setAttribute('aria-expanded', String(!expanded));
  panel.hidden = expanded;
});
```

- Focus stays on the button; the panel follows it directly in the DOM.
- Put `aria-expanded` on the button, never on the panel, and don't add `aria-haspopup`.
- Headings inside `<summary>` can lose their heading semantics in some browser and screen reader combinations; use the accordion pattern when sections need headings.
- APG: [Disclosure](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/).

## Accordion

```html
<h3>
  <button type="button" id="personal-btn" aria-expanded="true" aria-controls="personal-panel">Personal details</button>
</h3>
<section id="personal-panel" aria-labelledby="personal-btn">...</section>

<h3>
  <button type="button" id="address-btn" aria-expanded="false" aria-controls="address-panel">Address</button>
</h3>
<section id="address-panel" aria-labelledby="address-btn" hidden>...</section>
```

- Put the button inside the heading, never the heading inside the button.
- Enter and Space toggle a section; Tab moves through headers and panel content. Down Arrow, Up Arrow, Home, and End between headers are optional.
- A named `<section>` (exposed as a `region` landmark) helps with a few panels; use a plain `<div>` when many panels can be open at once, to avoid landmark noise.
- Let several sections stay open unless there's a strong reason not to.
- APG: [Accordion](https://www.w3.org/WAI/ARIA/apg/patterns/accordion/).

## Modal dialog

Use `<dialog>` with `showModal()`:

```html
<button type="button" id="open-delete">Delete project</button>

<dialog id="delete-dialog" aria-labelledby="delete-title" aria-describedby="delete-desc">
  <h2 id="delete-title">Delete project?</h2>
  <p id="delete-desc">This removes all 14 files. You can't undo this.</p>
  <form method="dialog">
    <button type="submit" value="cancel" autofocus>Cancel</button>
    <button type="submit" value="delete">Delete project</button>
  </form>
</dialog>
```

```js
const dialog = document.getElementById('delete-dialog');

document.getElementById('open-delete').addEventListener('click', () => {
  dialog.returnValue = '';
  dialog.showModal();
});

dialog.addEventListener('close', () => {
  if (dialog.returnValue === 'delete') deleteProject();
});
```

- `showModal()` makes the rest of the page inert, keeps Tab inside the dialog (and the browser's own controls), closes on Escape, and exposes the dialog as modal. Under the HTML specification, closing returns focus to the element that was focused before it opened.
- Reset `returnValue` before opening: closing with Escape leaves the previous value in place.
- Choose initial focus with `autofocus`: the least destructive action for destructive confirmations, the first field for forms, or a static element with `tabindex="-1"` at the start of long content.
- Name the dialog with `aria-labelledby`, and include a visible close or cancel button.
- Don't add `tabindex` to the `<dialog>` element itself.
- Use `role="alertdialog"` for urgent confirmations that need a response.
- Without `<dialog>`, a custom modal needs `role="dialog"`, `aria-modal="true"`, a name, focus moved in, Tab kept inside, Escape to close, the background made `inert`, and focus returned on close.
- Non-modal dialogs (`show()`) don't trap focus, but still close on Escape and return focus.
- APG: [Dialog (Modal)](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/), [Alert and Message Dialogs](https://www.w3.org/WAI/ARIA/apg/patterns/alertdialog/).

## Tabs

```html
<div role="tablist" aria-label="Account settings">
  <button type="button" role="tab" id="tab-profile" aria-selected="true" aria-controls="panel-profile">Profile</button>
  <button type="button" role="tab" id="tab-security" aria-selected="false" aria-controls="panel-security" tabindex="-1">Security</button>
</div>
<div role="tabpanel" id="panel-profile" aria-labelledby="tab-profile" tabindex="0">...</div>
<div role="tabpanel" id="panel-security" aria-labelledby="tab-security" tabindex="0" hidden>...</div>
```

```js
const tabs = [...tablist.querySelectorAll('[role="tab"]')];

function selectTab(tab) {
  for (const candidate of tabs) {
    const selected = candidate === tab;
    candidate.setAttribute('aria-selected', String(selected));
    candidate.tabIndex = selected ? 0 : -1;
    document.getElementById(candidate.getAttribute('aria-controls')).hidden = !selected;
  }
  tab.focus();
}

tablist.addEventListener('click', (event) => {
  const tab = event.target.closest('[role="tab"]');
  if (tab) selectTab(tab);
});

tablist.addEventListener('keydown', (event) => {
  const index = tabs.indexOf(document.activeElement);
  if (index === -1) return;
  const target = {
    ArrowRight: tabs[(index + 1) % tabs.length],
    ArrowLeft: tabs[(index - 1 + tabs.length) % tabs.length],
    Home: tabs[0],
    End: tabs[tabs.length - 1],
  }[event.key];
  if (!target) return;
  event.preventDefault();
  selectTab(target);
});
```

- Tab moves into the tab list at the selected tab and then to the panel. Give a panel `tabindex="0"` when it has no focusable content at its start.
- The example uses automatic activation (arrows select). If showing a panel is slow, use manual activation: arrows only move focus, and Enter or Space selects.
- For vertical tabs, add `aria-orientation="vertical"` and use Up and Down arrows.
- Tabs aren't site navigation; for pages, use links with `aria-current="page"`.
- APG: [Tabs](https://www.w3.org/WAI/ARIA/apg/patterns/tabs/).

## Menu button

For application-style action menus such as "More actions", not for navigation.

```html
<button type="button" id="actions-btn" aria-haspopup="menu" aria-expanded="false" aria-controls="actions-menu">Actions</button>
<ul id="actions-menu" role="menu" aria-labelledby="actions-btn" hidden>
  <li role="menuitem" tabindex="-1">Rename</li>
  <li role="menuitem" tabindex="-1">Duplicate</li>
  <li role="menuitem" tabindex="-1">Delete</li>
</ul>
```

| Key | Behavior |
|---|---|
| Enter, Space, Down Arrow on the button | Open the menu and focus the first item |
| Up Arrow on the button (optional) | Open the menu and focus the last item |
| Down Arrow, Up Arrow | Move between items |
| Home, End | First or last item |
| Enter | Activate the item and close the menu |
| Escape | Close the menu and return focus to the button |
| Tab | Close the menu and continue to the next element |
| Printable character (optional) | Move to the next item starting with that character |

- `role="menu"` tells screen readers to expect this whole keyboard model. For a few simple actions, a disclosure that reveals ordinary buttons is easier to get right.
- APG: [Menu Button](https://www.w3.org/WAI/ARIA/apg/patterns/menu-button/), [Menu and Menubar](https://www.w3.org/WAI/ARIA/apg/patterns/menubar/).

## Navigation with submenus

```html
<nav aria-label="Main">
  <ul>
    <li><a href="/" aria-current="page">Home</a></li>
    <li>
      <button type="button" aria-expanded="false" aria-controls="products-submenu">Products</button>
      <ul id="products-submenu" hidden>
        <li><a href="/products/payroll">Payroll</a></li>
        <li><a href="/products/time">Time tracking</a></li>
      </ul>
    </li>
  </ul>
</nav>
```

- Links stay links, and each submenu toggle is a disclosure button. Don't use `role="menu"` or `menubar` for site navigation; APG's [disclosure navigation example](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/examples/disclosure-navigation/) explains why.
- Escape closes the open submenu and returns focus to its button.
- Don't open submenus on hover alone. If hover opens them, also meet 1.4.13 and keep the button for keyboard and touch users.
- If a top-level item is both a page and a submenu parent, provide a link and a separate toggle button, such as "Show Products submenu".

## Combobox and custom select

Prefer native `<select>`. An `<input>` with `<datalist>` gives simple suggestions, but its rendering and screen reader support vary, so test it in the target browsers.

A custom editable combobox follows the APG pattern; focus stays in the input:

```html
<label for="city">City</label>
<input id="city" type="text" role="combobox" aria-autocomplete="list"
       aria-expanded="true" aria-controls="city-options" aria-activedescendant="city-option-2">
<ul id="city-options" role="listbox" aria-label="Cities">
  <li id="city-option-1" role="option" aria-selected="false">Dubai</li>
  <li id="city-option-2" role="option" aria-selected="true">Dublin</li>
</ul>
<div role="status" class="visually-hidden">2 results available</div>
```

- Down Arrow opens the list or moves to the next option; Up Arrow moves back; Enter accepts; Escape closes the list, and a second Escape may clear the input. Typing filters the options.
- Keep `aria-expanded` and `aria-activedescendant` in sync with what's shown, and announce the result count politely.
- Select-only custom dropdowns are hard to get right; use a well-tested library component or native `<select>`. Customizable native selects (`appearance: base-select`) are arriving in browsers; check support before relying on them.
- APG: [Combobox](https://www.w3.org/WAI/ARIA/apg/patterns/combobox/), [Listbox](https://www.w3.org/WAI/ARIA/apg/patterns/listbox/).

## Tooltip and toggletip

A tooltip is a short, supplementary description shown on hover **and** focus:

```html
<button type="button" aria-describedby="save-tip">Save</button>
<div role="tooltip" id="save-tip" hidden>Saves a draft without publishing</div>
```

- Show it on hover and focus. Hide it on Escape, blur, and pointer leave, but keep it open while the pointer is over the tooltip (1.4.13).
- Tooltips never contain interactive content or essential information, and never receive focus. For interactive content, use a non-modal dialog or a disclosure.
- For an icon-only button whose tooltip shows its name, reference the tooltip with `aria-labelledby` instead of `aria-describedby`.
- Disabled buttons can't receive focus, so keyboard users never see their tooltips; explain unavailability in visible text.
- Native `title` tooltips don't appear on keyboard focus or touch; don't use them for anything important.
- The APG [Tooltip](https://www.w3.org/WAI/ARIA/apg/patterns/tooltip/) pattern is still a work in progress.

A **toggletip** shows information on click, such as an "i" button. Use the disclosure pattern, with the content right after the button, or the `popover` attribute (`<button popovertarget="vat-info">` with `<div id="vat-info" popover>`), and test both keyboard and screen reader behavior.

## Toasts and notifications

- Keep one live region container rendered at all times and inject messages into it: `role="status"` for information and success, `role="alert"` for errors.
- Don't move focus to a toast.
- Don't auto-dismiss messages that users must read or act on, or pause the timer on hover and focus and allow enough time (2.2.1). Toasts with actions such as "Undo" need a non-timed way to reach the action.
- Show form errors inline next to their fields (3.3.1); a toast alone isn't enough.
- Give the close button a name, and keep toasts from covering the focused element (2.4.11).

## Carousel

- Avoid auto-rotation. If slides rotate, provide a pause button (first in the carousel's Tab order), pause while the pointer hovers over the carousel, and stop when keyboard focus enters it, resuming only when the user activates the rotation control (2.2.2).
- Name the previous, next, and slide-picker buttons, and offer them as an alternative to swiping (2.5.1).
- Container: `<section aria-roledescription="carousel" aria-label="Featured products">`. Each slide: `role="group"`, `aria-roledescription="slide"`, and `aria-label="2 of 5"`.
- Set the slide wrapper to `aria-live="off"` while rotating and `"polite"` when stopped.
- Hide inactive slides with `hidden` or `inert` so keyboard users can't tab into them.
- APG: [Carousel](https://www.w3.org/WAI/ARIA/apg/patterns/carousel/).

## Sortable table

```html
<th scope="col" aria-sort="ascending">
  <button type="button">Date</button>
</th>
<th scope="col">
  <button type="button">Amount</button>
</th>
```

- Put a button in each sortable header cell, and `aria-sort` only on the currently sorted column.
- Hide decorative sort icons with `aria-hidden="true"`; `aria-sort` carries the state.
- Screen reader support for `aria-sort` changes varies, so also announce "Sorted by date, ascending" through a live region.
- APG: [Table](https://www.w3.org/WAI/ARIA/apg/patterns/table/).

## Loading and progress

- Announce the start ("Loading invoices") and the result ("12 invoices loaded") through a live region, not every intermediate step.
- `aria-busy="true"` on the updating region can delay announcements, but support varies; don't rely on it alone.
- Hide skeleton placeholders with `aria-hidden="true"` and keep a status message.
- For determinate progress, use a labeled `<progress value="40" max="100">` and announce milestones sparingly.
- Keep focus on, or return it to, a logical element when loaded content replaces the control that started the load.

## Drag and drop alternatives

Every drag action needs a single-pointer alternative (2.5.7) and a keyboard path (2.1.1):

```html
<ol>
  <li>
    <span id="task-3">Write report</span>
    <button type="button" id="task-3-up" aria-labelledby="task-3-up task-3">Move up</button>
    <button type="button" id="task-3-down" aria-labelledby="task-3-down task-3">Move down</button>
  </li>
</ol>
```

- The names become "Move up Write report" and "Move down Write report": unique, and they keep the visible label first (2.5.3).
- Other alternatives: a "Move to" menu, click to select and click the destination, or a number input for the position.
- After a move, keep focus on the moved item's control and announce the new position, for example "Write report moved to position 2 of 8".
- When a library provides keyboard dragging (for example Space to lift, arrows to move, Space to drop, Escape to cancel), confirm it also provides screen reader announcements and the pointer alternative above.

## Breadcrumbs and pagination

```html
<nav aria-label="Breadcrumb">
  <ol>
    <li><a href="/">Home</a></li>
    <li><a href="/settings">Settings</a></li>
    <li><a href="/settings/profile" aria-current="page">Profile</a></li>
  </ol>
</nav>
```

- Draw separators with CSS so screen readers don't read them.
- For pagination, use `<nav aria-label="Pagination">`, name page links such as "Page 3" (the visible "3" stays in the name), mark the current page with `aria-current="page"`, and keep targets at least 24 × 24 CSS px.
- APG: [Breadcrumb](https://www.w3.org/WAI/ARIA/apg/patterns/breadcrumb/).
