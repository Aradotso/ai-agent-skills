---
name: agentation-visual-feedback
description: Agent-agnostic visual feedback tool for AI coding agents to identify and annotate UI elements with structured selectors
triggers:
  - add agentation feedback tool
  - set up visual annotation for AI agents
  - install agentation for element selection
  - configure agentation feedback toolbar
  - how do I use agentation to annotate UI elements
  - add visual feedback tool for AI coding
  - integrate agentation element selector
  - enable agentation annotation mode
---

# Agentation Visual Feedback

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

Agentation is an agent-agnostic visual feedback tool that helps developers capture precise UI element references for AI coding agents. It provides click-to-annotate functionality with automatic selector identification, text selection, multi-select, and structured markdown output that includes selectors, positions, and context.

## Installation

```bash
npm install agentation -D
```

For other package managers:

```bash
yarn add agentation -D
# or
pnpm add agentation -D
```

## Basic Usage

### React Integration

Add the Agentation component to your app:

```tsx
import { Agentation } from 'agentation';

function App() {
  return (
    <>
      <YourApp />
      <Agentation />
    </>
  );
}
```

### Next.js Integration

For Next.js apps, wrap in a client component:

```tsx
// components/AgentationWrapper.tsx
'use client';

import { Agentation } from 'agentation';

export function AgentationWrapper() {
  return <Agentation />;
}
```

Then add to your layout or page:

```tsx
// app/layout.tsx
import { AgentationWrapper } from '@/components/AgentationWrapper';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <AgentationWrapper />
      </body>
    </html>
  );
}
```

### Conditional Loading (Development Only)

Only load Agentation in development:

```tsx
import { Agentation } from 'agentation';

function App() {
  return (
    <>
      <YourApp />
      {process.env.NODE_ENV === 'development' && <Agentation />}
    </>
  );
}
```

## Configuration

### Basic Props

```tsx
<Agentation
  position="bottom-right"  // 'bottom-right' | 'bottom-left' | 'top-right' | 'top-left'
  theme="auto"            // 'auto' | 'light' | 'dark'
  defaultOpen={false}     // Start with toolbar expanded
/>
```

### Custom Styling

```tsx
<Agentation
  style={{
    zIndex: 10000,
    bottom: '20px',
    right: '20px'
  }}
/>
```

## Features & Usage Patterns

### Click to Annotate

1. Click the Agentation toolbar button (bottom-right by default)
2. Click any element on the page
3. Add your feedback/note in the modal
4. Copy the structured output with selectors

Example output format:
```markdown
## Element: .sidebar > button.primary

**Selector:** `.sidebar > button.primary`
**Position:** x: 120, y: 340
**Note:** This button should be blue (#0066FF) instead of green
```

### Text Selection Mode

Select specific text content for annotation:

1. Activate Agentation
2. Click and drag to select text
3. Add annotation
4. Get selector + text content in output

### Multi-Select Mode

Select multiple related elements at once:

1. Activate Agentation
2. Hold Shift (or use multi-select button)
3. Drag across multiple elements
4. Annotate all selected elements together

### Area Selection

Annotate regions or empty space:

1. Activate Agentation
2. Enable area mode
3. Drag to create a selection box
4. Annotate the entire region

### Animation Pause

Freeze animations to capture specific states:

1. Click the pause button in the toolbar
2. All CSS animations, transitions, JS animations, and videos freeze
3. Click elements in their paused state
4. Resume animations when done

```tsx
// Agentation automatically pauses:
// - CSS animations and transitions
// - requestAnimationFrame loops
// - Video playback
// - GIF animations
```

## Working with AI Agents

### Using Structured Output

When you copy annotations, Agentation provides:

```markdown
## Feedback Report

### Element 1: Navigation Button
**Selector:** `nav.header > button[aria-label="Menu"]`
**Position:** x: 24, y: 16
**Classes:** `.btn-menu`, `.mobile-only`
**Note:** Button should have hover state with 0.2s transition

### Element 2: Card Title
**Selector:** `.card-container > .card-header > h3`
**Position:** x: 340, y: 280
**Text Content:** "Featured Product"
**Note:** Font size should be 24px, currently 18px
```

### Providing Feedback to AI Agents

Best practices when using Agentation output with AI agents:

1. **Copy the full structured output** - includes selectors agents can grep for
2. **Add specific requirements** - color values, spacing, behavior
3. **Reference multiple elements** - select related components together
4. **Capture animation states** - pause animations at the exact frame you want

Example prompt to AI agent:
```
Here's feedback from Agentation:

[paste structured output]

Please update these components according to the notes. The selectors provided 
should help you locate the exact code to modify.
```

## TypeScript Support

Agentation is written in TypeScript and includes full type definitions:

```tsx
import { Agentation } from 'agentation';
import type { AgentationProps } from 'agentation';

const config: AgentationProps = {
  position: 'bottom-right',
  theme: 'dark',
  defaultOpen: false
};

function App() {
  return <Agentation {...config} />;
}
```

## Troubleshooting

### Toolbar Not Visible

**Issue:** Agentation toolbar doesn't appear

**Solutions:**
- Check z-index conflicts - default is 999999
- Verify the component is rendered in the DOM
- Check for CSS that might hide fixed position elements

```tsx
// Increase z-index if needed
<Agentation style={{ zIndex: 10000000 }} />
```

### Elements Not Selectable

**Issue:** Can't click or select certain elements

**Solutions:**
- Check for overlays or modals with higher z-index
- Verify elements aren't hidden by pointer-events: none
- Try area selection mode for hard-to-click elements

### Selectors Too Generic

**Issue:** Generated selectors like `div > div > span` aren't useful

**Solutions:**
- Add meaningful class names to your components
- Use semantic HTML elements
- Add data attributes for better selector generation

```tsx
// Better markup for Agentation
<div className="product-card" data-testid="product-card">
  <h3 className="product-title">Title</h3>
  <button className="add-to-cart-btn">Add to Cart</button>
</div>
```

### Performance Issues

**Issue:** App slows down when Agentation is active

**Solutions:**
- Only load in development environments
- Disable animation pause if not needed
- Check for extremely deep DOM trees

```tsx
// Load only in development
{process.env.NODE_ENV === 'development' && <Agentation />}
```

### Dark Mode Not Matching

**Issue:** Agentation theme doesn't match app theme

**Solutions:**
- Manually set theme prop
- Ensure prefers-color-scheme is set correctly
- Check for theme overrides in CSS

```tsx
// Force specific theme
<Agentation theme="dark" />
```

## Browser Support

Agentation requires:
- Modern desktop browser (Chrome, Firefox, Safari, Edge)
- JavaScript enabled
- No mobile support (desktop only)

## Common Integration Patterns

### With Vite

```tsx
// main.tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import { Agentation } from 'agentation';
import App from './App';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <App />
    {import.meta.env.DEV && <Agentation />}
  </StrictMode>
);
```

### With Create React App

```tsx
// index.tsx
import { Agentation } from 'agentation';
import App from './App';

root.render(
  <React.StrictMode>
    <App />
    {process.env.NODE_ENV === 'development' && <Agentation />}
  </React.StrictMode>
);
```

### With Remix

```tsx
// app/root.tsx
import { Agentation } from 'agentation';

export default function App() {
  return (
    <html>
      <head>
        <Meta />
        <Links />
      </head>
      <body>
        <Outlet />
        <ScrollRestoration />
        <Scripts />
        <LiveReload />
        {process.env.NODE_ENV === 'development' && <Agentation />}
      </body>
    </html>
  );
}
```

## License

PolyForm Shield 1.0.0 - Free for non-commercial use. Check the license for commercial usage terms.
