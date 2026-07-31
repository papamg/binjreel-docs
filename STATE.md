# STATE.md — binjreel

> ## ⛔ STEP ZERO — CANON FIRST (read before scoping, architecting, or retrofitting)
> **Pull and read canon (`/docker/v2s-canon/_v2s/`) as authoritative before acting:**
> `git -C /docker/v2s-canon pull`. That read includes
> `_v2s/procedures/chat-operating-instructions.md` **at session open** — canon is authoritative
> over any pasted copy, and where canon has moved ahead of a paste, canon governs for the rest
> of the session.
> **Everything in `/docs` is operational state, not doctrine; where they conflict, CANON WINS.**
> Canon for doctrine · STATE for project state. Where `/docs` and chat memory disagree, `/docs`
> wins — reconcile, don't reconstruct.
>
> **Addressing (DL-010).** Read `/docker/PROJECTS.md` **before** this file. Address every doc by
> full project-qualified path (`binjreel/docs/STATE.md`), never a bare `STATE.md` — those
> filenames collide across the fleet. *(This bit us once: a bare `TDD.md` from a different project
> was uploaded over binjreel's by name collision, caught and corrected 2026-07-25.)*

**Identity check.** This is **binjreel** — *Short-Form Vertical Drama Platform*. App class
**consumer content SaaS, multi-tenant-ready (single-tenant in practice)**, audit tier **high**.
Repo `papamg/binjreel`, build branch `binjreel-dev`. *If you opened this expecting a different
project, product, or version line: **STOP** — do not scope, reconcile, or write.*

---

## WHERE WE ARE

**RESTYLE ORDER — DELIVERED, LIVE (2026-07-30). STOPPED AT GATE-3: the Owner's viewing is
the test.** The ratified design package (unpacked to `docs/design-package/`, zip cleared from
cubby) is transcribed into the binding contract **`app/web/DESIGN.md` v2** and the deployed
face now wears it: the three-accent system (tangerine = tap, lime = free, pink = crowd),
Gabarito/Plus Jakarta Sans/JetBrains Mono via Google Fonts, and the five briefed surfaces
pixel-close (home hero + shelves + Top-10 rank medals · show page with the kit episode grid ·
glass-chromed player with act progress bar + 2400ms auto-hide · three-door wall, sheet <1024
/ 860px modal ≥1024 · admin login + forced change), plus the component kit states. IA,
routes, API, behavior: unchanged — **API 147 + web server 9 + admin-gate e2e 4 all green,
none modified; served == merged (bundle hash matched live).** One defect found & fixed under
the restyle: stage presses on the raw `<video>` were invisible to RNW's responder system
(tap-toggle/hold dead in browsers since v0.4) — web stage input now runs on DOM pointer
listeners. Divergences recorded in DESIGN.md v2 §9 (dark-only ship, ha-stat slots styled but
never faked, five tabs at all widths, production art still owed). Side-by-sides for the
Owner: `docs/design-package/conformance/01…08-*.side-by-side.png`. *As-built: L-041…L-043 ·
BUILD-REPORT (restyle engagement).* **GATE-3: browse binjreel.com on your device — taste is
the acceptance; the /admin temp credential remains untouched for your forced change.**

**v0.4 — the web face — BUILT, SECURED, DEPLOYED LIVE AT https://binjreel.com (2026-07-28).
STOPPED AT GATE-3: the Owner's live viewing is the final verification.** Phase 2 of the
combined Order. The Expo (D-019) web app serves at the domain root — five tabs, hero/shelf
home, SEO show pages (server-injected, verified live), search/tags/hubs/My List, the hls.js
vertical player (prefetch-fed swipe, **mid-session re-mint proven live**, captions/speed/
quality/drawer, tap-to-hold), the three-door wall rendered from the server offer verbatim
(purchase doors "get the app — coming soon", ad door fail-closed), share-a-moment + /s/
landings, For You teaser feed — while the API keeps `/v1/*` + `/health` by path rule (Bunny
callback verified unaffected; www→apex 301). **Real admin identity:** michael@journeyviral.com
seeded HASH-ONLY, forced first-login change (server-enforced — operator acts refuse until it
lands), eye-reveal fields, login rate-limited (429 proven live), sessions HttpOnly/Secure/
Strict cookie; the D-014 token never reaches a browser (bundle scanned in-suite and live: 0
hits) — the /admin panel drives shelves/schedule/tags through an allow-listed server-side
proxy. **147 API + 9 web-server tests green · money core diff across the whole v0.3+v0.4
engagement = 0 lines · security pass clean (0 new; 1 build-tooling advisory dispositioned).**
The temp admin credential is UNTOUCHED — the Owner's own first login performs the forced
change. DEFERRED-BY-ORDER (external accounts): Apple/Google sign-in providers, IAP (v0.5), ad
network (v0.6). *As-built: L-035…L-038 · BUILD-REPORT (combined engagement) · SECURITY-REPORT
v0.4 · design contract `app/web/DESIGN.md`.* **Pushed to `binjreel-dev`; remote `main`
untouched (W-002 parked).**

**GATE-3 for the Owner:** browse https://binjreel.com on your device · play "Fired & Furious"
Ep 1 (the proven test clip; free) · swipe · hit the wall on a locked episode · dismiss ·
share a moment · sign in at /admin with the temp credential and complete the forced change.

**v0.3 — the client contract — BUILT, GATE-2 PASSED, DEPLOYED (2026-07-28). The server can be
browsed.** Phase 1 of the combined v0.3+v0.4 Order (kickoff 2026-07-28; CONTEXT-v0.4.md placed
in /docs and goes live as the stage contract at this gate). All 14 ordered scopes landed:
schema migration (15 new tables — genre/nonfiction (D-018), tags, shelves+hero, follows, lists,
progress+resume, history, share links+resolutions, schedule, captions, cast), catalog reads
with per-caller lock states, shelf/home composition, show-page one-call, search+trending,
Top-10, engagement writes (merge-safe — resume survives anonymous→real), next-episode+prefetch,
the three-door offer (ad door data-present, fail-closed false per D-017), entitlement summary
(mayPlay-arbitrated), **mid-session re-mint (DL-B007 CLOSED FULLY: 900s code default + renew
route)**, share-the-moment + operator deep links (128-bit tokens, origin tags persisted),
caption manifests, scheduling honoured at every surface incl. the mint (ADDED guard), log-only
follow-fanout hook. **138/138 tests green (60 v0.1 + 27 v0.2 unbroken + 51 new); money core
diff = 0 lines (git-verified); security pass: found 1 LOW (F-004 trending injection), fixed
closed-loop, no high/critical open.** Deployed on-box: migration applied to the live DB, comedy
catalog seeded in place ("My Landlord Is a Billionaire (Unfortunately)" · "Fired & Furious"),
`/health` → 0.3.0, live leak-grep clean. *As-built: L-031…L-034 · BUILD-REPORT · SECURITY-REPORT
v0.3.* **Pushed to `binjreel-dev` per the combined Order (the push path L-022; remote `main`
untouched; W-002's two-`main` question stays parked).*

**v0.2 — media — GATE-3 COMPLETE (2026-07-27). The platform streams real, paywalled video.**
Commit `574275a` on `binjreel-dev` (v0.2 base `b0b67bc`; v0.1 rollback point `9c757af`).
The full arc: GATE 2 closed fail-closed → Director placed the six media slots and deployed →
the GATE-3 live test caught **DL-B006** (single-file token killed HLS; fixed same day —
directory-scoped HS256 token per Bunny's reference, D-016) and then **DL-B007** (30s TTL died
mid-episode; resolved operationally, `PLAYBACK_URL_TTL_SECONDS=3600` in `.env`) → **all four
GATE-3 DoD items proven live**: manifest 200 · segment 200 under the same token · expired 403 ·
master bucket refuses unauthenticated GET (400, public access disabled) — **and the Director
watched a real episode play in VLC.** Zone config corrected en route: Bunny's "Block direct URL
file access" was ON (blocked every no-referer client, protected against nothing) — now OFF;
CDN token auth remains the real gate, and the v0.3 player blocker is cleared. Real Bunny token-auth signing at the seam,
master storage we own (SigV4 client, no unsigned URLs), ingest + the async completion callback
driving the episode media-state machine — ALL FAIL-CLOSED: with the six new SECRETS.md slots
empty, playback mints nothing (503), registration refuses, the callback rejects everything.
**87/87 tests green (60 v0.1 unbroken + 27 media); money core diff vs v0.1 = 0 lines.** One
DIVERGENCE logged as the Order anticipated (the body swap needed the signing config threaded —
one argument at one call site; L-024). Media security pass: found 1 LOW (webhook secret would
have hit request logs), fixed closed-loop. **NOT pushed** (by order; W-002 two-`main` decision
still open). *As-built: `BUILD-LEDGER.md` L-023…L-028 · `BUILD-REPORT.md` · `SECURITY-REPORT.md`
v0.2 pass (GATE-3 items 1/2/4 now marked live-proven).*

**v0.1 — the money-and-access spine — is BUILT, tested, audited and deployed on-box.** Commit
`9c757af` on `binjreel-dev`. The Phase-1.3 Express scaffold is gone; the real Fastify/TypeScript
service is running with Postgres and Redis behind it.

**The correctness-critical properties are proven, not asserted** — 60 tests against a real
Postgres and a real Redis, plus 10 live checks through Traefik:
a replayed store notification credits **nothing further** · a replayed unlock does **not**
double-debit and concurrent spends cannot overdraw · an unentitled request is refused and yields
**no playback URL** · a cross-tenant read returns **nothing** · the anonymous→real merge preserves
balance and entitlements, and the stale token still on her phone follows the merge.

**GATE 2 passed.** 1C full-tier audit (Base + Auth & Access + Tenant Isolation): **found 2, fixed
2, 0 needing the Director**, no high/critical open. A third defect was caught and closed by the
test suite mid-build. All three are in `BUILD-REPORT.md`.

**Two decisions were taken in code and now constrain v0.2+.** **D-011** — the ledger is
append-only *in the database*, with no exception at all; a wallet movement is expressed as paired
compensating entries, never an UPDATE. **D-012** — idempotency keys stop replays but do *not*
prevent double-spend, so every wallet mutation takes a row lock; the concurrency test that proves
this is the only test that fails if the lock is ever refactored away.

**What is deliberately absent** (CONTEXT's out-of-scope list, all honoured): no Bunny or real
video · no app, player or paywall UI · no studio admin · no `events` ingestion · no Stripe · no
tenant-management machinery · no reward-earning surfaces.

**Standup posture unchanged.** V2S-bound, bridge live and scoped to `/docs` only, kept provisional
per D-009 pending IODE remote-write. Write path for Architect docs remains Director-run `scp`.

### Live state (verified 2026-07-25)

| | |
|---|---|
| Design docs | `IDEAL-SCENE.md` ✅ · `TDD.md` ✅ (roadmap advanced: v0.1 + v0.2 built; §11(2) divergence awaiting Architect reconciliation) |
| Stage contract | `CONTEXT.md` v0.2 ✅ **delivered in full — GATE-3 complete**, every DoD item proven (the two live-only items by the Director's own viewing); v0.1 contract archived at `CONTEXT-v0.1.md`, also delivered in full |
| `binjreel-app` | **healthy · v0.1 service** (Fastify/TS) · `https://binjreel.com` + `www` · non-root, read-only rootfs, all caps dropped, **zero host mounts** |
| `binjreel-postgres` | healthy · **no host port** · `internal: true` network only · 2 migrations applied · fixtures seeded (2 series, 14 episodes) |
| `binjreel-redis` | healthy · **no host port** · sessions (as digests) + rate-limit counters only |
| `binjreel-mcp-bridge` | healthy · `https://docs.binjreel.com` · scoped to `/docs` only · **wall re-verified after the topology change** · provisional (D-009) |
| DNS | `binjreel.com`, `www`, `docs.` all resolve to this box ✅ |
| Git | `binjreel-dev` @ **`574275a` (v0.2 + DL-B006 fix)** · committed locally · still **unpushed by order and by choice** — the credential is verified working (L-022), but the Order said no push and the two-`main` decision (W-002) is still open |
| Deployed vs built | **Container serves `574275a`** · healthy through Traefik · all six media slots placed · `PLAYBACK_URL_TTL_SECONDS=3600` (operational, DL-B007) · **GATE-3 complete: real video streams behind the paywall, verified by live viewing** |
| Tests | **87 passing, 0 failing** (60 v0.1 + 27 media) · `npm audit`: 0 vulnerabilities across 6 prod deps (v0.2 added none) |
| Decisions | D-001…**D-016** logged |
| Defects | DL-B001…**DL-B007**; B005 resolved · B006 closed-verified live · **B007 (TTL < viewing session) closed operationally — code-default fix open for the next code touch**; B001–B003 open on the canon route |
| Canon | **read path LIVE** · local clone current at `834fc98`, origin confirmed responding · advanced **2 commits** past the `ab07022` the v0.1 build read |

---

## NEXT ACTION

**v0.2 GATE 3 is CLOSED.** The baton passes to the Architect.

**Architect — the v0.3 stage contract** (player & funnel: vertical gapless player, deferred
deep-link, paywall interstitial, continue-watching). Three things this gate learned that v0.3's
order should carry:
1. **TTL is a session-length dial, not a seconds dial** (DL-B007) — the player consumes the
   signed directory token for the whole episode. If the Director ever wants a shorter horizon,
   the player needs mid-session re-mint; design that seam into v0.3 or explicitly defer it.
2. TDD §11(2) reconciliation — the "no call site moves" premise diverged as built (one argument
   threaded at one call site, L-024); fold the as-built into the next TDD revision.
3. The zone's referer blocking is OFF by deliberate decision — token auth is the gate. v0.3's
   player should NOT be built to depend on referer headers existing.

**Director — two small items, neither urgent:**
- **Final playback TTL** (product decision, TDD §12 shape): 3600s is the working interim;
  anti-sharing vs viewing-session is yours to set. The 30s CODE default in `config.ts` gets
  raised at the next code touch regardless (DL-B007 open item).
- **Push + two-`main` + branch protection** (W-002): v0.2 exists as a SINGLE COPY on this box —
  `574275a` is committed but unpushed, so a disk failure loses the build. The push itself is
  one command once you settle the two-`main` question.

**Before real money flows (unchanged, the last gate standing): the continuous ledger backup.**
Delivery is now real; the moment real receipts go live, the ledger stops holding fixtures.
Backlog #3, SECURITY-REPORT N-001. Apple/Google receipt keys stay fail-closed until then.

---

## WAITING ON — Director

### W-001 · Git credential is placed but matches nothing (canon read is DOWN) — ✅ CLEARED
**Status: RESOLVED 2026-07-26 · verified by observation, not by report**

**Cause, as corrected:** host-only placement on a path-keyed box. The Director has since moved both
paths to the intended path-keyed model — an account-level `papamg` write path, and a box-level read
path to `papamg007/v2s-canon`. **No credential was placed, read, or printed by the Worker in the
verification run.**

**Evidence (full raw output: `BUILD-LEDGER.md` L-022):**
- **Canon read — PASS.** `fetch` exit 0; `pull` → *Already up to date*; HEAD `834fc98`. Because
  "already up to date" alone cannot distinguish a current clone from an offline no-op, origin was
  forced to answer: `git ls-remote origin HEAD` returned `834fc987f…`, matching local HEAD, with
  `FETCH_HEAD` written at `21:46:12` in-run. **Origin responded.** Canon is **2 commits ahead** of
  the `ab07022` v0.1 read (`834fc98`, `9fa0e13`) — so the staleness this entry warned about was
  real. `_v2s/` and `_v2s/procedures/` fully populated (FRAMEWORK/BLUEPRINT/IMPLEMENTATION-GUIDE +
  all nine procedures); tree clean.
- **binjreel write — PASS.** `push origin HEAD:refs/heads/_writecheck` → `* [new branch]`, then
  `push origin --delete _writecheck` → `- [deleted]`. Post-check `ls-remote` shows the ref gone and
  `main` still at `ec2ef3d1`. **Nothing real was touched and nothing was left behind.**

**Still open from this entry, moved to W-002:** the two-`main` decision (remote `main` `ec2ef3d1`
vs this box's history) and the account-vs-repo scope of the write credential.

<details><summary>Original diagnosis, retained for the record</summary>

The PAT placed at 18:19 landed in `/root/.git-credentials` as a **single host-only entry**
(`https://<redacted>@github.com`, no repo path). This box sets `credential.usehttppath=true`
globally, so git looks credentials up by host **+ path** — a pathless entry matches no request.
Result: `git ls-remote` fails identically on **both** `papamg/binjreel` **and**
`papamg007/v2s-canon`.

**CONFIRMED by direct test, not inferred.** Running one command with host-only matching forced on
(`git -c credential.useHttpPath=false ls-remote`) **succeeded against `papamg/binjreel`** and
listed the repo. So the token is **good**; only its label was wrong. The same command against
canon returned `Repository not found` — GitHub saying *authenticated, but this account cannot see
that repo* — because canon lives under **`papamg007`**, a different account. Canon therefore needs
its **own** entry, and its token is not recoverable from this box.

Also confirmed: the remote `papamg/binjreel` **exists** and already carries a `main` at
`ec2ef3d1`, unrelated to this box's history. Harmless for now — `binjreel-dev` does not exist on
the remote and will push cleanly as a new branch without touching `main` — but the two `main`
branches will need a deliberate decision later, and that decision is the Director's.

Full diagnosis with both alternative causes falsified: **DL-B005**.

This is the failure `secret-manifest.md` ("Slot scope") and **D-005** predicted — a per-project
placement replacing the shared box credential — made worse by being host-only on a path-keyed box,
so it authenticates nothing at all.

**Ask (recommended):** re-place as **two path-keyed entries**, matching the remote URLs exactly,
including the `.git` suffix git actually sends:
```
https://<user>:<canon-read-token>@github.com/papamg007/v2s-canon.git
https://<user>:<binjreel-token>@github.com/papamg/binjreel.git
```
**Not recommended:** `credential.usehttppath=false` — it would make the current entry match, but
points one credential at every GitHub repo on the box (the fleet-wide blast radius D-005 exists to
prevent) and would still 403 on canon if the PAT is fine-grained to `papamg/binjreel`.

**Worker can do the partial repair on your word:** append the path to the existing entry in place
(a suffix edit; the value is never printed or handled). That restores **binjreel push only** —
canon's token is not recoverable on this box and is yours to place.

**Consequence while open:** the v0.1 build reads canon from the local clone at `ab07022` — clean
tree, the same commit `L-000` verified — but **unverified against `origin/main`**. Recorded, not
papered over.

</details>

### W-002 · Write path is proven for one repo, not for an account (open)
**Status: open · does NOT block · DOES bear directly on the Account-Aware Credential Routing build**

The Director's framing is an **account-level `papamg` write path covering the Journey/binjreel
repos**. What was actually observed is narrower: **`papamg/binjreel` pushes**. No second `papamg`
repo was named or tested.

This matters because `credential.useHttpPath=true` is still `true` on this box (re-confirmed in the
verification run). Under that setting git looks credentials up by host **+ path**, so an entry keyed
to `.../papamg/binjreel.git` matches **that repo only** — a sibling `papamg` repo would miss it in
exactly the way the host-only entry missed everything in DL-B005. Account-level routing is not a
property a path-keyed entry has for free; it is precisely what the routing brief sets out to build.

**Ask:** name a second `papamg` repo and it can be proven or disproven in one `ls-remote`. Until
then the plural claim is **unverified**, and the coming build must not assume routing is already
solved.

**Also still open (carried from W-001):** remote `papamg/binjreel` `main` sits at `ec2ef3d1`,
unrelated to this box's history. `binjreel-dev` will push cleanly as a new branch without touching
it, but the two-`main` reconciliation is a deliberate Director decision and has not been taken.

---

## BACKLOG

| # | Item | Status | Where |
|---|---|---|---|
| 1 | ~~Build v0.1 — money-and-access spine~~ | ✅ **DONE** 2026-07-25 · commit `9c757af` · 60 tests green · GATE 2 passed | `docs/BUILD-REPORT.md` |
| 2 | ~~Re-place git credential path-keyed~~ ✅ **DONE 2026-07-26** — canon read + binjreel write both verified live (`BUILD-LEDGER.md` L-022). **Residue still open:** branch protection on `main` never confirmed (the write check used a throwaway ref, which protection would not have challenged), the two-`main` reconciliation, and account-vs-repo credential scope | **partly open — Director (W-002)** | `DEFECT-LEDGER.md` DL-B005 |
| 3 | **Continuous ledger backup — now gating real-money go-live** (the last gate standing before real receipts) | open — Director/Treasury | `SECURITY-REPORT.md` N-001 |
| 4 | Apple ASN + Google RTDN verification keys (slots declared, **fail closed** while empty) | open — Director, at real-receipt go-live | `docs/SECRETS.md` |
| 5 | ~~v0.2 — media~~ | ✅ **DONE — GATE-3 COMPLETE 2026-07-27** · `574275a` · 87 tests green · all four live DoD items proven (incl. bucket privacy) · live VLC viewing confirmed · zone referer-block resolved (OFF; token auth is the gate) | L-028/L-029 · DL-B006/B007 |
| 6 | v0.3 — player & funnel (**next — Architect writes the CONTEXT**; carry the DL-B007 TTL/re-mint seam question) · v0.4 — studio admin (retires the interim `media-admin-token`, D-014) · v0.5 — retention + `events`→JVScope | queued | `docs/TDD.md` §11 |
| 7 | IODE remote-write capability (retires all remote per-project bridges) | open — Architect routes via Tone canon drop | D-009 |
| 8 | Canon gaps DL-B001/B002/B003/**B005** (empty app-class library · no token budget in orders · no false-Director-gate rule · **credential placement not required to be path-keyed / appended**) | open — route via Tone canon drop | `docs/DEFECT-LEDGER.md` |
| 9 | Product decisions: pricing, free-episode count, auto-unlock default, launch catalog size, **final playback TTL (DL-B007)**. *(Anonymous-first is now **built** per CONTEXT §3 — confirm or reverse.)* Coin values are **config-driven**, not compiled in: set them without a code change | open — Director (none blocked v0.1) | `docs/TDD.md` §12 |
| 10 | Compliance/Treasury: auto-renew rules, digital-goods tax, age-gating, privacy, DMCA | open — Director/Treasury | `docs/TDD.md` §12 |
| 11 | **5A scalability & tenancy audit has not run** against v0.1 — CONTEXT ordered only the 1C pass. Index/query-plan coverage under tenant-scoped load, pooling, and backup/DR *architecture* are unexamined | open — recorded so its absence is a known gap, not an assumed pass | `SECURITY-REPORT.md` seam note |

---

## Reading order for the next session

1. `/docker/PROJECTS.md` — which project, where its records (read first).
2. `git -C /docker/v2s-canon pull`, then canon's `_v2s/` — doctrine.
3. This file — project state.
4. `docs/IDEAL-SCENE.md` — the destination (what "done right" is).
5. `docs/TDD.md` — the as-designed technical intent.
6. `docs/CONTEXT.md` — the v0.2 stage contract (the live Order; v0.1's is `CONTEXT-v0.1.md`).
7. `docs/DECISIONS.md` D-001…D-015 — the settled forks.
8. `docs/BUILD-LEDGER.md` — the as-built trail (standup L-000…L-009 · v0.1 L-011…L-021 ·
   credential verification L-022 · v0.2 L-023…L-027).
