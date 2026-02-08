# Accessibility Standards

This document defines the WCAG 2.1 Level AA accessibility standards that all agents, skills, and generated code in this repository must follow.

## Target Compliance

**Primary Standard:** WCAG 2.1 Level AA
**Optional:** WCAG 2.1 Level AAA for critical user paths

## WCAG 2.1 Level AA Requirements

### Perceivable

Information and user interface components must be presentable to users in ways they can perceive.

#### 1.1.1 Non-text Content (Level A)

All non-text content must have a text alternative that serves the equivalent purpose.

**Requirements:**
- All `<img>` elements must have descriptive `alt` attributes
- Decorative images must use `alt=""` or `role="presentation"`
- Icon-only buttons must have `aria-label` or visually hidden text
- Complex images (charts, diagrams) need detailed descriptions via `aria-describedby` or adjacent text
- Audio/video content must have transcripts or captions

**Examples:**
```html
<!-- Informative image -->
<img src="profile.jpg" alt="Jane Doe, Software Engineer">

<!-- Decorative image -->
<img src="divider.svg" alt="" role="presentation">

<!-- Icon button -->
<button aria-label="Close dialog">
  <svg>...</svg>
</button>

<!-- Complex diagram -->
<img src="architecture.png" alt="System architecture diagram"
     aria-describedby="arch-description">
<div id="arch-description">
  The system consists of three layers: presentation, business logic, and data...
</div>
```

#### 1.3.1 Info and Relationships (Level A)

Information, structure, and relationships conveyed through presentation must be programmatically determined or available in text.

**Requirements:**
- Use semantic HTML elements (`<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>`)
- Use proper heading hierarchy (`<h1>` → `<h2>` → `<h3>`, no skipping levels)
- Use `<table>` with `<thead>`, `<tbody>`, `<th>` for data tables
- Form inputs must have associated `<label>` elements
- Related form controls must be grouped with `<fieldset>` and `<legend>`
- Use lists (`<ul>`, `<ol>`, `<dl>`) for list content
- Use `<button>` for buttons, `<a>` for links

**Bad:**
```html
<div class="button" onclick="submit()">Submit</div>
<div class="heading">Page Title</div>
<input type="text" placeholder="Email"> <!-- No label -->
```

**Good:**
```html
<button type="submit">Submit</button>
<h1>Page Title</h1>
<label for="email">Email</label>
<input type="email" id="email" name="email">
```

#### 1.4.1 Use of Color (Level A)

Color must not be the only visual means of conveying information, indicating an action, prompting a response, or distinguishing a visual element.

**Requirements:**
- Links must have underline, icon, or other non-color indicator
- Form errors must have icon and text, not just red color
- Required fields must have text label "(required)", not just red asterisk
- Status indicators must have icon and text, not just color
- Charts must use patterns, labels, or textures in addition to color

**Bad:**
```html
<p>Links are <a href="/terms" style="color: blue;">blue</a></p>
<input type="email" style="border-color: red;"> <!-- Error indicated only by color -->
<label>Email <span style="color: red;">*</span></label> <!-- Required indicated only by color -->
```

**Good:**
```html
<p>Links are <a href="/terms" style="color: blue; text-decoration: underline;">blue</a></p>

<input type="email" aria-invalid="true" aria-describedby="email-error">
<div id="email-error">
  <svg aria-hidden="true"><!-- Error icon --></svg>
  Please enter a valid email address
</div>

<label>
  Email <span aria-hidden="true">*</span> (required)
</label>
<input type="email" required aria-required="true">
```

#### 1.4.3 Contrast (Minimum) (Level AA)

Text must have a contrast ratio of at least 4.5:1 (normal text) or 3:1 (large text).

**Requirements:**
- **Normal text (< 18pt or < 14pt bold):** 4.5:1 minimum contrast ratio
- **Large text (≥ 18pt or ≥ 14pt bold):** 3:1 minimum contrast ratio
- **Text in buttons, inputs, and UI components:** Must meet text requirements (4.5:1 or 3:1), not UI component requirements
- Logotypes and incidental text are exempt

**Examples:**
- Black text (#000000) on white background (#FFFFFF): 21:1 ✅
- Dark gray (#595959) on white: 7:1 ✅
- Medium gray (#767676) on white: 4.5:1 ✅ (minimum for normal text)
- Light gray (#949494) on white: 2.6:1 ❌ (fails for all text)
- #5061ff on white: 4.67:1 ✅ (passes for normal text)
- #7c8aff on white: 3.03:1 ❌ (fails for normal text, passes for large text only)

**Critical Distinction:**
Text inside UI components (buttons, form fields) must meet **text contrast requirements** (4.5:1 normal, 3:1 large), NOT the 3:1 UI component boundary requirement.

#### 1.4.11 Non-text Contrast (Level AA)

Visual information required to identify UI components and states must have a contrast ratio of at least 3:1.

**Requirements:**
- UI component boundaries (borders, outlines) need 3:1 against adjacent colors
- Focus indicators need 3:1 against adjacent background
- Active/hover state indicators need 3:1
- Required graphical objects need 3:1
- This does NOT apply to text within components (use 1.4.3 instead)

**Examples:**
- Input border (#767676) on white background: 4.5:1 ✅
- Focus outline (2px solid #0066cc) on white: Must be 3:1 ✅
- Disabled components are exempt

### Operable

User interface components and navigation must be operable.

#### 2.1.1 Keyboard (Level A)

All functionality must be operable through a keyboard interface.

**Requirements:**
- All interactive elements (buttons, links, form controls) must be keyboard-operable
- Custom interactive components must handle `keydown`/`keypress` events
- Enter/Space must activate buttons
- Arrow keys for components like tabs, menus, sliders
- Escape key to close modals/dialogs
- No keyboard-only hidden functionality

**Examples:**
```javascript
// Button with keyboard support
<button onClick={handleClick}>Click me</button>

// Custom interactive div (use button instead!)
<div role="button" tabIndex="0"
     onClick={handleClick}
     onKeyDown={(e) => {
       if (e.key === 'Enter' || e.key === ' ') {
         e.preventDefault();
         handleClick();
       }
     }}>
  Click me
</div>

// Modal with Escape key support
useEffect(() => {
  const handleEscape = (e) => {
    if (e.key === 'Escape') closeModal();
  };
  if (isOpen) {
    document.addEventListener('keydown', handleEscape);
    return () => document.removeEventListener('keydown', handleEscape);
  }
}, [isOpen]);
```

#### 2.1.2 No Keyboard Trap (Level A)

Keyboard focus must not be trapped in any part of the content.

**Requirements:**
- Users must be able to move focus away from all components using only keyboard
- Modals/dialogs must trap focus within themselves but allow Escape to close
- Tab and Shift+Tab must work to move focus
- Document how to exit if standard methods don't apply

**Example (Focus Trap for Modal):**
```javascript
import FocusTrap from 'focus-trap-react';

function Modal({ isOpen, onClose, children }) {
  return isOpen ? (
    <FocusTrap>
      <div role="dialog" aria-modal="true">
        <button onClick={onClose} aria-label="Close">×</button>
        {children}
      </div>
    </FocusTrap>
  ) : null;
}
```

#### 2.4.3 Focus Order (Level A)

If content can be navigated sequentially, focusable elements must receive focus in an order that preserves meaning and operability.

**Requirements:**
- Tab order must follow visual order (left-to-right, top-to-bottom)
- Don't use positive `tabindex` values (>0)
- Use `tabindex="0"` to add items to tab order
- Use `tabindex="-1"` for programmatic focus only
- Source order should match visual order

**Bad:**
```html
<button tabindex="3">Submit</button>
<input type="text" tabindex="1">
<input type="email" tabindex="2">
```

**Good:**
```html
<input type="text">
<input type="email">
<button>Submit</button>
```

#### 2.4.4 Link Purpose (In Context) (Level A)

The purpose of each link must be determined from the link text alone or from the link text together with its programmatically determined context.

**Requirements:**
- Avoid generic link text: "click here", "read more", "learn more", "more"
- Make link text descriptive of the destination
- If generic text needed, add context via `aria-label`, `aria-labelledby`, or sr-only text
- Same link text should point to same destination

**Bad:**
```html
<a href="/article/1">Read more</a>
<a href="/article/2">Read more</a> <!-- Ambiguous -->
```

**Good:**
```html
<!-- Option 1: Descriptive text -->
<a href="/article/1">Read the full article: Understanding WCAG 2.1</a>

<!-- Option 2: sr-only text -->
<a href="/article/1">
  Read more
  <span class="sr-only">about Understanding WCAG 2.1</span>
</a>

<!-- Option 3: aria-label -->
<a href="/article/1" aria-label="Read more about Understanding WCAG 2.1">
  Read more
</a>

<!-- Option 4: Link the heading -->
<h3><a href="/article/1">Understanding WCAG 2.1</a></h3>
<p>Article excerpt...</p>
```

#### 2.4.7 Focus Visible (Level AA)

Any keyboard operable interface must have a mode of operation where the keyboard focus indicator is visible.

**Requirements:**
- Focus indicators must be visible (don't use `outline: none` without replacement)
- Minimum 2px outline or equivalent
- Must have 3:1 contrast ratio against adjacent colors (WCAG 1.4.11)
- Can customize but must remain visible

**Bad:**
```css
button:focus {
  outline: none; /* No replacement! */
}
```

**Good:**
```css
button:focus {
  outline: 2px solid #0066cc;
  outline-offset: 2px;
}

/* Or custom indicator */
button:focus {
  outline: none;
  box-shadow: 0 0 0 3px rgba(0, 102, 204, 0.5);
}
```

### Understandable

Information and the operation of user interface must be understandable.

#### 3.2.1 On Focus (Level A)

When a component receives focus, it must not initiate a change of context.

**Requirements:**
- Don't automatically submit forms when a field receives focus
- Don't automatically navigate when a link receives focus
- Don't open modals when focusing an element
- Context changes should require explicit action (click, Enter, button press)

**Bad:**
```javascript
<input onFocus={() => form.submit()} /> <!-- Auto-submits on focus! -->
```

**Good:**
```javascript
<input onChange={handleChange} />
<button onClick={() => form.submit()}>Submit</button>
```

#### 3.3.1 Error Identification (Level A)

If an input error is automatically detected, the item in error must be identified and described to the user in text.

**Requirements:**
- Error messages must be text (not just color/icon)
- Associate error messages with fields via `aria-describedby`
- Use `aria-invalid="true"` on fields with errors
- Provide clear, specific error messages

**Example:**
```html
<label for="email">Email</label>
<input
  type="email"
  id="email"
  aria-invalid="true"
  aria-describedby="email-error"
>
<div id="email-error" class="error">
  <svg aria-hidden="true"><!-- Error icon --></svg>
  Please enter a valid email address in the format: name@example.com
</div>
```

#### 3.3.2 Labels or Instructions (Level A)

Labels or instructions must be provided when content requires user input.

**Requirements:**
- Every `<input>`, `<select>`, `<textarea>` must have an associated `<label>`
- Or use `aria-label` or `aria-labelledby`
- Provide format hints for complex inputs (dates, phone numbers)
- Indicate required fields clearly (not just color)

**Example:**
```html
<label for="phone">
  Phone number (required)
  <span class="hint">Format: (555) 123-4567</span>
</label>
<input
  type="tel"
  id="phone"
  required
  aria-required="true"
  placeholder="(555) 123-4567"
  pattern="\\(\\d{3}\\) \\d{3}-\\d{4}"
>
```

### Robust

Content must be robust enough to be interpreted reliably by a wide variety of user agents, including assistive technologies.

#### 4.1.2 Name, Role, Value (Level A)

For all UI components, the name and role can be programmatically determined; states, properties, and values that can be set by the user can be programmatically set; and notification of changes is available to user agents, including assistive technologies.

**Requirements:**
- Use native HTML elements when possible (inherit correct roles)
- If using custom components, add appropriate ARIA roles
- Provide accessible names via label, `aria-label`, or `aria-labelledby`
- Update `aria-*` attributes when state changes
- Use `aria-live` for dynamic content updates

**Examples:**
```html
<!-- Native button (role="button" implicit) -->
<button>Submit</button>

<!-- Custom toggle button -->
<button
  role="button"
  aria-pressed="false"
  onClick={() => setPressed(!pressed)}
>
  Toggle
</button>

<!-- Custom checkbox -->
<div
  role="checkbox"
  aria-checked="false"
  aria-labelledby="agree-label"
  tabIndex="0"
  onClick={handleToggle}
  onKeyDown={handleKeyToggle}
>
  <span id="agree-label">I agree to terms</span>
</div>

<!-- Dynamic update -->
<div aria-live="polite" aria-atomic="true">
  {itemsAdded} items added to cart
</div>
```

#### 4.1.3 Status Messages (Level AA)

Status messages can be programmatically determined through role or properties such that they can be presented to the user by assistive technologies without receiving focus.

**Requirements:**
- Use `aria-live` regions for dynamic status updates
- Use `role="status"` for important status (implicit `aria-live="polite"`)
- Use `role="alert"` for urgent messages (implicit `aria-live="assertive"`)
- Don't move focus to status messages

**Examples:**
```html
<!-- Success message -->
<div role="status" aria-live="polite">
  Your changes have been saved.
</div>

<!-- Error alert -->
<div role="alert" aria-live="assertive">
  Connection lost. Please check your internet connection.
</div>

<!-- Loading indicator -->
<div role="status" aria-live="polite" aria-busy="true">
  Loading...
</div>
```

## Additional Guidelines

### Semantic HTML Priority

Always prefer semantic HTML over ARIA:

**Use semantic HTML:**
```html
<button>Click me</button>
<nav>...</nav>
<main>...</main>
```

**Not ARIA on generic elements:**
```html
<div role="button">Click me</div> <!-- Only if necessary! -->
<div role="navigation">...</div>
<div role="main">...</div>
```

### ARIA Best Practices

1. **First rule of ARIA:** Don't use ARIA. Use native HTML.
2. **Second rule:** Don't change native semantics unless necessary.
3. **Third rule:** All interactive ARIA controls must be keyboard-usable.
4. **Fourth rule:** Don't use `role="presentation"` or `aria-hidden="true"` on focusable elements.
5. **Fifth rule:** All interactive elements must have an accessible name.

### Form Accessibility Checklist

- [ ] Every input has an associated label
- [ ] Required fields indicated with text, not just color
- [ ] Error messages associated via `aria-describedby`
- [ ] Use `aria-invalid="true"` on fields with errors
- [ ] Group related inputs with `<fieldset>` and `<legend>`
- [ ] Provide format hints for complex inputs
- [ ] Don't use `placeholder` as the only label
- [ ] Submit button is `<button type="submit">`, not `<input type="submit">`

### Keyboard Interaction Patterns

**Buttons:**
- Enter/Space to activate

**Links:**
- Enter to activate

**Tabs:**
- Arrow keys to navigate between tabs
- Tab to move into tab panel
- Space/Enter to activate (if not auto-activating)

**Modals:**
- Focus moves to modal on open
- Focus trapped within modal
- Escape closes modal
- Focus returns to trigger on close

**Dropdowns/Comboboxes:**
- Arrow keys to navigate options
- Enter/Space to select
- Escape to close
- Type to filter (if applicable)

## Color Contrast Reference

### Text Contrast Requirements

| Text Type | Size | Contrast Ratio | Example |
|-----------|------|----------------|---------|
| Normal text | < 18pt (24px) | 4.5:1 | Body copy, labels |
| Normal text | < 14pt bold | 4.5:1 | Bold body text |
| Large text | ≥ 18pt (24px) | 3:1 | Headings, large buttons |
| Large text | ≥ 14pt (18.5px) bold | 3:1 | Bold headings |

### UI Component Contrast Requirements

| Component | Contrast Ratio | Example |
|-----------|----------------|---------|
| Borders, outlines | 3:1 | Input borders, button outlines |
| Focus indicators | 3:1 | Focus rings, focus highlights |
| Icons (no text) | 3:1 | Icon-only buttons, status icons |
| UI states | 3:1 | Hover/active state indicators |

**Important:** Text inside UI components (button text, input text) must meet **text contrast requirements**, not UI component requirements.

## Testing Requirements

### Automated Testing

Required for every component/page:
- Run axe-core with WCAG 2.1 Level AA rules
- Zero violations required
- Use Playwright + axe-core for integration tests

```typescript
test('component has no accessibility violations', async ({ page }) => {
  await page.goto('/component');

  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa'])
    .analyze();

  expect(results.violations).toHaveLength(0);
});
```

### Manual Testing

Required before release:
- [ ] Keyboard navigation (Tab, Enter, Space, Arrow keys, Escape)
- [ ] Screen reader testing (NVDA on Windows, VoiceOver on macOS)
- [ ] Zoom to 200% (layout remains usable)
- [ ] High contrast mode (Windows)
- [ ] Focus indicator visibility
- [ ] Color contrast verification with tool

### Screen Readers

Test with at least one:
- **NVDA** (Windows) - Free, widely used
- **JAWS** (Windows) - Industry standard
- **VoiceOver** (macOS/iOS) - Built-in
- **TalkBack** (Android) - Built-in

## Common Violations to Avoid

1. ❌ Images without alt text
2. ❌ Buttons without accessible names
3. ❌ Form inputs without labels
4. ❌ Insufficient color contrast
5. ❌ Color as the only indicator (links, errors, required fields)
6. ❌ Keyboard inaccessible interactive elements
7. ❌ Missing or invisible focus indicators
8. ❌ Generic link text ("click here", "read more")
9. ❌ Improper heading hierarchy (skipping levels)
10. ❌ Incorrect or excessive ARIA usage
11. ❌ Non-semantic HTML (divs as buttons)
12. ❌ Focus traps without Escape key exit
13. ❌ Auto-playing audio/video
14. ❌ Time limits without user control
15. ❌ Flashing content (can cause seizures)

## Resources

- [WCAG 2.1 Quick Reference](https://www.w3.org/WAI/WCAG21/quickref/)
- [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [axe DevTools](https://www.deque.com/axe/devtools/)
- [WAVE Browser Extension](https://wave.webaim.org/extension/)
- [Inclusive Components](https://inclusive-components.design/)

## Summary

All code generated or modified by this plugin must:
- ✅ Meet WCAG 2.1 Level AA standards
- ✅ Use semantic HTML
- ✅ Be fully keyboard-operable
- ✅ Have sufficient color contrast (4.5:1 normal, 3:1 large text)
- ✅ Provide text alternatives for non-text content
- ✅ Have visible focus indicators
- ✅ Include proper labels and ARIA attributes
- ✅ Pass automated axe-core scans
- ✅ Work with screen readers
