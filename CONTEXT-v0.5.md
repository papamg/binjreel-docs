# CONTEXT-v0.5.md — binjreel · the stage contract for v0.5

**"Native shell + money — launchable."**

*Authored by the Architect (Phase 1.2). This is the Order. Canon is doctrine; this is the stage
contract. Read `/docker/PROJECTS.md`, then canon `_v2s/`, then `docs/STATE.md`, then this file,
before any code. Identity check: binjreel · money-and-access spine · `binjreel-dev` · Bunny ·
comedy · Journey Viral. If the loaded STATE names a different project — STOP.*

**Baseline:** v0.4 BUILT + GATE-3 Owner-certified 2026-07-31 (design accepted at 390 and 1440;
forced admin password change performed). Repo `papamg/binjreel`, branch `binjreel-dev`, pushed.
Record repo `papamg/binjreel-docs` now carries `/docs`.

**Budget: ≤ $120 total across both phases** (Phase A ≤ $80 · Phase B ≤ $40). Record actual spend
per phase in `BUILD-REPORT.md` as a stat with variance against this line. *(This line exists
because its absence is a standing canon defect — DL-B002, which has now recurred on every order
this project has received. Do not proceed without it; if a future order omits it, log the
recurrence.)*

---

## 0 — Why this order is phased

v0.5 is the first version that cannot be finished by building. Two of its scopes require Apple
and Google **accounts** that are enrolling now and are not expected for one to two weeks
(Apple's D-U-N-S number is the long pole). Everything else is buildable today.

So this Order runs in two phases, and **Phase A does not wait**:

- **Phase A — everything that does not need a store account.** Native shell, native video,
  onboarding, the rewards half that has no ad network in it, the pricing configuration, the
  ledger backup mechanism, and the engine touch. Ends at its own gate.
- **Phase B — the money proof.** StoreKit and Play Billing client flows, sandbox purchase →
  real receipt → real unlock, push delivery, store sign-in providers. Opens when the accounts
  land.

**Standing rule, restated:** an external-account dependency is **DEFERRED-BY-ORDER**, never
BLOCKED. Build the seam, wire the client, fail closed on the absent credential, and say so.

---

# PHASE A

## A1 — Native shell (the app itself)

The Expo codebase at `app/web/` is one codebase for three faces (D-019). Stand up the **iOS and
Android targets** against it. No second codebase, no fork, no parallel design system.

- `VideoSurface` is the **one** platform-split module and this is its explicit v0.5 seam
  (recorded at v0.4). Implement the native side — `expo-video` or the equivalent current API —
  against the same props and the same contract the web `hls.js` implementation satisfies.
  The player's behaviour is already specified and shipped; match it, do not redesign it.
- **Cold start lands in the scene, not in a lobby.** A cold launch resolves session, restores the
  last position if one exists, and gets to playable video without an interstitial step.
- **Deferred deep link.** A viewer who taps a `/s/{token}` share link, installs the app, and
  opens it for the first time lands on the shared moment — not the home screen. The share-link
  token, its origin tag, and the timestamp all survive the install. The origin tag is the JVScope
  join key and has been minted since v0.3; do not drop it on this path.
- **Five tabs, unchanged.** IA is fixed. This is a shell, not a re-architecture.
- Build locally and on device. Store distribution is Phase B.

**Deliberately not in A1:** any change to routes, API, IA, or the design system.

## A2 — Onboarding (the parked conformance debt)

HANDOFF §9.3 and §9.4 specify onboarding screens. No onboarding feature existed at v0.4, so it
was logged as deferred conformance debt **to this version**. Build it now, to the package:
`docs/design-package/canvas-doc-with-six-screens/project/HANDOFF.md` **governs in full**, as it
did for the conformance-completion pass.

- The how-it-works pass and the taste-selection pass, per §9.3/§9.4.
- Taste selections write to real state and actually feed For You's heuristic. **A preference the
  product ignores is a lie told at first launch** — if a selection cannot yet influence anything,
  do not ask for it.
- Skippable, and skipping is not punished.
- Conformance captures at 390 and 1440, into `docs/design-package/conformance-v3/`, by the same
  mechanical route `scripts/conformance.mjs` already uses.

## A3 — The pricing configuration (D-022, Owner-ratified 2026-07-31)

All values are **config, not code** — that property already exists; this scope populates it and
proves it holds. No price is compiled in, and changing any of these must not require a code
change or a store review.

**The anchor: 1 coin = $0.01 nominal.**

| Item | Value |
|---|---|
| Episode unlock | **30 coins** — flat, every series, every position |
| Free episodes | **2 by default**, per-series overridable |
| Pack — starter | $1.99 → 200 coins |
| Pack | $4.99 → 550 coins (500 + 50 bonus) |
| Pack | $9.99 → 1,150 coins (1,000 + 150 bonus) |
| Pack | $24.99 → 3,000 coins (2,500 + 500 bonus) |
| Pack — best value | $49.99 → 6,500 coins (5,000 + 1,500 bonus) |
| Subscription — weekly | $5.99, first week $3.99 |
| Subscription — annual | $49.99 |
| Subscription — monthly | slot exists, **unset** — not offered at launch |

**The hard rule of this scope: the price is flat and visible before the tap.** No escalation
deeper into a series, no per-series variance, no surge on popularity. The category's dominant
players vary the price opaquely; that practice is the single most-cited reason viewers resent
them, and it is the opposite of the shipped voice rule — *numbers are facts, never bait*. A
per-series `episodeCoinPrice` override may exist in config for a future promotion, but every
surface must display the actual price of the actual episode before it is charged, and the
displayed free-episode count must equal the real one for that series.

## A4 — Rewards, the half with no ad network in it

Per TDD §9.2 and D-017, minus everything that needs an ad network (that is v0.6).

- **Daily check-in and streaks.** Check-in credits 10 coins; a completed 7-day streak credits a
  100-coin bonus. Streak state, freezes, and resets per the shipped Rewards design.
- **Task registry** — config-driven, server-enforced daily caps.
- **Referrals** — the graph and the credit path.
- Every reward credit is an **ordinary ledger entry** written under D-011 (append-only, no
  exception) and D-012 (row lock on every wallet mutation). No new money path, no bypass, no
  privileged write. The wallet is source-agnostic by design; this is a new *source*, not a new
  *mechanism*.
- **Rewarded-ad earning and the wall's third door stay closed and fail-closed** — v0.6.
- The Rewards hub leaves its coming-soon state only for the doors that actually open.

**D-023 applies here — reward coins do NOT expire.** See DECISIONS. `coin_class` remains on the
ledger, and expiry remains *mechanically* supported, but no expiry is set on reward credits and
no surface implies one. Do not build a reward-coin expiry sweep.

## A5 — The engine touch (N-008, folded in as promised)

The entitlement engine has been frozen since v0.1 and every order has correctly declined to
touch it. This is its next *ordered* touch, and N-008 rides along:

**`mayPlay` consults publish state.** A draft or unpublished episode must not be playable even
if its id is known — today the engine consults no `publish_state`, and only the fact that no
viewer surface emits draft ids keeps it safe. That is a defence in depth of one. Add the check,
scoped so **every existing v0.1/v0.2/v0.3 refusal keeps its exact shape and status code**, and
prove the full existing matrix unchanged.

**Money core diff target: 0 lines** in `wallet/ledger.ts` and `receipts/*`. The engine change is
the single sanctioned exception in this Order and must be git-verified and reported as a diff.

## A6 — Continuous ledger backup (the last gate before real receipts)

Open since v0.1 as `SECURITY-REPORT.md` **N-001** and STATE backlog #3. It gates real money, and
real money is Phase B, so it lands **here** — before, not alongside.

- Build the mechanism: scheduled, verified, restorable backup of the wallet and entitlement
  tables, with a **restore actually exercised** against a scratch database. A backup nobody has
  restored is a hope, not a backup.
- **The destination is the Director's** — an off-box location and its credential are an Owner
  act. Build to a configured destination, fail closed and loudly if unset, and report the slot in
  `SECRETS.md`. Do not invent a destination and do not write backups to this box only: the whole
  point is surviving this box.
- Do **not** back up to the `binjreel-docs` repo. That repo is the written record; ledger data is
  production data and does not belong in it.

## A7 — Phase A gate

Suites green (existing 147 API + 9 web + 4 admin-gate + 4 assets-gate + 2 resume, plus new).
Security pass on every new surface, 1C pattern, rewards first — **the rewards economy is this
platform's first adversarial surface**: caps enforced server-side, referral self-dealing refused,
check-in unfakeable by clock manipulation, every credit idempotent. Deploy the web face if it
changed. Ledger, report, and defect entries written **before any pause**. Commit and push
`binjreel-dev`. Remote `main` untouched — W-002's two-`main` reconciliation is still the
Director's and is not settled by this Order.

**STOP at the Phase A gate and report.** Do not open Phase B without the Director's word that the
store accounts are live.

---

# PHASE B — opens when Apple Developer and Play Console are live

## B1 — StoreKit and Play Billing, client side

The **server** half is built and has been proven fail-closed since v0.1: Apple ASN V2 (JWS,
ES256) and Google RTDN (Pub/Sub push, OIDC-verified), both normalised to one internal shape, both
double-locked for idempotency, both crediting **nothing** while their keys are absent. Do not
rebuild it. Do not weaken it. Wire the client to it.

- Coin packs and subscriptions as real store products, priced per A3.
- **The client never grants anything.** A purchase completes on the device, the store notifies our
  server, the server verifies the signature and writes the ledger entry, and only then does the
  entitlement exist. A device that claims a purchase our server has not verified gets nothing.
- Restore purchases. Subscription lapse and renewal reflected in entitlement state.
- **The web top-up lane (D-020) is not activated in this Order** and the app links to nothing
  resembling it — store policy.

## B2 — Sandbox proof (the actual definition of done)

A real sandbox purchase on **both** platforms produces a real store-signed receipt, our server
verifies it, the ledger moves, and a locked episode plays. Then: a replayed notification credits
**nothing further**; a subscription lapse refuses playback mid-session at the next re-mint; a
forged receipt is refused. These are the properties v0.1 proved against test keys — prove them
again against the real stores.

## B3 — Push and store sign-in

- Push notifications for scheduled episode drops, riding the follow-fanout hook that has existed
  as a log-only sweep since v0.3. Needs the Apple push key — an Owner act.
- Apple and Google sign-in providers: the UI and the anonymous→real merge are already wired and
  proven; this is provider configuration against the new accounts.
- **The merge must survive a purchase.** A viewer who buys as an anonymous account and then signs
  in keeps her coins and her entitlements. This is tested since v0.1; re-prove it on the real path.

## B4 — Phase B gate

All of A7's discipline, plus: the compliance set confirmed placed by the Director before any
store submission (auto-renew and click-to-cancel disclosure, digital-goods tax, age-gating,
privacy policy matching actual telemetry, DMCA). **No store submission happens inside this
Order** — submission is the Director's act, at the Director's timing.

---

## Out of scope — do not build

Ad network, rewarded-ad earning, the wall's third door (v0.6) · studio admin v2 and the §5 admin
screens with no product behind them (v0.7) · the events sink and the JVScope join (v0.8) ·
invited-studio tenancy and revenue share (v0.9+) · the Stripe web top-up lane (D-020, activation
is a Director timing decision) · DRM (parked on its v1 reopen criteria) · light theme (Owner
ratified dark-only) · any IA change, any redesign, any new tab.

## Standing constraints

- **The money core is frozen.** `wallet/ledger.ts` and `receipts/*`: 0-line diff, git-verified at
  each gate. A5's engine change is the one sanctioned exception and is reported as a diff.
- **D-011** append-only in the database, no exception. **D-012** row lock on every wallet
  mutation — the concurrency test that proves it is the only test that fails if the lock is ever
  refactored away. Neither is negotiable, in either phase.
- **The suite runs isolated.** `docker compose run --rm test` only. The test-database-name guard
  (DL-B011) stays fail-closed. The live database is never the suite's target.
- **Ledger before any pause.** `BUILD-LEDGER.md` is written before any stop, any reply to the
  Director, and any write to a doc the Worker does not own — whether or not the step produced
  code (the L-012 standing correction).
- **Defects get tracebacks, not first guesses.** Falsify the first hypothesis before naming a
  cause. Next free numbers: **DL-B012** onward (B009, B010, B011 are taken).
- **No credential value, hash, or secret is ever printed, logged, echoed, or committed** — in
  code, in docs, in a ledger, or in a report.

## Definition of done

**Phase A:** native app runs on both platforms against the one codebase · native player matches
the shipped web contract · cold start lands in the scene · a deferred deep link survives install
with its origin tag · onboarding built to HANDOFF with real taste state · pricing config
populated and flat-price-visible proven by test · check-in, streaks, tasks and referrals credit
through the ordinary ledger with server-side caps · reward coins do not expire and no surface
says otherwise · `mayPlay` consults publish state with every prior refusal shape unchanged ·
ledger backup mechanism built and a restore **exercised** · suites green · security pass with
rewards walked adversarially first · deployed where applicable · pushed.

**Phase B:** a real sandbox purchase on each platform verifies server-side and unlocks · a replay
credits nothing further · a lapse refuses at re-mint · a forged receipt is refused · the
anonymous→real merge survives a purchase · push delivers on a scheduled drop · store sign-in
works on both providers · compliance set confirmed placed.

## Stop point

**Phase A ends at its gate — stop and report.** **Phase B ends at GATE-3, which is the Owner's:**
the Owner buys coins with real money in sandbox on their own device, watches the balance move,
unlocks an episode, and sees it play. Nobody else can perform that test.
