# TDD.md — binjreel

*As-designed technical intent. What we intend to build and why, at a level a builder can execute
against and a reviewer can verify. Living document — advances with the STATE baton at each GATE.*

*Doctrine is canon; this is project state. Serves IDEAL-SCENE.md (the destination). Rationale for
the big calls lives in the two scoping docs; this states the design and the decisions that settle
the open forks the standup flagged (D-002, D-003).*

*Authored by the Architect, Phase 1.2. Current target: v0.1 — the money-and-access spine.*

---

## 1 — What this is

binjreel is a short-form vertical drama platform: a native mobile app (iOS + Android) that streams
1–2 minute episodes, freemium, monetized by coins and an unlimited subscription. App class:
**consumer content SaaS, designed multi-tenant-ready, high audit tier.** The operator produces or
licenses catalog; at scale the platform may carry multiple content clients (hence tenant-aware
from the data layer up), but there is no third-party creator marketplace.

The load-bearing engineering is not the video — it's the **money-and-access core**. Video delivery
is a solved, rented problem (Bunny). What we build and own is the control plane that decides who
may watch what, and keeps the books true.

## 2 — Two planes

**Control plane — owned, built here.** The API, database, auth, wallet ledger, entitlement engine,
admin. Stateless, containerized, horizontally scalable. This is the IP.

**Media plane — rented, Bunny from day one.** Ingest, transcode to an adaptive HLS/DASH ladder,
global CDN delivery via signed short-lived URLs. Masters are kept in **our own** object storage so
the delivery vendor is swappable by re-upload, never a rebuild. Video never touches the origin box.

The seam between them is the signed URL: the control plane decides *may this play*, and only then
mints a Bunny URL that dies in seconds. The paywall is enforced at this seam, not in the UI.

## 3 — Settled decisions (resolving the standup's open forks)

**D-002 → SETTLED. Stack: Node 22 + TypeScript, Fastify, Postgres, Redis.** The API is I/O-bound
(it brokers auth, entitlement lookups, ledger writes, and signed-URL minting — no heavy compute,
since transcode is Bunny's), so Node/TS is a correct fit, and it keeps the box on one runtime
shared with the MCP bridge. Fastify over Express for first-class TypeScript and lower per-request
overhead at scale. This supersedes the provisional D-002 scaffold. Reversible only by a future
TDD revision with a named reason.

**D-003 → SETTLED. Tenancy is path/header-based, not subdomain.** Tenants are resolved by a
`tenant_id` carried in the API layer, not by `tenant.binjreel.com` hostnames. This needs **nothing
from the Director** — no wildcard certificate, no DNS-01 resolver, no registrar API credentials.
Subdomain-per-tenant vanity URLs remain a later option if a client ever demands one, at the cost of
the DNS-01 machinery D-003 describes; nothing here forecloses it. For a mobile-app-first product,
tenants are invisible to the viewer anyway (the app targets its tenant via config), so subdomains
would buy nothing in V1.

## 4 — The money-and-access core

Three subsystems, one job each, clean seams (this is the heart; build it first, build it right).

### 4.1 Receipts — the only trusted source of "money arrived"
Purchases happen through **Apple StoreKit and Google Play Billing** (IAP-primary — locked). The
server never trusts the app's claim of a successful purchase. Truth comes only from the stores'
server-to-server notifications: **App Store Server Notifications V2** and **Google Play
Real-Time Developer Notifications** (Pub/Sub). Each notification is signature-verified, then drives
a ledger credit or an entitlement change. Two integrations, both mandatory.

### 4.2 Wallet — the coin ledger
An **append-only, double-entry ledger.** Credits on verified purchase/reward; debits on unlock.
Balance is **derived by summing the ledger**, never stored as a mutable field. Every write carries
an **idempotency key** (the store transaction id, or a client-generated unlock token) so a retry
is a no-op, never a double. Two coin classes: `paid` (no expiry) and `reward` (expiry timestamp).
This is the vault; correctness here is non-negotiable, and it is the one dataset whose backup
cadence is continuous, not periodic.

### 4.3 Entitlements — the access record
The server-side answer to *may this account play this episode now?* — yes iff the episode is free,
OR the account owns it (a permanent unlock record from a coin spend), OR the account holds an
active subscription. Ownership is **account-bound**, not device-bound, so it survives reinstall and
resolves identically across devices. Nothing else in the system may mint a playback URL.

## 5 — Media pipeline & delivery — **AS BUILT, v0.2 (2026-07-27)**

Master upload → Bunny transcodes to the adaptive ladder → stored + globally distributed. Playback:
the app requests episode N → API runs the §4.3 check → on yes, mints a **signed Bunny URL, seconds
of TTL** → player pulls manifest+segments, CDN validates the signature at the edge. A screen-grab
can't leak a working link because the link is dead before it can be shared. **DRM (Widevine/
FairPlay)** is a named later stage; signed URLs are the V1 baseline. Perfect anti-piracy is not a
goal — raising the cost of casual sharing is.

*As built (v0.2):* masters live in an S3-compatible bucket we own, reached by a narrow SigV4
client with no unsigned-URL capability (D-013); registration → `POST /library/{id}/videos/fetch`
with a ≤1h presigned source URL; the async completion webhook (query-token-authenticated, D-015)
drives the episode media-state machine (`none|uploading|transcoding|ready|failed`, DB CHECKs on
the terminal states); playback signing is Bunny CDN token auth, TTL config-driven. Everything is
FAIL-CLOSED with the six SECRETS.md media slots empty. Master registration + media-state read sit
behind an interim operator token (D-014) until v0.4 roles.

## 6 — Data model (first cut)

Tenant-aware from the root. Core tables:

- `tenants` — id, name, config.
- `users` — id, tenant_id, auth identities (Apple/Google/email), created_at. Anonymous-first: a
  device may hold a provisional user that later merges into a real account on first purchase.
- `series` — id, tenant_id, title, synopsis, genre tags, cover, publish_state, free_episode_count.
- `episodes` — id, series_id, order_index, master_ref, bunny_ref, unlock_price_coins.
- `wallet_entries` — id, user_id, delta_coins, coin_class, reason, idempotency_key, created_at
  (append-only; balance = SUM).
- `entitlements` — id, user_id, episode_id (nullable), scope (`episode` | `subscription`),
  source, granted_at, expires_at (nullable).
- `subscriptions` — id, user_id, store, store_sub_id, status, current_period_end.
- `events` — the in-app telemetry sink (see §9), high-volume, isolated from the money tables.

Every query that returns content or wallet state is scoped by `tenant_id` and `user_id`; no row
crosses either boundary. This is the isolation the audit tier demands.

## 7 — Scaling stages (named, not pre-built)

- **Stage 0 (now):** single box, stateless app + Postgres + Redis, containerized. The Journey 2
  host is a valid Stage 0 seed *because* the app is stateless.
- **Stage 1:** multiple app replicas behind Traefik (trivial, because stateless).
- **Stage 2:** Postgres read replicas + hard Redis caching of entitlement checks; managed DB.
  Trigger: DB load, not calendar.
- **Stage 3:** the `events` firehose moves to its own pipe; async work (transcode callbacks,
  receipt-retry, reward-expiry sweeps) on a background queue.
- **Stage 4:** multi-region control plane.

Each later stage is a migration triggered by a measured metric. Building Stage 3–4 at launch is
waste.

## 8 — Payments posture (IAP-primary, locked)

Apple/Google IAP is the primary and only V1 rail — chosen for frictionless in-app conversion at
the top of a social-fed funnel, accepting the ~15% store fee (Small Business tier). The wallet is
**source-agnostic**: coin credits carry their origin, so a Stripe web coin-store for high-spenders
can be added later as another credit source with zero change to entitlement or spend logic. Not in
V1; not walled off.

## 9 — Analytics

**JVScope handles the social funnel** — the clips on TikTok/YouTube/Instagram are videos on
connected accounts, JVScope's existing live capability; no new build. **binjreel owns a thin
in-app event store** (§6 `events`) for what happens *inside* the app — paywall shown, offer
chosen, unlock, binge depth, abandonment point — which JVScope has no ingestion path for today.
Joining the two ("which hook produces payers") is a valuable later loop, Stage-3 era, not V1.

## 10 — Security spine (audit-tier: high)

The wall guards two crown jewels — the money and the media. Non-negotiables: all secrets in the
box secret store, never in the repo, never bridge-reachable, never near Traefik's `acme.json`;
entitlement checks server-side only; signed short-TTL media URLs; idempotency on every money/coin
write; store-notification signatures verified on every call; rate-limiting on auth, purchase, and
unlock endpoints; per-tenant and per-user row isolation; continuous backup on the ledger above all
else. A formal 1C security audit runs at GATE 2 against the first real build.

## 11 — Build order

1. **v0.1 — the money-and-access spine: ✅ BUILT 2026-07-25** (commit `9c757af` on
   `binjreel-dev`). users/auth, the wallet ledger, the entitlement engine, and the two
   store-receipt integrations, with the signed-URL seam stubbed. All three proofs demonstrated by
   test against a real Postgres and Redis, not asserted: a verified purchase credits coins **once
   and a replay credits nothing further**; a spend writes an idempotent debit **and** a permanent
   entitlement atomically (and an underfunded spend writes **neither**); an unentitled request is
   refused **and yields no playback URL**. Plus cross-tenant isolation and the anonymous→real
   merge. 1C full-tier audit passed at GATE 2 with 0 high/critical open.
   *As-built: `BUILD-LEDGER.md` L-011…L-021 · `BUILD-REPORT.md` · `SECURITY-REPORT.md`.*
   *Two design decisions taken in code and worth carrying forward: **D-011** (append-only enforced
   by database trigger with no exception; wallet movement expressed as compensating entries) and
   **D-012** (idempotency keys do not prevent double-spend — every wallet mutation takes a row
   lock). Both constrain how v0.2+ may touch the ledger.*
2. **v0.2 — media: ✅ BUILT 2026-07-27, fail-closed at GATE 2** (`binjreel-dev`; commit in
   STATE). Real Bunny ingest/transcode contract-proven against the documented API + a simulated
   completion callback; real token-auth signed playback with the paywall enforced at the seam
   (unentitled → no URL, no `bunny_ref`; slots empty → mints NOTHING). 86 tests green, the 60
   v0.1 properties unbroken, money core diff = 0 lines. **The live end-to-end proof is GATE-3**,
   after the Director places the Bunny keys + master bucket (six slots, `SECRETS.md`).
   *As-built: `BUILD-LEDGER.md` L-023…L-027 · `BUILD-REPORT.md` · `SECURITY-REPORT.md` v0.2 pass.*
   *DIVERGENCE against this section as previously written: "real signing changes no call site"
   proved false in the code as built — the real mint needs the episode's `bunny_ref` (a DB read)
   and the signing config, so `getPlaybackUrl` gained one argument at its single call site;
   enforcement order and route shape unchanged (L-024). Decisions taken in code: **D-013**
   (SigV4 storage client, no vendor SDK), **D-014** (interim operator token on the media-admin
   surface), **D-015** (query-token webhook auth — vendor constraint, with mitigations).*
   **Still gated on the Director before real money flows:** the continuous ledger-backup
   mechanism (§10) and the Apple/Google verification keys — declared and fail-closed today.
3. **v0.3 — the player & funnel:** vertical gapless player, deferred deep-link, paywall
   interstitial, continue-watching.
4. **v0.4 — studio admin:** series/episode management, ingest status, merchandising.
5. **v0.5 — retention & telemetry:** push, low-balance prompts, the `events` sink → JVScope.

## 12 — Still the Director's to decide (product, not engineering)

None of these block v0.1; the architecture supports any value.
- Pricing: coin-pack tiers, per-episode coin cost, subscription price points.
- Free-episode threshold (start ~5–10, tune per series on real Stage-4-of-the-funnel data).
- Auto-unlock default at first purchase (on/off — affects binge depth and complaint rate).
- Confirm anonymous-first (recommended: yes) — accepts the session-to-account merge cost.
- Launch catalog size — a content-scheduling question that shapes the launch date.
- Compliance/Treasury flags: auto-renew rules (click-to-cancel), digital-goods sales tax,
  age-gating for mature genres, privacy policy, DMCA posture.
