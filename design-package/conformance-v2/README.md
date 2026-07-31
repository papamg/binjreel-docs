# Conformance v2 — GATE-3 REDIRECT (conformance completion)

Governing contract: `docs/design-package/canvas-doc-with-six-screens/project/HANDOFF.md`, in full,
plus `binjreel Handoff Package.dc.html` and `tokens.css`. Scope: `app/web` only. Dark-only ship
stands (Owner-ratified); the light tokens stay transcribed in `tokens.css` and `src/lib/theme.ts`
for the later toggle.

Everything here is produced mechanically by `app/web/scripts/conformance.mjs`:

1. **Reference halves** are screenshotted out of the package's own `.dc.html`, per
   `data-screen-label` frame, at each frame's declared size — not cropped by hand.
2. **Live halves** come from the **deployed** site (`https://binjreel.com`) at **390 and 1440**.
3. **Pairs** are composed `mockup | deployed`.

```
docker run --rm --network host -v /docker/binjreel:/repo -w /repo/app/web \
  mcr.microsoft.com/playwright:v1.49.1-noble \
  sh -c "npx playwright install chrome && node scripts/conformance.mjs"
```

Real Chrome, not bundled Chromium: Playwright's Chromium has no H.264, so the hero bed and the
player stage would both capture as still posters. Same constraint drives `e2e/resume.spec.mjs`.

## Side-by-sides

| Pair | Package frame | Deployed |
|---|---|---|
| `home--390.side-by-side.png` | §2.1 Home / dark | `/` @390, full length |
| `home--1440.side-by-side.png` | §4.1 Web home / dark | `/` @1440, full length |
| `show--390.side-by-side.png` | §2.2 Show page / dark | `/show/{id}` @390, full length |
| `player--390.side-by-side.png` | §2.3 Player / dark | `/watch/{id}` @390, chrome woken |
| `player--1440.side-by-side.png` | §4.3 Web player | `/watch/{id}` @1440, docked episode rail |
| `paywall--390.side-by-side.png` | §2.4 Paywall / dark | locked episode, server offer verbatim |
| `paywall--1440.side-by-side.png` | §4.4 Wall, 860px centred modal | same, @1440 |
| `rewards--390.side-by-side.png` | §8.1 Rewards hub / dark | `/rewards` @390, coming-soon state |
| `profile--390.side-by-side.png` | §8.3 Profile / dark | `/profile` @390 |
| `search-results--390.side-by-side.png` | §9.1 Search results / dark | `/search` @390, query "petty" |
| `admin-login--390.side-by-side.png` | §2.5 Admin login | `/admin` @390 |

`live--*` and `mockup--*` are the halves. Surfaces with **no package frame at that width** are
listed in `UNPAIRED.txt` — the package specifies their web derivations in prose (§9 tail), not as
mockups: For You, My List, search idle/empty at 1440, Show @1440, Rewards/Profile @1440.

## Automated gate — `app/web/e2e/assets-gate.spec.mjs`

The miss this redirect exists to cure was invisible to the suite: tokens shipped onto surfaces
whose art slots were all `NULL`. The gate now asserts it mechanically, against the deployed site,
at **both** widths, on **Home · Show · For You · Search**:

- every `<img>` has a non-empty `src`, decodes to a non-zero `naturalWidth`, and its URL answers
  `200` with an `image/*` content type;
- every `<video>` resolves **both** its `poster` and its source;
- every `/art/*` CSS background resolves `200`;
- a surface with **no** image slot at all fails — "no art anywhere" is the original defect, not a pass.

Two further tests assert the hero bed's hard rule as attributes: `autoplay muted loop playsinline
preload=metadata` + `poster` + `object-fit:cover`, and that `prefers-reduced-motion` swaps the
`<video>` for the still poster.

## Hard rules — swept explicitly

| Rule | Verified |
|---|---|
| **Pink never touches the paywall or Rewards — and never a button** | `grep -rn "colors\.heat" src/` returns **two** lines, both in `PosterCard.tsx`: rank 1 solid, rank 2 stroked. Nothing else in the app uses heat. Rewards is lime (earn, when a door is actually open) / tangerine (balance + unlimited); the wall has no heat token. The ONE sanctioned heat-on-text outside crowd metrics is Profile's `Sign out`, per the review. **Caught by this sweep:** the show page's Follow button turned `--heat-text` when active — pink on a thing you tap. Now `--act-text`, which is what the token table gives active controls. |
| **`--scrim-header` (118px, z-index 10) under any header over artwork** | `WebHeader` carries its own 118px scrim; the mobile home hero, the show hero and the player all render the 118px `scrimHeader` band. |
| **One heat stat per hero** | Web hero's badge row carries **access only** (the episode count moved to the meta line, so the two no longer duplicate) — no heat. See divergence D-1: the ha-chip has no data source, so nothing pink is invented. Mobile hero likewise carries no invented figure. Top-10 rank medals still use heat, from real ranks. |
| **The player is dark in both themes** | `styles.shell`/`styles.stage` are `#000`; all floating chrome is `--glass`; the docked rail sits on `--bg-base`. No light chrome anywhere in `watch/[episodeId].tsx`. |
| **No guilt copy, no countdowns, no fake scarcity** | Rewards states "Miss a day and nothing bad happens", "Coins never expire". The wall's dismiss ("Back to the show") is always visible. Search's no-results offers a live way onward. No countdown or timer copy on any surface. |
| **Shelves bleed past the right gutter, never centred, never wrapped** | Every shelf `ScrollView` uses `paddingLeft: contentInset(width)` and `paddingRight: 0`, inside a full-bleed container. |
| **IA is fixed — five tabs, restyle not re-architect** | The five TABS are the mobile contract and are unchanged: the 84px tab bar below 1024. At ≥1024 the 72px header carries the wider nav §4.1 draws (Home · Comedy · Blowing up ● · Free · Rewards · My List) with the avatar as Profile — restored on the Owner's instruction, and every item resolves to a real route (`/genre/comedy`, `/trending`, `/free`), none invented as dead nav. Three paywall doors, fixed order. |

## Recorded divergences

**D-1 — the hero ha-chip carries no number.** §4.1 puts a `12.4K LAUGHING` heat chip on the
featured card. The product records no laugh/ha count — there is no such field in `schema.prisma`
and no endpoint that returns one. Per the package's own voice rule ("Numbers are facts, never
bait") the chip is omitted rather than filled with a fabricated figure. Lands with the engagement
feature; the featured card is otherwise built to spec (9:16, 240×427 at a 560 hero).

**D-2 — no "Go unlimited" in the web header.** §4.1 shows a coin pill and a "Go unlimited" button.
*Superseded in part by the punch-list pass:* the coin pill now ships in BOTH headers, reading the
real `/v1/wallet/balance` — a balance of 0 is a fact, so nothing had to be invented. "Go unlimited"
is still absent: it is a purchase CTA with no economy behind it, and the right cluster is sized to
what's there rather than left with §2.14's hole. Lands with the economy (v0.5/v0.6).

**D-3 — the animated webp is not wired.** HANDOFF lists
`uploads/BRBGBlurredDarkVingflickerHigh.webp` under "in the mockups now" and the order says "the
webp where it fits". It is a **20 MB animated** webp. No shipped surface can carry that weight, and
`hero-poster.png` (1 MB, the same source) is the specified reduced-motion fallback. Not shipped;
recorded rather than silently dropped.

**D-4 — cast headshots render initials.** HANDOFF's "Still needed for production" lists cast
headshots, so the package ships no placeholder for them. The fallback renders **no `<img>`**, so it
cannot mask a slot that should have art, and the assets gate is not weakened by it.

**D-5 — 9:16 stills in 2:3 poster slots.** As HANDOFF warns, the ten placeholder stills lose ~25%
top and bottom under `center/cover` in poster slots. Swapping in real 2:3 key art is a URL change
(`prisma/seed.ts`, `still()`).

**D-6 — light theme not shipped.** Owner-ratified. Light tokens remain transcribed.

## Deferred-to-later conformance debt (logged by order, not built)

- **Onboarding (§9.3, §9.4)** — no onboarding feature exists. **Deferred to v0.5.**
- **Coins & plans admin (§5.4)** — no coin or plan management exists. **Deferred to v0.7.**
- **Admin §5 screens beyond the panel** — Dashboard, Shows table w/ bulk select, Show editor,
  Viewers, Moderation, Audit log are §5 mockups with no product behind them. The existing `/admin`
  panel is restyled onto §5's system (260 sidebar · 64 topbar · 28 gutter · 40 table rows ·
  tabular-nums); the absent screens are not drawn as dead nav. **Deferred with their features.**
- **Admin panel capture** — the signed-in panel needs the Owner's credential, which is theirs to
  spend at GATE-3. `admin-login` is captured; the panel is asserted structurally by
  `e2e/admin-gate.spec.mjs` (4/4).

## Residual data artifacts — NEEDS OWNER DECISION

While verifying, the API suite was run through the `app` service, which inherits the **live**
`DATABASE_URL`. Its `resetDatabase()` TRUNCATEs every table including `tenants`, so the deployed
catalog, the operator account and every media ref were destroyed mid-gate. Restored: reseeded,
app restarted to re-resolve the tenant, and the one real Bunny clip
(`b56ffcb4-6df8-4ce6-81e3-1ef908384ebf`) re-attached to *Fired & Furious* ep 1, with the suite's
placeholder `bunny_ref`s cleared so no episode advertises media that 404s.

**Guard added so it cannot recur:** `test/helpers.ts` now refuses to truncate a database that is
not named like a test database (override: `ALLOW_DESTRUCTIVE_DB_RESET=1`), and a compose `test`
service runs the suite against `${POSTGRES_DB}_test`:

```
docker compose run --rm test          # 147/147, isolated database
```

**Two artifacts remain in the live database and need the Owner's call** — removing them is a
destructive write to production data, so it was not performed unilaterally:

1. A duplicate shelf `slug = 'new'` ("New on binjreel", `order_index` 2) — visible as a second,
   identical shelf on Home in `home--390.side-by-side.png`.
2. A draft series `Unannounced Pilot` — `publish_state = 'draft'`, so absent from every viewer
   read and invisible on all surfaces.

Neither is created by `prisma/seed.ts`. Suggested remediation, once approved:

```sql
DELETE FROM shelf_items WHERE shelf_id IN (SELECT id FROM shelves WHERE slug = 'new');
DELETE FROM shelves WHERE slug = 'new';
DELETE FROM series  WHERE title = 'Unannounced Pilot';
```

## Suites at this gate

| Suite | Result |
|---|---|
| API (`docker compose run --rm test`) | **147 / 147** |
| Web server unit (`npm test` in `web/`) | **9 / 9** |
| Admin gate (`e2e/admin-gate.spec.mjs`) | **4 / 4** |
| Assets gate (`e2e/assets-gate.spec.mjs`) | **4 / 4** — new |
| DL-B009 resume (`e2e/resume.spec.mjs`) | **2 / 2** — new, 390 + 1440 |

Served == merged: the deployed bundle and a clean local `expo export` produce the same entry hash.

---

# Punch-list pass (Owner review of conformance-v2, waves 1–4)

Re-shot after the pass; all pairs above are the current deployed face.

## Wave 1 — cross-cutting
| # | Item | Done |
|---|---|---|
| 0.1 | Coin glyph app-wide | `components/CoinGlyph.tsx` — a DRAWN rotated diamond (not a codepoint, which is why the old `◉`/`*` varied by font). Wired into episode cells, paywall door 1, player rail + drawer, Rewards balance, Profile stat, both headers' coin pills. In `--act`/`--act-text` per the review; `--coin` gold stays for admin table money. |
| 0.2 | Lime ≠ disabled | `SoonPill`/`SoonButton` in `components/Chrome.tsx` are the ONLY way to say coming-soon, on `--surface-2` + `--text-secondary`. Rewards' three earn cards and the wall's earned door are now neutral; lime returns the moment a door actually opens (`live` flag / `adAvailable`). |
| 0.3 | Mono for prose | Paywall door sub-line, player `Ep 1 · Episode 1` and `Episodes 1 / 6` moved to Plus Jakarta Sans. Mono kept for counts, timecodes, table figures, overlines. |
| 0.4 | Duplicate titles | `useCollapsingTitle` — the bar title is the H1's collapsed state: opacity 0 at rest, fades in past 48px. Home drops its stack header entirely. |
| 0.5 | Wordmark | One `Wordmark` component, two-tone at every width. |
| 0.6 | 2-line card titles | `numberOfLines={2}` at 1.25 line-height with the space for BOTH lines reserved, so meta baselines align across a row. |
| 0.7 | Tab icons | `components/TabIcon.tsx` — five drawn icons, fill when active. The label is drawn in the same component: react-navigation's web label kept collapsing to ~4px no matter what heights the navigator got, so `tabBarShowLabel` is off and the tab item is one box we own. |
| 0.8 | Tab bar 84 | `height: TABBAR_H` + `minHeight`, 50 content + 34 safe area. |
| 0.9 | Section rhythm | Shelf-to-shelf 28 mobile / 48 web; title-to-first-card 12. |
| 0.10 | 2-show catalogue | **Seeded to ten** (`prisma/seed.ts`) — 82 episodes, all ten stills in play, `freeEpisodeCount` deliberately varied INCLUDING shows with none. Cards were left at 116×174. |
| 0.11 | Duplicate shelf | Still present — needs your approval (see "Residual data artifacts" above). |
| 0.12 | `-:--` duration | Unresolved: only one fixture episode has encoded media, a 12.8s clip, and duration is read off the element. Data, not chrome. |

## Wave 2 — restored components
Paywall: UP NEXT card w/ still + overline + meta · icon tile per door · real buttons (`Unlock` /
`Start 7 days free` / `Earn`), disabled not text-links · `$6.99 / month` at 24/900 in `--act-text` ·
`Cancel anytime` · footer `EP 1–2 ALWAYS FREE, FOREVER` · duplicate balance line deleted · glass ✕
over the paused frame · the paused frame itself now shows (a cold deep link gets the locked
episode's own still under `--scrim-hero` instead of flat black). **Web wall is now a 3-column 860px
grid, centred on the VIEWPORT, over a full-viewport scrim.**

Player: **full-bleed** (`object-fit: cover` — `contain` was letterboxing into the black bands) ·
back chevron + show title + `Ep N · title` + ⋮ overflow · social rail = 56px circles (Clip it /
List / Eps) · playback settings moved to a horizontal pill row (`CC ON` / `1×` / `Auto` /
`Share this moment`) · 4px scrubber with a 14px knob · grab handle + centred count · bottom scrim so
white-on-video reads · at ≥1024 the mobile rail is gone, the stage is CENTRED 9:16, and the rail is
a 380w `--surface-1` panel with 72h cards and 44×62 thumbs + keyboard hints.

Profile: identity row · 3-up stat row · §8.3 Free-plan card · both settings groups (`Sign out` in
`--heat-text`, the one sanctioned heat-on-text) · build stamp. Copy compressed out of release-notes
voice.

Search: scope tabs (`Shows N` live; Episodes/People/Tags disabled rather than showing a count no
endpoint can answer) · real 3-up grid · magnifier, clear ✕, `Cancel`.

Rewards: balance card gets the tangerine treatment and a real figure · notice moved out of it ·
icon tiles · 7-segment streak · tangerine on the BUTTON not the card.

Home: floating glass header over the hero (wordmark · coin pill · 44px round search) — the search
FIELD is gone, search is a destination · `--scrim-hero` + an 80px foot fade · full-width 56h CTA +
56×56 `+` · genre chip rail as its own row · `All` / `View all` + pager circles · `● LIVE` on heat
shelves · `NO COINS` on free shelves · continue-watching progress bar + `Ep N · Ms left` · badges
capped at one, priority HOT > FREE > NEW, suppressed on free shelves · rank numerals 90/110px
overlapping the card · 16×4 active dot.

## Wave 3 — geometry
Web nav restored to the wider set (Home · Comedy · Blowing up ● · Free · Rewards · My List) — and
`/trending` + `/free` were built so nothing links nowhere: both reuse existing reads (`/v1/top10`,
the server's own free-to-binge shelf). Header gutter 40, search 180w/44h, coin pill closes §2.14's
gap with the REAL balance. Hero title wraps to 3 lines at a 620 measure and never ellipsizes; the
featured card's title is gone (§4.1 carries badges only); the badge row is access-only so it no
longer duplicates the meta line. Web `My List` ghost button added.

## Not done — and why
- **§2.11 / §3.9 / §9.3 / §1.13's ha-rate metas** — every one needs the ha/laugh count. Still no
  such field (D-1). Omitted rather than fabricated.
- **§3.7 caption/quote block, §3.8 hint pill, §10.7 admin error toast** — P3, unbuilt, no data or
  no failure path exercised at this gate.
- **§4.9 rail range tabs** — not reachable at 12 episodes; required before a 20+ episode show.
- **§8.2 `HA'S GIVEN`** — replaced with `ON YOUR LIST`, which is a real figure.
- **§10** admin login deltas (top-anchored lockup, `Keep me in` row, glow anchor, 52h inputs, r20)
  — deferred: it was the closest surface to spec and the budget went to P1s.
