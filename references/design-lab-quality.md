# Качество вариантов в коде

Выдержки из Design and Refine: https://github.com/0xdesign/design-plugin, файл `design-and-refine/skills/design-lab/DESIGN_PRINCIPLES.md`. Используй релевантные задаче разделы; не добавляй продуктовые ветки ради полноты справочника.

Приоритет оформления: явные указания пользователя → актуальные core-ios-style.md и code-prototype.md → совместимые существующие компоненты → эти рекомендации. Числа, цвета, шрифты и примеры из источника не заменяют токены цепочки. Для движения учитывай частоту: частые действия и клавиатурная навигация без декоративных задержек, короткие переходы 150–300 мс, reduced motion. Не добавляй библиотеку ради одного эффекта.

## Part 1: UX Foundations

### Jakob Nielsen's 10 Usability Heuristics

1. **Visibility of system status** - Always keep users informed through appropriate feedback within reasonable time
2. **Match between system and real world** - Use familiar language, concepts, and conventions
3. **User control and freedom** - Provide clear "emergency exits" (undo, cancel, back)
4. **Consistency and standards** - Follow platform conventions; same words mean same things
5. **Error prevention** - Eliminate error-prone conditions or ask for confirmation
6. **Recognition over recall** - Minimize memory load; make options visible
7. **Flexibility and efficiency** - Provide accelerators for expert users (shortcuts, defaults)
8. **Aesthetic and minimalist design** - Remove irrelevant information; every element competes
9. **Help users recover from errors** - Plain language errors with constructive solutions
10. **Help and documentation** - Provide concise, task-focused help when needed

### Don Norman's Design Principles

- **Affordances** - Design elements should suggest their usage
- **Signifiers** - Visual cues that indicate where actions should happen
- **Mapping** - Controls should relate spatially to their effects
- **Feedback** - Every action needs a perceivable response
- **Conceptual model** - Users should understand how the system works

### Cognitive Load Principles

- **Limit choices** - 5-7 items max in navigation; 3-4 options in decisions
- **Progressive disclosure** - Show only what's needed at each step
- **Chunking** - Group related items; break long forms into steps
- **Visual hierarchy** - Guide attention with size, color, contrast, position
- **Reduce cognitive friction** - Minimize decisions, clicks, and reading

### URL & State Principles

- **URL state reflection** - Important UI state (filters, tabs, pagination) should be in the URL
- **Shareable links** - Users should be able to share/bookmark the current view
- **Browser navigation** - Back/forward buttons should work as expected

### Destructive Actions

- **Confirmation required** - Delete, remove, and irreversible actions need explicit confirmation
- **Clear consequences** - State exactly what will happen ("This will permanently delete 5 files")
- **Recovery path** - Prefer soft delete with undo over immediate permanent deletion
- **Visual distinction** - Destructive buttons use warning colors (red) and distinct styling

---


### Content Handling

**Text truncation:**

```css
/* Single line truncation */
.truncate {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* Multi-line truncation */
.line-clamp-2 {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
  overflow: hidden;
}

/* Break long words */
.break-words {
  overflow-wrap: break-word;
  word-break: break-word;
}
```

**Flex children with text:**

```css
/* IMPORTANT: Flex children with text need min-w-0 to truncate properly */
.flex-child-with-text {
  min-width: 0; /* Allows text to shrink below content size */
}
```

**Empty states:**

- Always design the empty state—it's the first thing users see
- Include helpful message + primary action
- Use illustration or icon to add visual interest

**Images:**

```jsx
// Always include explicit dimensions to prevent layout shift
<img
  src="/image.jpg"
  width={800}
  height={600}
  alt="Description"
  loading="lazy"  // Defer off-screen images
/>

// For above-the-fold images
<img src="/hero.jpg" width={1200} height={800} alt="Hero" priority />
```


## Part 3: Component Patterns

### Buttons (from Stripe, Linear)

**Hierarchy:**

1. **Primary** - One per view, main action, filled with brand color
2. **Secondary** - Supporting actions, outlined or ghost style
3. **Tertiary** - Low-emphasis actions, text-only with hover state
4. **Destructive** - Delete/remove actions, red with confirmation

**States:**

- Default → Hover (+shadow or darken) → Active (scale 0.97) → Disabled (50% opacity)
- Loading: replace text with spinner, maintain width
- Min width: 80px; min height: 36px (touch-friendly: 44px)

**Best practices:**

- **Specific labels:** "Save API Key" not "Continue" or "Submit"
- Verb + noun labels: "Create project" not "Create"
- Sentence case, not ALL CAPS
- Icon left of text (or icon-only with tooltip)
- Primary button right-aligned in forms/dialogs
- **Icon buttons require `aria-label`**

**Active state feedback:**

```css
button:active {
  transform: scale(0.97);
}
```

### Forms (from Airbnb, Stripe, Vercel)

**Input anatomy:**

```
┌─────────────────────────────────┐
│ Label                           │  ← Required (above input, not inside)
│ ┌─────────────────────────────┐ │
│ │ Placeholder...              │ │  ← Format hint only, ends with ...
│ └─────────────────────────────┘ │
│ Helper text or error message    │  ← Specific and actionable
└─────────────────────────────────┘
```

**Autocomplete attributes (required):**

```html
<!-- Always use appropriate autocomplete for user data -->
<input type="email" autocomplete="email" />
<input type="text" autocomplete="name" />
<input type="text" autocomplete="given-name" />
<input type="text" autocomplete="family-name" />
<input type="text" autocomplete="organization" />
<input type="text" autocomplete="street-address" />
<input type="text" autocomplete="postal-code" />
<input type="tel" autocomplete="tel" />
<input type="password" autocomplete="current-password" />
<input type="password" autocomplete="new-password" />
<input type="text" autocomplete="one-time-code" />
```

**Input types and modes:**

```html
<!-- Use correct type for validation and keyboard -->
<input type="email" inputmode="email" />
<input type="tel" inputmode="tel" />
<input type="url" inputmode="url" />
<input type="number" inputmode="numeric" />

<!-- Numeric input without spinners -->
<input type="text" inputmode="numeric" pattern="[0-9]*" />
```

**Disable spellcheck where inappropriate:**

```jsx
// Disable for codes, emails, usernames, URLs
<input type="text" spellCheck={false} autoComplete="username" />
<input type="email" spellCheck={false} />
<input type="text" spellCheck={false} placeholder="Enter code..." />
```

**Anti-patterns to avoid:**

```jsx
// NEVER block paste - this is hostile UX
<input onPaste={(e) => e.preventDefault()} />  // ❌ NEVER DO THIS

// NEVER use placeholder as label
<input placeholder="Email" />  // ❌ Placeholder disappears on focus

// NEVER validate on every keystroke
onChange={(e) => validateEmail(e.target.value)}  // ❌ Too aggressive
```

**Best practices:**

- Labels above inputs (not inside—accessibility)
- Placeholder ≠ label; use for format hints only, end with `...`
- Inline validation on blur, not on every keystroke
- Error messages: specific and actionable ("Email must include @")
- **Focus first error field** after form submission fails
- Success state: checkmark icon, green border (brief)
- Required fields: mark optional ones instead of required
- Single column forms outperform multi-column

**Unsaved changes warning:**

```jsx
// Warn users before leaving with unsaved changes
useEffect(() => {
  const handleBeforeUnload = (e) => {
    if (hasUnsavedChanges) {
      e.preventDefault();
      e.returnValue = '';
    }
  };
  window.addEventListener('beforeunload', handleBeforeUnload);
  return () => window.removeEventListener('beforeunload', handleBeforeUnload);
}, [hasUnsavedChanges]);
```

### Cards (from Material, Apple)

**Anatomy:**

```
┌────────────────────────────────┐
│ [Media/Image]                  │  ← Optional
├────────────────────────────────┤
│ Eyebrow · Metadata             │  ← Optional
│ Title                          │  ← Required
│ Description text that can      │  ← Optional
│ wrap to multiple lines...      │
├────────────────────────────────┤
│ [Actions]              [More]  │  ← Optional
└────────────────────────────────┘
```

**Best practices:**

- Entire card clickable for primary action
- Consistent padding (16-24px)
- Image aspect ratios: 16:9, 4:3, 1:1 (be consistent)
- Limit to 2 actions max; overflow to menu
- Hover: subtle lift (translateY -2px + shadow increase)

### Tables (from Linear, Notion)

**Best practices:**

- Left-align text, right-align numbers
- **Use `tabular-nums` for numeric columns** (consistent width digits)
- Zebra striping OR row hover, not both
- Sticky header on scroll
- Sortable columns: show current sort indicator
- Actions: row hover reveals action buttons (or kebab menu)
- Empty state: helpful message + action
- Pagination vs infinite scroll: pagination for data accuracy, infinite for browsing
- Min row height: 48px for touch; 40px for dense
- **Virtualize tables with >50 rows**

```css
.numeric-column {
  font-variant-numeric: tabular-nums;
  text-align: right;
}
```

### Navigation (from Apple HIG, Material)

**Patterns by scale:**

- **2-5 items**: Tab bar / horizontal tabs
- **5-10 items**: Side navigation (collapsible)
- **10+ items**: Side nav with sections/groups

**Best practices:**

- Current location always visible
- Breadcrumbs for deep hierarchy (not for flat structures)
- Mobile: bottom nav for primary actions (thumb-friendly)
- Icons + labels together; icon-only needs tooltip
- Consistent order across pages

---


### Feedback Patterns (from Dan Saffer's Microinteractions)

**Every action needs feedback:**

1. **Immediate** - Button press visual (scale, color change)
2. **Progress** - Loading states for anything >1s
3. **Completion** - Success confirmation (toast, checkmark, animation)
4. **Failure** - Clear error with recovery path

**Loading states:**

- 0-100ms: No indicator needed
- 100-300ms: Subtle change (opacity, skeleton)
- 300ms-1s: Spinner or progress bar
- 1s+: Skeleton screens + progress indication
- 10s+: Background processing with notification

### State Handling

**Every component needs these states:**

```
Default    → Base appearance
Hover      → Interactive hint (cursor change, highlight)
Focus      → Keyboard navigation (visible ring)
Active     → Being pressed/activated
Loading    → Async operation in progress
Disabled   → Not available (reduce opacity, remove pointer)
Error      → Invalid input or failed operation
Success    → Completed successfully (brief)
Empty      → No data to display (helpful message + action)
```


### Timing Guidelines

| Element Type | Duration | Notes |
|---|---|---|
| Micro-interactions | 100-150ms | Buttons, toggles, hover states |
| Tooltips, dropdowns | 150-250ms | Small UI appearing |
| Modals, drawers | 200-300ms | Larger surfaces |
| Page transitions | 300-400ms | Full view changes |
| Staggered items | 30-50ms delay | Between each item |

**Important:** Exit animations should be 20-30% faster than entrances.


## Part 6: Accessibility Essentials

### WCAG Quick Reference

**Perceivable:**

- Color contrast: 4.5:1 text, 3:1 UI components
- Don't rely on color alone (add icons, patterns)
- Text resizable to 200% without loss
- Captions for video; transcripts for audio

**Operable:**

- All functionality via keyboard
- No keyboard traps
- Skip links for repeated content
- Touch targets: 44x44px minimum

**Understandable:**

- Consistent navigation
- Identify input errors clearly
- Labels and instructions for forms

**Robust:**

- Semantic HTML elements
- ARIA only when HTML isn't enough
- Tested with screen readers

### Focus Management

**Use `:focus-visible` over `:focus`:**

```css
/* ✅ Only show focus ring for keyboard navigation */
:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

/* ❌ Don't remove outline without replacement */
:focus {
  outline: none; /* BAD - removes accessibility */
}
```

**Compound controls:**

```css
/* Highlight parent when any child is focused */
.input-group:focus-within {
  box-shadow: 0 0 0 2px var(--color-primary);
}
```

### Keyboard Navigation

**All interactive elements must be keyboard-operable:**

```jsx
// ❌ Click-only interaction
<div onClick={handleAction}>Click me</div>

// ✅ Keyboard accessible
<button onClick={handleAction}>Click me</button>

// ✅ If must use div, add keyboard support
<div
  role="button"
  tabIndex={0}
  onClick={handleAction}
  onKeyDown={(e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      handleAction();
    }
  }}
>
  Click me
</div>
```

**Keyboard patterns:**

- Tab order must match visual order
- Enter/Space activate buttons and links
- Escape closes dialogs and dropdowns
- Arrow keys navigate within components (tabs, menus)

### ARIA Patterns

**Icon buttons require `aria-label`:**

```jsx
// ❌ No accessible name
<button><CloseIcon /></button>

// ✅ Accessible
<button aria-label="Close dialog"><CloseIcon /></button>
```

**Form controls require labels:**

```jsx
// ❌ No label
<input type="email" placeholder="Email" />

// ✅ Visible label
<label>
  Email
  <input type="email" />
</label>

// ✅ Or visually hidden label
<label htmlFor="email" className="sr-only">Email</label>
<input id="email" type="email" placeholder="email@example.com" />
```

**Live regions for async updates:**

```jsx
// Announce dynamic content to screen readers
<div aria-live="polite" aria-atomic="true">
  {statusMessage}
</div>
```

**Semantic HTML before ARIA:**

```jsx
// ❌ ARIA role when native element exists
<div role="button" tabIndex={0}>Submit</div>

// ✅ Use native element
<button>Submit</button>

// ❌ ARIA for native functionality
<div role="navigation">...</div>

// ✅ Use native element
<nav>...</nav>
```

**Common ARIA patterns:**

```html
<!-- Modal -->
<div role="dialog" aria-modal="true" aria-labelledby="modal-title">
  <h2 id="modal-title">Dialog Title</h2>
</div>

<!-- Tab panel -->
<div role="tablist">
  <button role="tab" aria-selected="true" aria-controls="panel1">Tab 1</button>
</div>
<div role="tabpanel" id="panel1">Content</div>

<!-- Loading state -->
<button aria-busy="true" aria-describedby="loading-text">
  <span id="loading-text" className="sr-only">Loading...</span>
</button>
```

---


## Part 8: Content & Copy

### Writing Style

**Active voice over passive:**

```
✅ "Install the CLI"
❌ "The CLI will be installed"

✅ "Your changes were saved"
❌ "Changes have been saved by the system"
```

**Title Case for headings and buttons:**

```
✅ "Save API Key"
❌ "Save api key"

✅ "Getting Started"
❌ "Getting started"
```

**Use numerals:**

```
✅ "8 deployments"
❌ "eight deployments"

✅ "3 items selected"
❌ "three items selected"
```

### Labels & Messages

**Specific labels over generic:**

```
✅ "Save API Key"
❌ "Continue"

✅ "Create Project"
❌ "Submit"

✅ "Delete Repository"
❌ "Confirm"
```

**Error messages include fix/next step:**

```
✅ "Email must include @ symbol"
❌ "Invalid email"

✅ "Password must be at least 8 characters"
❌ "Password too short"

✅ "Could not connect. Check your internet connection and try again."
❌ "Network error"
```

### Internationalization

**Use Intl APIs for formatting:**

```jsx
// ❌ Hardcoded format
const date = `${month}/${day}/${year}`;
const price = `$${amount.toFixed(2)}`;

// ✅ Locale-aware formatting
const date = new Intl.DateTimeFormat('en-US', {
  dateStyle: 'medium'
}).format(new Date());

const price = new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD'
}).format(amount);

// Relative time
const rtf = new Intl.RelativeTimeFormat('en', { numeric: 'auto' });
rtf.format(-1, 'day'); // "yesterday"
```

---


## Part 9: Anti-Patterns Checklist

Flag these patterns during design review:

### Accessibility Violations

- [ ] `user-scalable=no` or `maximum-scale=1` in viewport meta
- [ ] `<div onClick>` instead of `<button>` for interactive elements
- [ ] Form inputs without associated labels
- [ ] Icon buttons without `aria-label`
- [ ] `outline: none` without focus replacement

### Performance Issues

- [ ] `transition: all` (animates layout properties)
- [ ] Images without explicit `width` and `height`
- [ ] Large arrays (>50 items) rendered without virtualization
- [ ] Layout reads (`getBoundingClientRect`) in render cycle

### UX Problems

- [ ] `onPaste` with `preventDefault()` (blocks paste)
- [ ] Hardcoded date/number formats (not using Intl)
- [ ] Placeholder used as label
- [ ] Validation on every keystroke
- [ ] No empty state designed

### Mobile Issues

- [ ] Touch targets smaller than 44x44px
- [ ] No `touch-action: manipulation` on buttons
- [ ] Hover effects without `@media (hover: hover)` query

### Interface Robustness Checklist

Test every interactive component against these stress conditions:

**Rapid User Input:**
- [ ] Scroll fast — does it break or show visual glitches?
- [ ] Spam click — does it trigger multiple actions or crash?
- [ ] Resize window rapidly — does layout break or cause errors?
- [ ] Type very fast in inputs — does it lag or lose characters?

**Animation Interruption:**
- [ ] Interrupt animations mid-way — does it recover gracefully?
- [ ] Close modal while opening — does it handle state correctly?
- [ ] Navigate away during transition — does it clean up properly?
- [ ] Trigger multiple animations simultaneously — do they conflict?

**Network Conditions:**
- [ ] Test with slow network (3G throttling) — does it show loading states?
- [ ] Test with offline mode — does it fail gracefully?
- [ ] Test with request timeout — does it retry or show error?

**Input Methods:**
- [ ] Test with keyboard only — is everything accessible?
- [ ] Test with screen reader — are states announced?
- [ ] Test with touch device — are hit areas adequate?
- [ ] Test with trackpad gestures — do scroll/swipe work correctly?

**State Combinations:**
- [ ] Test all combinations of loading + error + empty states
- [ ] Test disabled state during async operations
- [ ] Test hover + focus + active simultaneously
- [ ] Test with maximum and minimum content

**Memory & Performance:**
- [ ] Run for extended period — does memory grow unbounded?
- [ ] Test with large datasets — does it virtualize or paginate?
- [ ] Monitor for memory leaks in animations and observers
- [ ] Check for event listener cleanup on unmount

---

