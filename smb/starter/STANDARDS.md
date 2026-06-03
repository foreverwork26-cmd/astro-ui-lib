---
name: SMB Starter Standards
description: Structural presets and implementation standards for small and medium business websites. Defines layout, responsive behavior, navigation, performance, and accessibility requirements.
layout:
  container-max: 1200px
  prose-max: 720px
  container-padding:
    mobile: 16px
    tablet: 24px
    desktop: 32px
breakpoints:
  sm: 640px
  md: 768px
  lg: 1024px
  xl: 1280px
spacing:
  base: 4px
  scale:
    - 4px
    - 8px
    - 12px
    - 16px
    - 20px
    - 24px
    - 32px
    - 40px
    - 48px
    - 64px
    - 80px
sections:
  vertical-spacing:
    desktop: 80px
    mobile: 48px
  alternating-backgrounds: true
typography-scale:
  unit: rem
  sizes:
    - 0.75rem
    - 0.875rem
    - 1rem
    - 1.125rem
    - 1.25rem
    - 1.5rem
    - 2rem
    - 2.5rem
    - 3.5rem
  mobile-reduction: one-level-down
navigation:
  switch-breakpoint: lg
  mobile:
    pattern: hamburger-overlay
    position: fixed-top
    overlay: full-width
    animation: slide-down
    touch-target: 44px
  desktop:
    pattern: horizontal-sticky
    position: sticky-top
    max-items: 7
    cta-placement: right
images:
  formats:
    - avif
    - webp
    - fallback-jpg
  loading: lazy
  hero-loading: eager
  aspect-ratios:
    hero: "16/9"
    card: "3/2"
    avatar: "1/1"
performance:
  lcp-target: 2.5s
  cls-target: 0.1
  fid-target: 100ms
  font-display: swap
  font-preload: true
  critical-css: inline
accessibility:
  min-touch-target: 44px
  focus-ring: "2px solid currentColor, 2px offset"
  skip-nav: required
  landmarks:
    - header
    - nav
    - main
    - footer
  heading-hierarchy: strict
  color-contrast: WCAG-AA
  reduced-motion: respected
---

## Overview

This document defines the structural implementation standards for SMB starter websites. It covers layout, responsive behavior, navigation patterns, image handling, performance targets, and accessibility requirements. These presets represent modern best practices as of 2026 and are designed to work with any visual identity defined in a companion DESIGN.md file.

The philosophy is **mobile-first, performance-conscious, and accessibility-native**. Every structural decision prioritizes fast load times on mobile networks, comfortable interaction on touch devices, and compliance with WCAG AA standards.

## Layout

The layout uses a **centered container model** with a maximum width of 1200px. This width accommodates comfortable reading on large monitors without creating uncomfortably wide line lengths, while leaving enough room for multi-column layouts.

- **Container max-width:** 1200px. Content never stretches beyond this on any screen.
- **Prose max-width:** 720px. Long-form text blocks (paragraphs, articles) are constrained further for optimal line length (50–75 characters per line at body font size).
- **Container padding:** Responsive — 16px on mobile (every pixel counts), 24px on tablet, 32px on desktop. This provides breathing room from screen edges at every size.
- **Content centering:** The container is horizontally centered with `margin-inline: auto`. Never offset or left-aligned on desktop.

### Why these values

The 1200px container fits comfortably within a 1280px viewport (the most common laptop resolution) with room for scrollbar and browser chrome. The 720px prose width is derived from the readability research standard of 50–75 characters per line at 16px body text.

## Breakpoints

Breakpoints align with Tailwind v4 defaults and correspond to real device categories.

| Token | Value | Target devices |
|:------|:------|:---------------|
| `sm` | 640px | Large phones in landscape, small tablets |
| `md` | 768px | Tablets in portrait (iPad Mini, standard tablets) |
| `lg` | 1024px | Tablets in landscape, small laptops |
| `xl` | 1280px | Standard laptops and desktops |

Design mobile-first: write base styles for the smallest screen, then layer on complexity at each breakpoint. Never design desktop-first and remove features for mobile.

### What changes at each breakpoint

- **Below `sm`:** Single column. Full-width cards. Stacked navigation. Section spacing at 48px.
- **`sm`:** Two-column grid becomes available for cards. Container padding increases to 20px.
- **`md`:** Three-column layouts for feature grids. Tablet-optimized spacing.
- **`lg`:** Navigation switches from hamburger to horizontal. Full section spacing (80px). Prose container activates at 720px.
- **`xl`:** Container hits max-width (1200px) and centers. No layout changes — just more whitespace on the sides.

## Spacing

All spacing values derive from a **4px base grid**. Every margin, padding, gap, and offset is a multiple of 4px. This creates a consistent visual rhythm that the eye perceives as "designed" rather than "thrown together."

The spacing scale covers the full range of needs:

| Token | Value | Typical use |
|:------|:------|:------------|
| `4px` | 4px | Micro-adjustments, icon-to-text gap |
| `8px` | 8px | Tight padding, inline element gaps |
| `12px` | 12px | Button padding (vertical), input padding |
| `16px` | 16px | Card padding (compact), list item gaps |
| `20px` | 20px | Internal section padding |
| `24px` | 24px | Card padding (standard), content group gaps |
| `32px` | 32px | Large content gaps, section sub-headers |
| `40px` | 40px | Content group separation |
| `48px` | 48px | Mobile section spacing |
| `64px` | 64px | Desktop content blocks |
| `80px` | 80px | Desktop section vertical spacing |

### Why 4px and not 8px

An 8px grid is clean but too coarse for real-world UI. You frequently need 4px for icon-to-label gaps, 12px for button vertical padding, and 20px for form field spacing. A 4px base gives you every value you'd need from an 8px grid (since 8, 16, 24, etc. are all multiples of 4) plus the finer steps in between.

## Typography Scale

Font sizes follow a **typographic scale in rem**, not the spatial grid. Font sizing is a readability concern, not a spatial rhythm concern — these are fundamentally different problems.

| Size | rem | px equivalent | Role |
|:-----|:----|:-------------|:-----|
| `xs` | 0.75rem | 12px | Fine print, captions, timestamps |
| `sm` | 0.875rem | 14px | Labels, secondary text, metadata |
| `base` | 1rem | 16px | Body text default |
| `lg` | 1.125rem | 18px | Body large, lead paragraphs |
| `xl` | 1.25rem | 20px | Small headings, card titles |
| `2xl` | 1.5rem | 24px | H3 headings |
| `3xl` | 2rem | 32px | H2 headings |
| `4xl` | 2.5rem | 40px | H1 headings |
| `5xl` | 3.5rem | 56px | Display / hero headlines |

### Mobile type scaling

On screens below `lg`, every type level shifts **one step down** in the scale. A desktop `display` (3.5rem) becomes a mobile `h1` (2.5rem). This prevents hero text from dominating the small viewport while maintaining clear hierarchy.

### Why not grid-locked font sizes

Strict 4px increments for font sizes would give you 12, 16, 20, 24, 28, 32... — missing 14px (the most common label/small-text size) and 18px (the standard comfortable body-large). Typography scales are designed around readability and hierarchy, not spatial alignment. The spacing grid handles spatial rhythm; the type scale handles information hierarchy.

## Navigation

Navigation is the highest-impact structural decision on an SMB site. Most visitors will interact with the nav before anything else.

### Mobile (below `lg` / 1024px)

- **Pattern:** Hamburger icon (three horizontal lines, 44px touch target) in the top-right of a fixed header.
- **Behavior:** Tapping the hamburger reveals a full-width overlay panel that slides down from the header. The panel contains vertically stacked nav links with generous 48px row height for comfortable thumb tapping.
- **CTA:** The primary call-to-action (e.g., "Get a Quote") appears as a full-width button at the bottom of the overlay, visually separated from the nav links.
- **Scroll lock:** When the overlay is open, the page body does not scroll. The overlay itself scrolls if the nav list is long.
- **Close:** Tapping the hamburger again (now an X icon) or tapping outside the overlay closes it. Pressing `Escape` also closes it.

### Desktop (`lg` and above / 1024px+)

- **Pattern:** Horizontal navigation bar, sticky to the top of the viewport on scroll.
- **Behavior:** Links are arranged horizontally with equal spacing. The active/current page is indicated by the primary color and a 2px bottom border.
- **CTA:** The primary call-to-action sits at the far right as a filled button, visually distinct from the nav links.
- **Max items:** Keep navigation to 5–7 items. More than 7 creates decision paralysis and forces smaller touch targets.
- **Logo/brand:** Left-aligned, links to the homepage.

### Why `lg` (1024px) for the switch

Tablets in portrait (768px) don't have enough horizontal space for a full nav bar plus a CTA button without cramping. The switch at 1024px ensures the horizontal nav only appears when there's genuinely enough room for it to breathe.

## Sections

Page sections are the primary vertical rhythm unit. They create the visual "chapters" of a page.

- **Vertical spacing:** 80px between sections on desktop, 48px on mobile. This creates unmistakable separation without needing horizontal rules or dividers.
- **Alternating backgrounds:** Consecutive sections alternate between `background` and `surface` colors (e.g., light gray → white → light gray). This provides visual separation and rhythm without adding visual noise.
- **Section inner padding:** 80px vertical on desktop (matching the gap), 48px on mobile. This ensures content doesn't crowd the edges of its section.
- **Full-bleed sections:** Hero and CTA sections should span the full viewport width. Content inside them still respects the container max-width.

## Images

- **Format priority:** Serve AVIF first (smallest file size, 2026 browser support at 95%+), WebP as fallback, JPEG as final fallback. Use Astro's `<Image>` component or `<picture>` element with `srcset`.
- **Loading:** All images below the fold use `loading="lazy"`. The hero image (above the fold) uses `loading="eager"` and should be preloaded in the `<head>` for LCP.
- **Aspect ratios:** Define aspect ratios on image containers to prevent layout shift (CLS). Hero: 16/9. Cards: 3/2. Avatars: 1/1.
- **Sizing:** Always provide `width` and `height` attributes. Use Astro's built-in image optimization for automatic resizing and format conversion.

## Performance

These targets align with Google's Core Web Vitals thresholds for a "good" user experience.

| Metric | Target | What it measures |
|:-------|:-------|:-----------------|
| LCP | < 2.5s | Largest Contentful Paint — how fast the main content loads |
| CLS | < 0.1 | Cumulative Layout Shift — visual stability during load |
| FID | < 100ms | First Input Delay — responsiveness to first interaction |

### Implementation requirements

- **Font loading:** Use `font-display: swap` to prevent invisible text during font load. Preload the primary heading font (Outfit 600/700) in the `<head>`.
- **Critical CSS:** Inline above-the-fold CSS. Astro handles this automatically in production builds.
- **JavaScript:** Minimal. The only JS in the starter is the mobile nav toggle (~20 lines). No frameworks on the client unless the developer adds them.
- **Static generation:** Astro defaults to static HTML output (SSG). Every page is pre-rendered at build time with zero client-side JavaScript unless explicitly opted in.

## Accessibility

Accessibility is not optional — it's a structural requirement baked into the starter from the ground up.

- **Touch targets:** All interactive elements (buttons, links, nav items) have a minimum 44x44px touch target area. This matches both WCAG 2.5.8 and Apple's HIG recommendation.
- **Focus rings:** All focusable elements display a visible 2px focus ring using `currentColor` with a 2px offset. Never remove focus outlines without providing an alternative.
- **Skip navigation:** A "Skip to content" link is the first focusable element on every page. It's visually hidden until focused (keyboard navigation), then jumps focus to `<main>`.
- **Landmarks:** Every page has exactly one `<header>`, `<nav>`, `<main>`, and `<footer>`. Screen readers use these landmarks for quick navigation.
- **Heading hierarchy:** Headings follow strict order — one `<h1>` per page, then `<h2>`, `<h3>`, etc. Never skip levels (e.g., `<h1>` → `<h3>`).
- **Color contrast:** All text meets WCAG AA minimum contrast ratios (4.5:1 for normal text, 3:1 for large text and UI components).
- **Reduced motion:** Animations and transitions are wrapped in `@media (prefers-reduced-motion: no-preference)`. Users who prefer reduced motion see no animations.

## Do's and Don'ts

- **Do** design mobile-first — base styles target the smallest screen, breakpoints add complexity.
- **Do** use semantic HTML elements (`<header>`, `<nav>`, `<main>`, `<section>`, `<footer>`) instead of generic `<div>` wrappers.
- **Do** provide `alt` text for every image. If the image is decorative, use `alt=""` and `role="presentation"`.
- **Do** test with keyboard navigation — every interactive element must be reachable and operable without a mouse.
- **Do** use Astro's `<Image>` component for automatic optimization, format conversion, and responsive sizing.
- **Don't** use fixed pixel widths for layout containers. Use `max-width` with percentage or viewport-relative fallbacks.
- **Don't** rely on color alone to convey information (e.g., red for errors). Always pair color with text or icons.
- **Don't** disable zoom or set `maximum-scale=1` in the viewport meta tag.
- **Don't** load JavaScript frameworks (React, Vue, etc.) unless a specific interactive feature requires them. Astro's island architecture makes this opt-in.
- **Don't** use layout shift-prone patterns like images without dimensions, dynamically injected banners, or fonts that cause reflow.
