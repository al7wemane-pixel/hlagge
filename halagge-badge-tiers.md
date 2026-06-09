---
name: Halagge barber rank badges
description: Where the bronze/silver/gold barber rank emblems live and the duplicated tier logic that must stay in sync.
---

## Emblem assets
Three metallic rank emblems (barber tools — razor/scissors/comb crest): `assets/images/badge-bronze.png` (مبتدئ), `badge-silver.png` (متوسط), `badge-gold.png` (محترف). Transparent PNGs. Displayed as a medal overlapping the barber photo on the home card and profile hero.

## Tier is now driven by the DB `rank` string (single source)
Tier presentation (emblem img / color / scale) lives in ONE place: `rankTier(rank)` in `lib/catalog.ts`. `rank` is a DB string `"gold" | "silver" | "bronze"` (anything else → bronze). The old score-based `getBadge`/`badgeScore` logic was removed.

- `app/(tabs)/index.tsx` (home list) calls `rankTier(barber.rank)` and uses `tier.labelAr/labelEn`.
- `app/barber.tsx` (profile) calls `rankTier(barberRank)` for the emblem, plus a LOCAL `tierLabel(rank, lang)` helper for the label because `rankTier` only exposes ar/en (no tr). The home card passes `rank` as a route param so the profile badge stays consistent with the list.

**Why:** ranks come from the admin dashboard now; deriving from a fake per-barber `badgeScore` made the list and profile disagree. **How to apply:** to change a tier's emblem/color/scale, edit `rankTier` only. For new tr/ma rank wording, extend the local `tierLabel` in barber.tsx (and add tr/ma to `rankTier` if you want the home card localized beyond ar/en).

## Per-tier presentation
bronze (#E8893B, scale 0.82, smallest), silver (#7FC7E8, scale 1.0, medium), gold (#FFD700, scale 1.3, largest). Emblem base size multiplied by `tier.scale` at each render site.

## Points / loyalty state (lib/points.tsx)
Customer loyalty points live in a React context `PointsProvider`/`usePoints` (`lib/points.tsx`), wrapped in `app/_layout.tsx` inside `I18nProvider`. In-memory only (resets on reload), initial 340. `loyalty.tsx` consumes it (was a hardcoded `USER_POINTS=340` constant). `shareApp(lang)` uses RN `Share.share`, awards `SHARE_REWARD` (30) once per calendar day (returns "earned"|"already"|"cancelled"), and prepends a localized entry to `history`. On web (no native share sheet) the catch path treats the intent as shared so the demo still awards. Share entry points: a card in loyalty.tsx + a SettingRow in (tabs)/settings.tsx.
