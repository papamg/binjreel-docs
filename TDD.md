# TDD.md — binjreel · v2

*As-designed technical intent. What we intend to build and why, at a level a builder can execute
against and a reviewer can verify. Living document — advances with the STATE baton at each GATE.*

*Doctrine is canon; this is project state. Serves IDEAL-SCENE.md (the destination). **v2 rewrite
ratified 2026-07-28** against the design set (PLATFORM-DESIGN.md · BUILD-PLAN-v2.md ·
MARKET-LANDSCAPE.md — all in this /docs). v1 sections that shipped are preserved below as the
as-built record; v1's roadmap is superseded by §11.*

*Authored by the Architect (4A). Current target: **v0.5 — native shell + money (launchable).**
Baseline: v0.4 BUILT, DEPLOYED, and Owner-certified at GATE-3 2026-07-31.*

---

## 1 — What this is

binjreel is a **vertical micro-series platform for every genre — comedy first**, including
micro-docs: a native mobile app (iOS + Android, with a web face) streaming 1–3 minute episodes,
freemium, monetized on the category's proven three-legged economy (coins · subscription ·
free-earning rewards). Content is produced by **Journey Viral**; catalog opens with two original
comedy series and grows first-party, then by curated studio/creator onboarding (§9.5). App class:
**consumer content SaaS, designed multi-tenant-ready, high audit tier.** No open UGC marketplace.

The load-bearing engineering is not the video — it is the **money-and-access core** (built,
audited, live-proven) plus the **engagement layer** that turns it into a product people open
daily. Video delivery is rented (Bunny); masters are owned. The market context, competitors, and
the openings this design attacks are recorded in MARKET-LANDSCAPE.md and are normative input to
this document.

## 2 — Two planes *(unchanged, as built)*

**Control plane — owned.** API, database, auth, wallet ledger, entitlement engine, catalog,
rewards, admin. Stateless, containerized, horizontally scalable. This is the IP.
**Media plane — rented.** Bunny ingest/transcode/CDN via signed short-TTL URLs; masters in our
own object storage (SigV4 client, D-013); vendor swappable by re-upload. Video never touches the
origin box. The seam is the signed URL; the paywall is enforced at this seam. *(As built v0.2,
GATE-3 live-proven.)*

## 3 — Settled decisions

D-002 (Node 22/TypeScript, Fastify, Postgres, Redis) and D-003 (path/header tenancy, not
subdomain) stand as settled in v1 and as built. New settlements taken in the v2 ratification:

**D-017 → SETTLED. The three-legged economy is in-design.** Coins + subscription + the
free-earning rewards economy (check-ins, streaks, rewarded ads, rewarded-ad episode unlock,
tasks, referrals). Supersedes v1's "no reward-earning surfaces" exclusion. Constraint: every
reward credit is an ordinary ledger entry (coin_class `reward`, expiring) written under
D-011/D-012; **ad rewards credit only on the ad network's server-side verification callback**,
signature-verified and idempotent — the store-receipt pattern applied to ads. Rewarded video
only at launch; no interstitial/banner formats (a deliberate anti-pattern rejection).

**D-018 → SETTLED. Genre is a first-class dimension.** Series carry `genre` (Comedy, Micro-Doc,
Drama, …) and a `nonfiction` flag; genres drive hub pages, default free-episode counts, and
presentation. Tags remain many-per-series and are first-class navigation.

**D-019 → SETTLED. The client is Expo/React Native, web face first.** One codebase, three
faces: web (first visible product, the funnel catcher, SEO show pages), iOS, Android. The video
component is the one platform-split module. Confirms and extends the standing player decision.

**D-020 → SETTLED. Web top-up lane.** Stripe coin purchases on the web face at a bonus-coin
advantage (a new credit source into the source-agnostic wallet; zero entitlement change). The
app never links to it (store policy). Activation is a launch-window Director decision; the seam
is designed now.

**D-021 → SETTLED. Curated supply, phased.** Phase 1 Journey Viral produces; Phase 2 invited
studios as tenants/labels with ledger-derived revenue-share statements; Phase 3 a self-serve
studio portal. Open UGC is explicitly out.

## 4 — The money-and-access core *(as built; unchanged; constraints restated)*

§4.1 Receipts (Apple ASN V2 + Google RTDN, signature-verified, fail-closed) · §4.2 Wallet
(append-only double-entry ledger; D-011 no-exception append-only, D-012 row lock on every
mutation; coin classes `paid`/`reward` with expiry on reward; spend order reward-first,
oldest-expiry-first) · §4.3 Entitlements (the only URL gate; account-bound). All as built and
proven in v0.1; v2 adds **credit sources** (ad SSV, tasks, referrals, Stripe-web) and **spend
surfaces**, never core changes. VIP perk flags (ad-free, 1080p, offline, early-release) hang on
the subscription entitlement.

## 5 — Media pipeline & delivery *(as built v0.2, GATE-3 live-proven 2026-07-27)*

As recorded in v1 §5 as-built: owned masters (D-013), Bunny fetch-ingest + webhook-driven media
state machine (D-015), directory-scoped HS256 CDN token signing (D-016), config-driven TTL,
fail-closed on empty slots, interim operator token (D-014, retired at §11 v0.7). v2 additions at
the delivery layer: **caption tracks per episode** (day one of the client), quality ladder
exposure (540/720/1080 with 1080 as a VIP flag), offline-download packaging (VIP, native only),
and the **mid-session re-mint seam** (client renews its link; entitlement re-checked every
mint; closes DL-B007's code-default item).

## 6 — Data model (v2 additions in bold)

Tenant-aware from the root, as built: `tenants`, `users`, `series`, `episodes`,
`wallet_entries`, `entitlements`, `subscriptions`, `events`. v2 adds:

- **`series.genre` + `series.nonfiction` + per-series free_episode_count default by genre**;
  `series_tags` (many-to-many) — tags drive tag pages and shelf assembly.
- **`shelves` + `shelf_items`** — home merchandising as data (hero carousel slots included),
  studio-arranged, ordered, scheduled.
- **`follows`**, **`list_items`** (My List), **`watch_progress`** (per-user per-episode position
  + series resume pointer; isolated from money tables), **`watch_history`**.
- **`share_links`** — opaque non-enumerable tokens: series/episode/position + origin tag
  (platform, clip, campaign). **`link_resolutions`** feed the funnel join.
- **Rewards:** `checkin_state` (streaks, freezes), `reward_tasks` (config registry) +
  `task_claims`, `ad_reward_events` (network event id, SSV-verified, idempotency key →
  ledger credit), `referrals`. Daily caps enforced server-side from config.
- **`cast_members` + `series_cast`** — the fandom seam (name cards now, content shelf later).
- **`captions`** (per-episode track refs). **Notification tokens + preferences.**
- **`episode_schedule`** — timed release drops driving follow fanout.

Every read/write scoped by `tenant_id` (+ `user_id` where personal). Progress/engagement tables
never contend with the ledger. Recommendation state is derived data (events-era, §9.4), not a
new source of truth.

## 7 — Scaling stages *(unchanged)*

Stage 0 single box (now) → Stage 1 replicas → Stage 2 read replicas + entitlement caching →
Stage 3 events firehose + background queue + recommendation compute → Stage 4 multi-region.
Triggered by measured metrics, not calendar. The rewards economy adds background sweeps
(reward expiry, streak resets, notification fanout) that land as Stage-3-class async jobs but
run fine as scheduled tasks at Stage 0 volume.

## 8 — Payments posture

IAP-primary stands (locked, as built at the receipts layer). The wallet's source-agnosticism is
now exercised twice: **ad-SSV credits** (D-017) and the **Stripe web lane** (D-020). Coin packs,
prices, sub tiers, perk sets, reward values, caps, and expiries are all config — tunable without
a code change or store review.

**Prices are set (D-022, Owner-ratified 2026-07-31).** 1 coin = $0.01 nominal; an episode
unlock is **30 coins, flat across every series and every position**, displayed before the tap;
two free episodes by default, per-series overridable, with the displayed count always equal to
the real one. Packs $1.99/$4.99/$9.99/$24.99/$49.99, bonus rising by tier. Subscription $5.99
weekly ($3.99 first week) and $49.99 annual; monthly is an unset slot, not offered at launch.
The category's dominant players vary per-episode price opaquely and deeper into a series; this
platform does not, because the voice rule is *numbers are facts, never bait* and the
differentiator is being the platform a viewer does not resent. Every value is config — tunable
without a code change or a store review, so the decision is reversible at any time.

## 9 — The engagement layer (new in v2)

**9.1 Browse.** Catalog reads (series/episodes/genres/tags), shelf composition, Top-10
computation, search (with trending), show-page composition (cover, synopsis, tags, cast,
episode grid with lock states, related), tag pages, genre hubs. Publish-state honoured
(unpublished = absent); no media reference ever leaks in a catalog read.

**9.2 Rewards.** Check-in/streaks, task registry, referral graph, the ad-SSV endpoint, caps.
The wall gains its third door: coins / unlimited / **watch one ad** (time-boxed or permanent
entitlement, config). **Reward coins do not expire (D-023).** The v0.4 Rewards surface shipped
that promise publicly and it stands; an inverted dark pattern is still the dark pattern, and
vanishing earned balances are the category's most-cited grievance. `coin_class` and the expiry
machinery remain in the ledger and are untouched, so this is reversible in config — but no
expiry is set on reward credits, no expiry sweep is built, and no surface implies one.

**9.3 Retention.** Follows + new-episode push fanout on scheduled drops; continue-watching;
low-balance warning ahead of a failed unlock; share-the-moment links (timestamped, origin-
tagged) — the comedy distribution engine.

**9.4 For You & recommendations.** Heuristic at launch (trending, genre affinity, follow
graph); upgraded on the `events` substrate in the intelligence stage. A named seam, not a
launch build. **Analytics posture unchanged from v1 §9:** JVScope owns the social funnel;
binjreel owns the thin in-app event sink; the join key is the share-link origin tag, minted
from day one so history is never untagged.

**9.5 The studio path.** Admin v2 (series/episode mgmt, upload + status, scheduling, shelves,
tags, pricing, rewards config, per-operator identity + audit log — retires D-014) → invited
tenants with ledger-derived revenue-share statements → self-serve portal. The audited wallet is
the trust product that brings a partner's catalog.

**9.6 Localization seam.** Captions day one; UI strings externalized; dubs and multi-language
catalog as a later stage, never a rebuild.

## 10 — Security spine (audit tier: high; additions)

Everything in v1 §10 stands. New surfaces, same wall: the ad-SSV endpoint is
signature-verified, idempotent, rate-limited, fail-closed (the webhook pattern); reward caps
enforced server-side (the rewards economy is the platform's first *adversarial* surface —
farming is met with SSV-only crediting, caps, expiry, and anomaly logging to the audit line);
share-link tokens non-enumerable and rate-limited against catalog walking; web-Stripe (when
activated) verified by Stripe signatures with the same idempotent-credit discipline; the 5A
scalability/tenancy audit remains a named pre-traffic gate.

## 11 — Build order (v2 roadmap; v1's §11(1–2) preserved as-built)

1. **v0.1 — money-and-access spine: ✅ BUILT + GATE-2 audited 2026-07-25** (`9c757af`).
   As recorded in TDD v1 §11(1): 60 tests proven against real Postgres/Redis; D-011/D-012
   taken in code. *As-built: BUILD-LEDGER L-011…L-021 · BUILD-REPORT · SECURITY-REPORT.*
2. **v0.2 — media: ✅ BUILT + GATE-3 COMPLETE 2026-07-27** (`574275a`). As recorded in TDD v1
   §11(2) incl. the L-024 divergence (one argument threaded at the single mint call site —
   reconciled here: §5 as-built reflects the real signature). 87 tests; live-proven end to end.
   *As-built: L-023…L-029 · DL-B006/B007 · D-013…D-016.*
3. **v0.3 — the client contract (server): ✅ BUILT + GATE-2 passed + deployed 2026-07-28.**
   All 14 ordered scopes: catalog/browse reads, shelves + hero data, search + trending,
   show-page + tag-page composition, follows/lists/history, progress + resume, next-episode +
   entitled prefetch, mid-session re-mint (**DL-B007 closed fully** — 900s code default + the
   renew seam), three-door offer payload (D-017 shape, ad door fail-closed), entitlement
   summary (mayPlay-arbitrated), deep-link + share-moment with origin tags, caption manifests,
   genre/nonfiction schema (D-018), episode scheduling + log-only follow-fanout hook. 138
   tests; money core diff 0 lines. *As-built: BUILD-LEDGER L-031…L-034 · BUILD-REPORT ·
   SECURITY-REPORT v0.3 (F-004 fixed; N-008 notes the engine's publish-state check for its
   next ordered touch).*
4. **v0.4 — player & browse, web face (Expo): ✅ BUILT + DEPLOYED + OWNER-CERTIFIED AT GATE-3
   2026-07-31.** Design accepted at 390 and 1440; the forced admin password change performed.
   Beyond the original scope, this version also absorbed the restyle to the ratified design
   package, the conformance-completion redirect (assets wired, 1440 layouts implemented, hard
   rules swept, DL-B009 fixed), and the Owner's punch-list pass. Governing visual contract:
   `docs/design-package/…/project/HANDOFF.md` in full. Expo/expo-router codebase at `app/web/`
   (D-019; `VideoSurface` is the one platform-split module, native side an explicit v0.5 seam);
   five-tab shell, hero/shelf home, SEO show pages (server-injected), tags/hubs/search/My List,
   hls.js vertical player (prefetch-fed swipe, mid-session re-mint via token-prefix rewrite,
   captions/speed/quality/drawer), the three-door wall from the offer verbatim, share-moment +
   /s/ landing, For You teaser feed, REAL admin identity (bcrypt hash-only, forced first-login
   change, uniform failures, hard rate limit) + thin /admin panel behind a server-side
   allow-listed proxy (D-014 token never in a browser — proven). Design tokens:
   `app/web/DESIGN.md`. *As-built: L-036…L-046 · BUILD-REPORT · SECURITY-REPORT v0.4 · conformance-v2 ·
   DL-B009/B011 closed.*
5. **v0.5 — native shell + money (launchable). ← CURRENT TARGET.** Stage contract:
   `docs/CONTEXT-v0.5.md`, **phased**: Phase A needs no store account and does not wait —
   native iOS/Android against the one Expo codebase (`VideoSurface`'s native seam), cold start
   into the scene, deferred deep link with its origin tag surviving install, onboarding to
   HANDOFF §9.3/§9.4, the D-022 pricing configuration, the rewards half with no ad network in it
   (check-in, streaks, tasks, referrals), the continuous ledger backup mechanism, and the
   engine's publish-state touch (N-008). Phase B is the money proof and opens on the store
   accounts — StoreKit and Play Billing client flows against the already-built server
   verification, sandbox purchase → real receipt → real unlock, push, store sign-in providers.
   Gated: **store accounts (D-U-N-S lead time is the clock)** · ledger backup before real
   receipts · compliance set before submission.
6. **v0.6 — the ad economy. ← LAUNCH LINE.** Mediation SDK (rewarded video only), ad-SSV
   endpoint, watch-ads-for-coins, rewarded-ad unlock, caps, referrals. Gated: ad-network
   account.
7. **v0.7 — studio admin v2** (retires D-014; per-operator identity; scheduling; shelves;
   rewards config).
8. **v0.8 — retention & intelligence** (events sink live; JVScope join; For You data-driven;
   low-balance; smarter nudges).
9. **v0.9+ — the platform play** (invited-studio tenancy + revenue-share statements;
   localization dubs; fandom shelf; programmatic-ads decision; DRM stays parked on its v1
   reopen criteria).

## 12 — Still the Director's to decide (product, not engineering)

Carried and extended; none block v0.3–v0.4 code:
- **Start-now clocks:** Apple Developer + Play Console enrollment (gates v0.5) · push +
  two-`main` + branch protection (W-002) + /docs under version control (single-copy risk).
- Pricing config set: coin packs, per-episode price, sub tiers + perk set, per-genre free
  counts, reward values/caps/expiries, final playback TTL.
- Continuous ledger backup (the last gate before real receipts). Ad-network account (v0.6).
- Compliance set before store submission: auto-renew/click-to-cancel, digital-goods tax,
  age-gating per genre, privacy policy matching actual telemetry, DMCA.
- Brand/design package for the comedy voice (feeds v0.4). Web-top-up activation window
  (D-020). 5A audit timing (before real traffic). Launch catalog size/cadence.

---

*TDD v2 — ratified 2026-07-28. v1 (drama-framing, two-legged economy, player-first roadmap)
superseded; its shipped sections live on above as the as-built record. Nothing built was
discarded; the money core is untouched by every addition in this revision.*
