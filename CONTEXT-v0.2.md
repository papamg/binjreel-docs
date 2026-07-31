# CONTEXT.md — binjreel · v0.2 (media — real video, paywalled at the seam)

*The stage contract for the current build. This is the Order the Worker executes against: what to
build, the boundaries, and the definition of done. Scoped tightly — this is the whole blast radius
for v0.2. Serves TDD.md §2, §5, §4.3, §11(2). Advances with the baton at GATE.*

*Doctrine is canon; this is project state. Where this is more specific than the TDD, that
specificity is the Worker's starting point and is overridable by better construction at or above
the IDEAL-SCENE floor.*

*Supersedes the v0.1 CONTEXT as the live stage contract; archive the prior file as
`CONTEXT-v0.1.md` at placement (do not delete — it is the v0.1 as-ordered record).*

---

## Target

**v0.2 — media.** Turn the **stubbed** signed-URL seam from v0.1 into a **real** one: ingest a
master file, have Bunny transcode it to an adaptive ladder, and mint a **real, seconds-of-TTL
signed playback URL** — but only for a request the v0.1 entitlement engine already approves. The
paywall is enforced **at this seam**, not in any UI. This is the first version that streams real
video and the first that touches a paid external service.

Why this slice now: v0.1 proved the *decision* (who may watch) against real Postgres and Redis.
v0.2 makes the *delivery* real without disturbing that decision. Per TDD §11(2) the seam is a
**body swap** — `mintPlaybackUrl(tenant, episode, ttl, clock)` already has the right shape, so real
signing replaces the stub body and **no call site moves**. If that turns out to be false in the
code as built, that is a DIVERGENCE to log, not to route around silently.

## Ratified frame (from the TDD — do not re-litigate here)

- **Two planes (TDD §2).** Control plane owned; media plane rented. **Masters live in our own
  object storage**, so the delivery vendor is swappable by re-upload, never a rebuild. Video never
  touches the origin box.
- **Bunny from day one (TDD §5).** Bunny does ingest → transcode → CDN delivery. Signed short-TTL
  URLs are the v1 anti-sharing baseline; **DRM (Widevine/FairPlay) is a named later stage — NOT in
  v0.2.**
- **The entitlement engine is unchanged (TDD §4.3).** `mayPlay` is the only thing that may gate a
  URL mint. v0.2 does not touch its logic; it consumes it.
- **Ledger is off-limits to casual change (D-011 / D-012).** Append-only with no exception, wallet
  movements as compensating entries, and a row lock on every wallet mutation. v0.2 should not need
  to write the ledger at all; if any path does (e.g. a future failed-playback refund — NOT in
  scope here), it obeys both or it is not built.
- **Audit tier: high (TDD §10).** A dedicated security pass on the new media seam runs before
  complete.

## Build scope — what v0.2 IS

1. **Master storage (ours).** A place we own for master files (object storage — a bucket we
   control, distinct from Bunny's CDN copy). A path to register a master against an episode and
   persist `episodes.master_ref`. Masters are **not publicly reachable** — no unauthenticated URL
   ever returns a master.
2. **Bunny ingest + transcode.** On master registration, hand the master to Bunny, trigger
   transcode to the adaptive ladder, and handle the **asynchronous completion callback**: on
   success persist `episodes.bunny_ref` and a `ready` state; on failure record a `failed` state
   with the reason (never silently drop). All Bunny calls go through **one client module** behind
   the credential slot.
3. **Real signed playback URL.** Replace the stub body of `mintPlaybackUrl`: on an entitled
   request, mint a **real Bunny token-authenticated URL with a few-seconds TTL** and return the
   manifest URL. TTL is **config-driven**, not compiled in. No call site changes (body swap).
4. **Paywall at the seam, end-to-end.** The playback endpoint: run `mayPlay` (v0.1, unchanged) →
   on **yes** mint the real signed URL → on **no** refuse with **no URL and no leak** of the
   `bunny_ref`. This is the v0.1 enforcement property, now proven with real signing behind it.
5. **Minimal media-state read.** A server-side way to see an episode's media state
   (`none | uploading | transcoding | ready | failed`) — enough to test and to unblock v0.4 admin
   later. **Not** a studio UI, not merchandising — a status field and a read path only.
6. **Dedicated media security pass (TDD §10).** TTL is real (an expired link is dead at the edge —
   proven, not asserted); the Bunny signing key + storage credentials live **only in the box
   store**, never in repo / client / bridge-reachable / near Traefik `acme.json`; the mint endpoint
   is rate-limited; there is **no path to a working URL that bypasses `mayPlay`**; masters are not
   publicly reachable.

## Fail-closed posture (build now, switch on later)

The build proceeds **without** a live Bunny account. The Bunny client is wired and tested against
Bunny's documented API contract and a simulated transcode-complete callback; with the credential
slot empty it **fails closed** (mints nothing, surfaces a clear "media not configured" state) —
exactly as v0.1's receipt integrations fail closed today. The **live end-to-end proof** (a real
master transcodes and plays through a real signed URL, and an expired link dies) is a **GATE-3
step** taken once the Director places the Bunny key. Declare every new slot in `SECRETS.md`.

## Out of scope — what v0.2 is NOT (do not build; log ADDED if tempted)

- No mobile app, no player, no paywall interstitial, no deep-linking (v0.3).
- No studio admin UI (v0.4). Media state is a field + read path for testing, not a console.
- **No DRM** (Widevine/FairPlay) — named later stage; signed URLs are the v1 baseline.
- No change to `mayPlay` logic, the ledger, or the entitlement schema. If a change seems required,
  BLOCK and route it — do not edit the money core to serve media.
- No real store-receipt keys required; they stay fail-closed from v0.1 through this build.
- No `events` ingestion (v0.5), no Stripe, no new coin-earning surfaces.
- **No real-money go-live.** This build proves the media path against fixtures; flipping real
  receipts and real coins on is a separate Director-gated step, not this build.

## Definition of done (GATE 2 / GATE 3 verification points)

Each must be checked or explicitly failed, never skipped:

- [ ] A master registered against an episode is stored in our own object storage; `master_ref`
      persists; **no unauthenticated request can retrieve a master**.
- [ ] Master registration triggers Bunny transcode; the **async completion callback** sets
      `bunny_ref` + `ready`; a failed transcode sets `failed` + reason. *(Contract-tested at build;
      live-proven at GATE-3.)*
- [ ] `mintPlaybackUrl` mints a real Bunny signed URL on an entitled request with **no call site
      changed** (body-swap confirmed, or DIVERGENCE logged).
- [ ] Paywall at the seam: an entitled request gets a working short-TTL manifest URL; **an
      unentitled request is refused, yields no URL, and leaks no `bunny_ref`** (v0.1 property
      preserved under real signing).
- [ ] **A signed URL is dead after its TTL** — proven by an expired-link fetch failing at the
      edge, not asserted. *(Live at GATE-3 once the key is placed.)*
- [ ] Bunny signing + storage credentials live only in the box store; none in repo/client/bridge/
      near `acme.json`; slots declared in `SECRETS.md`, fail-closed while empty.
- [ ] The playback-mint endpoint is rate-limited.
- [ ] **The full v0.1 suite is still green** — ledger, entitlement, tenant-isolation, and merge
      properties unbroken; any ledger touch (none expected) obeys D-011/D-012.
- [ ] Dedicated media-seam security pass run; `SECURITY-REPORT.md` updated with severity counts;
      no high/critical open.
- [ ] `BUILD-LEDGER.md` written as-you-go; `BUILD-REPORT.md` closes the loop; STATE + TDD roadmap
      advanced at close (v0.2 marked built, the §11(2) seam de-stubbed, §5 moved to as-built).

## Build discipline (from the binding)

- Commit to `binjreel-dev` only, never `main`.
- Append to `BUILD-LEDGER.md` after each step and before any pause; never stop without writing it.
- Log a DIVERGENCE if built differently than this CONTEXT or the TDD specifies, naming the now-
  stale section. Log ADDED for anything built beyond this scope.
- Owner-level questions (scope, product, priority — not code choices) → BLOCKED in the ledger,
  question to STATE "Waiting On," and pause. Do not improvise around a blocker.
- Run the media-seam security pass after the build, before marking complete.

## Director acts this build waits on

- **Bunny account + API / storage / signing keys** (a spend + a secret). Build and contract-test
  proceed fail-closed without it; the **live end-to-end proof is GATE-3** once the key is placed.
  *This is the one new spend v0.2 introduces.*
- **Continuous ledger-backup mechanism** (Director / Treasury) — must be standing **before real
  money flows**. Gates real-money go-live, not this build. The money-safety gate.
- **Apple ASN + Google RTDN verification keys** — remain fail-closed through this build; placed at
  real-receipt go-live only.
- **Carried from W-002:** the two-`main` reconciliation and branch protection on `main` — settle
  before `binjreel-dev` is first pushed for real.
