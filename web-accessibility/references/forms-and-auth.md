# Forms and authentication

Forms are where most task-blocking barriers occur. Native controls with visible, associated labels solve most of them.

## Contents

- [Labels](#labels)
- [Groups](#groups)
- [Instructions and required fields](#instructions-and-required-fields)
- [Input purpose and autocomplete](#input-purpose-and-autocomplete)
- [Validation and error messages](#validation-and-error-messages)
- [Error prevention and redundant entry](#error-prevention-and-redundant-entry)
- [Accessible authentication](#accessible-authentication)
- [Time limits](#time-limits)
- [Multi-step forms](#multi-step-forms)
- [Custom and special controls](#custom-and-special-controls)

## Labels

```html
<label for="email">Email address</label>
<input id="email" name="email" type="email" autocomplete="email">

<label>
  <input type="checkbox" name="terms">
  I agree to the terms
</label>
```

- Every input has a visible label associated with `for`/`id` or by nesting (1.3.1, 3.3.2, 4.1.2).
- Placeholders aren't labels: they disappear on input, often fail contrast, and aren't reliably announced. Floating labels must stay visible and readable after typing.
- Put checkbox and radio labels after the control and keep them clickable.
- A search field next to a visible "Search" button may use `aria-label="Search"`; visually hidden labels are otherwise only for fields whose purpose is visually obvious.
- The accessible name must contain the visible label text (2.5.3).

## Groups

```html
<fieldset>
  <legend>Preferred contact method</legend>
  <label><input type="radio" name="contact" value="email"> Email</label>
  <label><input type="radio" name="contact" value="phone"> Phone</label>
</fieldset>
```

- Group radio buttons, related checkboxes, and multi-part fields (date parts, address blocks) with `<fieldset>` and a short `<legend>`, which many screen readers read with each control.
- If `<fieldset>` can't be used, use `role="group"` or `role="radiogroup"` with `aria-labelledby`.
- Native radio groups take one Tab stop, and arrow keys move the selection.

## Instructions and required fields

```html
<label for="dob">Date of birth</label>
<p id="dob-hint" class="hint">For example, 31/03/1990</p>
<input id="dob" name="dob" type="text" inputmode="numeric" autocomplete="bday" aria-describedby="dob-hint">
```

- Put hints and format examples between the label and the input, and associate them with `aria-describedby` (3.3.2).
- Indicate required fields in text: "Email (required)", or an asterisk explained before the form starts. If most fields are required, mark the optional ones instead.
- Add `required` (or `aria-required="true"`) so the state is announced. Hide a decorative asterisk with `aria-hidden="true"` when the state is already announced.
- Character counters: link the counter with `aria-describedby`, and announce the remaining count politely only near the limit, debounced.

## Input purpose and autocomplete

Fields that collect information about the user need the matching `autocomplete` token (1.3.5); it also lets browsers and password managers fill them in (3.3.8).

| Field | `autocomplete` | Notes |
|---|---|---|
| Full name, given name, family name | `name`, `given-name`, `family-name` | |
| Email | `email` | `type="email"` |
| Phone | `tel` | `type="tel"` |
| Organization | `organization` | |
| Street address | `street-address`, or `address-line1` and `address-line2` | |
| City, region, postal code, country | `address-level2`, `address-level1`, `postal-code`, `country-name` | Use `country` for country codes |
| Date of birth | `bday`, or `bday-day`, `bday-month`, `bday-year` | |
| Username | `username` | Also for email used as a login |
| Passwords | `current-password`, `new-password` | `type="password"` |
| One-time code | `one-time-code` | `inputmode="numeric"`; an HTML token that helps 3.3.8 |
| Payment card | `cc-name`, `cc-number`, `cc-exp`, `cc-csc` | `inputmode="numeric"` |

- Use `type="text"` with `inputmode="numeric"` for numbers that aren't quantities (card numbers, codes, postal codes); `type="number"` can alter values and changes on scroll.
- Don't set `autocomplete="off"` on fields about the user.

## Validation and error messages

Validate on submit. For format errors, you can also validate when the user leaves a field they've completed, but never on every keystroke or before they've finished. Remove an error as soon as the field becomes valid.

```html
<form id="signup" novalidate>
  <div id="error-summary" class="error-summary" tabindex="-1" hidden>
    <h2>There is a problem</h2>
    <ul></ul>
  </div>

  <label for="email">Email address</label>
  <p id="email-hint" class="hint">We'll send your receipt here.</p>
  <p id="email-error" class="error"></p>
  <input id="email" name="email" type="email" autocomplete="email" required
         aria-describedby="email-hint email-error">

  <button type="submit">Create account</button>
</form>
```

```js
const form = document.getElementById('signup');
const summary = document.getElementById('error-summary');

form.addEventListener('submit', (event) => {
  const errors = validate(form);
  renderFieldErrors(errors);
  if (errors.length === 0) {
    summary.hidden = true;
    return;
  }
  event.preventDefault();
  renderSummary(errors);
});

summary.addEventListener('click', (event) => {
  const link = event.target.closest('a[href^="#"]');
  if (!link) return;
  event.preventDefault();
  document.getElementById(link.hash.slice(1)).focus();
});

function renderFieldErrors(errors) {
  for (const input of form.elements) {
    const error = input.id && document.getElementById(`${input.id}-error`);
    if (!error) continue;
    const match = errors.find((item) => item.input === input);
    error.textContent = match ? match.message : '';
    if (match) input.setAttribute('aria-invalid', 'true');
    else input.removeAttribute('aria-invalid');
  }
}

function renderSummary(errors) {
  summary.querySelector('ul').replaceChildren(
    ...errors.map(({ input, message }) => {
      const link = document.createElement('a');
      link.href = `#${input.id}`;
      link.textContent = message;
      const item = document.createElement('li');
      item.append(link);
      return item;
    }),
  );
  summary.hidden = false;
  summary.focus();
}
```

`validate(form)` stands for the project's validation and returns `{ input, message }` pairs.

- The summary lists every error as a link to its field and receives focus on submit, so screen reader users hear it immediately (3.3.1).
- Each inline error is referenced by `aria-describedby` and is empty when there's no error, because hidden but referenced text is still read.
- Set `aria-invalid="true"` only on fields that failed.
- Write messages that say what's wrong and how to fix it: "Enter an email address in the correct format, like name@example.com" (3.3.3).
- Don't rely on color: pair red text or borders with an icon and the word "Error", visibly or in hidden text (1.4.1).
- Keep the user's input after an error; never clear valid fields.
- Don't disable the submit button until the form is valid; users can't find out what's missing (best practice).
- Browser validation bubbles disappear, can't be styled, and vary between browsers and screen readers. Prefer `novalidate` with custom messages, while keeping `required` and the right `type`.

## Error prevention and redundant entry

- For legal commitments, financial transactions, test answers, and changes to or deletion of user data, let users review and confirm before submitting, check input and allow correction, or allow the action to be undone (3.3.4).
- Don't ask again for information already given in the same process: carry values forward, offer "Same as billing address", or let users pick previously entered values (3.3.7). Confirming a newly created password and re-entry needed for security are allowed.
- Avoid "confirm your email" fields; show the address on the review step instead (best practice).

## Accessible authentication

No step of signing in, multi-factor authentication, or account recovery may rely only on a cognitive function test such as recalling a password, transcribing a code, or solving a puzzle (3.3.8).

- Let password managers and paste work: visible labels, `autocomplete="username"` and `current-password`, and no scripts that block paste or autofill.
- For each cognitive test, either provide a mechanism that helps users complete it, such as password-manager and paste support, or offer another authentication method such as passkeys (WebAuthn), single sign-on, an emailed sign-in link, or approval on a trusted device.
- Use one field for one-time codes, with `autocomplete="one-time-code"`. If the design needs one box per digit, pasting the whole code must fill every box.
- Never ask for selected characters of a password ("enter the 2nd and 5th characters").
- Avoid CAPTCHAs that require transcription or puzzles; prefer risk-based checks. Object recognition is allowed at AA but excludes blind users unless another method is available (1.1.1).
- A "show password" toggle helps everyone check what they typed (best practice).

## Time limits

- Warn before a session or form expires and let users extend it with a simple action, giving at least 20 seconds to respond (2.2.1). A modal `role="alertdialog"` with focus on "Stay signed in" works well.
- Save drafts or restore entered data after re-authentication (2.2.5 at AAA; good practice everywhere).

## Multi-step forms

- Show progress in text, such as "Step 2 of 4: Address", in the heading or in a step list that uses `aria-current="step"`.
- On each step change, update `document.title` and move focus to the step heading.
- Let users go back without losing data, and offer a review step before final submission (3.3.4).

## Custom and special controls

- Style native checkboxes and radios (with `accent-color`, or `appearance: none` plus custom styles) instead of rebuilding them. Boundaries and checked states need 3:1 contrast (1.4.11), plus a visible focus indicator.
- Date pickers: always allow typing the date in a text field with a format hint; the calendar popup is an optional extra. Native `<input type="date">` differs between browsers, so test it.
- File upload: keep a real "Browse" button (the native input, styled through its label); drop zones alone fail 2.1.1 and 2.5.7.
- `readonly` fields stay focusable and readable; `disabled` fields are skipped by Tab and have no contrast requirement, so don't use `disabled` for values users need to read.
- Don't change context automatically when a field is filled, such as submitting the form or jumping to the next box after a fixed number of characters, unless users are told beforehand (3.2.2).
