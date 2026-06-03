---
name: SMB Starter Decisions
description: Architecture decision record for the SMB starter template. Every design and structural choice, its value, rationale, and rejected alternatives.
decisions:
  aesthetic:
    choice: Modern & Confident
    description: Contemporary and assured — professional without being cold, warm without being casual.
    alternatives-rejected:
      - Clean & Professional (too cold for personal service businesses)
      - Warm & Approachable (too casual for B2B-leaning SMBs)
  typography-families:
    choice: Two families — geometric display + humanist body
    display-font: Outfit
    body-font: Inter
    max-families-per-page: 2
    max-weights-per-page: 3
    alternatives-rejected:
      - Single family (too plain for client-facing demos)
      - Serif + sans (too opinionated, harder to flex across niches)
  typography-sizing:
    choice: Typographic scale in rem
    unit: rem
    scale: [0.75, 0.875, 1, 1.125, 1.25, 1.5, 2, 2.5, 3.5]
    grid-locked: false
    rationale: Font sizes follow readability, not spatial rhythm. Strict 4px increments miss critical sizes like 14px and 18px.
  color-strategy:
    choice: True neutral base + bold primary
    primary: "#4F46E5"
    base-temperature: neutral
    rationale: No warm/cool lean in the base means the primary color does all the personality work. Swap primary per client; neutrals rarely change.
    alternatives-rejected:
      - Cool neutral base (fights warm brand colors)
      - Warm neutral base (fights cool brand colors)
  primary-color:
    choice: Indigo
    value: "#4F46E5"
    rationale: More personality than default blue, still universally professional. Easy to swap per client.
    alternatives-rejected:
      - Blue #2563EB (too generic, every template uses it)
      - Teal #0D9488 (too specific, some clients find it niche)
      - Slate blue #3B5998 (too heavy for light/modern aesthetic)
  shape-language:
    choice: Medium roundedness
    buttons: 12px
    cards: 16px
    inputs: 8px
    badges: 9999px
    alternatives-rejected:
      - Subtle 4-6px (too corporate, dated feel)
      - Fully rounded 16px+ pills (too casual for B2B SMBs)
  spacing-grid:
    choice: 4px base grid
    base: 4px
    scale: [4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80]
    rationale: 8px is too coarse — real UI frequently needs 4px, 12px, and 20px steps. 4px gives every 8px value plus finer steps.
    alternatives-rejected:
      - 8px base (misses 4px micro-gaps, 12px button padding, 20px form spacing)
  layout:
    choice: Centered container model
    container-max: 1200px
    prose-max: 720px
    container-padding-mobile: 16px
    container-padding-tablet: 24px
    container-padding-desktop: 32px
  breakpoints:
    sm: 640px
    md: 768px
    lg: 1024px
    xl: 1280px
    nav-switch: lg
    rationale: Aligned with Tailwind v4 defaults. Nav switches at lg (1024px) because tablets in portrait can't fit a full horizontal nav + CTA.
  css-framework:
    choice: Tailwind CSS v4
    config: CSS-native via @theme block
    all-tokens-as-css-vars: true
    daisyui: not-included
    daisyui-upgrade-path: All tokens are CSS custom properties — daisyUI can override any variable without restructuring.
    alternatives-rejected:
      - Tailwind v3 (older config model, @theme not available)
      - Vanilla CSS (no utility classes, slower development)
      - daisyUI now (adds opinions about component structure too early)
  dark-mode:
    choice: Deferred
    rationale: Most SMB sites are light-only. Adding dark mode later is trivial via daisyUI themes or a manual prefers-color-scheme layer.
    upgrade-path: Add @media (prefers-color-scheme dark) overrides to CSS custom properties, or install daisyUI and define a dark theme.
  distribution:
    choice: degit
    command: "npx degit user/smb-starter my-project"
    rationale: Simplest distribution — no npm publishing, no registry, no CLI to maintain. Just a GitHub repo.
    alternatives-rejected:
      - npm create (requires publishing to npm, more maintenance)
      - Custom CLI (more features but more to build and maintain)
  pages:
    choice: Minimal — index + 404
    rationale: The starter is a layout shell, not a content template. Developers fill in their own pages. A minimal set avoids deleting unused boilerplate.
    alternatives-rejected:
      - Standard set with about/services/contact (more to delete than to use for many clients)
      - Full set with blog (scope creep for a starter)
  component-strategy:
    choice: Flat HTML in layouts
    rationale: No Astro components yet — keeps the template simple and avoids premature abstraction. Components will be extracted when the component library is built.
    evolution-path: Extract repeated patterns (Section, Card, Button) into src/components/ as the library matures.
  static-generation:
    choice: Astro SSG (default)
    rationale: SMB sites are content-driven with no dynamic data. Static HTML is the fastest, most secure, and cheapest to host (Netlify, Vercel, Cloudflare Pages — all free tier).
  font-loading:
    choice: Google Fonts with preconnect + swap
    display: swap
    preload: true
    rationale: font-display swap prevents invisible text during load. Preconnect + preload the heading font for fastest LCP.
  accessibility:
    choice: WCAG AA baseline
    skip-nav: required
    min-touch-target: 44px
    focus-ring: 2px solid currentColor, 2px offset
    heading-hierarchy: strict
    reduced-motion: respected
    rationale: Accessibility is a structural requirement, not an afterthought. 44px touch targets match both WCAG 2.5.8 and Apple HIG.
  elevation:
    choice: Tonal layering with minimal shadows
    level-0: background (#F9FAFB)
    level-1: surface (#FFFFFF) + 1px border or soft shadow
    level-2: stronger shadow for modals/dropdowns
    rationale: Heavy shadows feel dated. Tonal contrast (gray to white) provides clean separation.
  images:
    choice: AVIF > WebP > JPEG fallback chain
    hero-loading: eager
    below-fold-loading: lazy
    aspect-ratios-required: true
    rationale: AVIF has 95%+ browser support in 2026 and is significantly smaller than WebP. Aspect ratios prevent CLS.
  performance-targets:
    lcp: "<2.5s"
    cls: "<0.1"
    fid: "<100ms"
    rationale: Google Core Web Vitals "good" thresholds. Astro SSG + minimal JS makes these achievable out of the box.
---

## Overview

This document records every design and structural decision baked into the SMB starter template. Each entry captures the choice, its concrete value, the rationale, and the alternatives that were considered and rejected.

The purpose is twofold: (1) give AI agents the context to understand *why* the template is built this way, so they make consistent decisions when extending it, and (2) give the developer a single reference when deciding whether to override a default.

## Design Decisions

### Aesthetic Direction

**Choice:** Modern & Confident.

The aesthetic sits between corporate coldness and consumer casualness. It's designed to feel like a firm handshake — professional and competent, but human. This middle ground flexes across SMB niches: a plumbing company and a yoga studio can both feel at home here by swapping the primary color and fonts.

Rejected: "Clean & Professional" (too sterile for personal service businesses) and "Warm & Approachable" (too casual for B2B-leaning SMBs). Both are too far toward their respective poles to serve as a generic starting point.

### Typography

**Choice:** Outfit (headings) + Inter (body). Two families maximum, three weights maximum per page.

Outfit is geometric and confident — it projects modernity without being trendy. Inter is the most battle-tested body font on the web, optimized for screen readability at every size. The pairing creates clear hierarchy without friction between the families.

Font sizes use a **typographic scale in rem** (0.75 through 3.5rem), not locked to the 4px spatial grid. Font sizing is a readability problem; spacing is a rhythm problem. These are fundamentally different concerns. Strict 4px increments for font sizes would miss 14px (the standard label size) and 18px (standard body-large), both critical for real-world UI.

On mobile (below `lg`), every type level shifts one step down in the scale to prevent hero text from dominating small viewports.

### Color Strategy

**Choice:** True neutral base (#F9FAFB background) with indigo primary (#4F46E5).

The neutral base has no warm or cool lean. This means the primary color alone determines the personality of the site. When you swap `primary` to a client's brand color, the neutrals never fight it — they just recede and let the new primary take over. This is the key to making the template genuinely niche-agnostic.

Indigo was chosen over blue (#2563EB) because every generic template defaults to blue — indigo says "this isn't a Bootstrap default" while remaining universally professional. It pairs well with both warm and cool secondary accents.

### Spacing

**Choice:** 4px base grid.

All spacing values are multiples of 4px: 4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80. This covers everything from micro icon-to-text gaps (4px) to major section separation (80px).

An 8px grid was considered but rejected as too coarse. Real UI constantly needs 4px (icon gaps), 12px (button vertical padding), and 20px (form field spacing) — all impossible on a strict 8px grid. Since every 8px value is also a multiple of 4px, the 4px grid is a strict superset with no downside.

### Shape Language

**Choice:** Medium roundedness. Buttons at 12px, cards at 16px, inputs at 8px, badges at full radius.

This is the current modern default — rounded enough to feel contemporary and approachable, not so rounded that it looks bubbly or childish. The different radii for different element types create subtle visual distinction: you can tell a button from an input from a card at a glance.

## Structural Decisions

### Layout

**Choice:** Centered container at 1200px max-width, 720px prose max-width.

1200px fits within 1280px viewports (the most common laptop resolution) with room for scrollbar. 720px for prose gives 50–75 characters per line at 16px body text — the readability sweet spot from typographic research.

Container padding is responsive: 16px mobile, 24px tablet, 32px desktop. Every pixel matters on small screens; desktop can afford more breathing room.

### Breakpoints

**Choice:** sm 640px, md 768px, lg 1024px, xl 1280px. Aligned with Tailwind v4 defaults.

Navigation switches from hamburger to horizontal at `lg` (1024px). Tablets in portrait (768px) don't have enough horizontal space for a full nav bar plus CTA button, so the switch waits until there's genuinely enough room.

### CSS Framework

**Choice:** Tailwind CSS v4 with CSS-native `@theme` configuration. No daisyUI.

All design tokens are expressed as CSS custom properties (`--color-*`, `--radius-*`, `--spacing-*`). This makes the system override-friendly — if daisyUI is added later, it can remap or override any variable without touching the source CSS. The upgrade path is trivial: install daisyUI, define a theme that references the existing CSS vars, done.

### Dark Mode

**Choice:** Deferred.

Most SMB sites are light-only. Adding dark mode later requires either (a) a `@media (prefers-color-scheme: dark)` block that overrides the CSS custom properties, or (b) installing daisyUI and defining a dark theme. Both are straightforward because all tokens are already CSS variables.

### Distribution

**Choice:** degit from a GitHub repository.

No npm publishing, no registry management, no CLI to maintain. The developer runs `npx degit user/smb-starter my-project` and gets a clean copy with no git history. This is the lowest-maintenance distribution method.

### Component Strategy

**Choice:** Flat HTML in layouts. No Astro components yet.

The starter avoids premature abstraction. Every element is plain HTML with CSS classes in the layout and page files. When the component library matures, repeated patterns (Section, Card, Button) will be extracted into `src/components/`. Starting flat means nothing needs to be refactored — only extracted.

### Performance

**Choice:** Astro SSG with zero client JS (except ~20 lines for mobile nav toggle).

Static HTML is the fastest, most secure, and cheapest deployment target. Core Web Vitals targets (LCP <2.5s, CLS <0.1, FID <100ms) are achievable out of the box. Astro's island architecture means JavaScript is only loaded when a specific interactive feature demands it — opt-in, never default.

### Accessibility

**Choice:** WCAG AA baseline baked into the structure.

Skip-to-content link on every page. 44px minimum touch targets. Visible focus rings. Strict heading hierarchy. `prefers-reduced-motion` respected. Semantic HTML landmarks (`<header>`, `<nav>`, `<main>`, `<footer>`). These are not nice-to-haves — they're structural requirements encoded in the starter's HTML and CSS.
