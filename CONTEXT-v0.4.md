# CONTEXT-v0.4.md — binjreel · v0.4 (the web face — player, browse, admin seed, LIVE at binjreel.com)

*Stage contract. Serves TDD.md v2 §9, §11(4), D-018/D-019. Department: **4B Build → 4C Deploy**.
Runs only after the v0.3 DoD is green (same engagement, phase-gated — see the kickoff Order).
The v0.3 contract (`docs/CONTEXT.md`) remains the prior phase's Order; this file is the live
contract once v0.3's gate closes.*

---

## Target

**The first visible product, live on the real domain.** An Expo (React Native) codebase whose
**web build deploys at https://binjreel.com** — vertical gapless player, full browse experience,
the three-door wall (ad door rendered as "coming soon"/fail-closed), and a seeded admin login
with a thin operator panel. Native iOS/Android builds of this same codebase are **v0.5** — do
not attempt store builds, IAP, push, or deferred deep links in this version.

## Ratified frame (do not re-litigate)

- **One codebase, web face first (D-019).** The video player is the only platform-split module;
  build the web implementation now (HLS playback — hls.js or equivalent where MSE is needed),
  leaving the native module seam explicit and documented.
- **The server contract is v0.3 — consume it, never bypass it.** No client-side entitlement
  logic; the wall renders the server offer payload verbatim; playback uses grants + the
  re-mint seam; no media URL is ever constructed client-side.
- **Design is the Worker's to author in this Order** (Owner directive 2026-07-28): skew
  ReelShort/NetShort — dark theme, vertical poster cards with badges, hero carousel, Top-10
  rank medals, shelf-scroll home — voiced for **comedy** (bold, bright accent color, playful
  type; an offer-never-a-scold wall). Read `/mnt/skills`-equivalent house frontend guidance if
  present on the box; record the design tokens in `app/web/DESIGN.md` so a later Claude Design
  restyle has a source of truth. GATE-3 visual verification is the Owner's, live.
- **Fail-closed inheritance.** Web build ships with no secrets; it talks only to the public API.
  The ad door in the wall renders from the offer payload's `adUnlock:false` as a disabled
  "watch-an-ad — coming soon" door (shape proven end to end, economy lands v0.6).

## Build scope — what v0.4 IS

1. **App shell (Expo, web output):** five-tab structure — Home · For You · Rewards · My List ·
   Profile. Rewards tab renders a "coming soon" state fed by config (the tab exists so the
   information architecture is final; the economy lands v0.5/v0.6). Anonymous session on first
   visit (existing endpoint); sign-in optional; the merge flow wired.
2. **Home:** hero carousel from hero slots; shelves from the shelf-composition endpoint
   (Continue Watching, New, Top-10 with rank badges, genre hubs Comedy-first, tag shelves,
   Free to Binge); poster cards with badges.
3. **Show pages:** full composition per v0.3 — cover, synopsis, tag chips (tappable), cast
   block, episode grid with per-caller lock states, Start/Resume, related. Server-rendered or
   pre-rendered enough that **show pages carry SEO metadata** (title/description/OpenGraph).
4. **Tag pages, genre hubs, search** (with trending), **My List + Following + history** UI.
5. **The player:** full-bleed vertical, swipe up/down between episodes, tap-to-hold, episode
   drawer, captions toggle (tracks from v0.3 manifests), speed 0.75–2×, quality selector
   honoring the ladder, **prefetch-fed gapless transitions** (next entitled episode buffered
   before the swipe), **mid-session re-mint** (playback never dies to TTL), progress reporting
   as she watches, resume on return. No referer dependency anywhere.
6. **The wall:** triggered at the swipe into a locked episode; renders the server offer —
   coins door and unlimited door present but **purchase buttons render a "get the app —
   coming soon" state** (IAP is v0.5; web-Stripe is D-020, later); ad door disabled per
   payload. Dismiss returns her exactly where she was. This proves the wall's UX and timing
   with zero money movement.
7. **For You (web teaser feed):** swipeable discovery of episode-1 hooks for logged-out and
   logged-in visitors, tap-through to the show; heuristic source (trending/genre) per v0.3.
8. **Deep links + share-the-moment (web half):** /s/{token} resolves and plays at the exact
   moment (entitled) or lands on the show page with the wall primed (not); a Share control in
   the player mints a moment link. Origin tags flow.
9. **Admin seed + thin panel (Owner scope directive):**
   - Real admin identity table (separate from viewer users): seed
     **michael@journeyviral.com** with temp password **michael_temp123** stored ONLY as an
     argon2/bcrypt hash, `must_change_password=true` forcing a change on first login.
     **Password fields (login + change) carry the eye-reveal toggle.** Rate-limited login,
     server-side sessions, uniform failure responses. Never log or echo the password; the
     temp value transited chat and is invalidated by the forced change.
   - Thin panel at /admin (admin session required): series/episode list with media state,
     shelf + hero arrangement (drag or ordered list), episode schedule view, tag assignment.
     Reads/writes go through the existing D-014-token'd API routes via a server-side proxy
     bound to the admin session — the interim token stays server-side only, never in the
     browser. Full admin (roles, upload UI, pricing) remains v0.7; do not build it.
10. **Deployment (4C):** web app container behind Traefik at binjreel.com root; API keeps
    `/v1/*` + `/health` by path rule (verify the Bunny callback URL is unaffected); www
    redirect intact; SSL green; both containers healthy; **served == merged verified** (the
    running container demonstrably carries the built tree).
11. **Fixture demo content:** the seeded comedy-shaped catalog from v0.3 plus the proven
    test-clip media path, sufficient for the Owner to browse, play, swipe, hit the wall, and
    share a moment — end to end, live.
12. **Tests + security pass:** component/E2E coverage for the wall, player grant/re-mint flow,
    admin auth (forced change, rate limit, no-secret-in-browser), SEO presence; the 1C-pattern
    pass on the new surfaces (admin auth foremost); SECURITY-REPORT updated.

## Out of scope (log ADDED if tempted)

Native builds, TestFlight/Play, IAP, StoreKit/Play Billing, push delivery, deferred deep link
(all v0.5) · ad network/SSV/check-ins/tasks/referrals (v0.5/v0.6) · web-Stripe (D-020) · full
studio admin (v0.7) · events sink (v0.8) · localization beyond the captions toggle · any change
to `mayPlay`, the ledger, receipts, or media pipeline · any new secret beyond the admin
password hash (SECRETS.md: declare the admin-session signing secret slot; fail-closed empty).

## Definition of done (GATE 2 / GATE 3)

- [ ] https://binjreel.com serves the web app; `/v1/*` + `/health` still serve the API; Bunny
      callback path unchanged and verified; SSL green; served == merged proven.
- [ ] Cold anonymous visit → browsing → playing fixture video → gapless swipe → the wall at
      the right swipe → dismiss returns in place — all live, no signup anywhere in the path.
- [ ] Player: re-mint keeps a long session alive past the TTL (proven live); captions, speed,
      quality, episode drawer functional; progress + resume round-trips.
- [ ] Show pages live with SEO metadata; tags/hubs/search/My List functional.
- [ ] /s/{token} moment links resolve to the exact beat; share control mints; origin tags
      persisted.
- [ ] Admin: seeded login works; **forced password change on first login enforced**; eye
      toggle present on password fields; rate limiting proven; no admin/API token reaches the
      browser (verified); thin panel operates shelves/schedule/tags live.
- [ ] Wall renders entirely from server data; all three doors present; zero purchase or ad
      code paths active; nothing hardcodes a price.
- [ ] Full server suite still green (v0.1+v0.2+v0.3); money core diff across v0.4 = 0 lines.
- [ ] Security pass clean (no high/critical open); SECURITY-REPORT updated; BUILD-LEDGER
      as-you-go; BUILD-REPORT closes the loop; STATE + TDD roadmap advanced; **v0.4 GATE-3 =
      the Owner browses, plays, swipes, hits the wall, and logs into /admin on their own
      device, live.**

## Token / cost budget (Blueprint §08)

This is the largest single slice yet (full client + deploy). Budget: **≤ $100 for this phase**
(the combined engagement budget lives in the kickoff Order). Log spend as a stat per phase in
BUILD-REPORT; if projected to exceed, checkpoint (commit + push + ledger) and report the
projection — do not silently continue.

## Build discipline (from the binding — reinforced for the long run)

- `binjreel-dev` only; **push the branch at every phase gate and before any pause** (the push
  path is verified, L-022; pushing binjreel-dev touches nothing else on the remote).
- BUILD-LEDGER after each step; DIVERGENCE/ADDED logged as they occur; owner-level questions →
  BLOCKED + STATE "Waiting On" + pause — with one standing exception class: **anything
  requiring an external account (Apple, Google, Stripe, ad network) is by prior Owner order
  DEFERRED, not BLOCKED** — note it in the ledger as DEFERRED-BY-ORDER and continue.
- Security pass before complete. Stop at GATE-3 for the Owner's live viewing.
