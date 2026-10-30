# Testing

Automated tools find only part of the problem: they're good at missing names, invalid ARIA, and many contrast failures, but they can't judge focus order, meaningful text alternatives, announcements, or whether a flow makes sense. Combine automated layers with manual checks, and include disabled users in research when possible.

## Contents

- [Strategy](#strategy)
- [Reading automated results](#reading-automated-results)
- [Linting](#linting)
- [Component tests](#component-tests)
- [End-to-end tests](#end-to-end-tests)
- [Storybook and command-line scans](#storybook-and-command-line-scans)
- [Browser checks with Chrome DevTools MCP](#browser-checks-with-chrome-devtools-mcp)
- [Manual testing](#manual-testing)
- [Screen reader commands](#screen-reader-commands)
- [Evidence](#evidence)

## Strategy

| Layer | Catches | Tools |
|---|---|---|
| Lint | Missing `alt`, invalid ARIA attributes, click handlers on static elements | `eslint-plugin-jsx-a11y`, `eslint-plugin-vuejs-accessibility`, angular-eslint template accessibility rules |
| Component tests | Names, roles, states, keyboard behavior, focus moves, axe rules per state | Testing Library, `jest-axe` or `vitest-axe` |
| End-to-end tests | Page-level axe rules, real layout (contrast, target size), keyboard journeys | `cypress-axe`, `@axe-core/playwright` |
| Browser audit | Rendered issues in the running app | Lighthouse, axe DevTools, Chrome DevTools MCP |
| Manual | Focus order and visibility, announcements, reflow, meaning, cognitive load | Keyboard, screen readers, zoom, emulation |

Use the layers the repository already has. Add a tool only when the user asks or agrees.

## Reading automated results

- axe returns `violations` (failures), `incomplete` (needs manual review), `passes`, and `inapplicable`. Report `incomplete` items under "Needs verification", not as failures.
- Impact levels (`critical`, `serious`, `moderate`, `minor`) match the [severity](../SKILL.md#severity) scale.
- Tags: `wcag2a`, `wcag2aa`, `wcag21a`, `wcag21aa`, and `wcag22aa` mark WCAG levels; `wcag143`-style tags map to the success criterion (here 1.4.3); `best-practice` rules aren't WCAG requirements; `experimental` rules are off by default. For WCAG 2.2 AA, run all five WCAG tags.
- axe doesn't test hidden content. Open menus, dialogs, error states, and other states, then scan again.
- JSDOM has no layout, so component-level axe can't check color contrast, target size, or reflow; jest-axe turns color contrast off. Cover those in a real browser.
- A Lighthouse accessibility score is a weighted subset of axe checks. A score of 100 isn't conformance; read its "Additional items to manually check" list as well.

## Linting

- React: `eslint-plugin-jsx-a11y` with the `recommended` or `strict` config (flat config: `jsxA11y.flatConfigs.recommended`).
- Vue: `eslint-plugin-vuejs-accessibility`. Angular: the `templateAccessibility` config from angular-eslint.
- Configure custom components that render native elements (for example, a `Button` that renders `<button>`) in the plugin settings so their rules apply.

## Component tests

Jest or Vitest with React Testing Library. Register the matcher once in the test setup file: `expect.extend(toHaveNoViolations)` from `jest-axe` (it works with Vitest's `expect.extend` too), or use `vitest-axe`.

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { axe } from 'jest-axe';
import { ShippingOptions } from './ShippingOptions';

describe('ShippingOptions', () => {
  it('should have no detectable axe violations when expanded', async () => {
    const user = userEvent.setup();
    const { container } = render(<ShippingOptions />);
    await user.click(screen.getByRole('button', { name: 'Shipping options' }));

    expect(await axe(container)).toHaveNoViolations();
  });

  it('should toggle the panel with the keyboard', async () => {
    const user = userEvent.setup();
    render(<ShippingOptions />);
    const toggle = screen.getByRole('button', { name: 'Shipping options' });

    await user.tab();
    await user.keyboard('{Enter}');

    expect(toggle).toHaveFocus();
    expect(toggle).toHaveAttribute('aria-expanded', 'true');
    expect(screen.getByText('Standard delivery takes 3 to 5 working days.')).toBeVisible();
  });
});
```

- Scan each meaningful state (closed, open, error, loading), not only the initial render.
- Components rendered through portals (dialogs, popovers) live outside `container`; scan `baseElement` instead.
- axe-core times out under fake timers. Switch to real timers around the `axe()` call (`jest.useRealTimers()` or `vi.useRealTimers()`), then restore fake timers.
- Page-level rules such as `region` (content inside landmarks) don't apply to isolated components; jest-axe documents turning `region` off for component tests with `configureAxe`. Keep page-level rules in end-to-end tests, and don't disable any other rule to get a pass.
- Query by role and accessible name (`getByRole('button', { name: 'Save' })`). A failing role query is often an accessibility bug, not a test problem.
- Useful jest-dom matchers: `toHaveAccessibleName`, `toHaveAccessibleDescription`, `toHaveFocus`, `toBeVisible`, `toBeDisabled`, `toHaveAttribute('aria-expanded', 'true')`.
- Assert focus after opening, closing, deleting, and submitting with errors.

## End-to-end tests

### Cypress

Import `cypress-axe` in the support file, then:

```js
describe('Payment step', () => {
  it('should have no detectable axe violations after a failed submit', () => {
    cy.visit('/checkout/payment');
    cy.injectAxe();
    cy.contains('button', 'Pay now').click();

    cy.checkA11y(null, {
      runOnly: { type: 'tag', values: ['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa', 'wcag22aa'] },
    });
  });

  it('should move focus to the error summary after a failed submit', () => {
    cy.visit('/checkout/payment');
    cy.contains('button', 'Pay now').click();

    cy.focused().should('have.attr', 'id', 'error-summary');
  });
});
```

- `cy.injectAxe()` must run after every `cy.visit()`.
- For real Tab presses, use `cy.press(Cypress.Keyboard.Keys.TAB)` in recent Cypress versions, or `cypress-real-events` (`cy.realPress('Tab')`).

### Playwright

```ts
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('should have no detectable axe violations on the settings page', async ({ page }) => {
  await page.goto('/settings');

  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa', 'wcag22aa'])
    .analyze();

  expect(results.violations).toEqual([]);
});

test('should close the actions menu with Escape and return focus', async ({ page }) => {
  await page.goto('/settings');
  const trigger = page.getByRole('button', { name: 'Actions' });

  await trigger.click();
  await page.keyboard.press('Escape');

  await expect(trigger).toBeFocused();
  await expect(trigger).toHaveAttribute('aria-expanded', 'false');
});
```

- `toHaveAccessibleName`, `toHaveAccessibleDescription`, and `toHaveRole` assert semantics; `toMatchAriaSnapshot` locks the accessibility tree of a region.
- Also scan at a narrow viewport (`page.setViewportSize({ width: 320, height: 640 })`), because mobile layouts often have their own target-size and contrast problems.

## Storybook and command-line scans

- Storybook's `@storybook/addon-a11y` runs axe on each story. With the Vitest addon, `parameters.a11y.test` set to `'error'` fails tests on violations.
- Lighthouse: `npx lighthouse <url> --only-categories=accessibility --output=json --output-path=./lighthouse-a11y.json`.
- axe CLI: `npx @axe-core/cli <url> --tags wcag2a,wcag2aa,wcag21a,wcag21aa,wcag22aa`.

## Browser checks with available browser tools

Use browser checks only when a running app is available and the user asked for or agreed to them. Tool names and capabilities vary by environment: discover the available browser tools and map them to these checks rather than assuming a particular MCP API. If a capability is unavailable, report it as not tested.

| Check | Browser capability |
|---|---|
| Automated audit with failing nodes | Run axe or Lighthouse against the current page state; preserve the state (for example, signed in or with a dialog open) when possible |
| Names, roles, states, landmarks, headings | Inspect the accessibility tree |
| Keyboard walk-through | Send `Tab`, `Shift+Tab`, `Enter`, `Escape`, and relevant arrow keys; inspect the focused element and resulting state |
| Reflow at 320 CSS px | Resize the viewport to 320 CSS px wide |
| Forced colors, reduced motion, dark mode, more contrast | Emulate each setting when supported |
| Text spacing (1.4.12) | Apply the [text-spacing overrides](visual-design.md#zoom-reflow-and-text-spacing), then inspect the page |
| Colors and focus styles | Inspect computed styles and contrast against the rendered background |
| Visual evidence | Capture a screenshot |

Re-check active emulation settings after changing them; some tools replace previous settings. Reset emulation when finished.

## Manual testing

### Keyboard

1. Put the mouse aside and Tab from the browser's address bar through the whole page.
2. Confirm every interactive element is reachable in a logical order, focus is always visible, and nothing focused is hidden behind sticky content.
3. Operate each control: Enter and Space on buttons, arrow keys in composite widgets, Escape on overlays.
4. Open and close every overlay; focus moves in and returns to the trigger.
5. Submit forms with errors; focus or an announcement leads to the errors.
6. Check for traps, a working skip link, and no focus on hidden or off-screen elements.

### Screen readers

Common pairings: NVDA with Chrome or Firefox and JAWS with Chrome or Edge on Windows, VoiceOver with Safari on macOS and iOS, and TalkBack with Chrome on Android.

Check that:

- The page title is announced on load and after route changes.
- The headings and landmarks lists describe the page.
- Each control announces its name, role, and state, and state changes (expanded, selected, checked) are announced.
- Errors and status messages are announced without hunting for them.
- Dialogs announce their name, and the content behind them is unreachable.
- Images, tables, and form groups make sense when heard out of visual context.
- The reading order matches the visual order.

### Zoom, reflow, and text spacing

- At a 1280 px wide window zoomed to 400% (or a 320 CSS px viewport): no horizontal scrolling except for exempt content, nothing cut off, and sticky elements don't cover the content.
- At 200% zoom, all text is readable and every control works.
- With the text-spacing overrides applied, nothing is clipped or overlapping.
- With a larger default font size in browser settings, text sized in `rem` scales (best practice).

### Visual

- Spot-check contrast on text, borders, icons, focus indicators, and states, in every theme.
- Use DevTools vision-deficiency emulation (Rendering panel) to find color-only cues.
- Check forced-colors mode and reduced motion.

### Flows and cognition

- Errors are understandable and fixable, and entered data is kept.
- Time limits warn and can be extended.
- Sign-in works with paste and a password manager, and offers an alternative to memory tests.
- Help and navigation stay in the same place across pages, and no information has to be entered twice in one process.

## Screen reader commands

NVDA key: Insert (or Caps Lock with the laptop layout). JAWS key: Insert. VO: Control+Option.

| Action | NVDA | JAWS | VoiceOver (macOS) |
|---|---|---|---|
| Start, stop | Ctrl+Alt+N, NVDA+Q | Start menu, Insert+F4 | Cmd+F5 |
| Stop speaking | Ctrl | Ctrl | Ctrl |
| Next item | Down Arrow | Down Arrow | VO+Right Arrow |
| Activate | Enter | Enter | VO+Space |
| Next heading | H | H | VO+Cmd+H |
| Next landmark | D | R | Rotor (VO+U), Landmarks |
| Next form field | F | F | VO+Cmd+J |
| Lists of elements | NVDA+F7 | Insert+F6 headings, Insert+F7 links, Insert+F5 form fields | Rotor (VO+U) |
| Switch between reading and interaction | NVDA+Space (browse or focus mode) | Enter on a field starts forms mode; Insert+Z toggles the virtual cursor | VO+Shift+Down Arrow interacts with an element |

Mobile:

- **VoiceOver (iOS)**: Settings > Accessibility > VoiceOver, or the Accessibility Shortcut (triple-click the side button). Swipe right or left to move, double-tap to activate, rotate two fingers for the rotor, and swipe up with two fingers to read everything.
- **TalkBack (Android)**: Settings > Accessibility > TalkBack, or hold both volume keys for about 3 seconds if the shortcut is on. Swipe right or left to move and double-tap to activate.

## Evidence

- **Automated**: tool and version, rule ID, impact, WCAG tags, and the selector or HTML snippet.
- **Manual**: the steps, environment (browser, assistive technology and versions, viewport, zoom), expected and actual results, and a screenshot, accessibility-tree excerpt, or announcement transcript (for example, NVDA with Chrome: "Save, button, unavailable").
