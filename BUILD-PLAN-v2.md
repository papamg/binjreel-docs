# BUILD-PLAN-v2.md — binjreel (gap analysis + the complete build-out)

*Compares PLATFORM-DESIGN.md against the standing TDD.md and IDEAL-SCENE.md and the as-built
state (v0.2 GATE-3 complete), determines keep / add / remove, and lays the full version road to
launch and beyond. On ratification this drives: TDD v2, the story-map rewrite, and the v0.3
stage contract.*

---

## 1 — What we compared

As-built: v0.1 money-and-access spine (60 tests, GATE-2 audited) + v0.2 real paywalled media
(87 tests, GATE-3 live-proven). Designed-but-unbuilt: the v0.3+ roadmap as it stood (player &
funnel → studio admin → retention/events). Vision: IDEAL-SCENE.md. Against: the market-standard
product surface and economy (MARKET-LANDSCAPE.md §3–4) and the Owner's directives (all genres
comedy-first, micro-docs, show pages, rewards economy, creator path later, investor readiness).

## 2 — KEEP (validated by the market, unchanged)

Everything load-bearing survives contact with the market intact:

- **The money core and both its constraints** (append-only ledger D-011, row-lock D-012) — now
  also the substrate for reward credits and future partner revenue share. The market's loudest
  complaints (double-charges, lost purchases) are the exact failures this core makes impossible.
- **Two-plane architecture; Bunny + owned masters; signed short-TTL playback; fail-closed
  posture everywhere.**
- **IAP-primary payments** — confirmed as the category's rail; the source-agnostic wallet
  already anticipated the web-Stripe lane.
- **Entitlement engine as the only URL gate; account-bound ownership; anonymous-first with
  merge.**
- **Multi-tenant data layer** — promoted from "insurance" to a named strategic asset (the
  creator/studio path).
- **JVScope funnel join; share-link origin tags; the events sink design.**
- **The v0.3 client-contract slice already scoped** (catalog reads, next+prefetch, re-mint,
  offer payload, entitlement summary, progress, deep links, share links) — every card survives;
  several grow (below).
- **IDEAL-SCENE.md** — still true; it gains a rewards passage and a comedy voice at TDD-v2 time,
  it does not change direction.

## 3 — ADD (the gaps the market exposed + Owner directives)

1. **The free-earning economy** (PLATFORM-DESIGN §2.3): check-ins/streaks, rewarded ads with
   server-side verification, rewarded-ad episode unlock (the wall's third door), task center,
   referrals, daily caps, expiring reward coins with visible expiry. *New services + one ad-SSV
   endpoint; zero ledger-logic change.*
2. **The browse layer**: home shelves + hero carousel (merchandising as data), Top-10, genre
   hubs, tag pages, search, **show pages**, follows, My List, watch history. *The largest pure
   addition; entirely reads + engagement writes.*
3. **For You feed** (heuristic first; recommendation seam named for the events era).
4. **VIP perks beyond access**: ad-free flag, 1080p gate, offline downloads, early release.
5. **Genre as a first-class dimension** incl. `nonfiction` (micro-docs); per-genre free-episode
   defaults.
6. **Web top-up lane** (Stripe on the web face, bonus-coin advantage; app never links to it).
7. **Captions day one; localization seam** (strings externalized; dubbing later).
8. **Notifications**: push + new-episode fanout for follows (extends the scoped push card).
9. **Release scheduling + shelf merchandising in studio admin.**
10. **Cast/fandom seam** (cast blocks on show pages now; content shelf later).
11. **Creator/studio path** (Phase-2 invited tenants → Phase-3 portal) as named roadmap stages.

## 4 — REMOVE / AMEND

- **Remove** TDD §1's drama-only framing → "vertical series platform, all genres, comedy-first,
  including micro-docs."
- **Remove** the v0.1-era out-of-scope line "no reward-earning surfaces" — superseded by Owner
  decision; the *fail-closed until built* discipline stays.
- **Amend** "no third-party creator marketplace" → "no open UGC marketplace; curated
  studio/creator onboarding is a named later phase."
- **Amend** the wall from two doors to three (coins / unlimited / one ad).
- **Nothing built is discarded.** Every shipped line remains load-bearing.

## 5 — The road (revised versions; each = one stage contract, one gate)

Sequencing logic: server contracts before surfaces; the store-enrollment clock starts NOW and
runs under everything; the web face makes the product visible early; ads land only where ad
SDKs can live (native); launch = native app + rewards.

- **v0.3 — The client contract (server).** Catalog/browse reads (series, episodes, genres,
  tags, shelves, search), show-page composition, follows/lists/history, progress + resume,
  next-episode + prefetch, mid-session re-mint (closes DL-B007's code default), three-door
  offer payload, entitlement summary, deep-link + share-moment mint/resolve with origin tags,
  caption manifests. *No Director dependency. The whole app rests on this.*
- **v0.4 — The player & browse, web face (Expo).** Vertical gapless player consuming §v0.3;
  home/shelves/show pages/tags/search/My List; the wall rendering the offer (ad door shown as
  "get the app" on web); web teaser feed; SEO show pages. *The Director watches real comedy
  swipe on a phone browser. Store enrollments proceed in parallel.*
- **v0.5 — Native shell + money (launchable).** Store builds via the same Expo codebase;
  StoreKit/Play Billing coin + subscription flows (server-verified, as built); deferred deep
  link; push; cold-start straight into the scene; check-in/streak + task center (the no-ad-
  network half of rewards). *Gates: store accounts, ledger backup before real receipts,
  pricing decisions, compliance set.*
- **v0.6 — The ad economy.** Mediation SDK (rewarded video only), ad-SSV endpoint, watch-ads-
  for-coins, rewarded-ad episode unlock, daily caps, referral program. *Gate: ad-network
  account (Director act, small).*
- **v0.7 — Studio admin v2.** Full console: series/episode mgmt, upload + transcode status,
  scheduling, shelves/hero merchandising, tags, pricing/free-counts, rewards config; retires
  the interim media-admin token (D-014); per-operator identity + audit log.
- **v0.8 — Retention & intelligence.** Events sink live; JVScope join (per-hook payer rates);
  For You upgraded from heuristic to data-driven; low-balance warning; smarter nudges.
- **v0.9+ — The platform play.** Invited-studio tenancy with ledger-derived revenue-share
  statements; localization (dubs); fandom shelf; programmatic ads decision; DRM stays parked
  on its existing reopen criteria.

**Launch line: end of v0.6** (native app, both stores, three-legged economy, two comedy shows,
Free-to-Binge funnel live). v0.4 is the first *visible* milestone and the investor demo.

## 6 — Director acts on the critical path (start-now items bolded)

1. **Apple Developer Program + Google Play Console enrollment** — the one uncompressible clock
   (D-U-N-S can take weeks). Gates v0.5, not v0.3/v0.4. **Start now.**
2. **Push + two-`main` + branch protection (W-002)** and **/docs under version control** —
   two shipped versions and the entire project record still live on one disk. **Start now.**
3. Continuous ledger backup — unchanged: the last gate before real receipts (v0.5 go-live).
4. Pricing set (coin packs, per-episode price, sub tiers + perks, free-episode counts per
   genre, reward values/caps/expiries — all config).
5. Compliance set (auto-renew/click-to-cancel, digital-goods tax, age gating per genre,
   privacy policy matching actual telemetry, DMCA) — before store submission.
6. Ad-network account (v0.6 gate; trivial spend).
7. 5A scalability/tenancy audit — before real traffic (window unchanged).
8. Brand/design direction for the comedy voice (a Claude Design package, the JVScope pattern) —
   feeds v0.4.

## 7 — What does NOT change

Gate discipline, one stage contract per version, fail-closed on every new secret slot, ledger
untouchable except under D-011/D-012, tests-as-proof, the 1C audit at each gate, and the seam
rule: the entitlement engine remains the only thing in the system that may open the video door
— now with three ways to earn the key, and still exactly one lock.
