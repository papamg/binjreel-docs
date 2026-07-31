# Security Report — binjreel

*Cumulative, per-build sections, newest first. Closed-loop record, not a to-do list: every
finding with a determinable fix was fixed, re-verified, and logged on the spot
(`procedures/decision-routing.md`). Only genuine four-item tradeoffs rise, as CSW packets.*

---

# 2026-07-28 (later) · DL-B008 — screen-level admin auth (fix pass, web face only)

**Order:** DEFECT FIX DL-B008 · **Branch:** `binjreel-dev` · **Scope:** app/web only — no API
change, no ledger, no design work.

## The finding (F-005 · UI gate · fixed closed-loop)

The v0.4 pass verified admin auth exhaustively AT THE API SEAM and never asserted it at the
SCREEN — the surface a human actually meets. As found (Architect, deployed code): the server
hands the full SPA shell + admin bundle to anyone at /admin; the only gate was client component
state that no test exercised; the Owner had not been routed through the forced password change,
so the seeded temp credential was still live.

**What was and was not reachable (stated precisely).** Reachable by a stranger: the admin
screen SCAFFOLDING (client-rendered UI states) and — confirmed by headless reproduction — a
viewer-session bootstrap firing on the /admin route (`POST /v1/auth/anonymous`, one anonymous
viewer account minted per visit). NOT reachable: any operator read or write (every
`/admin/api/ops/*` call 401s without a session — re-verified), any private data, any secret
(the bundle scan held: 0 secret strings). Reproduction on a clean profile showed the login card
rendering on fresh load — the exposure was a furnished-lobby and unasserted-surface defect, not
an open door. The temp credential's liveness was by design pending the Owner's GATE-3 login;
the defect is that nothing STRUCTURAL forced that flow at the screen.

## The fix

- **Structural gate** (`src/app/admin/_layout.tsx`): every /admin route renders through one
  layout that resolves `GET /admin/api/me` FIRST and renders ONLY login (no session), ONLY the
  forced-change screen (must-change; success revokes all sessions and lands at login), or the
  panel routes. A future admin screen is gated by construction.
- **Bare lobby:** the /admin surface fires exactly ONE call before authorization — the session
  resolve. The root layout no longer bootstraps the viewer session on /admin (the reproduced
  leak, closed). Panel ops meeting 401/403 mid-session re-enter the gate — no stale panel.
- **The missing test class:** a Playwright screen-level suite drives the REAL exported bundle
  in a REAL browser: fresh /admin = login-only + exactly-one-network-call (asserted against
  both the browser's request log AND the stub API's receive log); must-change = change-only;
  the full temp→change→re-login journey ends at the panel; the eye-reveal is asserted. 4/4.

## Re-verified live (screen level, clean profile — not curl)

https://binjreel.com/admin renders the login form and nothing else (login 1 · panel 0 ·
change 0); network = `GET /admin/api/me` alone; served == merged (bundle hash match container
vs tree); viewer face regression-checked (bootstrap + catalog render intact).

**Traceback lesson (for the next Order's DoD):** name the SURFACE with the property —
"enforced" got proven where it was cheapest to test. Recorded in DEFECT-LEDGER DL-B008.

---

# 2026-07-28 · v0.4 — the web face (dedicated pass, CONTEXT v0.4 §12 — admin auth foremost)

**Build:** v0.4, Expo web + admin + deploy · **Branch:** `binjreel-dev` · **Gate:** GATE 2/3,
run against 147 API tests + 9 web-server tests (all green) and the LIVE deployment at
binjreel.com. Money core diff across the whole v0.3+v0.4 engagement: **0 lines** (re-verified
by git diff at this gate).

## Summary — v0.4 pass

| | |
|---|---|
| **Found** | **0 new defects** (the design review below closed its questions in-build) |
| **Fixed** | — |
| **Needing you** | **0** |

**By severity:** Critical **0** · High **0** · Medium **0** · Low **0** ·
Informational **2 noted** (N-009, N-010) · **1 dependency advisory dispositioned** (below)

**Open high/critical: none.** CONTEXT v0.4 DoD "no high/critical open" — met.

## Admin auth — the ordered focus, walked line by line

| Check | Result | Evidence |
|---|---|---|
| **Hash-only storage** | ✅ | `admin_users.password_hash` is bcrypt cost 12; no code path stores/returns/logs a plaintext; the seed carries ONLY the hash (the temp value exists nowhere in the repo — it was hashed out-of-band from the Order document, never echoed); tested (`$2` prefix, plaintext absent from row) |
| **Forced change on first login** | ✅ + live | `must_change_password` seeded true; server-enforced — the web ops proxy refuses (403) any must-change session, so the flag gates OPERATOR ACTS, not just UI; a change requires the current password, a 10+ char different new one; **live: seeded login answers `mustChangePassword: true`** and the temp credential remains for the Owner's own first login |
| **Change revokes everything** | ✅ | `passwordChangedAt` kills every session issued before it (including the one that made the change) — proven by test on two parallel sessions; the temp password is dead after the change (tested) |
| **Uniform failures** | ✅ | Unknown email, wrong password, malformed body → byte-identical 401s; the unknown-email path runs a REAL bcrypt compare against a burn-in hash so timing does not reveal which emails exist |
| **Rate limiting** | ✅ + live | 5/min default; suite proves 429 under storm; **live storm: 401×4 then 429×3** |
| **Sessions** | ✅ | Server-side, Redis, SHA-256 of token stored (a session-store dump yields nothing), 12h TTL, separate namespace from viewer sessions; an admin token is NOT a viewer token and NOT the D-014 token (both cross-uses tested) |
| **No secret reaches the browser** | ✅ + live | Login proxy moves the session token INTO an `HttpOnly; Secure; SameSite=Strict; Path=/admin` cookie and OUT of the body (tested + live: body carries only email + flag); the D-014 token lives in the web container env and rides only proxy-to-API hops; the served bundle scanned for token names/values/temp-credential fragments — **0 hits, in the suite AND against the live bundle** |
| **The ops proxy cannot be a general forwarder** | ✅ | Explicit method+path allow-list (8 operator routes); unlisted paths (incl. the media-pipeline master registration and a traversal probe) answer 404 WITHOUT touching the API — proven by a stub-API assertion that no request arrived |

## The rest of the ordered surfaces

| Check | Result | Evidence |
|---|---|---|
| **Wall renders server data only; zero purchase/ad code paths** | ✅ | `WallOverlay` lays out the offer payload verbatim; no price/pack/copy constant exists client-side; purchase doors render a static get-the-app state; grep: no billing SDK, no ad SDK, no fetch from the wall |
| **No client-side entitlement logic** | ✅ | The player asks `/playback` and renders the answer: URL plays, 403 walls; lock states come from the server's episode grid; no media URL is ever constructed client-side (the only URLs touching the video element are grant URLs, and re-mint rewrites use the grant's own prefix) |
| **SEO injection is escape-safe** | ✅ | Every interpolated value HTML-escaped (tested with `<script>`-shaped synopsis); show-page fetch failure serves the plain shell, never an error page |
| **Share-card SEO does not touch the funnel** | ✅ | `/s/*` gets a static card WITHOUT resolving the token server-side — no double-counted `link_resolutions` (tested) |
| **Web container posture** | ✅ | Non-root, read-only rootfs, all caps dropped, no-new-privileges, NOT on the DB network (talks only to the API like any client), zero runtime npm dependencies (node stdlib server + static files) |
| **Traefik seam** | ✅ + live | API keeps `/v1/*` + `/health` at priority 100; web at root; **Bunny callback path verified live (401 = reachable + refusing, exactly as before)**; www→apex 301 live; HSTS/nosniff/frame-deny middleware on both routers |
| **Player/renew surfaces unchanged** | ✅ | The web face consumes v0.3 routes as-is; live grant + live renew verified against the real zone |

## Dependency advisory — dispositioned, not ignored

`npm audit` (web workspace): 11 moderate, ALL one root cause — `uuid < 11.1.1` (buffer bounds
in v3/v5/v6-with-buf) via `xcode`/config-plugin BUILD tooling under `expo-splash-screen`/
`expo-linking`. Disposition: **build-time only, never shipped** — the runtime web image
contains the exported static bundle + a zero-dependency server (no `node_modules` at all), and
the bundler does not include the affected code paths. The non-breaking `npm audit fix` cannot
resolve it without desyncing the Expo SDK. **Revisit at the next SDK bump (v0.5 native work).**
API workspace (`bcryptjs` added this build): 0 vulnerabilities.

## Informational

- **N-009 · Safari re-mint caveat.** Browsers without MSE use native HLS, where per-request
  URL rewriting isn't available; a token shorter than an episode could die there. Mitigated
  operationally (TTL 3600 on the box) and recorded in the player module; the native app (v0.5)
  owns its own renewal.
- **N-010 · The seeded bcrypt hash is committed.** Deliberate and Order-sanctioned (hash-only
  storage; the temp value "transited chat and is invalidated by the forced change"). Offline
  cracking of a strong hash of a one-time credential that dies at first login is a
  non-economic attack; noted so nobody mistakes the hash for a secret worth keeping.

---

# 2026-07-28 · v0.3 — the client contract (dedicated pass, CONTEXT v0.3 §14)

**Build:** v0.3, catalog/engagement/offer/share · **Branch:** `binjreel-dev` · **Gate:** GATE 2,
run against the built code and the 138-test suite on real engines (60 v0.1 + 27 v0.2 + 51 new,
all green). Money core diff verified **0 lines** by `git diff` (`wallet/`, `entitlements/
engine.ts`, `receipts/` untouched).

## Summary — v0.3 pass

| | |
|---|---|
| **Found** | **1** |
| **Fixed (closed-loop, re-verified)** | **1** |
| **Needing you** | **0** |

**By severity:** Critical **0** · High **0** · Medium **0** · Low **1 (fixed)** ·
Informational **2 noted** (N-007, N-008)

**Open high/critical: none.** CONTEXT v0.3 DoD "no high/critical open" — met.

## Closed-loop — v0.3

### F-004 · LOW · Trending searches were an arbitrary-string injection channel
**Found by:** walking every place a CALLER-SUPPLIED value becomes VIEWER-VISIBLE data. Search
queries are recorded for trending and echoed to every viewer's search screen.

**What was wrong.** Every non-empty query was recorded, so anyone could publish arbitrary
strings ("visit evil.example") onto the trending list of every other viewer at rate-limit speed
(300/min). No XSS (JSON API, client renders text), but a free defacement/abuse channel.

**Fix.** Only a query that MATCHED at least one published series may trend — the trending
vocabulary is now bounded by the catalog itself. Re-verified by test (a no-result query never
appears in trending); suite 138/138.

## The ordered surfaces (CONTEXT v0.3 §14) — walked, not assumed

| Check | Result | Evidence |
|---|---|---|
| **No media ref / playable URL in any viewer read for an unentitled caller** | ✅ | Explicit allow-list selects in `catalog/reads.ts` (a new column cannot leak by default) + a sweep test walking EVERY viewer surface (home, lists, show page, hubs, tags, search, trending, top-10, offer, captions, next-episode-unentitled) against the fixture bunny GUIDs, field names, storage keys and the CDN host — sessionless AND session-holding |
| **Unpublished + scheduled content absent, never 403'd** | ✅ | Draft series: 404 show page, unsearchable, silently dropped from merchandised shelves; future-scheduled episode: absent from grids/next/counts, unmintable (playback + renew 404), unbuyable (unlock 404), unshareable (moment mint 404) — all by test |
| **Tenant + caller scoping on every new query** | ✅ | Every new table carries `tenant_id`; every read/write threads `TenantContext`; cross-tenant follow/progress/share-resolve return nothing (tested); cross-USER engagement reads return nothing (tested); admin shelf items cannot plant another tenant's series (tested) |
| **Share tokens non-enumerable + resolution rate-limited** | ✅ | 128-bit `randomBytes` base64url; uniform 404 for miss/malformed (no oracle); dedicated resolve limiter proven engaging under burst (429); guessing test in suite |
| **Every new list/read rate-limited** | ✅ | Four new buckets (catalog 300 · engagement 300 · share-mint 30 · share-resolve 60), Redis-backed (replica-safe), limiter engagement tested for catalog + resolve |
| **`mayPlay` remains the only gate; offer/prefetch/re-mint consume it** | ✅ | Next-episode calls `mayPlay` then the UNCHANGED seam; unentitled prefetch never reaches the mint (no URL in response — tested); re-mint is the same `getPlaybackUrl` with full re-check (lapsed sub → 403 mid-session — tested); summary uses `mayPlay` as arbiter (expired-entitlement divergence test); lock-state mirror pinned to `mayPlay` by a full-matrix agreement test |
| **Ledger isolation of engagement writes** | ✅ | No FK into money tables; no engagement transaction touches a wallet row; no user-row lock taken (progress storms cannot queue money); re-mint proven to move not one coin (ledger row-count unchanged — tested) |
| **Session handling on the new optional-auth reads** | ✅ | Absent header → public shape; PRESENT-but-invalid header → 401, never a silent downgrade (tested) |
| **Admin surfaces (D-014)** | ✅ | Every admin route behind `requireMediaAdmin` (constant-time compare, uniform 401); viewer session explicitly NOT accepted (tested); operator catalog view carries media STATE but still no `bunny_ref` (tested) |
| **Fanout hook** | ✅ | Log line carries ids + counts only — no PII, no secrets; claim-once via atomic `updateMany` (double-sweep tested) |
| **New dependencies** | ✅ | None added. `npm audit --omit=dev`: 0 vulnerabilities |

## Informational — noted, not defects

- **N-007 · `link_resolutions` grows one row per resolve** (bounded by the resolve limiter).
  Fine at Stage 0; a retention/rollup policy belongs to the v0.8 events-sink design.
- **N-008 · Draft-series playback.** `mayPlay` does not consult `publish_state` (v0.1 as-built,
  engine frozen this Order): a FREE episode of a DRAFT series would mint for a caller who
  somehow holds its id. No viewer surface emits draft ids (proven by the absence tests), and
  the v0.3 schedule guard covers the *scheduled* case at the mint. Fold a publish-state check
  into the engine at its next ordered touch (v0.5 shape), not as an unordered edit now.

---

# 2026-07-27 · v0.2 — the media seam (dedicated pass, CONTEXT v0.2 §6)

**Build:** v0.2, media · **Branch:** `binjreel-dev` · **Gate:** GATE 2, run against the built
code and the 86-test suite on real engines. **The live half of two checks (edge-expiry, bucket
privacy) is structurally GATE-3** — no Bunny key or bucket exists yet — and is listed under
"deferred to GATE-3", not silently passed.

## Summary — v0.2 pass

| | |
|---|---|
| **Found** | **1** |
| **Fixed (closed-loop, re-verified)** | **1** |
| **Needing you** | **0** |

**By severity:** Critical **0** · High **0** · Medium **0** · Low **1 (fixed)** ·
Informational **2 new noted** (N-005, N-006) · **1 v0.1 note resolved** (N-003)

**Open high/critical: none.** CONTEXT v0.2 DoD "no high/critical open" — met.

## Closed-loop — v0.2

### F-003 · LOW · The webhook secret would have transited request logs
**Found by:** adversarially walking the new secret's life-cycle — where does this value ever
exist? Answer: `.env`, the Bunny webhook config, and **every callback's request URL**.

**What was wrong.** Bunny Stream webhooks can carry no custom header, so the callback
authenticates by a query token (D-015 — the strongest form the vendor offers). Fastify's default
request logging records `req.url`, and the error handler's `access_refused` audit event falls
back to `request.url` for unmatched routes — so in production every legitimate callback (and
every attacker probe with a guessed token) would have written a URL-borne secret candidate into
the box-local container logs.

**Contrary test before logging it.** *If this were not real, the prod config would disable
request logging or the URL would be redacted somewhere.* Neither: `disableRequestLogging` is
test-only, and no redaction existed. Confirmed real; bounded (logs are box-local, rotated 10MB×3,
the secret gates only media-state transitions — never money).

**Fix.** `redactSecretParams()` — every URL that reaches a log line passes through it: the
request-log serializer and the audit-event fallback path. The token VALUE is replaced with
`[REDACTED]`; nothing else in the URL is touched.

**Re-verified.** Unit-tested (token-only redaction, pass-through for clean URLs); suite 86/86.

## The ordered checklist (CONTEXT v0.2 §6) — run, not assumed

| Check | Result | Evidence |
|---|---|---|
| **No path to a working URL that bypasses `mayPlay`** | ✅ | Exactly ONE mint path: `signPlaybackUrl` is called only from the seam, behind `mayPlay` (grep-audited + the seam throws on refusal so omission cannot ship). The admin status route returns `bunny_ref` but no signed URL — and a `bunny_ref` alone cannot become a working URL without the token key, which exists only in the app container's env. Tests: unentitled → 403 with no URL/ref/host; entitled-unready → 409 leaking neither ref nor reason nor state name |
| **TTL is real — an expired link is dead at the edge** | ✅ algorithm / ⏳ edge | The token binds `sha256(key + path + expires)`, so the expiry is INSIDE the signature — proven by independent recomputation in test, plus determinism and config-driven seconds-scale TTL. The edge's actual refusal of an expired link is Bunny's evaluation of the same algorithm and is **GATE-3** (no key exists to prove it against) |
| **Signing key + storage creds only in the box store** | ✅ | Env-only slots, all empty today; no value in any tracked file (`git grep` secret-pattern sweep clean); `.env` gitignored/0600 and never in history (v0.1 check unchanged); the docs bridge reaches `/docs` only (unchanged topology — re-affirmed by zero compose/Dockerfile diff); nothing mounts near Traefik's `acme.json` |
| **Key never in a URL, response, or log** | ✅ | Signer test: key not in minted URL; storage test: secret not in presigned URL; F-003 closed the log path for the webhook token; error bodies carry no slot names (tested: `media_not_configured` body names no env key) |
| **Mint endpoint rate-limited** | ✅ | Redis-backed, per-route cap, real 429s driven in test. Registration (30/min) and callback (300/min) capped too |
| **Masters not publicly reachable** | ✅ client / ⏳ bucket | The client offers NO unsigned-URL method; presign TTL clamps ≤1h in code even under hostile config; the contract fixture REFUSES unsigned requests so the SigV4 is load-bearing in test; storage keys are allowlist-validated (traversal shapes rejected before any network call). The bucket's own privacy policy is Director-placed at GATE-3 and proven live then (SECRETS.md carries the note at the slot) |

## Adversarial items examined and their outcomes

- **Can a forged callback corrupt state?** No token → uniform 401, state untouched (tested).
  Valid token + wrong library → 400 (tested). Valid token + guessed GUID → matches nothing
  (vendor-issued GUIDs; tested no-op) or names exactly the row Bunny issued it for. Residual: a
  holder of the secret can flip media states — bounded, never money — N-005.
- **Can registration reach anything but our bucket/vendor?** Storage endpoint and API base are
  Director-placed config, never request input; the only request-supplied value (`storageKey`)
  is allowlist-validated and rejected before any network call when traversal-shaped (tested).
- **Does a viewer session open the admin surface?** No — 401 (tested). Does an empty admin slot
  fail open? No — uniform 401 (tested).
- **Does one user's URL differ from another's?** Only by time — nothing user-shaped in a
  shareable URL, so a leaked link identifies nobody and a revoked user's already-minted link
  still dies at TTL (tested for the first property; the second is the TTL check).
- **Vendor detail leakage:** a Bunny 500's body stays off the wire (tested); the episode row
  records `bunny_ingest_failed` + status only.

## Noted — v0.2

### N-005 · The webhook secret is URL-borne — accepted vendor constraint (D-015)
Redaction (F-003) closes the log path; the residual is the secret's transit in the URL itself
(TLS-covered end-to-end; Traefik does not log query strings in its default access-log-off
config). Blast radius if held: media-state flips only. Rotation free. Revisit if Bunny ships
signed webhooks.

### N-006 · The callback's GUID lookup is deliberately cross-tenant
The webhook carries no tenant. `bunny_ref` is vendor-issued per video, unique by construction,
and indexed; the row it matches IS the row Bunny is reporting on. A stale GUID after
re-registration matches nothing (tested). Recorded so the un-scoped `findFirst` reads as design,
not oversight.

### N-003 (v0.1) · RESOLVED — the stub URL's tenant-UUID echo is gone
The v0.2 signer emits `host/{guid}/playlist.m3u8?token&expires` — no tenant id, no user id, no
episode id beyond the vendor GUID. The v0.1 note existed precisely so this signer would be
written knowing about it; it was.

### N-002 (v0.1) · SUPERSEDED — playback URLs are no longer placeholders
Real signing landed; the fail-closed posture replaced the stub (a stub would now be a lie —
nothing is minted without the key).

## Deferred to GATE-3, explicitly (the Director's key placement unlocks these)

1. ~~A real master transcodes and **plays** through a real signed URL (DoD).~~ ✅ **PROVEN LIVE
   2026-07-27** after the DL-B006 signing fix (single-file → directory token): manifest 200,
   sub-playlist AND media segment 200 under the same token. Evidence: BUILD-LEDGER L-028.
2. ~~**An expired link dies at the edge** — fetch after TTL → CDN refusal (DoD).~~ ✅ **PROVEN
   LIVE 2026-07-27**: expired mint → 403 at the edge (L-028).
3. ~~The master bucket refuses an unauthenticated GET (bucket-policy half of "not publicly
   reachable").~~ ✅ **PROVEN LIVE 2026-07-27 (Director):** unauthenticated S3 GET returns
   non-200 (400 — no object served); public access disabled at the bucket; and the signed path
   demonstrably works — the presigned fetch is how the master reached Bunny and transcoded.
   Both halves of "masters not publicly reachable" (client + bucket policy) now hold.
4. ~~Bunny's live response shapes match the contract fixtures.~~ ✅ Held live: ingest returned a
   GUID the callback later named; video reached status 4 with a full ladder (L-028).

**ALL FOUR GATE-3 ITEMS PROVEN — v0.2 GATE-3 COMPLETE 2026-07-27** (live VLC playback
confirmed by the Director; L-029).

**RESOLVED — the empty-`Referer` 403 (DL-B006 discovery).** Cause found zone-side: Bunny's
**"Block direct URL file access" was ON**. It blocked every no-referer client (curl and VLC
today; native mobile players in v0.3) while providing effectively no protection — any non-empty
referer passed, even a foreign one. **Now OFF (Director).** CDN token authentication remains
the real gate, which is the correct posture: the token is cryptographic and expiring; the
referer check was neither. This clears the v0.3 player blocker flagged in the prior GATE-3
report.

**Posture change to a claim in this report — playback TTL (DL-B007).** The v0.2 pass described
"seconds of TTL". Live viewing proved 30s shorter than a viewing session (the directory token
expires as a whole while the player is still fetching segments), and the Director set
`PLAYBACK_URL_TTL_SECONDS=3600` operationally. The anti-sharing property is therefore now
"a leaked URL dies within the hour", not "within seconds" — an accepted interim posture; the
final TTL is a named Director product decision (anti-sharing vs viewing session), and the
30s CODE default in `config.ts` is an open fix for the next code touch so a fresh deploy
cannot silently regress to mid-episode death.

*(Historical note: the v0.2 checklist row "TTL is real" below described the token as
`sha256(key+path+expires)` over the manifest path — the construction DL-B006 corrected. The
property claimed there — expiry inside the signed digest — held through the fix and is now
edge-proven; the algorithm as-built is D-016's.)*

---

# 2026-07-25 · v0.1 — the money-and-access spine (FULL tier)

**Build:** v0.1 · **Branch:** `binjreel-dev`
**App class:** consumer content SaaS, multi-tenant-ready · **Audit tier: FULL**
(Base + Auth & Access + Tenant Isolation — `_library/div1-cco/1C-inspection/skills/SKILL.md`)
**Gate:** GATE 2, run against the built and deployed stack (CONTEXT §"Audit tier", TDD §10).

---

## Summary

| | |
|---|---|
| **Found** | **2** |
| **Fixed (closed-loop, re-verified)** | **2** |
| **Needing you** | **0** |

**By severity:** Critical **0** · High **0** · Medium **2** · Low **0** · Informational **2**

**Open high/critical: none.** CONTEXT's definition of done requires "no high/critical open" — met.

*Two items are carried below as **noted, not findings**: one is a Director act CONTEXT
deliberately scoped out of this build, one is an accepted v0.1 posture with its v0.2 successor
already named. Neither is a departure being quietly absorbed; both are stated so they cannot
arrive as a surprise.*

---

## Closed-loop — handled, re-verified, logged

### F-001 · MEDIUM · Receipt endpoints disclosed configuration state to unauthenticated callers
**Found by:** live probe of `POST /v1/receipts/apple` during end-to-end verification.

**What was wrong.** The endpoint answered with `{"message":"Apple receipt verification is not
configured"}`. Distinct messages were also returned for a bad signature (`"Apple signature
verification failed"`) and a malformed body (`"Malformed Apple payload"`).

**Why it matters.** These two endpoints *must* be unauthenticated — the stores call them and
carry no session — so anyone on the internet can probe them freely. The differing messages are an
oracle: they reveal which payment rails are live, which are still dark, and how far a crafted
payload got through validation. That is a map of the money path's readiness, handed out for free.

**Contrary test before logging it (1C: verify the diagnosis).** *If this were not really
reachable, what would I expect?* — that the route required auth, or sat behind an allowlist.
Neither is true: the probe was sent with no credential from outside the container, through
Traefik, and got the disclosing message back. Confirmed exploitable as an information leak, not a
pattern that merely looks like one.

**Fix.** `DomainError` gained a `publicMessage` distinct from `message`. `ReceiptVerificationError`
now carries the specific reason internally and returns a uniform `"Receipt rejected"`. The real
reason is logged server-side, where it is genuinely useful.

**Re-verified.** Live through Traefik: two different failure causes now return byte-identical
bodies. Plus a regression test asserting three distinct causes (unparseable JWS, malformed body,
bad Google envelope) produce **one** distinct response body — the assertion is on the *set size*,
because the differences between messages are the vulnerability, not any single message.

---

### F-002 · MEDIUM · Authorization failures were not logged for audit
**Found by:** walking the Tenant Isolation checklist item *"security-relevant events (auth,
authorization failures, admin actions, tenant-boundary denials) are logged for audit."*

**What was wrong.** Refusals were *behaviourally* correct — 401 on a missing session, 403 on an
unentitled play, 404 on a cross-tenant episode — and left **no record**. The application logged
requests, but nothing marked a refusal as a security event.

**Why it matters.** This is the failure mode that looks like success. Every control worked; the
system simply could not tell you afterwards that anyone had tried. An enumeration campaign against
episode ids — the classic multi-tenant probe — would be invisible in retrospect, and the first
sign of trouble would be its consequences.

**Fix.** Every `401` / `402` / `403` now emits a structured event: `event: "access_refused"`, with
code, internal reason, method, matched route pattern, and client IP (Traefik is trusted as the
single ingress, so the IP is the real one, not the proxy's).

**Re-verified.** Live: an unentitled playback request and an unauthenticated balance request were
issued through Traefik, and both `access_refused` entries were confirmed present in the container
log with the correct codes and routes.

---

## Checklist results

Every line was **run**, not assumed. A check not actually performed is not a pass (1C).

### Base — all app classes

| Check | Result | Evidence |
|---|---|---|
| No secret hardcoded in source or any committed file | ✅ | `git grep` for PAT/key/PEM/AWS patterns across tracked files → none |
| `.env` git-ignored and **never committed** | ✅ | Checked full history (`git log --all --diff-filter=A`), not just the working tree → no `.env` ever added |
| Secrets read from environment at runtime, never baked into image | ✅ | `loadConfig()` reads `process.env` only; `.dockerignore` excludes `.env`; image carries no values |
| Each key least-privilege, rotation possible without rebuild | ✅ | Postgres credential scoped to one DB on an internal network; store keys are env slots swappable by restart |
| No secret in logs, error output, or a URL | ✅ | Log lines carry codes/reasons, never values; DB URL redacted in every tool invocation |
| All external input validated against an expected shape | ✅ | `zod` at every boundary: sign-in, merge, unlock, both receipt payloads, UUID params |
| Parameterised queries / ORM — no string-built SQL | ✅ | Prisma throughout; `grep` for `$queryRawUnsafe`/`$executeRawUnsafe` in `src/` → **none**; both raw sites are parameterised tagged templates |
| Output encoded for context | ✅ | JSON-only API, no HTML rendering surface in v0.1 |
| Input never reaches a shell, file path, or template | ✅ | No `child_process`, no `fs` path construction from input |
| Lockfile present and committed | ✅ | `package-lock.json` (45.9 KB) present and tracked |
| Known-vulnerability scan run and triaged | ✅ | `npm audit --omit=dev` → **0 vulnerabilities** |
| No abandoned/unvetted packages on the critical path | ✅ | 6 production deps, all first-tier: `fastify`, `@fastify/rate-limit`, `@prisma/client`, `ioredis`, `jose`, `zod` |
| HTTPS enforced end to end | ✅ | Traefik `websecure` + `letsencrypt`; HTTP→HTTPS **301 confirmed live** |
| No debug endpoint, admin route, or directory listing reachable | ✅ | `GET /admin` → 404; no wildcard route; unrouted host → 404 (no catch-all) |
| Errors generic to the user; detail logged server-side | ✅ | Test asserts no stack frames, internal paths or library names in a body; F-001 closed the one exception |
| Every MCP bridge scoped to its project `/docs` only | ✅ | Re-tested after the topology change: bridge mount is `/docker/binjreel/docs → /docs` and nothing else; inside the container `/docker`, `/app/src`, `/docker/traefik` all **do not exist** |
| Canon access from chat read-only; writes via the gate | ✅ | Unchanged by this build. *(Note as written 2026-07-25: canon read was down for an unrelated credential defect — DL-B005 / STATE W-001. RESOLVED 2026-07-26, verified live; see BUILD-LEDGER L-022.)* |

### + Auth & Access

| Check | Result | Evidence |
|---|---|---|
| Authentication real; tokens issued, validated, expired properly | ✅ | 32-byte CSPRNG tokens, Redis-backed with TTL; forged token → 401 (tested) |
| Session store holds no usable credential | ✅ | Redis keys are the **SHA-256 digest** of the token, never the token; asserted by test |
| Authorization checked **server-side on every protected action** | ✅ | `requireActor()` on every protected route; `mayPlay` is the single gate on playback and cannot be bypassed client-side because there is no client |
| CSRF protection on state-changing requests | ✅ | Bearer-token API with no cookie auth — the CSRF precondition (ambient credentials) does not exist. Noted as N-002 below so the posture is deliberate, not accidental |
| Rate limiting on auth and abusable endpoints | ✅ | `@fastify/rate-limit` on anonymous-signup, sign-in, merge, unlock, both receipt routes. **Redis-backed**, so the cap is shared across replicas rather than multiplied by them. Four tests drive real 429s |
| Baseline security headers present | ✅ | HSTS (1y, includeSubdomains, preload), `X-Content-Type-Options: nosniff`, `frameDeny`, `referrerPolicy` — applied at the Traefik edge so they cover responses the app never generates |
| Sign-in failures reveal nothing about provider configuration | ✅ | Every provider failure mode collapses to one identical 401 (fixed during build — see BUILD-LEDGER L-018) |

### + Tenant Isolation — the unrecoverable tier

| Check | Result | Evidence |
|---|---|---|
| Authorization is tenant-aware on every read and write | ✅ | Every query threads a `TenantContext`; `tenant_id` is **never** taken from client input — resolved server-side from config (D-003/D-010), so a client cannot address another tenant by asking to |
| Every table carries `tenant_id` | ✅ | Asserted against `information_schema` at test time, not against the model file — the claim is about the database, so the database is what gets asked |
| No insecure direct object reference (IDOR) | ✅ | An episode id from another tenant is **not found**, not "forbidden" — identical to a nonexistent id, so no probe distinguishes them. Tested for playback (403, no URL) and unlock (404, **and no debit written**) |
| Cross-tenant wallet read returns nothing | ✅ | The same user id scoped to the other tenant sums to **0** (tested) |
| Cross-tenant payment attribution refused | ✅ | A verified store notification naming an account in another tenant is recorded and applied to **nothing** (tested) |
| Shared infrastructure partitioned | ✅ | Redis keys namespaced by tenant (`binjreel:sess:<tenantId>:…`); Postgres rows scoped by `tenant_id`; rate-limit keys namespaced |
| Security-relevant events logged for audit | ✅ | **F-002, fixed and re-verified** |
| Backups encrypted and access-controlled | ⚠️ | **No backup mechanism exists yet** — see N-001. Not a v0.1 build defect; CONTEXT scoped it as a Director item |

### Container & network posture (audit tier high)

| Check | Result | Evidence |
|---|---|---|
| App runs as non-root | ✅ | `uid=1000(node)`, verified inside the running container |
| Read-only root filesystem | ✅ | `ReadonlyRootfs=true`, `/tmp` tmpfs only |
| All capabilities dropped, no privilege escalation | ✅ | `CapDrop=[ALL]`, `no-new-privileges:true`, `Privileged=false` |
| App container has no host filesystem access | ✅ | **Zero mounts.** Cannot read `/docker`, cannot read `/docs`, does not hold `BRIDGE_TOKEN` |
| Data layer unreachable from outside | ✅ | Postgres and Redis publish **no host port** and sit only on `binjreel-internal`, confirmed `internal=true` |
| Data layer unreachable from the document bridge | ✅ | Connection from `binjreel-mcp-bridge` to `postgres:5432` **did not establish** (timed out) — the bridge is not on that network |
| Log rotation bounded | ✅ | `json-file`, 10 MB × 3 on every service |

---

## Noted — stated deliberately, not findings

### N-001 · Continuous ledger backup does not exist *(Director item, correctly out of this build)*
The wallet ledger is the one dataset that cannot be reconstructed if lost (TDD §10, IDEAL-SCENE:
*"a lost server never means a lost ledger"*). No backup mechanism is in place.

This is **not a v0.1 defect**: CONTEXT §8 scopes it as *"continuous backup posture for the ledger
**noted** for the Director"*, and STATE already carries it as backlog #3, gating v0.2 rather than
v0.1. It is recorded here because a security report that stayed silent about it would be reporting
a clean sheet on a system with no disaster recovery.

**Nothing is at risk today** — v0.1 holds no real money and no real viewer data; the tables carry
fixtures. It becomes load-bearing the moment a verified store notification credits a real coin,
which is v0.2. Not escalated as a CSW packet because it is not a decision awaiting the Director's
judgment: it is already an accepted, scheduled item on the Director's list.

### N-002 · Signed playback URLs are placeholders in v0.1
`getPlaybackUrl` returns a labelled `stub://` string (`stub: true` in the payload). The
**enforcement** is real and tested — an unentitled request yields no URL at all — but the URL
carries no cryptographic protection because there is nothing behind it to protect yet. Real
short-TTL Bunny signing is v0.2 (TDD §5). Stated so nobody reads "playback works" as "playback is
secured".

### N-003 · Informational — the stub URL echoes the tenant UUID
The placeholder query string includes `tenant=<uuid>`. Not a finding: tenancy is resolved
server-side and never accepted from the client, so knowing the id grants no leverage. Flagged only
so the v0.2 signer is written knowing it is there, rather than inheriting it unnoticed.

### N-004 · Informational — the two write-paths to `/docs` still differ by uid
Unchanged from DL-B004/D-008 and not touched by this build. The bridge's atomic-write path holds.

---

## Seam note (SaaS class — who owns what)

1C owns *can a user reach another tenant's data*; **5A owns *is the data layer built to keep them
apart*** (Blueprint §12). Naming the line so nothing falls in the crack and nothing is audited
twice:

- **Covered by this report (1C, the access lens):** tenant-aware authorization on every read and
  write; IDOR resistance; cross-tenant read, spend and payment-attribution refusals; partitioning
  of Redis and rate-limit namespaces; audit logging of boundary denials; secret custody; the wall.
- **Owned by the 5A scalability & tenancy audit, NOT run here:** whether the *data layer
  architecture* scales and keeps tenants apart structurally — index coverage and query plans under
  tenant-scoped load, connection-pool sizing, the read-replica and managed-DB path (TDD §7 Stages
  2–4), and backup/DR **architecture** (1C confirmed only that no backup store exists to be
  insecure — N-001).

**5A has not run against this build.** CONTEXT names the 1C audit at GATE 2 and does not order a
5A pass for v0.1; recorded here so its absence is a known gap rather than an assumed pass.

---

## The honest limit

Self-review is not independent review: the same Worker that wrote this code ran this audit. That
is structurally true regardless of how carefully the checks were run, and it does not change once
this product holds real money — which it will in v0.2. The value of this report is that it makes
an eventual human pass **short and cheap**: a clean, fully-logged trail with every check named and
its evidence recorded. It is not a substitute for that pass.

*(Not legal or security-professional advice; decision-useful structure.)*
