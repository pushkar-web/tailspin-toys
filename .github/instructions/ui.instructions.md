---
description: 'Central UI strategy and component development philosophy'
---

# UI Component Strategy

This file defines the central UI development strategy for Tailspin Toys. Technology-specific guidance is in separate instruction files.

## Commenting and Documentation Philosophy

### Comment Intent, Not Mechanics

Comments should explain **why** a piece of code exists or clarify non-obvious decisions — not restate what the code already says. Remove comments that merely paraphrase the line below them.

**Bad:**
```ts
// Increment the counter
count++;

// Check if the rating is valid
if (rating > 0 && rating <= 5) {
  // Store the rating in the database
  saveRating(rating);
}
```

**Good:**
```ts
count++;

// Ratings must be in the 1-5 range; the UI enforces this, but we double-check at the data layer.
if (rating > 0 && rating <= 5) {
  saveRating(rating);
}
```

### When to Comment

- **Design decisions**: Why a non-obvious approach was taken
- **Workarounds**: Why we didn't use the "obvious" solution
- **Edge cases**: Why certain values or paths are handled specially
- **Non-local context**: When code references behavior elsewhere in the system

### Document API Contracts

- **Functions**: Use TSDoc/JSDoc to document purpose, parameters, and return values
- **Components**: Document the `Props` interface so the component API is self-explanatory
- **Data structures**: Comment complex types to clarify their structure and usage

### Keep Comments Current

Treat outdated comments as bugs — update or delete them in the same change that touches the related code.

## Component Architecture

### Technology Separation

- **Astro** (`.astro` files): Pages, layouts, components, routing, and static content. The site is fully prerendered (`output: 'static'`), so components render to HTML at build time.
- **Tailwind CSS** (utility classes): Styling
- **Astro `<script>`**: Reach for a small client-side script only when genuine interactivity is required — there is no client-side UI framework.

Refer to technology-specific instruction files:
- [`astro.instructions.md`](astro.instructions.md) - Astro pages, layouts, and components
- [`style.instructions.md`](style.instructions.md) - Tailwind CSS styling patterns

## Core Principles

### Testability

- Every interactive element MUST include a `data-testid` attribute
- Use descriptive test IDs that identify the element's purpose and context
- Examples: `data-testid="game-card-{game.id}"`, `data-testid="submit-button"`, `data-testid="nav-home"`

### Accessibility

- Use semantic HTML elements (`<nav>`, `<main>`, `<article>`, `<button>`)
- Provide ARIA labels and roles where semantic HTML isn't sufficient
- Use plain `<nav>` with `<a>`/`<button>` elements for site navigation — do **not** add `role="menu"`. Reserve `role="menu"` / `role="menuitem"` for true application-style menus that implement full composite keyboard semantics (arrow-key roving focus, Home/End, type-ahead)
- Loading states should use `role="status"` and `aria-live="polite"` for screen reader announcements
- Include Escape key handlers for dismissible elements (menus, modals)
- Ensure keyboard navigation works for all interactive elements, with proper focus management
- Include visible focus states: `focus:ring-2 focus:ring-blue-500 focus:outline-none`
- Maintain sufficient color contrast (especially in dark theme)

### Design Consistency

- Dark theme throughout the application
- Modern, clean UI with rounded corners and smooth transitions
- Consistent spacing and visual hierarchy
- Responsive design that works on mobile, tablet, and desktop

### Component Reusability

- Create reusable components for common UI patterns
- Keep components focused on a single responsibility
- Use props for configuration, not duplication
- Document component APIs with TypeScript types

## Development Workflow

1. **Choose the right tool**: 
   - Content & structure → Astro components/pages
   - Styling → Tailwind
   - Client interactivity (rare) → a scoped Astro `<script>`

2. **Follow technology-specific patterns**: 
   - Refer to the appropriate instruction file

3. **Ensure testability**: 
   - Add `data-testid` to all interactive elements

4. **Verify accessibility**: 
   - Test keyboard navigation
   - Check focus states
   - Validate semantic structure
