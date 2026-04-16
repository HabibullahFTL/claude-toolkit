---
name: frontend-design
description: Create distinctive, production-grade frontend interfaces. Automatically detects framework, styling method, component libraries, and TypeScript usage before generating code. Adapts to the project's existing conventions while producing visually striking, accessible, and responsive output.
---

This skill guides creation of distinctive, production-grade frontend interfaces that avoid generic "AI slop" aesthetics. Implement real working code with exceptional attention to aesthetic details, accessibility, responsiveness, and creative choices — while always integrating cleanly into the existing codebase.

The user provides frontend requirements: a component, page, application, or interface to build. They may include context about the purpose, audience, or technical constraints.

---

## 0. Pre-Implementation Checklist (Mandatory — run every check before writing code)

Scan the project and resolve each item before generating any code. State your findings briefly at the top of your response.

### Framework Detection

Check `package.json`, config files, and directory structure to identify the framework:

| Signal                                   | Framework                                   |
| ---------------------------------------- | ------------------------------------------- |
| `next.config.*` or `"next"` in deps      | Next.js                                     |
| `remix.config.*` or `"@remix-run/react"` | Remix                                       |
| `nuxt.config.*` or `"nuxt"` in deps      | Nuxt (Vue)                                  |
| `svelte.config.*` or `"svelte"` in deps  | SvelteKit / Svelte                          |
| `astro.config.*` or `"astro"` in deps    | Astro                                       |
| `"solid-js"` in deps                     | SolidJS                                     |
| `angular.json` or `"@angular/core"`      | Angular                                     |
| `react-native` + `nativewind`            | React Native (NativeWind) — mobile, not web |
| Only `index.html` + no bundler config    | Vanilla HTML/CSS/JS                         |

If React Native / NativeWind is detected, shift to mobile layout patterns (Flexbox column, no `hover:`, touch targets ≥ 44px). All other guidance in this skill applies to web platforms unless explicitly noted.

### Styling Method Detection

Check in this order — use the first match:

1. `"tailwindcss"` in `package.json` → **Tailwind CSS** — then detect the version:
   - **Tailwind v4**: `@import "tailwindcss"` in `globals.css` and/or `@theme { }` block present in CSS — **no `tailwind.config.js`**
   - **Tailwind v3**: `tailwind.config.js` or `tailwind.config.ts` present at the project root, with `@tailwind base/components/utilities` directives in CSS
2. `*.module.css` files or `styles/` directory → **CSS Modules**
3. `"styled-components"` or `"@emotion/react"` → **CSS-in-JS**
4. `"vanilla-extract"` or `"linaria"` → **Zero-runtime CSS-in-JS**
5. `"sass"` or `*.scss` files → **SASS/SCSS**
6. None of the above → **Plain CSS**

### Component Library Detection

Check `package.json` for these and adapt accordingly — do not reinvent what's already available:

- `"@radix-ui/*"` or `"shadcn"` → Use existing shadcn/ui components as base primitives or add/install shadcn component that is needed
- `"@mantine/*"` → Use Mantine components and `useMantineTheme`
- `"@chakra-ui/*"` → Use Chakra components and `useColorMode`
- `"antd"` or `"@ant-design/*"` → Use Ant Design components; customize with `theme.token`
- `"@headlessui/react"` → Use for accessible interactive primitives
- None → Build from scratch using semantic HTML

### TypeScript Detection

Check for `tsconfig.json`. If present:

- All components must use `.tsx` extension
- Define explicit `interface` or `type` for all props
- Use proper return type annotations (`React.FC` is optional; explicit JSX return types are preferred)
- Avoid `any` — use `unknown` + type guards where needed

### Icon Library Detection

Check `package.json` for icon libraries and use what's already installed:

- `"lucide-react"` → Use Lucide icons
- `"@heroicons/react"` → Use Heroicons
- `"@phosphor-icons/react"` → Use Phosphor
- `"react-icons"` → Use React Icons
- None → Use inline SVG only for 1–2 icons; otherwise ask the user to install one

### State Management Detection

Check `package.json` for state libraries before writing any component logic:

| Library                   | Usage pattern                                                                                                          |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `"zustand"`               | Global client state — use a store, not `useState`, for shared or persistent UI state                                   |
| `"jotai"`                 | Atomic global state — use atoms; avoid duplicating state in local `useState`                                           |
| `"@tanstack/react-query"` | Server state — use `useQuery` / `useMutation` for all data fetching; no manual `useEffect` + `useState` fetch patterns |
| `"swr"`                   | Server state — use `useSWR` for data fetching                                                                          |
| `"@reduxjs/toolkit"`      | Global store — use existing slices/selectors; do not add local state for data that belongs in the store                |
| None detected             | `useState` / `useReducer` is appropriate for local component state                                                     |

**Rule**: if a state library is present, use it for the right category of state. Do not default to `useState` for data that should live in a server-state cache (`react-query`/`swr`) or global store (`zustand`/`jotai`/`redux`).

### Animation Library Detection

Check for `"framer-motion"` or `"motion"` in `package.json`:

- If present → use **Framer Motion** (`motion/react`) for React animations
- If absent → use **CSS-only** animations (`@keyframes`, `transition`, `animation`) — do not add Framer Motion as a new dependency without asking

### Font Loading Method Detection

Check how fonts are currently loaded in the project:

| Signal                                          | Method                                                                            |
| ----------------------------------------------- | --------------------------------------------------------------------------------- |
| `import { ... } from 'next/font/google'`        | Next.js font optimization                                                         |
| `<link>` tag in `_document.tsx` or `index.html` | CDN (Google Fonts / Bunny Fonts)                                                  |
| `@font-face` in CSS                             | Self-hosted webfont                                                               |
| `fontsource` packages in `package.json`         | NPM-based font loading                                                            |
| No font setup found                             | You must add font loading using the method appropriate for the detected framework |

### Existing Component Audit

Before writing any new code, read 2–3 existing components to extract:

- Naming convention (PascalCase files, kebab-case, etc.)
- Import organization (absolute paths, barrel files, aliases like `@/components`)
- Prop pattern (object destructuring, spread props, compound components)
- Shared primitives already in use (Button, Card, Input, etc.)
- **Color usage pattern** — are components using literal color classes (`text-gray-900 dark:text-white`) or semantic token classes (`text-foreground`, `text-primary`, `bg-background`, `bg-card`)? This is critical — see Semantic Color Tokens below.

Match these conventions exactly in your output.

### Semantic Color Token Detection

Many projects — especially those using shadcn/ui or a custom design system — define semantic CSS custom properties that swap value between light and dark mode automatically. Components then use semantic utility classes instead of literal color + dark variant pairs.

**Detect this pattern by:**

1. Opening `globals.css` (or the global CSS file) and checking for a `@layer base` block that defines CSS variables on `:root` and `.dark` (or `[data-theme="dark"]`)
2. Grepping existing components for classes like `text-foreground`, `text-primary`, `bg-background`, `bg-card`, `bg-muted`, `border-border`, `text-muted-foreground`, `ring-ring` — if these appear, the project uses semantic tokens

**If semantic tokens are detected:**

- **Never use** `text-gray-900 dark:text-white` style pairs — use the project's semantic classes instead
- Map your color intentions to the existing token vocabulary:

  | Intent                    | Likely token class                    |
  | ------------------------- | ------------------------------------- |
  | Main page background      | `bg-background`                       |
  | Card / surface background | `bg-card`                             |
  | Slightly muted surface    | `bg-muted`                            |
  | Primary body text         | `text-foreground`                     |
  | Subdued / secondary text  | `text-muted-foreground`               |
  | Brand / interactive color | `text-primary` / `bg-primary`         |
  | Borders and dividers      | `border-border`                       |
  | Destructive / error       | `text-destructive` / `bg-destructive` |

- Read the full token list from `globals.css` before writing code — the project may have custom tokens beyond the table above (`text-accent`, `bg-sidebar`, etc.)
- When adding a **new** color not covered by existing tokens, add it to both `:root` and `.dark` in `globals.css`, then wire it into `@theme` (v4) or `tailwind.config.js` `theme.extend.colors` (v3) so it generates a utility class — do not use an arbitrary value for a repeated color

**If semantic tokens are NOT detected:**

- Use literal Tailwind color classes with explicit `dark:` variants: `text-gray-900 dark:text-gray-100`
- Or define semantic tokens yourself in `globals.css` if the component will be reused across the app — document the tokens you added in your response

### Clarification Protocol

If requirements are ambiguous, **do not ask multiple questions** — state your chosen direction explicitly at the start of your response and proceed. Format it as:

> **Chosen direction:** [aesthetic tone] — [one sentence rationale]. Let me know if you'd like a different direction.

Only ask a clarifying question if a decision would require significant rework if wrong (e.g., unclear data schema, unknown routing structure).

---

## 1. Implementation Strategy

### IF TAILWIND v4 IS DETECTED:

Tailwind v4 replaces `tailwind.config.js` entirely. All configuration lives in `globals.css` (or equivalent global CSS file). Read that file before touching any styles.

- **Imports**: the file uses `@import "tailwindcss"` — do not add `@tailwind` directives
- **Design tokens**: custom colors, fonts, spacing, and radii are defined in an `@theme {}` block using CSS custom properties:
  ```css
  @theme {
    --color-brand: oklch(0.65 0.22 260);
    --font-display: 'Cabinet Grotesk', sans-serif;
    --radius-card: 1.25rem;
  }
  ```
  Always extend the existing `@theme` block — do not create a second one
- **Using tokens in classes**: v4 auto-generates utilities from `@theme` tokens — `bg-brand`, `font-display`, `rounded-card` etc. are available immediately
- **Arbitrary values**: still valid for one-off bold choices (`text-[11rem]`, `bg-[#e8ff00]`)
- **Dark mode in v4**: by default `dark:` variants respond to `prefers-color-scheme: dark`. For class-based toggling, check for `@variant dark` in `globals.css`:
  ```css
  @variant dark (&:where(.dark, .dark *));
  ```
  If absent and class toggling is needed, add that line to `globals.css`
- **Semantic token check**: before writing any color class, check whether the project uses semantic CSS variable tokens (see Semantic Color Token Detection above). If it does, use `text-foreground`, `bg-background`, etc. — never `text-gray-900 dark:text-white` pairs
- **Plugins**: imported via `@plugin "..."` in CSS, not via the JS config
- **No `@apply`** — it defeats the utility-first model and creates specificity issues

### IF TAILWIND v3 IS DETECTED:

- Use Tailwind utility classes for all styling
- Prioritize color/spacing tokens from `tailwind.config.js` `theme.extend` over arbitrary values
- Use `[...]` arbitrary values for bold one-off choices (`text-[11rem]`, `bg-[#e8ff00]`)
- Check `darkMode` in `tailwind.config.js` — `'class'` means toggling adds `.dark` to `<html>`; `'media'` means it follows system preference
- **Semantic token check**: before writing any color class, check whether the project uses semantic CSS variable tokens (see Semantic Color Token Detection above). If it does, use `text-foreground`, `bg-background`, etc. — never `text-gray-900 dark:text-white` pairs
- Do not add extra `.css` files unless required for complex `@keyframes` animations
- Never use `@apply` — it defeats the utility-first model and creates specificity issues

### IF CSS MODULES ARE DETECTED:

- Use `*.module.css` files co-located with each component
- Define custom properties on `:root` for shared design tokens (colors, spacing, type scale)
- Use `composes:` sparingly and only for genuine shared base styles

### IF CSS-IN-JS IS DETECTED:

- Follow the existing library's pattern (styled-components template literals vs emotion's `css` prop)
- Define a theme object with design tokens if one doesn't exist; extend the existing one if it does
- Avoid mixing CSS-in-JS with inline `style={}` props

### IF PLAIN CSS / SCSS IS DETECTED:

- Define all design decisions as `:root` CSS custom properties for easy tweaking
- Use BEM naming (`block__element--modifier`) unless the project uses a different convention
- Scope styles tightly — avoid global selectors that will leak

---

## 2. Design Thinking

Before coding, commit to a BOLD aesthetic direction based on context:

- **Purpose**: What problem does this interface solve? Who uses it? (consumer app, enterprise tool, creative portfolio, e-commerce, data dashboard — each demands a different register)
- **Tone**: Pick a clear extreme and execute it with precision. Options: brutally minimal, maximalist editorial, retro-futuristic, organic/natural, luxury/refined, playful/toy-like, brutalist/raw, art deco/geometric, soft/pastel, industrial/utilitarian, cyberpunk/neon, Swiss grid, Japanese wabi-sabi, Y2K nostalgia, etc.
- **Audience fit**: Bold maximalism works for a creative agency. It fails for a medical records dashboard. Match aesthetic intensity to the user's emotional context.
- **Differentiation**: What is the ONE thing someone will remember about this design?

**CRITICAL**: Choose a direction and execute it with intentionality. The failure mode is not "too bold" — it is "no opinion."

---

## 3. Responsive Design (Non-Negotiable)

Every implementation must be responsive. Treat mobile as a first-class citizen, not an afterthought.

- Default to **mobile-first**: base styles target small screens; use `md:`, `lg:` (Tailwind) or `@media (min-width: ...)` to progressively enhance
- **Touch targets**: interactive elements must be ≥ 44×44px
- **Fluid typography**: use `clamp()` for headline sizes (e.g., `font-size: clamp(1.5rem, 4vw, 3.5rem)`) rather than fixed breakpoint jumps
- **Fluid spacing**: use `clamp()` for section padding and gap values where proportional spacing matters
- **No horizontal overflow**: always verify that no element causes `overflow-x` on mobile
- **Images**: use `width: 100%; height: auto` as the base; use `aspect-ratio` to prevent layout shift
- **Test breakpoints**: mentally validate at 375px (mobile), 768px (tablet), 1280px (desktop), 1920px (wide)

---

## 4. Performance (Non-Negotiable)

Performance is part of production quality. Apply these rules to every implementation:

### Images

- Always define explicit `width` and `height` (or `aspect-ratio`) on every `<img>` and media element — this reserves space before load and eliminates layout shift (CLS)
- In **Next.js**: use the `<Image>` component from `next/image` for all images
  - Add `priority` prop to the **largest above-the-fold image** (hero, banner, LCP element) — this triggers `<link rel="preload">` and directly improves Largest Contentful Paint
  - Use `fill` + a positioned wrapper for images where dimensions are unknown; always provide a `sizes` prop
  - Never use a plain `<img>` tag in a Next.js project unless the image is a base64 data URI or an external URL that cannot go through the optimizer
- In non-Next.js projects: use `loading="lazy"` on below-the-fold images; `loading="eager"` on the LCP image

### Fonts

- Always use `font-display: swap` (or `display: 'swap'` in `next/font`) — prevents invisible text during font load
- Preload the primary font file with `<link rel="preload" as="font" crossorigin>` when using self-hosted fonts
- Prefer variable fonts (single file, full weight range) over loading multiple static weight files

### JavaScript

- Do not import an entire icon library — import only the specific icons used (`import { ArrowRight } from 'lucide-react'`, not `import * as Icons`)
- Prefer CSS transitions over JavaScript-driven animations for simple hover/focus states
- In Next.js: use `next/dynamic` with `{ ssr: false }` for heavy client-only components (charts, maps, rich text editors) to keep the initial bundle small

### Avoiding Layout Shift

- Reserve space for async content (images, embeds, ads) with explicit dimensions or `aspect-ratio` before data loads
- Use skeleton loading states (see Loading States section) instead of conditional renders that change layout dimensions
- Avoid inserting content above existing content after load

---

## 5. Accessibility (Non-Negotiable)

Production-grade means accessible. Apply these rules without exception:

### Semantic HTML

- Use landmark elements: `<header>`, `<main>`, `<nav>`, `<footer>`, `<section>`, `<article>`, `<aside>`
- Use heading hierarchy correctly: one `<h1>` per page, followed by `<h2>` → `<h3>` in order
- Use `<button>` for actions, `<a>` for navigation — never `<div onClick>`

### ARIA

- Add `aria-label` to icon-only buttons and links
- Add `aria-expanded`, `aria-controls`, `aria-haspopup` to disclosure patterns (dropdowns, accordions)
- Use `role="dialog"` + `aria-modal="true"` + focus trap for modals
- Add `aria-live="polite"` to regions that update dynamically (toasts, status messages)

### Keyboard Navigation

- All interactive elements must be reachable via `Tab` and operable via `Enter`/`Space`
- Custom components (sliders, carousels) need arrow key support
- Never remove `:focus-visible` outline — style it to match the design instead

### Color Contrast

- Body text: minimum **4.5:1** contrast ratio (WCAG AA)
- Large text (≥ 18px bold or ≥ 24px regular): minimum **3:1**
- Interactive states (focus rings, active borders): minimum **3:1**
- Never convey information by color alone — pair color with shape, text, or icon

### Motion

- Wrap all non-trivial animations in `@media (prefers-reduced-motion: reduce)` — either disable them or reduce to a simple fade
- In Framer Motion: use `useReducedMotion()` hook to conditionally apply variants

---

## 6. Typography

Typography is the fastest path from generic to distinctive. Apply these rules:

### Font Selection

**Default rule**: Avoid defaulting to Inter, Roboto, Arial, or system fonts when a more distinctive choice serves the design. These fonts signal "default AI output" and dilute the aesthetic.

**Exception — when to keep Inter (or any project font)**:
If the project is already using Inter (or another neutral sans-serif) **and** the use case is data-dense, enterprise, or utility-focused (dashboards, admin panels, SaaS tools, developer tools, form-heavy UIs), Inter is a legitimate choice — it was designed for screen legibility at small sizes in complex interfaces. In this case:

- **Do not replace it** — changing the base font breaks consistency across the product
- **Differentiate elsewhere**: pair Inter with a distinctive display font for headings only (`font-display: swap`)
- Apply tight tracking (`letter-spacing: -0.03em`) on large Inter headlines to add personality
- Use weight contrast aggressively (Inter 800 vs 400) to create visual hierarchy without changing the font
- Focus all typographic expression on scale, weight, and spacing — not the face itself

The rule is not "never use Inter" — it is "never use Inter because it is the default."

### Font Loading

Load fonts using the method matching the detected framework:

- **Next.js**: `import { FontName } from 'next/font/google'` — always pass `display: 'swap'` and `subsets: ['latin']`
- **Vite / CRA / Astro**: use a `<link rel="preconnect">` + `<link rel="stylesheet">` from Google Fonts or Bunny Fonts (privacy-respecting alternative)
- **Self-hosted**: use `@font-face` with `font-display: swap` and include `woff2` format
- **NPM Fontsource**: `import '@fontsource-variable/font-name'` — use the variable font variant when available

### Typographic Pairing

For projects where typography can be expressive:

- Pair a **distinctive display font** (headlines, hero text) with a **legible body font**
- Establish a type scale: display → h1 → h2 → h3 → body → caption — with meaningful size jumps (1.25× or 1.5× ratio)
- Use `font-variant-numeric: tabular-nums` for any numeric data
- Use `text-wrap: balance` on headings to prevent orphaned words

---

## 7. Color & Dark Mode

### Color Strategy

- Commit to a dominant color palette (2–3 colors max) with one sharp accent
- **Semantic tokens first**: if the project uses semantic color tokens (`text-foreground`, `bg-background`, etc.), express all color decisions through the token layer — never bypass it with literal colors or `dark:` pairs
- When introducing a new color: add the CSS variable to both `:root` and `.dark` in `globals.css`, then expose it as a utility via `@theme` (v4) or `tailwind.config.js` `theme.extend.colors` (v3)
- Avoid even distribution — **dominant + accent + neutral** outperforms five equally-weighted colors

### Dark Mode

Check how the project handles dark mode before implementing:

**Tailwind v4:**

- Default behavior: `dark:` variants respond to `prefers-color-scheme: dark` automatically — no config needed
- Class-based toggling: look for `@variant dark (&:where(.dark, .dark *));` in `globals.css`; if missing and class toggling is required, add it
- Define dark token overrides inside the `@theme` block or using a separate `@layer base` with `[data-theme="dark"]` / `.dark` selectors:
  ```css
  @layer base {
    :root {
      --color-surface: oklch(0.98 0 0);
    }
    .dark {
      --color-surface: oklch(0.12 0 0);
    }
  }
  ```

**Tailwind v3:**

- `darkMode: 'class'` in `tailwind.config.js`: add `dark:` variants; toggling adds `.dark` to `<html>`
- `darkMode: 'media'`: `dark:` variants respond to `prefers-color-scheme` automatically

**Non-Tailwind projects:**

- CSS variables approach: define two token sets — apply the dark set inside `[data-theme="dark"]` or `@media (prefers-color-scheme: dark)` — works in any framework
- No dark mode in project: implement using `prefers-color-scheme` media query unless the user specifies class-based toggling

Semantic token naming for dark mode: use `--color-surface`, `--color-text`, `--color-border` etc. rather than `--color-white` / `--color-black` — semantic names survive theme switching; literal names do not.

---

## 8. Loading States & Error Handling

Every component that fetches data or performs async work must account for three states: loading, error, and success. Do not implement only the success state.

### Loading States

**Detect the project's loading pattern before implementing:**

- Check if the component library has a `Skeleton` component (shadcn/ui, Mantine, Chakra all do) — use it rather than building custom skeletons
- Check if the project uses React `Suspense` boundaries — if so, implement a `fallback` prop with a skeleton that matches the loaded component's layout dimensions exactly (same height, same grid structure)
- If no library skeleton exists: build a skeleton using the semantic token `bg-muted` (or equivalent) with a CSS `animate-pulse` shimmer

**Rules:**

- Skeleton dimensions must match the loaded content — a card skeleton should be the same height as a populated card; mismatched skeletons cause layout shift on load
- For lists: render 3–5 skeleton items, not 1 — a single skeleton item implies the list only ever has one entry
- For `@tanstack/react-query` or `swr`: use the library's `isLoading` / `isPending` flag, not a manual loading state variable
- Never show a blank white space while loading — it reads as broken

### Error States

- Every async component needs an error state that is visually distinct from the loading state
- For `@tanstack/react-query`: use `isError` + `error` from `useQuery`
- For React `Suspense` + `ErrorBoundary`: wrap the component tree; provide a `fallback` that shows a human-readable error message and a retry action
- Error messages must be actionable: "Failed to load posts. Try again." with a retry button — not raw error strings or empty UI
- Style error states using `text-destructive` / `bg-destructive/10` (or equivalent semantic token) so they are visually consistent with the design system

### Optimistic Updates

- For mutation-heavy UIs (forms, toggles, drag-and-drop), implement optimistic updates using `react-query`'s `onMutate` or `swr`'s `mutate` — do not wait for server confirmation before updating the UI

---

## 9. Motion & Animation

- **CSS-first**: `transition`, `@keyframes`, `animation` for most effects — no library needed
- **Framer Motion** (only if already installed): use for complex orchestrated sequences, exit animations, drag, and layout transitions
- **High-impact moments**: one well-orchestrated page-load with staggered reveals creates more delight than scattered micro-interactions
- **Hover states**: use `transition-duration: 150–200ms` — fast enough to feel responsive, slow enough to be intentional
- **Always add**: `@media (prefers-reduced-motion: reduce) { * { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; } }` in global CSS, or use `useReducedMotion()` in Framer Motion

---

## 10. Spatial Composition

- Unexpected layouts. Asymmetry. Overlap. Diagonal flow. Grid-breaking elements.
- Generous negative space OR controlled density — never the middle ground
- Use CSS Grid for two-dimensional layout; Flexbox for one-dimensional alignment
- Avoid equal gutters everywhere — use spatial rhythm (e.g., 8/16/32/64px scale)
- Backgrounds: gradient meshes, noise textures, geometric patterns, layered transparencies, grain overlays create atmosphere — default solid colors do not

---

## 11. Output Structure

Always structure your response as follows:

1. **Project scan summary** (2–4 lines): framework, styling method, component library, TypeScript, chosen aesthetic direction
2. **File tree** (if creating multiple files): show which files will be created or modified
3. **Code**: complete, copy-paste-ready implementation
4. **Design notes** (3–5 bullets): explain the key aesthetic choices made — font pairing rationale, color decisions, motion approach — so the user can adjust intentionally

---

## Anti-Patterns (Never Do These)

- `<div onClick>` instead of `<button>` or `<a>`
- Removing `:focus-visible` outline without replacing it
- `@apply` in Tailwind
- Hardcoded color hex values outside of CSS variables, `@theme {}` (v4), or `tailwind.config.js` `theme.extend` (v3)
- Font choices made without checking what's already loaded in the project
- Animations without `prefers-reduced-motion` fallback
- Fixed pixel widths on containers (use `max-width` + `width: 100%`)
- `any` type in TypeScript
- Building a component that already exists in the detected component library
- Importing an icon library that is not in `package.json`
- Using `text-gray-900 dark:text-white` literal pairs in a project that uses semantic color tokens — always use `text-foreground`, `bg-background`, etc. when the token system is in place
- Adding a new color as an arbitrary value (`bg-[#e8ff00]`) when it will be used more than once — define it as a token instead

---

Remember: Claude is capable of extraordinary creative work. State your direction, commit fully, and execute with precision. The best designs are remembered not because they were loud, but because every detail served the intent.
