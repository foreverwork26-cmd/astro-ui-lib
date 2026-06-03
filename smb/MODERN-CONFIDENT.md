---
name: Modern Confident
description: A generic, niche-agnostic design system for small and medium business websites. Swap the primary color and fonts to match any client's brand.
colors:
  primary: "#4F46E5"
  on-primary: "#FFFFFF"
  primary-hover: "#4338CA"
  primary-subtle: "#EEF2FF"
  secondary: "#6B7280"
  on-secondary: "#FFFFFF"
  accent: "#F59E0B"
  on-accent: "#1F2937"
  background: "#F9FAFB"
  surface: "#FFFFFF"
  surface-dim: "#F3F4F6"
  on-surface: "#111827"
  on-surface-secondary: "#4B5563"
  on-surface-muted: "#9CA3AF"
  border: "#E5E7EB"
  border-subtle: "#F3F4F6"
  error: "#DC2626"
  on-error: "#FFFFFF"
  success: "#16A34A"
  on-success: "#FFFFFF"
typography:
  display:
    fontFamily: Outfit
    fontSize: 3.5rem
    fontWeight: "700"
    lineHeight: 1.1
    letterSpacing: -0.02em
  h1:
    fontFamily: Outfit
    fontSize: 2.5rem
    fontWeight: "700"
    lineHeight: 1.2
    letterSpacing: -0.02em
  h2:
    fontFamily: Outfit
    fontSize: 2rem
    fontWeight: "600"
    lineHeight: 1.25
    letterSpacing: -0.01em
  h3:
    fontFamily: Outfit
    fontSize: 1.5rem
    fontWeight: "600"
    lineHeight: 1.3
  body-lg:
    fontFamily: Inter
    fontSize: 1.125rem
    fontWeight: "400"
    lineHeight: 1.75
  body-md:
    fontFamily: Inter
    fontSize: 1rem
    fontWeight: "400"
    lineHeight: 1.7
  body-sm:
    fontFamily: Inter
    fontSize: 0.875rem
    fontWeight: "400"
    lineHeight: 1.6
  label:
    fontFamily: Inter
    fontSize: 0.875rem
    fontWeight: "500"
    lineHeight: 1.4
    letterSpacing: 0.01em
  label-sm:
    fontFamily: Inter
    fontSize: 0.75rem
    fontWeight: "600"
    lineHeight: 1.3
    letterSpacing: 0.02em
rounded:
  sm: 4px
  DEFAULT: 8px
  md: 8px
  lg: 12px
  xl: 16px
  full: 9999px
spacing:
  base: 8px
  xs: 4px
  sm: 8px
  md: 16px
  lg: 24px
  xl: 40px
  2xl: 64px
  section: 80px
  gutter: 16px
  container-max: 1200px
components:
  button-primary:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.on-primary}"
    typography: "{typography.label}"
    rounded: "{rounded.lg}"
    padding: 12px 24px
    height: 44px
  button-primary-hover:
    backgroundColor: "{colors.primary-hover}"
  button-secondary:
    backgroundColor: transparent
    textColor: "{colors.primary}"
    typography: "{typography.label}"
    rounded: "{rounded.lg}"
    padding: 12px 24px
    height: 44px
  button-secondary-hover:
    backgroundColor: "{colors.primary-subtle}"
  card:
    backgroundColor: "{colors.surface}"
    rounded: "{rounded.xl}"
    padding: "{spacing.lg}"
  card-hover:
    backgroundColor: "{colors.surface}"
  input-field:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.on-surface}"
    typography: "{typography.body-md}"
    rounded: "{rounded.md}"
    padding: 12px 16px
    height: 44px
  badge:
    backgroundColor: "{colors.primary-subtle}"
    textColor: "{colors.primary}"
    typography: "{typography.label-sm}"
    rounded: "{rounded.full}"
    padding: 4px 12px
  nav-link:
    textColor: "{colors.on-surface-secondary}"
    typography: "{typography.label}"
  nav-link-hover:
    textColor: "{colors.primary}"
---

## Overview

Modern Confident is a versatile, niche-agnostic design system for small and medium business websites. The aesthetic is contemporary and assured — clean enough to feel professional, warm enough to feel human. It avoids both the coldness of enterprise SaaS and the casualness of consumer apps, landing in a middle ground that works for any SMB from a dental practice to a roofing company.

The personality is **competent, approachable, and direct**. Pages should feel like a firm handshake — confident without being aggressive. Whitespace is generous, hierarchy is clear, and every element earns its place on the page. The goal is conversion: every page exists to move a visitor toward a specific action (call, book, buy, inquire).

Typography pairs **Outfit** for headings (geometric, modern, high visual confidence) with **Inter** for body text (neutral, highly legible, battle-tested across screen sizes). This pairing creates a clear visual hierarchy without introducing friction between the two families.

## Colors

The palette is built on true neutrals with a single bold primary. The neutral base has no warm or cool lean — it adapts to whatever primary color is swapped in per client.

- **Primary (#4F46E5):** Indigo — used exclusively for interactive elements: buttons, links, active states. This is the conversion color. Never use it decoratively.
- **Primary Hover (#4338CA):** Darkened primary for hover and pressed states.
- **Primary Subtle (#EEF2FF):** A tinted background used for badges, highlights, and soft callouts. Derived from the primary at ~5% opacity on white.
- **Secondary (#6B7280):** Neutral gray for supporting text, icons, and secondary UI elements.
- **Accent (#F59E0B):** Warm amber reserved for urgent callouts, limited-time offers, or star ratings. Use sparingly — at most once per page.
- **Background (#F9FAFB):** The page canvas. True neutral, slightly off-white to reduce screen glare.
- **Surface (#FFFFFF):** Cards, modals, and elevated containers sit on pure white to create subtle depth against the background.
- **On-Surface (#111827):** Primary text color. Near-black for maximum readability without the harshness of `#000000`.
- **On-Surface Secondary (#4B5563):** Body text, descriptions, and supporting copy.
- **On-Surface Muted (#9CA3AF):** Placeholders, captions, timestamps, and tertiary information.

When customizing for a client, swap `primary`, `primary-hover`, and `primary-subtle` to their brand color. The neutral stack rarely needs to change.

## Typography

The type system uses two families with a strict hierarchy. **Outfit** handles display and heading levels — its geometric letterforms project confidence and modernity. **Inter** handles everything else — body text, labels, inputs, navigation — optimized for screen readability at all sizes.

- **Display / H1:** Outfit Bold. Used for hero headlines only. Large, tight letter-spacing draws the eye immediately.
- **H2 / H3:** Outfit SemiBold. Section headings and card titles. Clear hierarchy without shouting.
- **Body:** Inter Regular at 16px–18px. Generous line height (1.7) for comfortable reading on all devices.
- **Labels:** Inter Medium/SemiBold at smaller sizes. Used for buttons, navigation, form labels, and metadata. Slight letter-spacing improves legibility at 12–14px.

Never use more than three font weights on a single page. The typical combination is Outfit 700 for headings, Inter 400 for body, and Inter 500 for labels.

## Layout & Spacing

The layout follows a centered, max-width container model (1200px) with consistent gutters. All spacing derives from an 8px base grid.

- **Sections:** Major page sections (hero, features, testimonials, CTA) are separated by 80px vertical spacing to create clear breathing room.
- **Content groups:** Related elements within a section use 24px–40px gaps.
- **Component internals:** Padding inside cards, buttons, and inputs follows the 8/12/16/24 scale.
- **Mobile:** Container padding collapses to 16px. Section spacing reduces to 48px. Typography scales down by one level (display becomes h1 size, h1 becomes h2 size).

The page should never feel cramped. When in doubt, add more space, not less.

## Elevation & Depth

Depth is achieved through **tonal layering** rather than heavy shadows. The background-to-surface contrast (gray to white) provides the primary sense of elevation.

- **Level 0:** Page background (`#F9FAFB`).
- **Level 1:** Cards and containers on white (`#FFFFFF`) with a 1px `border` tone or a very soft shadow (`0 1px 3px rgba(0,0,0,0.06)`).
- **Level 2:** Modals, dropdowns, and floating elements use a slightly stronger shadow (`0 4px 12px rgba(0,0,0,0.08)`).

Avoid dark or colored shadows. Keep them small, diffused, and nearly invisible. The goal is separation, not drama.

## Shapes

The shape language is **medium-rounded** — modern and approachable without being bubbly.

- **Buttons:** 12px radius (`rounded-lg`). Substantial and clickable.
- **Cards:** 16px radius (`rounded-xl`). Soft containers that feel inviting.
- **Inputs:** 8px radius (`rounded-md`). Clean and structured, clearly distinct from buttons.
- **Badges / pills:** Full radius (`9999px`). Used for status indicators and tags.
- **Icons:** Outline style with rounded caps (1.5–2px stroke). Match the weight of body text for visual harmony.

Never mix sharp and rounded corners in the same view. All interactive elements follow the same shape language.

## Components

### Buttons

Primary buttons use the full primary color — they are the loudest element on the page and should correspond to the single most important action per viewport. Secondary buttons are ghost-style (transparent with a primary text color and primary-subtle hover fill) for supporting actions. Button height is fixed at 44px to maintain a comfortable touch target.

### Cards

Cards sit on white surfaces with a 16px radius. They use either a subtle 1px border or a soft shadow for separation — never both. Internal padding is 24px. Cards should contain a single idea or action.

### Inputs & Forms

Input fields have the same height as buttons (44px) for visual alignment in forms. They use a lighter border that darkens to the primary color on focus. Labels sit above inputs, never inside them as disappearing placeholders.

### Navigation

Navigation links use the secondary text color and shift to primary on hover. The active/current page link uses the primary color at full weight. Keep the nav to 5–7 items maximum.

### Badges

Badges use the primary-subtle background with primary text for informational tags. They use full-rounded corners to visually distinguish them from buttons. Keep badge text to 1–2 words.

## Do's and Don'ts

- **Do** use the primary color only for the single most important action per viewport — never for decoration.
- **Do** keep hero sections tight: one headline, one supporting sentence, one CTA. No paragraphs.
- **Do** alternate between background and surface tones for consecutive sections to create visual rhythm.
- **Do** maintain WCAG AA contrast ratios (4.5:1 for text, 3:1 for large text and UI elements).
- **Do** ensure every page has a clear conversion action — if you can't name it, the page lacks purpose.
- **Do** use whitespace generously. Empty space is a design element, not wasted space.
- **Don't** use more than two font families or three font weights on a single page.
- **Don't** use the accent color (amber) for more than one element per page. It's a highlight, not a second brand color.
- **Don't** use generic stock photos as filler. Every image must support the content's message or demonstrate the service.
- **Don't** center-align body text. Left-align paragraphs for readability; center-align only headlines and short taglines.
- **Don't** stack multiple CTAs of equal visual weight. One primary, the rest secondary or text links.
- **Don't** use colored or patterned backgrounds behind text-heavy sections. Rely on the neutral palette for readability.
