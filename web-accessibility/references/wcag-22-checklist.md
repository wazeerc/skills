# WCAG 2.2 checklist

Every Level A and AA success criterion (SC) in WCAG 2.2, phrased as a check with common failures. Each SC links to its official *Understanding* document, which defines intent, exceptions, and sufficient techniques; read it before deciding an edge case.

**Automation**: *partial* means axe, Lighthouse, or linters catch some failures, but a person must still verify; *manual* means it needs human or assistive-technology judgement.

## Contents

- [Conformance essentials](#conformance-essentials)
- [1 Perceivable](#1-perceivable)
- [2 Operable](#2-operable)
- [3 Understandable](#3-understandable)
- [4 Robust](#4-robust)
- [Level AAA](#level-aaa)
- [What changed in WCAG 2.2](#what-changed-in-wcag-22)

## Conformance essentials

- A conformance claim needs every Level A and AA SC satisfied (or not applicable) for **full pages** and **complete processes**. If one step of a checkout or sign-up fails, the whole process fails.
- 1.4.2 Audio Control, 2.1.2 No Keyboard Trap, 2.2.2 Pause, Stop, Hide, and 2.3.1 Three Flashes apply to all content on a page, even content not relied on for conformance.
- Content that conforms to WCAG 2.2 also conforms to 2.1 and 2.0. ISO/IEC 40500:2025 is the October 2023 version of WCAG 2.2.
- Legal and procurement requirements may reference specific WCAG versions, editions of EN 301 549, or jurisdictional rules. Verify the applicable requirements using current official sources, such as [Web Accessibility Laws & Policies](https://www.w3.org/WAI/policies/) and the [ETSI standards catalogue](https://www.etsi.org/standards), and leave legal interpretation to the project's compliance owner.
- For full audits, follow the [WCAG-EM evaluation methodology](https://www.w3.org/WAI/test-evaluate/conformance/wcag-em/) to choose a representative sample of pages and states.

## 1 Perceivable

### 1.1 Text alternatives

- **1.1.1 [Non-text Content](https://www.w3.org/WAI/WCAG22/Understanding/non-text-content)** (A, partial). Images, icons, charts, image buttons, image maps, and CAPTCHAs have text alternatives that serve the same purpose; decorative or redundant images are hidden from assistive technology. *Fails:* missing `alt`; a file name or "image" as `alt`; an icon-only button or link with no name; an informative CSS background image with no text equivalent; noisy `alt` on decorative images.

### 1.2 Time-based media

- **1.2.1 [Audio-only and Video-only (Prerecorded)](https://www.w3.org/WAI/WCAG22/Understanding/audio-only-and-video-only-prerecorded)** (A, manual). Prerecorded audio-only content has a transcript; prerecorded video-only content has a text alternative or an audio track that describes it.
- **1.2.2 [Captions (Prerecorded)](https://www.w3.org/WAI/WCAG22/Understanding/captions-prerecorded)** (A, manual). Prerecorded video with audio has synchronized captions for dialogue, speaker changes, and meaningful sounds. Auto-generated captions usually need editing to be accurate.
- **1.2.3 [Audio Description or Media Alternative (Prerecorded)](https://www.w3.org/WAI/WCAG22/Understanding/audio-description-or-media-alternative-prerecorded)** (A, manual). Prerecorded video has audio description of important visual information, or a full text alternative such as a descriptive transcript.
- **1.2.4 [Captions (Live)](https://www.w3.org/WAI/WCAG22/Understanding/captions-live)** (AA, manual). Live video with audio, such as a webinar or stream, has real-time captions.
- **1.2.5 [Audio Description (Prerecorded)](https://www.w3.org/WAI/WCAG22/Understanding/audio-description-prerecorded)** (AA, manual). Prerecorded video has audio description for visual information the soundtrack doesn't convey; at AA a text alternative alone isn't enough.

### 1.3 Adaptable

- **1.3.1 [Info and Relationships](https://www.w3.org/WAI/WCAG22/Understanding/info-and-relationships)** (A, partial). Structure and relationships shown visually are in the markup: headings, lists, data tables with header cells, labels tied to inputs, grouped controls, and landmarks. *Fails:* styled `<div>` elements as headings; data tables without `<th>`; label text not associated with its input; a radio group without a group label; ARIA roles missing their required parent or child roles.
- **1.3.2 [Meaningful Sequence](https://www.w3.org/WAI/WCAG22/Understanding/meaningful-sequence)** (A, manual). The DOM order gives the correct reading order when styles are ignored. *Fails:* CSS `order`, `*-reverse` flex directions, grid placement, or absolute positioning that makes the reading order contradict the visual meaning.
- **1.3.3 [Sensory Characteristics](https://www.w3.org/WAI/WCAG22/Understanding/sensory-characteristics)** (A, manual). Instructions don't rely only on shape, color, size, visual location, orientation, or sound. *Fails:* "Press the round button on the right" with no text label to refer to.
- **1.3.4 [Orientation](https://www.w3.org/WAI/WCAG22/Understanding/orientation)** (AA, partial). Content isn't locked to portrait or landscape unless a specific orientation is essential.
- **1.3.5 [Identify Input Purpose](https://www.w3.org/WAI/WCAG22/Understanding/identify-input-purpose)** (AA, partial). Fields that collect information about the user have the matching `autocomplete` token from the [WCAG input purposes list](https://www.w3.org/TR/WCAG22/#input-purposes) (name, email, telephone, address parts, username, passwords, birthday, payment). *Fails:* missing, wrong, or `off` values on these fields. Fields about other people are out of scope.

### 1.4 Distinguishable

- **1.4.1 [Use of Color](https://www.w3.org/WAI/WCAG22/Understanding/use-of-color)** (A, partial). Color isn't the only way to convey information, indicate an action, prompt a response, or distinguish an element. *Fails:* errors or required fields marked only in red; links in body text distinguished only by a color that has less than 3:1 contrast with the surrounding text and no other cue; charts with color-only legends.
- **1.4.2 [Audio Control](https://www.w3.org/WAI/WCAG22/Understanding/audio-control)** (A, partial). Audio that plays automatically for more than 3 seconds can be paused or stopped, or its volume controlled separately from the system volume. Prefer no autoplay.
- **1.4.3 [Contrast (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum)** (AA, partial). Text and images of text have at least 4.5:1 contrast; large text (at least 24 CSS px, or 18.66 CSS px bold) at least 3:1. Disabled controls, decoration, incidental text in photos, and logos are exempt. Check hover, focus, visited, and selected states, placeholder text, text over images or gradients (worst case), and every theme. Tools miss text over images and most non-default states.
- **1.4.4 [Resize Text](https://www.w3.org/WAI/WCAG22/Understanding/resize-text)** (AA, partial). Text scales to 200% without loss of content or function. *Fails:* zoom disabled in the viewport `<meta>`; fixed-height containers that clip text; text that overlaps at 200%.
- **1.4.5 [Images of Text](https://www.w3.org/WAI/WCAG22/Understanding/images-of-text)** (AA, manual). Real text styled with CSS is used instead of images of text, except logos or where the presentation is essential or user-customizable.
- **1.4.10 [Reflow](https://www.w3.org/WAI/WCAG22/Understanding/reflow)** (AA, manual). At 320 CSS px wide (1280 px at 400% zoom), content needs only vertical scrolling and nothing is lost; content that scrolls horizontally works at 256 CSS px high. Maps, diagrams, data tables, video, games, and toolbars may need two-dimensional layout, but the rest of the page must still reflow. *Fails:* fixed-width layouts; content cut off or overlapping; controls pushed off screen.
- **1.4.11 [Non-text Contrast](https://www.w3.org/WAI/WCAG22/Understanding/non-text-contrast)** (AA, manual). Visual information needed to identify components and their states (input borders, checkbox and toggle states, focus indicators, icon-only buttons) and the meaningful parts of graphics (chart lines, informative icons) have at least 3:1 contrast with adjacent colors. Inactive components and unmodified browser defaults are exempt. *Fails:* light-gray input borders; a selected tab shown only by a faint tint; a pale focus ring.
- **1.4.12 [Text Spacing](https://www.w3.org/WAI/WCAG22/Understanding/text-spacing)** (AA, partial). Nothing is lost or overlaps when users set line height to 1.5, paragraph spacing to 2, letter spacing to 0.12, and word spacing to 0.16 times the font size. *Fails:* fixed heights or `overflow: hidden` that clip text; inline `!important` spacing that blocks user styles.
- **1.4.13 [Content on Hover or Focus](https://www.w3.org/WAI/WCAG22/Understanding/content-on-hover-or-focus)** (AA, manual). Content that appears on hover or focus (tooltips, popovers, submenus) is **dismissible** without moving the pointer or focus (usually Escape), **hoverable** (the pointer can move onto it without it disappearing), and **persistent** (it stays until dismissed, the trigger is removed, or it's no longer valid). Browser-controlled `title` tooltips are exempt.

## 2 Operable

### 2.1 Keyboard accessible

- **2.1.1 [Keyboard](https://www.w3.org/WAI/WCAG22/Understanding/keyboard)** (A, manual). All functionality works with a keyboard, without specific timings for keystrokes, except input that depends on the path of movement (such as freehand drawing). *Fails:* click handlers on non-focusable elements; hover-only menus; drag-only interactions; custom widgets that ignore Enter, Space, or arrow keys; scrollable regions that can't receive focus.
- **2.1.2 [No Keyboard Trap](https://www.w3.org/WAI/WCAG22/Understanding/no-keyboard-trap)** (A, manual). Focus can always leave a component with standard keys; if non-standard keys are needed, users are told how. *Fails:* embedded editors, players, or iframes that capture Tab; a focus trap that stays active after its dialog closes.
- **2.1.4 [Character Key Shortcuts](https://www.w3.org/WAI/WCAG22/Understanding/character-key-shortcuts)** (A, manual). Single-character shortcuts (letters, numbers, punctuation, symbols) can be turned off, remapped to include a modifier key, or are active only while the relevant component has focus. *Fails:* global `j`, `k`, or `/` shortcuts that fire while speech-input users dictate.

### 2.2 Enough time

- **2.2.1 [Timing Adjustable](https://www.w3.org/WAI/WCAG22/Understanding/timing-adjustable)** (A, partial). Each time limit can be turned off, adjusted to at least 10 times the default, or extended: users are warned before it expires, get at least 20 seconds to extend it with a simple action, and can extend it at least 10 times. Real-time events, essential limits, and limits over 20 hours are exempt. *Fails:* session timeouts without warning; auto-dismissing toasts or messages that users need to read or act on; delayed `<meta http-equiv="refresh">` redirects.
- **2.2.2 [Pause, Stop, Hide](https://www.w3.org/WAI/WCAG22/Understanding/pause-stop-hide)** (A, partial). Moving, blinking, or scrolling content that starts automatically, lasts more than 5 seconds, and appears alongside other content can be paused, stopped, or hidden; auto-updating content can be paused, stopped, hidden, or have its update frequency controlled. *Fails:* auto-rotating carousels without a pause control; looping animations, GIFs, or background videos longer than 5 seconds without controls.

### 2.3 Seizures and physical reactions

- **2.3.1 [Three Flashes or Below Threshold](https://www.w3.org/WAI/WCAG22/Understanding/three-flashes-or-below-threshold)** (A, manual). Nothing flashes more than three times in any one-second period, unless the flashes are below the general and red flash thresholds. Analyze video and animation with a photosensitive-epilepsy analysis tool.

### 2.4 Navigable

- **2.4.1 [Bypass Blocks](https://www.w3.org/WAI/WCAG22/Understanding/bypass-blocks)** (A, partial). There is a way to skip blocks repeated on multiple pages, such as landmarks and headings. Add a visible-on-focus skip link to `<main>` as well, because keyboard users without a screen reader can't jump by landmark.
- **2.4.2 [Page Titled](https://www.w3.org/WAI/WCAG22/Understanding/page-titled)** (A, partial). Each page and single-page-app view has a `<title>` that describes its topic or purpose, for example "Edit profile - Settings - Product".
- **2.4.3 [Focus Order](https://www.w3.org/WAI/WCAG22/Understanding/focus-order)** (A, manual). Focus order preserves meaning and operability: it follows the reading order, dialogs receive focus when they open and return it when they close, and hidden or off-screen content isn't focusable. *Fails:* positive `tabindex`; DOM order that differs from the visual order; focus that lands behind an open modal.
- **2.4.4 [Link Purpose (In Context)](https://www.w3.org/WAI/WCAG22/Understanding/link-purpose-in-context)** (A, partial). Each link's purpose is clear from its text, or from its text together with its programmatic context (sentence, paragraph, list item, table cell and headers). *Fails:* several "Read more" links without distinguishing context; empty icon links.
- **2.4.5 [Multiple Ways](https://www.w3.org/WAI/WCAG22/Understanding/multiple-ways)** (AA, manual). There is more than one way to find a page in a set (navigation, search, site map, related links), except pages that are a step in, or the result of, a process.
- **2.4.6 [Headings and Labels](https://www.w3.org/WAI/WCAG22/Understanding/headings-and-labels)** (AA, manual). Headings and labels that exist describe their topic or purpose. *Fails:* duplicate "Details" headings for different sections; labels such as "Field 1".
- **2.4.7 [Focus Visible](https://www.w3.org/WAI/WCAG22/Understanding/focus-visible)** (AA, manual). Every keyboard-operable element shows a visible focus indicator. *Fails:* `outline: none` without a replacement; indicators clipped by `overflow: hidden`; focus styles indistinguishable from the unfocused state.
- **2.4.11 [Focus Not Obscured (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/focus-not-obscured-minimum)** (AA, manual). A focused component isn't entirely hidden by author-created content such as sticky headers or footers, cookie banners, or chat widgets. Content the user opened and can dismiss without moving focus doesn't count. *Fix:* `scroll-padding` on the scroll container (W3C technique [C43](https://www.w3.org/WAI/WCAG22/Techniques/css/C43)).

### 2.5 Input modalities

- **2.5.1 [Pointer Gestures](https://www.w3.org/WAI/WCAG22/Understanding/pointer-gestures)** (A, manual). Functions that use multipoint gestures (pinch, two-finger swipe) or path-based gestures (swipe, drawing a path) also work with a single pointer without a path, such as a tap or click, unless the gesture is essential. *Fails:* swipe-only carousels; pinch-only zoom without buttons.
- **2.5.2 [Pointer Cancellation](https://www.w3.org/WAI/WCAG22/Understanding/pointer-cancellation)** (A, manual). Single-pointer actions don't complete on the down-event, or can be aborted or undone, or the up-event reverses the down-event, or completing on down is essential. The native `click` event meets this. *Fails:* actions on `mousedown`, `pointerdown`, or `touchstart`.
- **2.5.3 [Label in Name](https://www.w3.org/WAI/WCAG22/Understanding/label-in-name)** (A, partial). A component's accessible name contains its visible label text, ideally at the start, so speech-input users can say what they see. *Fails:* `<button aria-label="Submit form">Send</button>`; a visible "Search" button named "Find".
- **2.5.4 [Motion Actuation](https://www.w3.org/WAI/WCAG22/Understanding/motion-actuation)** (A, manual). Functions triggered by moving the device or by user gestures detected by a camera (shake, tilt) also work through UI controls, and motion response can be disabled, unless the motion is essential.
- **2.5.7 [Dragging Movements](https://www.w3.org/WAI/WCAG22/Understanding/dragging-movements)** (AA, manual). Anything done by dragging (sortable lists, sliders, kanban boards, map panning, resizable panes) can also be done with a single pointer without dragging, such as buttons, menus, or click-to-pick-then-click-to-place, unless dragging is essential. A keyboard alternative meets this only if it also offers controls that can be clicked or tapped.
- **2.5.8 [Target Size (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum)** (AA, partial). Pointer targets are at least 24 × 24 CSS px, or an undersized target is spaced so a 24 px diameter circle centered on it doesn't intersect another target or another undersized target's circle. Exceptions: an equivalent control meets the size; inline links in text; unmodified browser controls; size that is essential, such as map pins. *Fails:* 16 px icon buttons packed into a toolbar; tiny "×" close buttons next to other controls.

## 3 Understandable

### 3.1 Readable

- **3.1.1 [Language of Page](https://www.w3.org/WAI/WCAG22/Understanding/language-of-page)** (A, partial). `<html lang>` has the correct primary language, such as `en` or `fr-CA`. *Fails:* missing, invalid, or wrong values, such as a template default of `en` on a French page.
- **3.1.2 [Language of Parts](https://www.w3.org/WAI/WCAG22/Understanding/language-of-parts)** (AA, partial). Passages in another language have their own `lang`, except proper names, technical terms, words of indeterminate language, and words that have become part of the surrounding language. In a language switcher, give each option `lang`, for example `<a lang="de">Deutsch</a>`.

### 3.2 Predictable

- **3.2.1 [On Focus](https://www.w3.org/WAI/WCAG22/Understanding/on-focus)** (A, manual). Moving focus to a component doesn't trigger a change of context, such as submitting a form, opening a new window, or moving focus elsewhere.
- **3.2.2 [On Input](https://www.w3.org/WAI/WCAG22/Understanding/on-input)** (A, manual). Changing a setting (choosing an option, checking a box, typing) doesn't automatically change the context unless users are told beforehand. *Fails:* a `<select>` that navigates on change; a form that submits when the last field is filled. Showing or hiding related fields in place is fine.
- **3.2.3 [Consistent Navigation](https://www.w3.org/WAI/WCAG22/Understanding/consistent-navigation)** (AA, manual). Navigation repeated across pages appears in the same relative order each time, unless the user changes it.
- **3.2.4 [Consistent Identification](https://www.w3.org/WAI/WCAG22/Understanding/consistent-identification)** (AA, manual). Components with the same function across pages have consistent labels, icons, and accessible names; for example, not "Search" on one page and "Find" on another.
- **3.2.6 [Consistent Help](https://www.w3.org/WAI/WCAG22/Understanding/consistent-help)** (A, manual). When help is repeated across pages (human contact details, a contact mechanism such as chat or a form, a self-help option such as an FAQ, or an automated contact mechanism), it appears in the same order relative to the other page content, unless the user changes it. Help itself isn't required.

### 3.3 Input assistance

- **3.3.1 [Error Identification](https://www.w3.org/WAI/WCAG22/Understanding/error-identification)** (A, manual). When an input error is detected automatically, the field in error is identified and the error is described in text. *Fails:* only a red border; a generic "The form has errors" message; errors shown only in a toast that disappears.
- **3.3.2 [Labels or Instructions](https://www.w3.org/WAI/WCAG22/Understanding/labels-or-instructions)** (A, partial). Inputs have labels or instructions, including required-field indicators and expected formats. *Fails:* a placeholder as the only label; an unexplained asterisk; an unstated date or phone format.
- **3.3.3 [Error Suggestion](https://www.w3.org/WAI/WCAG22/Understanding/error-suggestion)** (AA, manual). When a correction is known, it's suggested, such as "Enter a date like 31/12/2026", unless that would compromise security or the purpose of the content.
- **3.3.4 [Error Prevention (Legal, Financial, Data)](https://www.w3.org/WAI/WCAG22/Understanding/error-prevention-legal-financial-data)** (AA, manual). Submissions that create legal commitments or financial transactions, modify or delete user data, or submit test answers are reversible, checked for errors with a chance to correct them, or reviewed and confirmed before they're final.
- **3.3.7 [Redundant Entry](https://www.w3.org/WAI/WCAG22/Understanding/redundant-entry)** (A, manual). Information the user already entered or was given in the same process is auto-populated or available to select, unless re-entry is essential, needed for security, or the earlier information is no longer valid. Confirming a newly created password is allowed. *Fails:* asking for the shipping address again on the payment step without a "same as" option.
- **3.3.8 [Accessible Authentication (Minimum)](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication-minimum)** (AA, manual). No step of authentication requires a cognitive function test (remembering a password, transcribing a code, solving a puzzle) unless another method is available, a mechanism helps (password managers, paste), or the test is recognizing objects or identifying content the user provided. *Fails:* paste blocked in password or code fields; scripts that block autofill; one-digit-per-box codes that reject a pasted code; text CAPTCHAs without an alternative; "enter the 3rd and 5th characters of your password".

## 4 Robust

- **4.1.1 [Parsing](https://www.w3.org/WAI/WCAG22/Understanding/parsing)** is obsolete and removed in WCAG 2.2. Duplicate IDs still matter when they break `for`, `aria-labelledby`, or `aria-describedby` references; report those under 1.3.1 or 4.1.2.
- **4.1.2 [Name, Role, Value](https://www.w3.org/WAI/WCAG22/Understanding/name-role-value)** (A, partial). Every UI component exposes its name and role; states, properties, and values that users can set are exposed and settable; and changes are announced. Native elements do this; custom widgets need correct ARIA plus keyboard support. *Fails:* unnamed buttons, links, inputs, or iframes; `<div>` buttons; toggles without `aria-pressed`, `aria-expanded`, or `aria-checked`; custom selects without listbox semantics; invalid ARIA; `aria-hidden` on focusable elements.
- **4.1.3 [Status Messages](https://www.w3.org/WAI/WCAG22/Understanding/status-messages)** (AA, manual). Status messages (success, result counts, progress, errors that don't move focus) are exposed with `role="status"`, `role="alert"`, `role="log"`, `aria-live`, or `<output>`, so assistive technology announces them without a focus change. *Fails:* a "Saved" toast without a live region; a search-results count that updates silently.

## Level AAA

AAA isn't required for AA conformance, and W3C doesn't recommend requiring it for entire sites because some content can't meet every AAA criterion. Adopt individual criteria where they're feasible; **bold** ones are cheap wins for most products.

| SC | Requirement in brief |
|---|---|
| 1.2.6 [Sign Language (Prerecorded)](https://www.w3.org/WAI/WCAG22/Understanding/sign-language-prerecorded) | Sign language interpretation for prerecorded audio in media |
| 1.2.7 [Extended Audio Description (Prerecorded)](https://www.w3.org/WAI/WCAG22/Understanding/extended-audio-description-prerecorded) | Video pauses to fit audio description where gaps are too short |
| 1.2.8 [Media Alternative (Prerecorded)](https://www.w3.org/WAI/WCAG22/Understanding/media-alternative-prerecorded) | Full text alternative for all prerecorded media |
| 1.2.9 [Audio-only (Live)](https://www.w3.org/WAI/WCAG22/Understanding/audio-only-live) | Text alternative for live audio-only content |
| 1.3.6 [Identify Purpose](https://www.w3.org/WAI/WCAG22/Understanding/identify-purpose) | Purpose of components, icons, and regions is programmatically determinable |
| **1.4.6 [Contrast (Enhanced)](https://www.w3.org/WAI/WCAG22/Understanding/contrast-enhanced)** | 7:1 for text, 4.5:1 for large text |
| 1.4.7 [Low or No Background Audio](https://www.w3.org/WAI/WCAG22/Understanding/low-or-no-background-audio) | Speech audio has no or quiet background sound |
| 1.4.8 [Visual Presentation](https://www.w3.org/WAI/WCAG22/Understanding/visual-presentation) | User-selectable colors, line length up to 80 characters, no justified text, generous spacing |
| 1.4.9 [Images of Text (No Exception)](https://www.w3.org/WAI/WCAG22/Understanding/images-of-text-no-exception) | Images of text only for decoration or where essential |
| 2.1.3 [Keyboard (No Exception)](https://www.w3.org/WAI/WCAG22/Understanding/keyboard-no-exception) | Everything keyboard-operable, with no path-dependent exception |
| 2.2.3 [No Timing](https://www.w3.org/WAI/WCAG22/Understanding/no-timing) | No time limits except real-time events |
| 2.2.4 [Interruptions](https://www.w3.org/WAI/WCAG22/Understanding/interruptions) | Users can postpone or suppress interruptions |
| 2.2.5 [Re-authenticating](https://www.w3.org/WAI/WCAG22/Understanding/re-authenticating) | No data loss when re-authenticating after a session expires |
| **2.2.6 [Timeouts](https://www.w3.org/WAI/WCAG22/Understanding/timeouts)** | Warn users about inactivity timeouts that lose data |
| 2.3.2 [Three Flashes](https://www.w3.org/WAI/WCAG22/Understanding/three-flashes) | No flashing more than three times per second at all |
| **2.3.3 [Animation from Interactions](https://www.w3.org/WAI/WCAG22/Understanding/animation-from-interactions)** | Motion triggered by interaction can be disabled (`prefers-reduced-motion`) |
| 2.4.8 [Location](https://www.w3.org/WAI/WCAG22/Understanding/location) | Users can tell where they are in a set of pages (breadcrumbs, current-page state) |
| 2.4.9 [Link Purpose (Link Only)](https://www.w3.org/WAI/WCAG22/Understanding/link-purpose-link-only) | Link text alone identifies the purpose |
| 2.4.10 [Section Headings](https://www.w3.org/WAI/WCAG22/Understanding/section-headings) | Sections are organized with headings |
| **2.4.12 [Focus Not Obscured (Enhanced)](https://www.w3.org/WAI/WCAG22/Understanding/focus-not-obscured-enhanced)** | No part of the focused component is hidden by author content |
| **2.4.13 [Focus Appearance](https://www.w3.org/WAI/WCAG22/Understanding/focus-appearance)** | Indicator area at least a 2 CSS px perimeter of the component, with 3:1 contrast between focused and unfocused states |
| **2.5.5 [Target Size (Enhanced)](https://www.w3.org/WAI/WCAG22/Understanding/target-size-enhanced)** | Targets at least 44 × 44 CSS px |
| 2.5.6 [Concurrent Input Mechanisms](https://www.w3.org/WAI/WCAG22/Understanding/concurrent-input-mechanisms) | Content doesn't restrict users to one input type (such as touch only) |
| 3.1.3 [Unusual Words](https://www.w3.org/WAI/WCAG22/Understanding/unusual-words) | Definitions for idioms and jargon |
| 3.1.4 [Abbreviations](https://www.w3.org/WAI/WCAG22/Understanding/abbreviations) | Expanded forms of abbreviations available |
| 3.1.5 [Reading Level](https://www.w3.org/WAI/WCAG22/Understanding/reading-level) | Supplemental content when text needs more than lower-secondary reading ability |
| 3.1.6 [Pronunciation](https://www.w3.org/WAI/WCAG22/Understanding/pronunciation) | Pronunciation available where meaning is ambiguous without it |
| 3.2.5 [Change on Request](https://www.w3.org/WAI/WCAG22/Understanding/change-on-request) | Context changes only on user request (warn before opening new windows) |
| 3.3.5 [Help](https://www.w3.org/WAI/WCAG22/Understanding/help) | Context-sensitive help available |
| 3.3.6 [Error Prevention (All)](https://www.w3.org/WAI/WCAG22/Understanding/error-prevention-all) | All submissions reversible, checked, or confirmed |
| **3.3.9 [Accessible Authentication (Enhanced)](https://www.w3.org/WAI/WCAG22/Understanding/accessible-authentication-enhanced)** | Like 3.3.8, without the object-recognition and personal-content exceptions |

## What changed in WCAG 2.2

| Change | Success criterion | Level |
|---|---|---|
| Added | 2.4.11 Focus Not Obscured (Minimum) | AA |
| Added | 2.4.12 Focus Not Obscured (Enhanced) | AAA |
| Added | 2.4.13 Focus Appearance | AAA |
| Added | 2.5.7 Dragging Movements | AA |
| Added | 2.5.8 Target Size (Minimum) | AA |
| Added | 3.2.6 Consistent Help | A |
| Added | 3.3.7 Redundant Entry | A |
| Added | 3.3.8 Accessible Authentication (Minimum) | AA |
| Added | 3.3.9 Accessible Authentication (Enhanced) | AAA |
| Removed | 4.1.1 Parsing (obsolete) | A |

Details: [What's New in WCAG 2.2](https://www.w3.org/WAI/standards-guidelines/wcag/new-in-22/).
