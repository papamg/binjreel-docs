# BUILD-REPORT.md — binjreel · RESTYLE ORDER (the ratified design package, live)

**Closed-loop record.** 2026-07-29/30 · Hat: 4B Build → 4C Deploy · Order: RESTYLE — conform
the live web face to the ratified design package · Scope: **app/web only** (tokens, styles,
assets, component visuals; no IA, no API, no ledger, no features) · Branch: `binjreel-dev`.

*Per Blueprint §10 this file is written per engagement. The v0.3+v0.4 report's substance is
preserved in `BUILD-LEDGER.md` L-030…L-040 and the SECURITY-REPORT sections (never
overwritten).*

---

## The record, in one line

The design package was unpacked into the permanent visual record, transcribed into a binding
DESIGN.md v2, and the deployed face at **https://binjreel.com** now wears it — tokens, fonts,
the five briefed surfaces, and the component kit — with all three suites green and zero
behavior drift; side-by-sides sit in `docs/design-package/conformance/` for the Owner's eye.

## The steps, with evidence

- **STEP 0 — unpacked & inventoried (L-041).** `Canvas doc with six screens-handoff.zip` →
  `docs/design-package/canvas-doc-with-six-screens/` (zip removed from cubby). Token sheet
  (`tokens.css` v1.0 FINAL) complete and unambiguous — hex both themes, named fonts, type
  scale, space, radii, elevation, motion. No STOP needed.
- **STEP 1 — the contract.** `app/web/DESIGN.md` rewritten as **v2**: faithful token
  transcription, font loading strategy (Google Fonts, preconnect + display=swap), per-screen
  conformance notes, the §7 voice note **verbatim**, and every order-scope divergence recorded
  (dark-only ship; ha-stats absent from the API so never faked; five-tab IA kept per the
  order over §4's web header; wall copy stays the server's verbatim; player side-rail stays
  the restyled drawer). DESIGN.md v2 + the package are the binding contract; package wins.
- **STEP 2 — the conform (L-042).** `src/lib/theme.ts` rewritten to the package palette /
  type / radii / 4pt space / motion; Gabarito + Plus Jakarta Sans + JetBrains Mono loaded via
  a post-export inject step (`scripts/inject-html.mjs`, wired into `export:web` and the
  Dockerfile — Expo's `output:"single"` ignores `+html.tsx`). All five briefed surfaces
  conformed pixel-close (home hero/shelves/Top-10 medals · show page with the kit episode
  grid · glass-chromed player with act progress bar and 2400ms auto-hide · the three-door
  wall, sheet <1024 / 860px modal ≥1024 · admin login + forced change), plus the component
  kit states. Routes, API, behavior: unchanged.
- **FOUND & FIXED under the restyle:** the raw `<video>` element is invisible to
  react-native-web's responder system, so stage presses (tap chrome-toggle, tap-to-hold,
  touch swipe) had been **dead in browsers since v0.4** — masked because chrome never
  auto-hid. The package's 2400ms auto-hide would have stranded viewers chrome-less. Web stage
  input now runs on DOM pointer listeners (the file's existing wheel/keys pattern); the
  Pressable path stays for native v0.5 behind Platform guards. Verified headless, mouse and
  touch: show → auto-hide → tap returns → hold pauses.
- **STEP 3 — proof.** **API 147/147** (ephemeral PG17+Redis7, torn down) · **web server
  9/9** (incl. the served-bundle secret scan) · **Playwright admin-gate 4/4** (real bundle,
  real browser). Conformance pairs shot from the **deployed** site beside their mockups:
  `docs/design-package/conformance/01…08-*.side-by-side.png` (08 forced-change drives the
  hash-identical bundle locally — reaching it live would spend the Owner's one-time temp
  credential, which stays theirs).
- **STEP 4 — deploy + verify (L-043).** Web image rebuilt, container healthy. Live checks
  through the real edge: served bundle hash == tree (`entry-5fafbd2d…`) · Google Fonts links
  live at root · served bundle secret scan **0 hits** · `/health` 200 (API untouched) ·
  `/v1/app-config` intact · admin ops bare 401 · bad login 401 · www→apex 301 · show-page
  SEO title injection intact · viewer home renders with live catalog.

## Divergences (all recorded in DESIGN.md v2 §9)

Dark-only (no toggle exists; light tokens carried in the contract) · ha-stats slots styled
but empty until the API grows the numbers · five tabs at all widths (order's "no IA changes"
outranks the §4 web header — a later pass) · hero bed video / production key art still owed
(package "Assets"); art slots render the fallback treatment · purchase doors keep the web
"Get the app — coming soon" state.

## Stats

| | |
|---|---|
| Ordered budget | ≤ $40 |
| **Estimated spend** | **≈ $25** — estimated from session token volume; this box has no direct API meter (flagged, not hidden) |
| Tests | 147 API + 9 web server + 4 e2e — all green, none modified |
| New dependencies | 0 |
| API / routes / schema diff | **0 lines** |
| New secrets | none |

## GATE-3 — the Owner's viewing is the test

Taste is the acceptance. Open **https://binjreel.com** on your device (hard-refresh if it
looks cached): browse home (medals, badges, the tangerine/lime/pink system) · open a show ·
play "Fired & Furious" Ep 1 · let the player chrome hide, tap it back, hold to pause · hit
the wall on a locked episode (three doors, door out visible) · glance at /admin (login card
only; your forced change still awaits your first login). The side-by-sides are at
`docs/design-package/conformance/` — 08 explains why the forced-change pair was shot from
the same bundle rather than live.
