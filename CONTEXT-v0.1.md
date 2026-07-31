# CONTEXT.md — binjreel · v0.1 (the money-and-access spine)

*The stage contract for the current build. This is the Order the Worker executes against: what to
build, the boundaries, and the definition of done. Scoped tightly — this is the whole blast radius
for v0.1. Serves TDD.md §4, §6, §11(1). Advances with the baton at GATE.*

*Doctrine is canon; this is project state. Where this is more specific than the TDD, that
specificity is the Worker's starting point and is overridable by better construction at or above
the IDEAL-SCENE floor.*

---

## Target

**v0.1 — the money-and-access spine.** The correctness-critical core, built and proven before any
video or player exists: accounts, the coin ledger, the entitlement engine, and the two store-
receipt integrations, with the signed-URL media seam **stubbed** (a placeholder that returns a
fake signed URL on an entitled request and refuses on an unentitled one — real Bunny wiring is
v0.2). Nothing here depends on Bunny, the app, or DNS.

Why this slice first: it is the riskiest and least forgiving part of the whole platform. If the
ledger can double-charge, or an unentitled request can obtain a playback URL, the product is
broken in the one way it must never be. Everything downstream stands on this being right.

## Ratified frame (from the TDD — do not re-litigate here)

- **Stack (D-002):** Node 22 + TypeScript, Fastify, Postgres, Redis. Stateless, containerized.
- **Tenancy (D-003, confirmed by Director 2026-07-25):** **tenant-aware from the first commit —
  every table carries `tenant_id`, every query is scoped by it — but single-tenant in practice.**
  Seed exactly one tenant (`binjreel`, us). No tenant-management UI, no client onboarding, no
  per-tenant billing in v0.1. The column and the query-scoping are in the foundation; the
  machinery that sits on them waits for a real second client. Resolve `tenant_id` from a single
  configured default for now; the request-time resolution seam exists but points at the one tenant.
- **Payments:** IAP-primary. Apple StoreKit + Google Play Billing. Receipts trusted **only** via
  verified store server notifications, never the client (TDD §4.1).
- **Audit tier:** high. The 1C security audit runs at GATE 2 against this build (TDD §10).

## Build scope — what v0.1 IS

1. **Foundation.** Fastify + TypeScript app, Dockerized, stateless. Postgres + Redis via compose.
   Prisma (or equivalent typed migrations). Health endpoint. Replaces the standup scaffold
   (`app/src/server.ts`) with the real service skeleton.
2. **Data model** (TDD §6), all tenant-scoped: `tenants`, `users`, `series`, `episodes`,
   `wallet_entries`, `entitlements`, `subscriptions`. Seed one `binjreel` tenant and a handful of
   fixture series/episodes (free + paid) for testing. `events` table may be created empty; its
   pipe is v0.5.
3. **Auth.** Account identity with Apple/Google/email sign-in *interfaces* and server-side
   sessions. Anonymous-first: a provisional user can exist and later merge into a real account on
   first purchase (build the merge path deliberately — TDD IDEAL-SCENE, the funnel). Full social
   sign-in secrets need not be placed in v0.1 if they gate on Director secret acts; stub the
   provider verification behind a clear interface and log the secret slots to SECRETS.md.
4. **The wallet ledger** (TDD §4.2). Append-only, double-entry. Balance = SUM over
   `wallet_entries`, never a stored field. Every credit/debit carries an **idempotency key**;
   a repeated key is a no-op. Two coin classes: `paid` (no expiry), `reward` (expiry). Provide:
   credit (from a verified receipt), debit (an episode unlock), and balance-read.
5. **The entitlement engine** (TDD §4.3). Server-side `mayPlay(user, episode)` → true iff episode
   is free, OR user owns it (a permanent `entitlements` row from a coin spend), OR user has an
   active subscription. An unlock is one transaction: idempotent debit **and** entitlement write
   together, or neither. Ownership is account-bound.
6. **Store-receipt verification** (TDD §4.1). Endpoints that ingest **App Store Server
   Notifications V2** and **Google Play RTDN**, verify the signature, and drive the ledger/
   subscription state. A coin-pack notification → ledger credit; a subscription notification →
   subscription row + entitlement. Idempotent on the store transaction id.
7. **The media seam, stubbed.** `getPlaybackUrl(user, episode)` runs `mayPlay` and, on yes,
   returns a placeholder signed URL; on no, refuses. This proves the seam and the enforcement
   point without Bunny. Real signing is v0.2.
8. **Security pass** (TDD §10). Rate-limit auth/purchase/unlock endpoints. All secrets in the box
   store, never in the repo or client. Per-tenant + per-user row isolation verified. Continuous
   backup posture for the ledger noted for the Director (the one dataset that is unrecoverable if
   lost).

## Out of scope — what v0.1 is NOT (do not build; log if tempted)

- No Bunny, no transcode, no real video (v0.2).
- No mobile app, no player, no deep-linking, no paywall UI (v0.3).
- No studio admin UI (v0.4). Fixture data is seeded directly for testing.
- No push, no low-balance prompts, no `events` ingestion (v0.5).
- No Stripe web coin-store (the wallet is source-agnostic so it slots in later untouched).
- No tenant-management UI, onboarding, or per-tenant billing (future, once a second client exists).
- No rewarded ads / check-in / referral reward flows (the `reward` coin class exists; the earning
  surfaces are later).

## Definition of done (GATE 2 verification points)

Each must be checked or explicitly failed, never skipped:

- [ ] App builds, runs containerized, health endpoint green; app holds no in-memory state.
- [ ] All tables carry `tenant_id`; every content/wallet query is tenant-scoped; the one seeded
      tenant resolves. A cross-tenant read returns nothing (proven by a second fixture tenant used
      only in tests).
- [ ] A verified coin-pack store notification credits the correct coins **once**; a replayed
      notification credits **nothing further** (idempotency proven, not assumed).
- [ ] An episode unlock writes an idempotent debit **and** a permanent entitlement atomically; a
      replayed unlock does not double-debit; balance never goes negative.
- [ ] `mayPlay` returns true only for free / owned / active-subscription; **an unentitled request
      is refused and yields no playback URL** (the stub proves the enforcement point).
- [ ] A verified subscription notification sets an active subscription + entitlement; a lapse
      notification revokes it; entitlement is account-bound and survives a simulated reinstall.
- [ ] Anonymous → real-account merge preserves wallet balance and entitlements.
- [ ] Auth/purchase/unlock endpoints are rate-limited; no secret appears in the repo or client;
      secret slots declared in SECRETS.md.
- [ ] 1C security audit run; `SECURITY-REPORT.md` written with severity counts; no high/critical
      open.
- [ ] `BUILD-LEDGER.md` written as-you-go; `BUILD-REPORT.md` closes the loop; STATE + TDD roadmap
      advanced at close.

## Build discipline (from the binding)

- Commit to `binjreel-dev` only, never `main`.
- Append to `BUILD-LEDGER.md` after each step and before any pause; never stop without writing it.
- Log a DIVERGENCE if built differently than this CONTEXT or the TDD specifies, naming the now-
  stale section. Log ADDED for anything built beyond this scope.
- Owner-level questions (scope, product, priority — not code choices) → BLOCKED in the ledger,
  question to STATE "Waiting On," and pause. Do not improvise around a blocker.
- Run the security audit after the build, before marking complete.

## Director acts this build waits on (if any surface)

- Store credentials for real receipt verification (Apple ASN key, Google Pub/Sub) — may be stubbed
  behind interfaces in v0.1 and placed when the integration goes live; declare the slots regardless.
- Confirm the continuous-ledger-backup mechanism (Director/Treasury) — noted, not blocking the
  build, but the ledger's backup is the one thing that must exist before real money flows in v0.2.
