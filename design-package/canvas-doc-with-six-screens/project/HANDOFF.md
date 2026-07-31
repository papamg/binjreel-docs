# binjreel — implementation handoff

**Visual source of truth:** `binjreel Handoff Package.dc.html` (open in a browser)
**Tokens, ready to ship:** `tokens.css`
**Status:** v1.1 — all consumer + admin screens specified. Nothing outstanding.

---

## Read this first

1. **Work from `binjreel Handoff Package.dc.html`, not `binjreel Design Package.html`.** The
   latter is an offline bundle with every asset base64-inlined — fine for review, useless as a
   source. The `.dc.html` has readable markup and real file references.
2. **`tokens.css` is the contract.** Ship it as-is. Never hardcode a hex outside it.
3. **The mockups are static reproductions, not app code.** Don't port their markup. Read the
   geometry and values off them and build in whatever the app already uses.

---

## The three-accent system — the one thing to get right

| Token | Owns | Never |
|---|---|---|
| `--act` tangerine | Primary CTA, subscription door, progress bars, active tab, active chip, focus ring, coin-locked cells | — |
| `--free` lime | FREE/NEW badges, rewarded-ad door, "no coins", unlocked cells, streak progress, success | — |
| `--heat` pink | HOT, rank 1–2, HA-meter, ha-counts, LIVE, unread dot | A button. A price. The paywall. Rewards. |

**Fills keep their bright value in both themes; only accent *text* drops to its darker twin**
(`--act-text`, `--free-text`, `--heat-text`). That single rule is why dark and light read as
the same product.

---

## Hard rules

- **Pink never touches the paywall or Rewards.** Heat beside a price is pressure. Rewards is a
  personal ledger, not a crowd.
- **No guilt copy, no countdowns, no fake scarcity.** The streak explicitly says missing a day
  costs nothing. Coins never expire. The dismiss affordance is always visible.
- **Light mode: nothing but a filled badge or a progress bar sits on poster art.** Titles go
  below the card in `--text-primary`. Same rule for rank rails and search grids in *both*
  themes — the rank numeral overlaps the card and would occlude the first characters.
- **The player is dark in both themes.** Light chrome around lit video reads as a bug.
- **Any header over artwork gets `--scrim-header`** (118px, `z-index:10`) beneath it.
  `--scrim-hero` alone is only ~0.35 alpha at the header row and accent type fails on a photo.
- **One heat stat per hero.** Web heroes have a featured card, so the ha-chip lives there and
  the title badge row carries access/season only. Mobile has no card, so its chip stays.
- **Web hero bed is a `<video>`**, never a CSS background: `autoplay muted loop playsinline
  preload="metadata"` + `poster` + `object-fit:cover`. Pause to the poster under
  `prefers-reduced-motion`.
- **IA is fixed:** five tabs, three paywall doors in a fixed order (coins → unlimited →
  earned), four episode lock states. Restyle, don't re-architect.
- **Onboarding uses a different order on purpose.** Step 2 runs free → coins/earned →
  unlimited, so a first-run viewer learns the cheapest door first; the paywall runs coins →
  unlimited → earned, optimised for someone already mid-episode. Both are fixed. Do not
  "correct" one to match the other.

---

## Screen inventory

### Consumer mobile (390) — §2 dark, §3 light
Home · Show page · Player (dark only, both themes use it) · Paywall · Admin login ·
Forced password change — all in both themes.

### Consumer web (1440) — §4
Home (both themes) · Player with episode rail · Paywall as 860px centred modal.

### Admin backend (1440) — §5
Dashboard · Shows table w/ bulk select · Show editor + episode manager (all lock and encoding
states) · Coins & plans · light-theme variant.

### Component kit — §6 · Voice note — §7

### Rewards & Profile — §8
Rewards hub (both themes) · Profile: Unlimited state dark, Free-plan state light.

### Search & Onboarding — §9
Search results (dark) · Search idle + no-results (light) · Onboarding step 1 taste (dark) ·
Onboarding step 2 how-it-works (light). Web derivations for all four are specified in prose at
the end of §9 — no separate mockups needed.

---

## Layout numbers

```
mobile gutter 20 · web gutter 40 · web header 72 · tab bar 84 (34 safe area) · min hit 44
poster 2:3 — 116×174 mobile · 150×225 web · 168×252 web-wide · search grid 3-up @169h
hero — 520h mobile · 452–560h web · featured card 9:16 (240×427 dark / 191×340 light)
episode cell 1:1, 5-up grid, 8px gap · video 9:16 · avatar 1:1 round
admin — 260 sidebar · 64 topbar · 28 gutter · 40 table rows · tabular-nums on all figures
breakpoints 390 / 768 / 1024 / 1440
```

Below 1024: tab bar returns, web hero drops the Up Next rail, player's episode rail becomes
the mobile swipe-up drawer. Shelves always bleed past the right gutter — never centred,
never wrapped.

---

## Assets

**Fonts** — Google Fonts: `Gabarito` (400–900, display/UI/numerals), `Plus Jakarta Sans`
(400–800, body), `JetBrains Mono` (400–600, metadata/tables).

**In the mockups now — all placeholder:**
- `uploads/Screenshot 2026-07-29 *.jpg` — ten 9:16 stills standing in for poster art.
- `uploads/BRBGBlurredVing.mp4` + `uploads/hero-poster.png` — web hero bed. Decodes at
  **1280×1080** (not 16:9) with black vignette bands baked in; `cover` crops them in the wide
  hero but they'd show in a taller container. Worth a re-encode.

**Still needed for production:**
- **2:3 key art per show.** The 9:16 stills lose ~25% top and bottom in poster slots.
- **16:9 banner art per show**, if you want per-show web heroes instead of the ambient wall.
  Both can coexist: wall for browse/logged-out, show art when selling one title.
- Cast headshots (1:1), show logos/title treatments if any.

Every image slot uses `center/cover`, so swapping in real art is a URL change.

---

## Voice, for judgment calls

binjreel is a premium cinema that happens to be run by funny people. The frame is serious —
true black or clean paper, tight type, generous poster art, nothing wobbly or bubbly — and the
wit lives only in the words and in the colour. When unsure, keep the layout calm and let the
copy be the joke. Sentence case except badges and overlines. Contractions always. One joke per
screen, in the smallest text that can carry it. Numbers are facts, never bait. If a screen
feels like it's shouting, take away a colour, not a joke.
