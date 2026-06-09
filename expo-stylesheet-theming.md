---
name: Expo StyleSheet theming pitfall
description: Why bare imported color constants crash in StyleSheet.create, and the correct theming pattern for this project.
---

## The rule
Never use imported color constants (`BG`, `CARD`, `TEXT`, `MUTED`) inside `StyleSheet.create()` calls. Use **literal hex/rgba values** as dark-mode fallbacks instead. Apply theme overrides via inline style arrays using `const C = useColors()`.

## Why
`StyleSheet.create()` is evaluated at **module initialization time** (before any component mounts). If the imported module (`@/components/ui`) has any async/circular dependency, the constants resolve to `undefined` at that moment → `ReferenceError: BG is not defined` at runtime.

## How to apply
In every new or updated screen:
1. `import { useColors } from "@/lib/theme"` (alongside GOLD from ui.tsx)
2. `const C = useColors()` inside the component function (after hooks)
3. In `StyleSheet.create`: use literal values: `"#000000"` (BG), `"#1A1A1A"` (CARD), `"#222222"` (CARD2), `"#FFFFFF"` (TEXT), `"rgba(255,255,255,0.54)"` (MUTED)
4. In JSX: override with `[styles.container, { backgroundColor: C.BG }]`, `{ color: C.TEXT }`, etc.
5. Sub-components that render independently (like `BarberCard`, `QuickAction`) each need their **own** `const C = useColors()` call.

## Gold-tinted accent surfaces
For surfaces that use a dark gold tint in dark mode (`"#1A1200"` — e.g. total bars, reminder cards, success-icon circles), don't hardcode them. Override inline with translucent gold `GOLD + "1A"`, which renders as a subtle dark-gold tint over a dark bg and a soft light-gold tint over a light bg — so the one value works in both themes.

## Intentionally fixed (do NOT theme)
WhatsApp chat bubbles (`#0d1f0d` green) and colored status badges (waiting/sending/sent) in booking-confirm are self-contained brand-colored elements — leave them fixed in both themes.

## Color map (dark → light)
- BG: `"#000000"` → `"#F5F5F5"`
- CARD: `"#1A1A1A"` → `"#FFFFFF"`
- CARD2: `"#222222"` → `"#EEEEEE"`
- TEXT: `"#FFFFFF"` → `"#0A0A0A"`
- MUTED: `"rgba(255,255,255,0.54)"` → `"rgba(0,0,0,0.54)"`
- BORDER: `"#2A2A2A"` → `"#E0E0E0"`
- GOLD: `"#D4B15A"` (unchanged)
