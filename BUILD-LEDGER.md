# BUILD-LEDGER.md — binjreel

*Append-only, never overwritten (Blueprint §10). The as-built redline against the as-designed
drawing: what was actually added, what was decided in code, what diverged and why. Written **as
the build happens**, not after — a ledger that lags the code has already broken the document
layer (4B Build skill). Only the Worker sees app code, so this is the only place the truth about
what was built can be recorded.*

---

# Phase 1 — standup (V2S new-project flow)

**Order:** `/docker/binjreel/docs/CONTEXT.md` (IODE, orchestrated provisioning v0.6.5).
**Date:** 2026-07-25 · **Hat:** 4B Build · **App class:** Multi-tenant SaaS, audit tier high.
**Budget:** none stated in the order — see DEFECT-LEDGER DL-B002.

## L-000 · Step zero — canon read before scaffolding
`git -C /docker/v2s-canon pull` → *Already up to date* (fetched `procedure/standup-refinements`
as a new remote branch; `main` at `ab07022`). Read as authoritative, live, not from a frozen
copy: `procedures/project-binding.md`, `procedures/new-box-standup.md`,
`procedures/secret-manifest.md`, `procedures/chat-operating-instructions.md`,
`procedures/defect-register.md`, Blueprint §13 (the wall) + §14 (folder structure),
Implementation Guide Phase 1 + the Customization Pass, `_library/div4-production/4B-build`,
`_library/div1-cco/1C-inspection`.

**Canon carried three requirements the order does not name.** Canon wins, so all three were
built:
1. **Origin remote wired at standup** (Impl. Guide 1.3, DL-001) — not deferred to first deploy.
2. **`docs/SECRETS.md`** secret manifest (`procedures/secret-manifest.md`) — a project is not
   "fully stood up" without it.
3. **`docs/DEFECT-LEDGER.md`** (`procedures/defect-register.md`, per-project tier) — every kept
   defect is recorded there.

**Box-standup gate checked before beginning project standup** (`new-box-standup.md`: "Only when
all five read true does Phase 1.3 begin"). Result in DEFECT-LEDGER DL-B003 — canon clone present
and pulling, parent pointer present, canon-write absent; two items are not verifiable from this
box and are surfaced rather than assumed.

## L-001 · Step 1 — registry first
Created `/docker/PROJECTS.md` (the file did not exist on this box; path per Blueprint §14) with
the binjreel entry: app + docs paths, all `/docs` record paths, MCP bridge, GitHub repo, active
branch `binjreel-dev`, deploy branch, production URL, stack, status, next action.
Per the enrollment-at-standup clause in `project-binding.md` the entry also carries the DL-010
items the order does not enumerate: **identity-check phrases** (the words a correct binjreel
STATE contains, and the words that mean WRONG PROJECT), the **write path + dispatch blockers**,
and the **starting baton**. Registry written before any other scaffold act, as ordered.

## L-002 · Step 2 — folder structure
`/docker/binjreel/app/` (real repo root) + existing sibling `/docker/binjreel/docs/` +
`/docker/binjreel/mcp-bridge/`. Matches the fleet as-built layout in Blueprint §14. Docs sit
**outside** the repo on purpose — that is what makes the §13 wall structural (DECISIONS D-001).

## L-003 · Step 3 — the committed V2S binding
Read canon's **current** template live from `procedures/project-binding.md` ("Template — project
root `CLAUDE.md`") and placed it, project name substituted, at the real repo root
`app/CLAUDE.md` — the load-bearing copy, committed. Confirmed the repo root is in fact `app/`
via `git rev-parse --show-toplevel` before treating it as such.

Brought the bare breadcrumb `/docker/binjreel/CLAUDE.md` in step with it (reconciled, not
overwritten): preserved its working content and provenance comment, aligned the body to canon's
live template, and resolved the one real divergence — IODE's copy says *Director* where canon's
template says *Owner*. Named as the same seat rather than silently rewriting canon's words
(DECISIONS D-007). Added to both copies: app class + audit tier, the two branch roles, and the
`PROJECTS.md`/full-path addressing pointer (DL-010).

Both binding seams are now closed: the **Worker's walk** (root `CLAUDE.md`) and the
**Architect's read path** (the canon-first STEP-ZERO banner at the top of `STATE.md`, L-005).

## L-004 · Step 4 — git, first commit, dev branch, origin remote
`git init -b main`; repo-local `user.name`/`user.email` set (this box had no global git identity
— a global default was **not** written, since that would silently sign other projects' commits).
First commit `1948227` covering the binding, the scaffold, and the wiring. Then created and
checked out **`binjreel-dev`**; `main` remains at the same commit as the deploy branch.

**Verified `.env` was excluded from the staged tree before committing**, rather than trusting
`.gitignore` to be right.

Wired `origin` → `https://github.com/papamg/binjreel.git`. `git remote -v` confirms. **Not
pushed:** the remote repo does not exist and no credential resolves for it — read-only check
`git ls-remote` fails with *could not read Username*. Two Director acts (create repo, place PAT).
Branch protection cannot be set until the repo exists; carried forward in STATE.

## L-005 · Step 5 — docs layer seeded
- **`DECISIONS.md`** — D-001…D-007, each with the reasoning that settled it.
- **`SECRETS.md`** — manifest per `secret-manifest.md`. `github-pat` declared as a **consumed
  box-level shared slot, not placed per-project** (D-005); `anthropic-api-key` as `env`, empty
  and correct at standup. Remaining multi-tenant class slots deliberately **not invented** — see
  the file's reasoning; inventing slots reproduces the DL-001 failure wearing a schema.
- **`DEFECT-LEDGER.md`** — created; DL-B001…DL-B003 logged.
- **`BUILD-LEDGER.md`** — this file.
- **`STATE.md`** — with the canon-first STEP-ZERO banner at the top, which is the
  Architect-side half of project binding (`project-binding.md`, "Binding the Architect's read
  path").

**`IDEAL-SCENE.md` and `TDD.md` were NOT created.** The order says "if the Architect has staged
them"; they are not staged, and `/docs` contained only `CONTEXT.md`. Writing them would be the
Worker authoring 4A's product from inside 4B — the Worker has no design authority and inventing
an ideal scene would give Qualification a measuring stick nobody agreed to. Registered as the
Architect's next action in `STATE.md` and `PROJECTS.md`.

## L-006 · Step 6 — MCP bridge, scoped to `/docs` only
Built `/docker/binjreel/mcp-bridge/` as a first-party Node 22 + TypeScript MCP server
(`@modelcontextprotocol/sdk` 1.29.0, zod 4, express 5), containerised, on `traefik-public`
behind `Host(docs.binjreel.com)` with the `letsencrypt` certresolver.

**Transport: Streamable HTTP, stateless** — a fresh MCP server + transport per POST, torn down
on every exit path (`res.on('close')`), so no session state can leak between callers and a
failed request cannot poison the next. `GET`/`DELETE /mcp` → 405 (no stream to resume, no
session to end).

**Tools (5):** `list_docs`, `read_doc`, `write_doc`, `append_doc`, `search_docs`.
**Deliberately no delete/move tool** — the docs layer is the project's record and the ledgers
are append-only (Blueprint §10); destroying history should not be one prompt away. `append_doc`
exists precisely so the append-only records have a tool that cannot overwrite them.

**The wall is the MOUNT, not application logic** (D-004). The container receives exactly one
bind: `/docker/binjreel/docs → /docs:rw`. Verified from inside the container that `/docker`,
`/docker/binjreel/app` and `/docker/traefik/acme.json` **do not exist** there — so no bug and no
prompt can reach app source, another project, or the SSL private keys. `docs-root.ts` is the
second layer, stopping traversal and symlink escape *within* the mount.

**Auth: bearer token, FAIL-CLOSED.** A read-write bridge to the record layer, reachable from the
public internet once DNS exists, must not be open. `/mcp` requires
`Authorization: Bearer <BRIDGE_TOKEN>`, compared over SHA-256 digests so the comparison is
constant-time regardless of length. **With `BRIDGE_TOKEN` unset the bridge answers every `/mcp`
request 503** rather than serving open. Token generated on the box at mode 600, outside `/docs`
and outside git; slot declared in `SECRETS.md` with the reasoning for Worker placement. `/health`
is unauthenticated by necessity and returns only `{"status":"ok"}` — no project, path or version
detail (1C: a health endpoint is an information-disclosure surface). Failed auth is logged,
never the presented credential. `/mcp` rate-limited to 240 req/min.

**Runtime hardening:** uid 1000 (non-root; the host docs dir was chowned to 1000:1000 so the
bridge writes its one mount without ever running as root), read-only rootfs + tmpfs `/tmp`,
`cap_drop: ALL`, `no-new-privileges`, no published host port, log rotation.

**Defect found and fixed mid-build (closed-loop, not escalated).** The first build leaked the
in-container path in filesystem errors — `read_doc nope.md` returned
`ENOENT ... stat '/docs/nope.md'`, handing the caller the container layout for free. Fixed with a
`guard()` wrapper mapping fs errors to caller-safe relative-path messages (`not found: nope.md`),
full detail retained in the server log only; rebuilt and re-verified.

## L-007 · Step 7 — Docker / Traefik per app class
`app/Dockerfile` (multi-stage: the TypeScript toolchain and dev deps never reach the runtime
image) + `app/docker-compose.yml` on `traefik-public` with the `letsencrypt` certresolver.

Built to the **observed fleet as-built pattern** because canon's class bundle is empty
(DEFECT-LEDGER DL-B001): read `/docker/traefik/traefik.yml` + its compose file and matched them —
external `traefik-public`, `websecure` entrypoint, `letsencrypt` resolver, `exposedByDefault:
false` so every route is an explicit opt-in label.

Audit-tier-high hardening on both services: **no published host port** (reachable only across
`traefik-public`, so neither service has an ingress of its own), non-root uid 1000, read-only
rootfs + tmpfs, `cap_drop: ALL`, `no-new-privileges`, HSTS + `nosniff` + `frameDeny` +
referrer-policy applied **at the Traefik edge** (so the guarantee also covers responses the app
does not generate — Traefik error pages and redirects), `x-powered-by` disabled, log rotation.
`helmet` in-app as the second layer.

**Tenant subdomains deliberately not routed** — the router matches apex + `www` only. The
`letsencrypt` resolver uses an HTTP-01 challenge, which cannot issue a wildcard certificate, so
routing `*.binjreel.com` would attach a router to a cert that can never exist. Recorded as
DECISIONS D-003 with the DNS-01 consequence for the TDD to decide knowingly.

`app/src/server.ts` is a standup scaffold, not product: `/health`, a placeholder root, a 404
catch-all, and graceful SIGTERM/SIGINT shutdown with the force-exit timer `unref`'d so it can
never itself hold the process open (4B async-cleanup guard).

## L-008 · Step 8 — health confirmed, wall verified, live state reported
Both services **healthy** (Docker healthcheck via the Node runtime already in the image — no
curl/wget added just to probe).

Verified end to end rather than assumed — a check not actually performed is not a pass (1C).
26 checks (22 here, 4 more after the L-009 fix); full table in `BUILD-REPORT.md`. Highlights:
- Bridge + app both reachable **through Traefik** by Host header (`-k`, since the public cert
  needs DNS that does not exist yet); HTTP→HTTPS 301 confirmed on the `web` entrypoint.
- MCP `initialize` and `tools/list` succeed with the token; **401 with no header and with a wrong
  token**.
- Traversal refused: `../app/CLAUDE.md`, `../../traefik/acme.json`, `../../../etc/passwd`,
  absolute `/etc/passwd`.
- **Symlink escape refused for reads AND writes.** First symlink attempt (→ `/docker/binjreel/app`)
  failed with ENOENT because the target does not exist *inside* the container — the mount caught
  it, meaning the second layer had not actually been exercised. Re-tested with targets that DO
  exist in the container (`/etc/passwd`, `/srv`): all refused by the containment check, and
  `/etc/passwd` confirmed unmodified after the write attempt. *Noted because the first result
  looked like a pass and was not evidence of one.*
- `write_doc evil.sh` refused (extension allowlist); read-write path confirmed by writing through
  the bridge and reading the file on the host; scratch file removed.
- App container has **no mounts at all**, cannot see `/docs`, `/docker` or the SSL keys, and does
  **not** hold `BRIDGE_TOKEN`.
- Unrouted host (`nope.binjreel.com`) → 404: no accidental catch-all.

`STATE.md` written with live state; `BUILD-REPORT.md` written as a closed-loop record.

## L-009 · Post-verification fix — atomic writes (DL-B004)
A probe run *before* hand-off found that the bridge (uid 1000) could not write any file the Worker
had created as root — including `STATE.md`, the one file canon requires the Architect to rewrite
at **every** session close. Reproduced inside the container (`Permission denied`), then fixed at
the write path rather than with a chown ritual: both write tools now use temp-file + `rename()`
(`docs-root.ts:atomicWrite`), which needs directory permission, not file permission. `append_doc`
became read-modify-atomic-write. Writes are now atomic as a side effect — a crash cannot truncate
a record — and in-flight temp files are hidden from `list_docs`/`search_docs` and removed on every
failure path. Rebuilt; re-verified cross-ownership write **and** append; re-ran the containment
probes to confirm the fix did not widen the wall (both still refused; `/etc/passwd` unmodified).
Full traceback: DEFECT-LEDGER DL-B004 · rationale: DECISIONS D-008.

## L-010 · Document placement — Architect's v0.1 staging docs moved from the cubby
No code touched; `app/`, canon, and everything outside `docs/` untouched. Placement only, on
Director instruction, of the three v0.1 docs the Architect staged in `/docker/binjreel/cubby/`
(delivered by Director-run `scp` — the provisional path recorded in D-009 pending IODE
remote-write).

- `cubby/CONTEXT.md` → `docs/CONTEXT.md` (**overwrote** the Phase-1 standup order, "Phase 1
  standup (V2S new-project flow)"). The stage contract is now v0.1, "the money-and-access spine."
- `cubby/STATE.md` → `docs/STATE.md` (**overwrote**).
- `cubby/DECISIONS-append.md` **appended** to `docs/DECISIONS.md` — not overwritten. D-001…D-008
  verified intact ahead of the write and again after; **D-009** (MCP bridge kept provisional;
  IODE remote-write is the real fix) and **D-010** (multi-tenant-ready from the first commit,
  single-tenant in practice) now follow them. File grew 7380 → 9915 bytes; one blank line added
  as the separator to match the existing entry convention.
- All three cubby copies removed. `Rules for a video - Sheet1.csv` deliberately **left in place**
  — out of scope for this task.

**Copied, not moved (`cp` + `rm`, not `mv`).** `mv` would have replaced the inodes and left the
three files root-owned. `cp` onto an existing file truncates in place and preserves the
destination's ownership, so all three remain uid 1000 — the same cross-ownership write failure
L-009/DL-B004 fixed at the write path, kept from being reintroduced at the placement step.
Verified after the write: `1000:1000` on all three.

---

# v0.1 — the money-and-access spine

**Order:** `/docker/binjreel/docs/CONTEXT.md` (Architect, Phase 1.2 · v0.1).
**Date:** 2026-07-25 · **Hat:** 4B Build · **App class:** consumer content SaaS, multi-tenant-ready,
audit tier high. **Budget:** none stated in the order — the DL-B002 class defect recurring one tier
up (see L-011).

## L-011 · Session open — canon read, order read, credential defect found, NO CODE YET
**Nothing was built in this step.** It is logged because the order requires the ledger current
before any pause, and two pauses have already occurred.

**Canon read — degraded, and recorded as such.** `git -C /docker/v2s-canon pull` **FAILED**:
*could not read Username for `https://github.com/papamg007/v2s-canon.git`*. Doctrine was therefore
read from the local clone: `main`, working tree **clean**, at **`ab07022`** — the same commit L-000
verified at standup. **Unverified against `origin/main`.** Read: canon `CLAUDE.md`, `V2S-FRAMEWORK`
v1.9, `V2S-BLUEPRINT` v1.6, `V2S-IMPLEMENTATION-GUIDE` v1.3, procedures `project-binding`,
`decision-routing`, `four-item-escalation`, `defect-traceback`, `secret-manifest`; and the two hats
worn for this build — `4B-build/SKILL.md`, `1C-inspection/SKILL.md`. Then `/docker/PROJECTS.md`
(first, per DL-010), `STATE`, `IDEAL-SCENE`, `TDD`, `CONTEXT`, `DECISIONS` D-001…D-010,
`DEFECT-LEDGER`, `SECRETS`, `app/CLAUDE.md`. Identity check passed (binjreel · Multi-tenant SaaS ·
`binjreel-dev` · `papamg/binjreel`).

**Environment facts established before building.** Node is **not installed on the host** — every
build and test must run in Docker. npm registry reachability confirmed from `node:22-alpine`.
`binjreel-app` + `binjreel-mcp-bridge` both healthy, 6h uptime. Repo root confirmed
`/docker/binjreel/app`, on branch `binjreel-dev`, clean, one commit (`1948227`, the standup).

**DL-B005 found and named.** The Director reported placing the project PAT; the Worker had reported
canon unreachable. Investigation established both facts share one cause: `/root/.git-credentials`
holds a **single host-only entry** (no repo path, mtime 18:19) while the box sets
`credential.usehttppath=true` globally, so git looks up by host **+ path** and a pathless entry
matches nothing — `git ls-remote` fails identically on **both** `papamg/binjreel` and
`papamg007/v2s-canon`. Two alternative causes were falsified before the finding was logged
(per the always-loaded verify-before-escalate rule and Traceback step 5): *"the PAT is bad"* — the
error is `could not read Username` (no credential found) not `Authentication failed`/403 (credential
found, refused), so it is a **lookup miss, not a rejection**; and *"canon is public so nothing
regressed"* — unauthenticated, canon returns **404**, so it is private and its clean pull at L-000
proves a working canon credential existed in that same file and is now gone. One direct
credential-helper probe that would have demonstrated the path/no-path mismatch in a single
observation was **blocked by the tooling permission layer and not worked around**; the diagnosis
rests on error shape + file state + canon's regression, and is recorded at that confidence, not
higher. Written to `DEFECT-LEDGER.md` **DL-B005**, `STATE.md` **W-001**, and `SECRETS.md`
(`github-pat` → `filled`, **not** `verified`; a slot advances only by observing the secret work).
**Not escalated as blocking:** v0.1 commits to `binjreel-dev` locally; STATE already carried push
confirmation as a non-blocking Director act.

**DIVERGENCE — none.** No code written, nothing built differently than specified.

**Order defect noted, not escalated: DL-B002 recurrence.** This `CONTEXT.md` again carries **no
token/cost budget**, which Impl. Guide Customization Pass step 5 and the 4B skill both require. Not
a four-item escalation. Handling per the DL-B002 precedent: build the least resource the result
truly requires, record actual spend as a stat in `BUILD-REPORT.md` in place of a variance. Sharper
than DL-B002's diagnosis, which named a Phase-1.2-skipping standup path as the cause — this order
came *from* a Phase 1.2 pass and still carries no budget, so the gap is not confined to the standup
path. To be appended to the DEFECT-LEDGER on the same canon route as DL-B001/B002/B003/B005.

**Registry drift noted for close.** `/docker/PROJECTS.md` is stale — still says `TDD.md` and
`IDEAL-SCENE.md` are "not yet staged", names the stack as Express, and calls the product *Short Form
Video Delivery Platform* where `STATE.md` says *Short-Form Vertical Drama Platform*. Registry
enrollment is doctrine (`project-binding.md`); to be advanced at close with STATE and the TDD
roadmap, with the name discrepancy surfaced to the Director rather than silently resolved.

## L-012 · Ledger discipline breach — caught by the Director, not by the Worker
**What happened.** The Worker completed the reading step and **paused** for a go-ahead without
writing this ledger, then ran the DL-B005 investigation and **paused again** — writing
`DEFECT-LEDGER.md`, `STATE.md` and `SECRETS.md` in between, but still not `BUILD-LEDGER.md`. The
order states the rule twice ("append after each step and before any pause; never stop without
writing it") and the Director's brief stated it a third time. The Director caught it; the Worker did
not.

**Why it matters beyond the rule.** `BUILD-LEDGER.md` is the one artifact only 4B can produce
(Framework §7 — only the Worker sees app code) and the reason the document layer functions at all.
Writing the Architect-maintained `SECRETS.md` and the shared `STATE.md` *before* the Worker's own
append-only record inverts the ownership order the whiteboard table (Blueprint §10) sets out.

**Named cause.** The Worker treated the ledger as a record of *code written* and concluded a
reading-and-diagnosis step with no commits had nothing to log — when the order conditions the write
on **pausing**, not on producing code. The two steps that were skipped are precisely the ones whose
findings (degraded canon read; DL-B005) a later reader most needs and could not reconstruct from
the repo, since neither left a commit.

**Artifact fix (closed).** L-011 written retroactively above, covering both unlogged steps and
labelled as such. **Standing correction for the rest of this build:** the ledger is written before
any pause, before any reply to the Director, and before any write to a doc the Worker does not own —
whether or not the step produced code.

## L-013 · Steps 1–2 — Fastify/TS foundation and the tenant-scoped data model
**Foundation (CONTEXT §1).** Replaced the Express standup scaffold with the real service.
Node 22 · TypeScript (strict, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`) ·
Fastify 5 · Prisma 6 · Postgres 17 · Redis 7 · `jose` for JWS · `zod` for input shapes. The
`tsconfig` `rootDir` moved from `src` to `.` so `src`, `test` and `prisma` compile as one tree;
the served entrypoint is therefore `dist/src/server.js` and the Dockerfile `CMD` moved with it.

**Compose (`docker-compose.yml`).** The app keeps its standup Traefik wiring untouched. Added
Postgres and Redis on a **separate `internal: true` network** with no published host port — the
app is the only member of both networks, so the money tables have no route to the internet even
if the application layer is wrong. Added a one-shot `migrate` service the app `depends_on` with
`condition: service_completed_successfully`, so a container can never serve against an
unmigrated schema. The Dockerfile gained a `tools` target (Prisma CLI + tests) kept out of the
served `runtime` image; the generated Prisma client is copied across as a build artefact so
runtime needs no Prisma CLI.

**Data model (CONTEXT §2, TDD §6).** All seven tables plus `events` (created empty; its pipe is
v0.5) and `store_notifications` (the receipt idempotency ledger — see L-016). Every table
carries `tenant_id`; that claim is tested against `information_schema`, not against the model
file. Two migrations: `init` (Prisma-generated) and `ledger_append_only` (hand-written).

**ADDED — hardening beyond the letter of CONTEXT, in the second migration.** Logged as ADDED per
build discipline. All four make a stated invariant enforceable by the database rather than by
code being correct:
1. **Append-only triggers on `wallet_entries`** rejecting UPDATE and DELETE. TDD §4.2 calls the
   ledger append-only; in code that is a promise every future call site must keep, and one bug
   away from broken. Row-level triggers do not fire on TRUNCATE, so test suites still reset
   cleanly without an escape hatch existing in normal operation.
2. **`CHECK (delta_coins <> 0)`** — a zero-delta row would consume an idempotency key while
   moving no money, which reads as a completed purchase in the history.
3. **Partial unique index** `(tenant_id, user_id) WHERE scope='subscription'`. The Prisma-level
   unique cannot enforce this: `episode_id` is nullable and Postgres treats NULLs as distinct,
   so an account could otherwise accumulate unlimited subscription entitlements.
4. **Entitlement shape trigger** — episode scope must name an episode, subscription scope must
   not.

**Seed (`prisma/seed.ts`).** One `binjreel` tenant, two fixture series, 14 episodes, free/paid
split. Idempotent, so it is safe on every deploy rather than a one-shot to remember not to
repeat. Coin prices and free-episode counts are **fixtures, not product decisions** — pricing is
the Director's (TDD §12) and is read from config, never decided in code.

## L-014 · Step 3 — accounts, anonymous-first, and the merge
Sessions are Redis-backed (the app holds no session state — TDD §7 Stage 0). Two details worth
the record: Redis stores the **SHA-256 of the token, never the token**, so a dump of the session
store yields nothing usable; and session resolution **follows `merged_into_id`**, so a stale
token held by the app on a viewer's phone lands on the surviving account rather than a drained
one.

Sign-in providers (Apple/Google/email) ship as an **interface with unconfigured implementations
that refuse everything** (CONTEXT §3 — credentials are Director acts). The unconfigured state is
a refusal, never a bypass.

**Merge — a design change made mid-build, and why.** The first implementation moved wallet rows
by `UPDATE wallet_entries SET user_id = ...`, which the append-only trigger blocks, so it carried
a `set_config` escape hatch the trigger would honour. That was rejected before it ran: an
append-only ledger with a privileged bypass is not append-only, and the bypass is exactly the
kind of thing a future caller finds and reuses. Replaced with **paired compensating entries** —
a debit on the source and a matching credit on the destination, one pair per (coin class,
expiry) bucket so a reward coin's expiry is not laundered into a permanent one. The ledger now
has **no exception at all**, both accounts' histories stay literally true, and conservation is
asserted in the test.

## L-015 · Steps 4–5 — the coin ledger and the entitlement engine
**Ledger.** Balance is `SUM(delta_coins)` over non-expired rows; a test asserts no column named
`balance`/`coins` exists anywhere, so there is nothing to drift out of step with history. Every
write carries an idempotency key unique per tenant; a replay returns the original entry and
reports itself a replay rather than erroring.

**Concurrency — the part idempotency does NOT cover.** An idempotency key stops a *replay* of
one act; it does nothing about two *different* spends interleaving. Both `credit` and `spend`
therefore take `SELECT ... FOR UPDATE` on the user row before reading the balance they depend
on, and re-check the key *inside* the lock (checking once leaves a TOCTOU window on money).
Proven by a test that fires five concurrent 30-coin spends against a 100-coin balance and
asserts at most three succeed and the balance agrees exactly with what succeeded. Without the
lock all five commit — and no single-threaded test would ever show it.

**Entitlement engine.** `mayPlay` grants on exactly three routes — free / owned / active
subscription — every check tenant-scoped. `unlockEpisode` is ONE transaction: the debit and the
entitlement commit together or neither does. A test proves the negative case explicitly (an
underfunded unlock leaves **no debit and no entitlement**), which is the half that matters — a
crash between the two is what takes a viewer's money without giving her the scene.

## L-016 · Step 6 — the two store-receipt integrations
Apple ASN V2 (JWS, ES256, `compactVerify`) and Google Play RTDN (Pub/Sub push, OIDC bearer token
verified with `jwtVerify`; the message body is not signed, so the token *is* the signature).
Both normalise to one internal shape, so a later Stripe rail (TDD §8) slots in as another
producer rather than a second code path.

**Fail-closed, and tested as such.** An unconfigured verifier **refuses every notification** — a
test builds an app with the key removed and asserts a validly-signed notification is still
rejected and credits nothing. This is the failure that matters: shipping before the Director
places store keys must not mean accepting unverified money.

**Idempotency has two independent locks**, because store redelivery is normal operation, not an
edge case: `store_notifications (tenant_id, store, notification_id)` for the notification, and
the ledger's own key on the store transaction id. Either alone would do; both are present
because they fail independently — a store can redeliver one purchase under a fresh notification
id, and a test covers exactly that case.

**Attribution is tenant-scoped**: a notification naming an account in another tenant is recorded
(so it is not retried forever) and applied to nothing. A purchase made under a since-merged
anonymous identity is credited to the survivor.

## L-017 · Step 7 — the media seam, stubbed
`getPlaybackUrl` runs the real `mayPlay` and, on refusal, **throws** — it does not return a null
URL, an empty string, or a URL carrying a "do not use" flag, each of which is a shape a careless
caller can ship past. The URL itself is a labelled placeholder (`stub://`, `stub: true` in the
payload) so nothing can mistake it for a real signed URL. The function signature already takes
what a real signer needs (tenant, episode, ttl, clock), so v0.2 is a body swap, not a call-site
migration.

## L-018 · Step 8 + verification — security pass and the proof suite
Rate limiting via `@fastify/rate-limit` backed by **Redis**, so the limit is shared across
replicas rather than per-process (which would silently multiply the real limit by the replica
count). Applied to auth, sign-in, unlock and both receipt endpoints. Error responses are generic
with detail logged server-side; a test asserts no stack frames, internal paths or library names
appear in a response body.

**59 tests, all passing**, run against a real Postgres and a real Redis — not mocks. The
properties at stake are properties of transactions, constraints and triggers; a mock would prove
only that the mock agrees with the code driving it.

**DEFECT FOUND AND FIXED DURING THE RUN (closed-loop).** The suite's first run was 58/59. The
failure — "sign-in failures do not reveal whether a provider is configured" — returned **500**
where a bad token should return 401. Cause: `signInWithProvider` only handled the two error
types it expected, so any other throw from a provider fell through to the 500 handler. This was
a real defect, not a test artefact, and it was fixed in the application rather than by relaxing
the test: **every** provider failure mode now collapses to one uniform 401 with the original
error preserved as `cause` for the server-side log. Two things were wrong with the old
behaviour — differing responses let an attacker map which providers are wired up, and a 500 is
the wrong answer to a bad credential, turning a routine refusal into an alert. Re-verified:
59/59.

**DIVERGENCE — none.** Everything in CONTEXT §§1–8 was built as specified. The `tsconfig`
`rootDir` change and the `dist/src/` entrypoint move are mechanical consequences of compiling
tests alongside source, not design departures. The four database constraints in L-013 are logged
as **ADDED**, not divergences — they enforce invariants CONTEXT and the TDD already state.

## L-019 · Deployment of the v0.1 stack, and the scaffold cut over
`docker compose build && up`. The Phase-1.3 Express scaffold container was stopped and removed —
this is the replacement CONTEXT §1 ordered, not a regression. All four containers healthy:
`binjreel-app`, `binjreel-postgres`, `binjreel-redis`, `binjreel-mcp-bridge`.

**ADDED — `name: binjreel` pinned in `docker-compose.yml`.** The standup stack ran under
Compose's directory-derived project name, `app`. That is the same name every project on the
fleet would derive, and it made the correct teardown command depend on which directory you were
standing in — which is how the scaffold container ended up holding the `binjreel-app` name
against a different project during cutover. Pinning it gives the stack one stable identity.

**Live verification through Traefik** (real network path, `-k` because the public cert still
needs DNS), not `inject`: health 200 · HTTP→HTTPS 301 · anonymous session issued ·
**unentitled playback on a paid episode → 403 with no URL of any kind in the body** · free
episode → 200 with a `stub://` URL flagged `stub: true` · balance 0 · unlock on an empty wallet
→ 402 · unsigned Apple receipt → 400 · unauthenticated protected route → 401 · unrouted host →
404 (no accidental catch-all).

Fixture catalogue seeded: 2 series, 14 episodes.

## L-020 · GATE 2 — the 1C security audit (full tier), and two findings closed
Ran the full battery for this app class: **Base + Auth & Access + Tenant Isolation**.

**Two findings, both fixed and re-verified during the audit — closed-loop, 0 reaching the
Director.** Both were found by running the checks, not by reading the code:

1. **Receipt refusals disclosed configuration state.** `POST /v1/receipts/apple` answered an
   *unauthenticated* caller with `"Apple receipt verification is not configured"`. These
   endpoints must be unauthenticated (the stores carry no session), so anyone can probe them —
   and the distinct messages for *unconfigured* / *bad signature* / *malformed payload* map out
   which store rails are live and which are still dark. **Fix:** `DomainError` gained a
   `publicMessage`; the specific reason now goes to the server log and every receipt rejection
   returns an identical `"Receipt rejected"`. **Re-verified** live and by a new test that asserts
   three different failure causes produce byte-identical bodies — because the *differences* are
   the oracle, not any single message.
2. **Authorization failures were not logged for audit.** The Tenant Isolation tier requires auth
   failures, authorization refusals and tenant-boundary denials to be logged. They were silently
   correct: the refusals worked, and left no trace. An enumeration campaign against episode ids
   would have been invisible after the fact. **Fix:** every 401/402/403 emits a structured
   `access_refused` event (code, reason, method, route, ip). **Re-verified** live — refusals
   triggered through Traefik and the corresponding entries confirmed in the container log.

**Everything else passed on inspection**, confirmed rather than assumed. Highlights: no `.env`
ever committed (checked against full git history, not the working tree); no credential-shaped
strings in any tracked file; zero `npm audit` vulnerabilities across 6 production dependencies;
no `$queryRawUnsafe` anywhere in `src/` and both raw call sites parameterised tagged templates;
app container runs as **uid 1000 non-root, read-only rootfs, ALL capabilities dropped,
no-new-privileges, and no host mounts at all**; Postgres and Redis publish **no host port** and
sit on an `internal: true` network the bridge provably cannot reach; the wall re-tested after the
topology change — the bridge still sees `/docs` and nothing else, and the app container can read
neither `/docker` nor `/docs` and does not hold `BRIDGE_TOKEN`.

Full detail, severity counts and the seam note: `SECURITY-REPORT.md`.

**Noted, not fixed — and correctly so.** Continuous ledger backup does not exist. CONTEXT §8
scopes it as *"noted for the Director"*, not built, and STATE already carries it as the gate on
v0.2 before real money flows. Recorded in the security report as an open Director item rather
than silently passed.

## L-021 · Close — commit, documents advanced, hand-off
**Committed to `binjreel-dev` only, never `main`** (binding + CONTEXT build discipline).
`9c757af` — 31 files, +5,931/−95. Staged tree checked for secret material before committing: no
`.env`, no `.pem`, no `.key`. **Not pushed** — the box git credential is broken for every repo
(DL-B005 / W-001), which is a Director act and does not affect the build.

**Documents advanced at close:**
- `BUILD-LEDGER.md` — L-011…L-021 (this file, append-only, never overwritten).
- `BUILD-REPORT.md` — rewritten per build (Blueprint §10). The Phase-1 standup report it replaced
  is superseded in substance by L-000…L-009, which are permanent.
- `SECURITY-REPORT.md` — new. Full-tier 1C record with severity counts, every checklist line and
  its evidence, the 5A seam note, and four items stated as *noted, not findings*.
- `DECISIONS.md` — **D-011** (append-only in the database, no exception; merge by compensating
  entries) and **D-012** (row lock, because idempotency keys do not stop double-spend).
- `DEFECT-LEDGER.md` — DL-B005 logged earlier this session.
- `SECRETS.md` — three new slots declared (`postgres-credentials` verified;
  `apple-asn-public-key` and `google-rtdn-public-key` empty and **fail-closed**). Replaced the
  "still to be derived" placeholder section, which the TDD's existence made answerable. Also
  recorded three slots that are **not needed** and why, so their absence reads as a design
  consequence rather than an oversight — notably a session signing secret, which does not exist
  because sessions are opaque server-side tokens rather than JWTs.
- `TDD.md` §11 — v0.1 marked built with its commit; v0.2's Director gates named.
- `STATE.md` — WHERE WE ARE, live state, NEXT ACTION and backlog advanced; W-001 updated with the
  confirmed diagnosis.
- `/docker/PROJECTS.md` — registry advanced (it was still describing an Express scaffold with no
  TDD staged).

**Registry defect found and fixed while advancing it.** `PROJECTS.md` carried the identity-check
phrase *"Short Form Video Delivery Platform"* while `STATE.md` and `app/CLAUDE.md` say
**"Short-Form Vertical Drama Platform"**. Under the DL-010 addressing rule an agent verifies the
loaded STATE against those phrases before acting — so the mismatch could have produced a **false
WRONG PROJECT stop** on a correctly-loaded STATE, which is the opposite of the failure the rule
exists to prevent. Reconciled to the Architect's design-doc name, with both phrases explicitly
accepted and a note that a name mismatch alone is not a wrong-project signal. `app/CLAUDE.md`
still carries the older phrase; it is committed, outside the bridge, and cosmetic, so it is raised
for the Director rather than changed unilaterally mid-close.

**Blocked: nothing.** No owner-level question surfaced during the build. The two Director items
(credential repair, ledger backup) were both already on the list and neither gated the work.

**Final state:** four containers healthy; 60 tests passing; `binjreel-dev` clean at `9c757af`.

## L-022 · 2026-07-26 · Credential paths verified live — canon read + binjreel write both proven
**Not a build entry.** No code touched, no credential placed, no build started. This entry records
two verifications the Director ordered before the v0.1-successor build may open, and their raw
evidence.

**Standing order honoured:** *do not place any new credential.* Nothing was written to
`/root/.git-credentials`; no credential file was read, `cat`ed, or `credential fill`ed; no value
was printed at any point.

### V-1 · Canon read path — PASS (live, not an offline no-op)
```
git -C /docker/v2s-canon fetch origin      → exit 0, silent
git -C /docker/v2s-canon pull              → "Already up to date."   exit 0
git -C /docker/v2s-canon log -1 --oneline  → 834fc98 New-Box Standup — walk-one refinements
                                              (S2 read-key-only, S4 auto-registry) (#47)
git -C /docker/v2s-canon rev-parse HEAD    → 834fc987fdcbf3bb6a5d43ca84ae397b9bf37efe
```
**Canon has advanced past the standup snapshot.** L-000/W-001 recorded the local clone pinned at
`ab07022`, unverifiable against `origin/main`. HEAD is now `834fc98`; `git log ab07022..HEAD` shows
**two** commits ahead (`834fc98`, `9fa0e13`). The doctrine the v0.1 build read at `ab07022` was
therefore genuinely stale, exactly as W-001 warned.

**"Already up to date" was NOT an offline no-op — proven, not assumed.** The pull reported no
change because the clone had *already* been advanced (procedures/ mtime `21:01`, before this run).
That is indistinguishable from an offline failure on its own, so origin was forced to answer
directly: `git ls-remote origin HEAD` — a network round-trip that cannot be served from local
objects — returned `834fc987fdcbf3bb6a5d43ca84ae397b9bf37efe`, byte-identical to local HEAD.
`.git/FETCH_HEAD` mtime `21:46:12` against a wall clock of `21:46:52` confirms the fetch in this
run reached the remote. **Origin responded; the clone is current, not merely unchanged.**

**Canon content complete.** `_v2s/` carries `V2S-FRAMEWORK.md` (52,575 b), `V2S-BLUEPRINT.md`
(33,409 b), `V2S-IMPLEMENTATION-GUIDE.md` (12,937 b), `SYSTEM-IDEAL-SCENE.md`, `DEFECT-REGISTER.md`.
`_v2s/procedures/` carries all nine: `chat-operating-instructions`, `csw-packet-template`,
`decision-routing`, `defect-register`, `defect-traceback`, `four-item-escalation`,
`new-box-standup`, `project-binding`, `secret-manifest`. Working tree **clean** — `git status
--porcelain` empty, `git diff HEAD` empty. *(`new-box-standup.md` carries mode 0600 and a newer
mtime than its siblings; it is tracked, matches HEAD exactly, and the mode is a umask artifact of
the 21:01 checkout — noted so a later reader does not mistake it for a local edit.)*

### V-2 · binjreel write path — PASS (push + delete clean, nothing left behind)
```
git -C /docker/binjreel/app ls-remote origin | head -3
  → ec2ef3d1a4fb3bc70fb404b0099535df2de9a310  HEAD
    ec2ef3d1a4fb3bc70fb404b0099535df2de9a310  refs/heads/main

git -C /docker/binjreel/app push origin HEAD:refs/heads/_writecheck
  → * [new branch]  HEAD -> _writecheck        exit 0

git -C /docker/binjreel/app push origin --delete _writecheck
  → - [deleted]     _writecheck                exit 0
```
**Write proven end-to-end and reversed.** Post-delete `ls-remote` shows zero `_writecheck` refs and
the remote back to exactly its prior state — `main` still at `ec2ef3d1`, untouched. No 403, no
branch-protection challenge on the throwaway ref. `binjreel-dev` was **not** pushed; the v0.1
commit `9c757af` remains local and unpushed, and the two-`main` decision W-001 flagged
(remote `main` `ec2ef3d1` vs this box's history) is still open and still the Director's.

### Finding carried forward — "account-level" is asserted, not demonstrated
The Director's framing is an **account-level `papamg` write path covering the Journey/binjreel
repos**. What V-2 proves is narrower: **`papamg/binjreel` specifically pushes**. With
`credential.useHttpPath=true` (re-confirmed `true` this run), git looks up by host **+ path**, so a
store entry keyed to `.../papamg/binjreel.git` matches *that repo only* — a sibling `papamg` repo
would miss it exactly as the host-only entry missed everything in DL-B005. Account-level routing
under `useHttpPath=true` is not something a path-keyed entry gives you for free; it is the thing the
Account-Aware Credential Routing brief exists to build. **No second `papamg` repo was named or
tested, so the plural claim is unverified — recorded as an open question for the Director, not as a
defect.** This is load-bearing for the coming build: if routing is assumed already-solved, the build
starts from a false premise.

---

# v0.2 — media (the Order: docs/CONTEXT.md, placed 2026-07-26)

## L-023 · 2026-07-26 · Order placed — CONTEXT v0.2 installed, v0.1 contract archived
Canon pulled current (`834fc98`, verified live at L-022). The v0.1 stage contract archived to
`docs/CONTEXT-v0.1.md` (copied with timestamps intact, NOT deleted — it is the v0.1 as-ordered
record). `cubby/CONTEXT-v0.2.md` moved into place as `docs/CONTEXT.md` and read in full. It is the
authoritative Order for this build: master storage we own, Bunny ingest+transcode with the async
callback, a real signed short-TTL playback URL as a body-swap of `mintPlaybackUrl`, the paywall at
the seam, minimal media-state read, a dedicated media security pass — all FAIL-CLOSED with no live
Bunny key. Out of scope honoured as written: no player, no admin UI, no DRM, no `mayPlay`/ledger/
entitlement-schema change, no real-money go-live. STOP at GATE 2; the live end-to-end proof is
GATE-3 after the Director places the Bunny key.

**Pre-build survey of what must not break.** Read before writing: the whole media seam
(`src/media/seam.ts`), `app.ts`, `config.ts`, `errors.ts`, `context.ts`, the entitlement engine's
exported surface, the schema, both migrations, and all six test files. Findings that shape the
build:
- `episodes` already carries `master_ref` and `bunny_ref` (nullable, from v0.1). v0.2 adds a
  `media_state` enum (`none|uploading|transcoding|ready|failed`) + `media_fail_reason` — an
  additive migration touching only `episodes`. Ledger tables and entitlement schema untouched.
- `mintPlaybackUrl(ctx, episodeId, ttl, now)` as built is sync and PURE — but a real Bunny token
  needs the episode's `bunny_ref` (a DB read) and the signing key + CDN host (config), neither of
  which its signature carries. The body-swap-with-zero-signature-change premise is therefore
  **false in the code as built** — a DIVERGENCE per the Order, to be logged precisely when the
  swap lands (the enforcement path and route shape do NOT move; what threads through is the media
  config).
- Exactly ONE v0.1 assertion is about the stub itself rather than the property it proves:
  `test/entitlement.test.ts` "a free episode plays" asserts `stub === true` and a `stub://` URL.
  The Order replaces the stub, so that assertion updates to the real-URL shape — named here
  in advance so the change is read as ordered evolution, not as quietly bending a green suite.
  Every OTHER playback test (403s, no-URL-in-refusal, cross-tenant, 401s) asserts properties that
  must and will survive unchanged.

## L-024 · 2026-07-27 · v0.2 built — the media plane, fail-closed end to end
All new code in `src/media/` behind the existing seams; nothing in `mayPlay`, the ledger, or the
entitlement schema was touched (verified at commit by diff, see L-027).

**Schema (additive migration `20260726220000_episode_media_state`).** `MediaState` enum
(`none|uploading|transcoding|ready|failed`) + `episodes.media_state`, `episodes.media_fail_reason`,
an index on `bunny_ref` (the callback's only lookup key), and two CHECK constraints in the v0.1
hand-written style — `ready` requires a `bunny_ref`, `failed` requires its reason — so the seam
cannot be lied to by a half-written row. ADDED (hardening beyond the letter of CONTEXT): the two
CHECKs.

**Master storage (`media/storage.ts`) — ours, vendor-agnostic (TDD §2).** An S3-compatible client
speaking AWS SigV4 directly (~200 lines, `node:crypto`, no vendor SDK — one auditable module per
external credential). It offers exactly two operations: a SIGNED existence check at registration,
and a short-TTL presigned GET minted only to hand a master to Bunny — clamped to ≤1h in code, no
method that returns an unsigned URL. Storage keys are allowlist-validated (no traversal shapes).

**Bunny client (`media/bunny.ts`) — the ONE module that speaks to Bunny.** Documented Stream API:
`POST /library/{id}/videos/fetch` under the AccessKey header, returning the video GUID the
completion webhook later names. API base overridable so contract tests run against a local fixture
speaking the documented shape; the shape itself is re-verified live at GATE-3.

**Pipeline (`media/pipeline.ts`).** Registration: verify the master EXISTS in our storage → set
`master_ref`+`uploading` → Bunny fetch → `bunny_ref`+`transcoding`; a vendor refusal records
`failed` + reason and answers a generic 502 (detail server-side only). Callback: library id
checked, terminal statuses move the machine (4→ready, 5/6→failed+reason), intermediates
acknowledged as no-ops, unknown GUIDs acknowledged (`updated:false`) so redeliveries after a
re-register cannot error-storm.

**The seam (`media/seam.ts`) — stub body swapped for real Bunny CDN token signing.** Order
preserved exactly: `mayPlay` FIRST (unchanged, tenant-scoped, refusal throws), and only an
entitled request ever reaches the media checks — so an unentitled caller learns nothing about
media state, configuration, or `bunny_ref`. Then fail-closed ladder: slots empty → 503 minting
NOTHING; not `ready` → 409 leaking neither ref nor reason; ready → the documented Bunny token
(`base64url(sha256(key+path+expires))`), TTL config-driven seconds.

**DIVERGENCE (CONTEXT §3, logged as the Order requires).** "Real signing replaces the stub body
and no call site moves" proved FALSE in the code as built: the v0.1 stub was pure, and a real mint
needs the episode's `bunny_ref`+state (a DB read) and the signing key+CDN host (config), neither
in the stub's signature. What actually moved: `getPlaybackUrl` gained ONE argument (the signing
config) at its single call site in `app.ts`, and `mintPlaybackUrl` now takes the media row's GUID
rather than the episode id. The enforcement path, the route shape, and the module's public
surface are otherwise as v0.1 built them. TDD §11(2)'s "changes no call site" is now stale as
written — flagged for the Architect at reconciliation.

**Routes.** Playback now rate-limited (DoD). New: `POST /v1/episodes/:id/master` and
`GET /v1/episodes/:id/media`, both gated by the `media-admin-token` slot (constant-time compare,
uniform 401 whether the slot is empty, the header is absent, or the token is wrong — an operator
surface a viewer session can NEVER open); `POST /v1/media/bunny/callback?token=…` gated by the
webhook-secret slot the same way. All three rate-limited. ADDED (named): the operator token gate —
CONTEXT names no auth for the registration path, and shipping it open or viewer-reachable were
both wrong; v0.4's real roles replace it. NOTE for the security report: the webhook secret rides
a query parameter because Bunny Stream webhooks can carry no custom header — it will appear in
server-side request logs (box-local); accepted and recorded rather than hidden.

**Config/env.** Ten new env keys, every one optional and fail-closed empty (`.env.example`
documents each; SECRETS.md declares the six secret-bearing slots). The box `.env` was NOT touched
— absent keys ARE the fail-closed configuration. `docker-compose.yml` needed no change.
`/health` version and package.json advance to 0.2.0.

## L-025 · 2026-07-27 · Proof — 85/85 green against real engines (60 v0.1 + 25 new)
Run in Docker (`node:22-alpine`) against an EPHEMERAL Postgres17+Redis7 stack on a scratch
network — never the live containers (the suite TRUNCATEs; the live DB holds the seeded fixtures).
Migrations applied to the scratch DB and verified present (`\d episodes`: new columns, both
CHECKs, the bunny_ref index) before the run.

**All 60 v0.1 tests pass unmodified in substance.** The single ordered exception (named in
advance, L-023): the free-episode playback test's stub assertions became real-URL assertions —
same property, real signer. Fixture episodes now seed media-`ready` (the OTHER tenant's episode
deliberately too, so the cross-tenant refusal keeps coming from tenancy, never masked by a media
409).

**25 new tests, the DoD items each proven not asserted** — highlights:
- Signer: deterministic; byte-equal to an independent recomputation of the documented algorithm;
  key never in the URL; positive-integer TTL enforced.
- Storage: traversal-shaped keys rejected before any network call; presign TTL clamps at 3600s
  even under hostile config; secret never in a presigned URL; unconfigured → both operations
  refuse. The fixture S3 REFUSES unsigned requests, so the client's SigV4 is load-bearing in the
  test, not decorative.
- Pipeline over real sockets: registration verifies the master in OUR storage first (a missing
  object persists NOTHING and never calls Bunny); the fetch call carries the AccessKey header and
  a PRESIGNED source URL; a Bunny 500 lands `failed`+reason with the vendor detail kept off the
  wire; callbacks: 4→ready, 5/6→failed+reason, 0–3 no-op, wrong/missing token → uniform 401 with
  state untouched, wrong library → 400, unknown guid → acknowledged no-op.
- The paywall under real signing: entitled → manifest URL whose token verifies against an
  independent recomputation and whose TTL is the configured seconds; unentitled → 403 with no
  URL, no bunny_ref, no CDN host; entitled-but-unready → 409 leaking neither ref nor reason nor
  even the state name; ALL SLOTS EMPTY → 503 `media_not_configured`, no URL, missing-slot names
  kept off the wire, and the admin+callback surfaces equally closed; mint endpoint rate-limits
  (429 under a tiny cap); two mints differ only by time — nothing user-shaped in a shareable URL.

**Tooling note.** `node --test dist/test/` (directory form) stopped working on node 22.23.1 — the
runner tried to require the directory as a module. `npm test` now uses the explicit
`dist/test/*.test.js` glob. Same command shape, same 85/85 result, pinned in package.json.

## L-026 · 2026-07-27 · Dedicated media-seam security pass — found 1, fixed 1, 0 needing the Director
CONTEXT §6's checklist run line by line, plus an adversarial walk of the new secret surfaces.
Full record: `SECURITY-REPORT.md`, new v0.2 section (the report is now cumulative, newest first;
the stale canon-availability note in the v0.1 section is annotated resolved per L-022).

**F-003 (LOW, fixed closed-loop).** The webhook secret is URL-borne (D-015 — Bunny Stream
webhooks carry no header and no signature), and BOTH log sinks would have recorded it: the
request logger records `req.url`, and the `access_refused` audit event falls back to
`request.url`. Fixed with `redactSecretParams()` at both sinks; unit-tested; 86/86. Found by
walking the secret's life-cycle ("where does this value ever exist?"), contrary-tested against
the prod logging config before being logged as real.

**Checklist outcomes** (each run, not assumed): one mint path only, behind `mayPlay`, grep-audited
and refusal-tested · TTL inside the signed digest, proven by independent recomputation — edge
refusal itself is GATE-3 · keys env-only, tracked-file sweep clean, bridge topology unchanged ·
key never in URL/response/log (three tests + F-003) · mint/registration/callback all rate-limited
with a driven 429 · masters unreachable at the client (no unsigned method, ≤1h clamp, allowlisted
keys, fixture refuses unsigned) — bucket policy is the Director's half at GATE-3.

**Adversarial outcomes:** forged callback → uniform 401, state untouched; valid-secret + wrong
library → 400; guessed GUID → no-op; storage endpoint/API base are config not input; traversal
keys die before any network call; viewer session ≠ operator; empty admin slot fails CLOSED;
vendor error bodies stay off the wire; URLs carry nothing user-shaped. Residuals recorded as
N-005 (URL-borne secret, accepted with mitigations) and N-006 (cross-tenant GUID lookup is
design). v0.1's N-002 and N-003 are resolved by this build. Severity: Critical 0 · High 0 ·
Medium 0 · Low 1 (fixed). `npm audit`: 0 across prod deps (still 6 — no new dependency was
added by this build).

## L-027 · 2026-07-27 · Close — commit, documents advanced, GATE 2, STOPPED as ordered
**Committed to `binjreel-dev` only, never `main`: `b0b67bc`** — 17 files, +1,739/−56 across
source, schema, tests. Staged tree checked for secret material before committing (no `.env`, no
key files, no credential-shaped strings). **NOT pushed** — the Order says do not push; the write
path itself is verified working (L-022), so this is discipline, not inability. **NOT deployed** —
the Order stops at GATE 2 and CONTEXT v0.2's DoD carries no deploy item; the running container
still serves v0.1 `9c757af`, and v0.2 changes nothing viewer-visible until the GATE-3 keys exist.
Deploy is one command whenever the Director chooses.

**Documents advanced at close:** CONTEXT-v0.1.md archived + CONTEXT.md placed (L-023) ·
BUILD-LEDGER L-023…L-027 (this file, append-only) · BUILD-REPORT rewritten per build (v0.2) ·
SECURITY-REPORT — cumulative now, v0.2 pass prepended, v0.1 notes N-002/N-003 resolved ·
DECISIONS D-013/D-014/D-015 · SECRETS.md — six media slots declared fail-closed with GATE-3
placement notes, the v0.1 "class slots expected" placeholder superseded · TDD §5 as-built +
§11(2) marked BUILT with the DIVERGENCE named against its stale "no call site" line ·
STATE.md advanced (WHERE WE ARE, live state, NEXT ACTION, backlog).

**Ephemeral test infrastructure torn down** (scratch Postgres/Redis/network) — the live stack
was never touched by the suite.

**Blocked: nothing.** No owner-level question surfaced; the GATE-3 dependencies were all known
to the Order in advance and are listed in BUILD-REPORT "What GATE-3 needs from you".

**Final state:** 4 containers healthy (serving v0.1) · `binjreel-dev` clean at `b0b67bc` ·
86/86 tests · canon current at `834fc98`.

## L-028 · 2026-07-27 · DL-B006 fixed and PROVEN LIVE — directory token, HLS playback works at the edge
**The defect (GATE-3 catch, Director-ordered fix):** the v0.2 signer signed a single-FILE token
over `/{guid}/playlist.m3u8`; HLS references sub-playlists and segments at sibling paths, none
covered, so playback died at the edge. Full diagnosis: `DEFECT-LEDGER.md` **DL-B006**. Decision
correcting the scheme (and superseding the old signer's directory-scope comment assumption):
**D-016**.

**Verification-first, as ordered.** The vendor's CURRENT reference was fetched before coding:
docs (advanced = HMAC-SHA256 "HS256-" tokens; basic = deprecated MD5) and the reference
implementation (BunnyWay/BunnyCDN.TokenAuthentication nodejs/token.js) — which contradicted the
order's own summary formula in two ways the edge later confirmed mattered: the token is an HMAC
(not a plain concat-hash), and `token_path` participates in the signed data. Signer rewritten to
mirror the reference byte-for-byte for our parameter set; unit tests recompute it independently;
plus a new test that a relative segment reference resolves INSIDE the authenticated prefix.
Suite: **87/87** on the ephemeral stack.

**Live iteration at the edge (the only honest oracle).** First deployed mint 403'd. Rather than
guess, candidate constructions were tested directly against the edge — the signing key read into
container memory exactly as the app reads it, **statuses printed, never values**. Findings:
1. **Both** the HS256 HMAC form and the legacy sha-concat form pass — WITH `token_path` in the
   signed data. Without it: 403. The implemented (HS256) form is correct and current.
2. The blanket 403s were the zone's **hotlink gate: empty `Referer` → 403, any referer → 200**
   (even a foreign one). Zone setting, not code — raised to the Director in DL-B006/D-016
   because native mobile players typically send no referer (v0.3 concern).
3. API cross-check: video `b56ffcb4…` status 4 (finished), 5-rung ladder 240p–1080p — the
   pipeline's own ingest genuinely transcoded.

**The three ordered proofs, against the DEPLOYED seam** (mint via `POST /v1/auth/anonymous` →
`GET /v1/episodes/e58a8a9e…/playback`, curls carrying a referer per finding 2):
1. **Manifest: HTTP 200**, valid `#EXTM3U` master playlist, 5 stream variants.
2. **Referenced sub-playlist `360p/video.m3u8`: HTTP 200** under the SAME token — the exact
   request class the single-file token 403'd — and one level deeper, media segment `video0.ts`:
   **HTTP 200, 351,560 bytes**.
3. **Expired mint: HTTP 403** — the TTL-death property survived the scheme change (grant TTL
   observed ~30s, config-driven).

**Deployed:** `docker compose up -d --build` — healthy, `/health` 200 v0.2.0 through Traefik.
The deployed image was built from the exact tree committed below. Scope discipline: `mayPlay`,
ledger, entitlement schema untouched (the diff touches `src/media/signer.ts` + two test files).

**GATE-3 DoD items this closes:** "a real master transcodes and plays through a real signed URL"
✅ live · "an expired link dies at the edge" ✅ live. Still open at GATE-3: the master bucket
refuses anonymous reads (bucket-policy half), and the Director's call on the empty-referer zone
setting.

## L-029 · 2026-07-27 · v0.2 GATE-3 COMPLETE — recorded (docs-only pass, no code, by order)
**The Director closed the gate live.** All four GATE-3 DoD items now proven, the last one this
session: the **master bucket refuses an unauthenticated GET** — non-200 (400, no object served),
public access disabled at the bucket, while the signed presigned fetch demonstrably works (it is
how the master reached Bunny and transcoded). With L-028's three (manifest 200 · segment 200
under the same token · expired 403), the DoD's live column is fully checked — and beyond the
checklist, **the Director watched a real episode play in VLC**. The platform now streams real,
paywalled video end to end: v0.1 decides, v0.2 delivers.

**Two operational findings folded in on the way to green:**
- **DL-B007 (new):** the 30s playback-TTL default is shorter than a viewing session — the
  directory token expires as a whole while the player is still fetching segments, so playback
  died mid-episode. Resolved operationally (`PLAYBACK_URL_TTL_SECONDS=3600` in `app/.env` —
  config-driven by design, no code change, live-verified). OPEN residue, next code touch: raise
  the CODE default in `config.ts` (still 30) so a fresh deploy without the env var cannot
  silently regress. Final TTL is the Director's product decision (anti-sharing vs viewing
  session) — added to the TDD §12 shape of things.
- **DL-B006 referer mystery resolved zone-side:** Bunny's **"Block direct URL file access" was
  ON** — it 403'd every no-referer client (curl, VLC; would have been native mobile players in
  v0.3) while any non-empty referer passed, i.e. friction without protection. **Now OFF**
  (Director). CDN token authentication remains the real gate — cryptographic and expiring,
  which the referer check was neither. The v0.3 player blocker from the prior GATE-3 report is
  cleared.

**Documents advanced:** STATE (baton rewritten — WHERE WE ARE / NEXT ACTION / BACKLOG to current
truth; baton passes to the Architect for the v0.3 CONTEXT) · SECURITY-REPORT (GATE-3 item 3
proven, all four marked complete; referer resolution; the "seconds of TTL" claim honestly
amended to the hour-scale interim posture) · DEFECT-LEDGER (DL-B007 logged) · this ledger.

**Carried forward, still open, restated so nothing arrives as a surprise:** receipts remain
fail-closed (no Apple/Google keys — placed at real-receipt go-live) · **continuous ledger backup
remains THE gate before real receipts** (SECURITY-REPORT N-001) · two-`main` reconciliation +
branch protection on `main` remain open (W-002) · v0.2 is **unpushed — a single copy on this
box** (`574275a`); the push is one command once the two-`main` question is settled.

**No commit this pass.** The Order says commit to `binjreel-dev`, but every artifact it touches
lives in `/docker/binjreel/docs` — OUTSIDE the git repo at `app/` (the wall separates the record
layer from source by design; `/docs` has never been git-tracked). Nothing in `app/` changed
(`git status` clean before and after), so there is nothing to commit and no new SHA to mint —
recording that plainly rather than minting an empty commit. HEAD remains **`574275a`**.

## L-030 · 2026-07-28 · Docs placement — TDD v2 + CONTEXT v0.3 placed from cubby; v1/v0.2 archived. No build started.

## L-031 · 2026-07-28 · v0.3 BUILD OPEN — combined Order (v0.3 + v0.4 phased, ≤$150 total)
Canon pulled current (`834fc98`, already up to date). Orders read: `docs/CONTEXT.md` (v0.3, live)
and `docs/CONTEXT-v0.4.md` (placed this session from cubby, becomes live at the v0.3 gate).
Repo at `574275a` on `binjreel-dev`, clean. Plan for this phase, in order: schema migration
(genre/nonfiction/tags/shelves/engagement/share/schedule/captions/cast) → config (TTL code
default raised per DL-B007) → catalog/engagement/offer/share/re-mint/admin route modules →
comedy fixture seed → full suite in Docker against ephemeral engines (87 prior must stay green)
→ security pass → deploy on-box → ledger/report/STATE/TDD advance → commit AND push (per this
Order; L-022 verified the push path; remote `main` untouched). Money-core diff target: 0 lines
(`wallet/ledger.ts`, `entitlements/engine.ts`, `receipts/*` unchanged — verified by git diff at
gate). Standing rule noted: external-account needs are DEFERRED-BY-ORDER, never BLOCKED.

## L-032 · 2026-07-28 · v0.3 BUILT — all 14 ordered scopes, 138/138 green, money core 0-line diff
The full client contract, in the Order's own numbering:
**(1) Migration** `20260728193016_v0_3_client_contract` — genre/nonfiction on series (column
default backfills BOTH launch series → comedy), tags + series_tags, shelves + shelf_items
(hero via `kind`), follows, list_items, watch_progress + series_resume pointer, watch_history,
share_links + link_resolutions, episode_schedule, captions, cast_members + series_cast, episode
duration/thumbnail display columns. No FK anywhere into money tables. **(2) Catalog reads**
`src/catalog/reads.ts` — explicit allow-list selects (a new column cannot leak by default);
series list/hubs/tags/search+trending (Redis rolling window)/Top-10 (config window, history +
unlock counts). **(3) Shelf composition** `src/catalog/home.ts` — hero + ordered shelves from
merchandising rows; rule-kinds (continue_watching per caller · free_to_binge by free-count ·
top10) composed at read; shelf CRUD behind D-014. **(4) Show page** one call: card, tags, cast,
grid with per-caller lock states (`src/catalog/access.ts`, pinned to `mayPlay` by a full-matrix
agreement test), resume point, related-by-tag/genre. **(5) Engagement writes**
`src/engagement/writes.ts` — follow/list/progress/history; progress upsert atomic-first-watch
(`createMany skipDuplicates` — a read-then-create race found BY the rapid-repetition test and
fixed structurally); merge moves all engagement (accounts.ts extended; newer position wins).
**(6) Next-episode + prefetch** grant rides along when entitled; offer when not; end-of-series
explicit; unentitled prefetch never reaches the mint. **(7) Three-door offer** `src/offers/` —
identical shape anonymous/signed-in, ad door data-present fail-closed false (D-017), every value
from config incl. server-side comedy-voice copy. **(8) Entitlement summary** `mayPlay` is the
arbiter per candidate row (engine frozen; divergence test with an expired entitlement).
**(9) Re-mint** `POST /v1/episodes/:id/playback/renew` — same seam, full re-check (lapsed sub →
403 mid-session by test), zero ledger movement; config default TTL 30→900 (DL-B007 CLOSED
FULLY — see DEFECT-LEDGER). **(10) Share/deep links** 128-bit tokens, moment + operator mints,
resolve → target + entitlement + grant|offer, unavailable → clean state, resolutions + origin
tags persisted (JVScope join), enumeration guessing test + dedicated resolve limiter.
**(11) Captions** manifest read, metadata only, empty valid. **(12) Scheduling** publish_at
honoured by EVERY surface incl. mint/unlock/share (ADDED, guard scoped so every v0.1/v0.2
refusal keeps its exact shape); follow-fanout hook = claim-once log-only sweep (60s interval,
replica-safe). **(13) Fixtures** comedy catalog seeded IN PLACE (legacy titles rebranded, live
media rows preserved): "My Landlord Is a Billionaire (Unfortunately)" + "Fired & Furious",
tags/cast/captions/6 shelves. **(14) Security pass** — L-033.
**Proof: 138/138 in Docker vs ephemeral PG17+Redis7 (60 v0.1 + 27 v0.2 unbroken + 51 new).
Money core diff = 0 lines (git-verified). npm audit: 0 across unchanged deps.**

## L-033 · 2026-07-28 · v0.3 security pass — found 1 (LOW), fixed 1, 0 needing the Director
CONTEXT §14 / 1C pattern on all new surfaces. **F-004 (LOW, fixed closed-loop):** trending
searches recorded every query — an arbitrary-string injection channel onto every viewer's
search screen at rate-limit speed. Fixed: only catalog-matching queries trend; proven by test.
Informational N-007 (link_resolutions growth, bounded) and N-008 (draft-series free-episode
mint if an id ever leaked — engine consults no publish_state; v0.1 as-built, engine frozen this
Order; no viewer surface emits draft ids, proven; fold into the engine's next ordered touch)
recorded. Full walk: `SECURITY-REPORT.md` v0.3 section. No high/critical open.

## L-034 · 2026-07-28 · v0.3 DEPLOYED on-box — migrate → seed → live-verified
`docker compose build && up`: migrate one-shot applied the v0.3 migration to the LIVE database,
app redeployed healthy (`/health` → 0.3.0). Seed evolved the catalog in place — 2 series (both
comedy), 14 episodes (media rows untouched), 6 shelves. Live through Traefik: home payload
serves hero + shelves; catalog/genre/search/offer verified; leak-grep across live viewer reads:
0 hits for bunny/b-cdn/master_ref/storageKey. Anonymous session → episode grid with lock
states → offer payload walked end to end live. Ephemeral test stack (scratch pg/redis) left
running for phase 2's suite runs; the live stack was never touched by tests.

## L-035 · 2026-07-28 · v0.3 GATE CLOSED → PUSHED · v0.4 PHASE OPEN (the web face)
Phase-1 gate: every CONTEXT v0.3 DoD box checked (BUILD-REPORT) · 138/138 · security clean ·
deployed · docs advanced. **Pushed `binjreel-dev` → origin as a NEW branch (`4ff70e1`), per the
combined Order and L-022's verified path. Remote `main` untouched (still `ec2ef3d1`); W-002
two-`main` question stays parked.** `docs/CONTEXT-v0.4.md` now governs as the live stage
contract (v0.3's `CONTEXT.md` remains the prior phase's as-ordered record, per its own header —
NOT archived, per the kickoff Order).
Phase-2 plan: (a) API side — real admin identity table + admin-auth routes (bcrypt hash-only,
forced first-login change, hard rate limit, uniform failures) + app-config read; (b) Expo
(D-019) web codebase under `app/web/` — five tabs, ReelShort/NetShort-skew dark comedy design
(tokens recorded in `app/web/DESIGN.md`), hls.js player (gapless prefetch, re-mint, captions,
speed, quality), the three-door wall from the server offer verbatim, share-moment, /s/{token};
(c) web server — static serve + SEO meta injection for show pages + /admin panel with
server-side session and a proxy that keeps the D-014 token out of the browser; (d) Traefik
re-wiring — web at root, API keeps /v1/* + /health by path rule (Bunny callback unaffected);
(e) tests + 1C pass (admin auth foremost); (f) deploy live at binjreel.com. Store builds, IAP,
push, deferred links: v0.5 by Order. Sign-in providers (Apple/Google) need external accounts →
will be wired UI-side and logged DEFERRED-BY-ORDER.

## L-036 · 2026-07-28 · v0.4 BUILT — Expo web face + real admin identity, 147+9 tests green
**API half:** `admin_users` migration (identity SEPARATE from viewer users by design) ·
`src/admin/auth.ts` (bcrypt-12 hash-only, burn-in-hash uniform failures, Redis sessions with
issued-at, `passwordChangedAt` revokes every earlier session) · `/v1/admin-auth/*` routes
(login hard-limited 5/min, uniform 401s; change-password enforces floor + difference and kills
all sessions incl. its own) · `/v1/app-config` (Rewards tab state from config, fail-closed
off) · caption track CONTENT route (presigned redirect via the owned-storage seam, 503
unconfigured / 404 no track) · seeded admin michael@journeyviral.com HASH-ONLY with
`must_change_password=true` (the plaintext temp value exists nowhere in the repo — hashed
out-of-band from the Order, never echoed). One new dependency: bcryptjs (pure JS; audit 0).
**Web half (`app/web/`, D-019 one codebase):** Expo SDK 57 + expo-router; five tabs (Home ·
For You · Rewards[config-fed coming-soon] · My List · Profile); design authored per the Owner
directive and RECORDED in `app/web/DESIGN.md` + `src/lib/theme.ts` (dark, marigold comedy
accent, poster cards/badges, hero carousel, Top-10 rank medals); show/tag/genre/search screens;
the PLAYER (hls.js, full-bleed vertical, swipe up/down via touch+wheel+keys, tap-to-hold,
episode drawer, captions toggle, 0.75–2× speed, quality ladder, prefetch-fed transitions —
/next's grant is in hand before the swipe, **mid-session re-mint adopts fresh tokens via
xhrSetup URL-rewrite without interrupting playback**, 5s progress reporting, resume);
THE WALL rendered from the offer verbatim (three doors always present, purchase doors
"get the app — coming soon", ad door disabled from payload, dismiss returns in place);
share-a-moment mint + /s/{token} landing; anonymous-first session, merge wired.
`VideoSurface` is the ONE platform-split module — the native side is an explicit v0.5 seam.
**Web server (`web/server/index.mjs`, zero runtime deps):** static bundle + SEO meta injection
for /show/{id} (HTML-escaped, crawler-safe) + static share cards (no funnel double-count) +
the /admin proxy: login moves the session token into an HttpOnly/Secure/SameSite=Strict cookie,
ops calls re-validate + refuse must-change sessions + attach the D-014 token SERVER-SIDE behind
an explicit 8-route allow-list. **Proof: API suite 147/147 (prior 138 unbroken + 9 admin-auth)
· web-server suite 9/9 incl. a bundle scan proving no token/credential string in the shipped
JS.** Thin admin panel at /admin: catalog + media state, schedule set/clear, tag assignment,
shelf/hero ordering — all through the proxy.

## L-037 · 2026-07-28 · v0.4 security pass — 0 new defects, 1 advisory dispositioned, admin auth walked first
CONTEXT §12 / 1C pattern, admin auth foremost — full record in SECURITY-REPORT v0.4 section.
Highlights: uniform login failures with burn-in bcrypt on the unknown-email path; forced change
gates OPERATOR ACTS server-side (proxy 403), not just UI; password change revokes every prior
session (tested on parallel sessions); ops proxy allow-list proven unable to forward unlisted
paths (stub-API assertion: zero requests arrived); served bundle scanned for secrets in-suite
AND live (0 hits). Dependency advisory: uuid<11.1.1 via Expo BUILD tooling — build-time only,
runtime web image ships zero npm deps; dispositioned, revisit at the v0.5 SDK touch. N-009
(Safari re-mint caveat) and N-010 (committed hash of the one-time temp credential — Order-
sanctioned) recorded. Money core diff across the ENTIRE engagement re-verified: 0 lines.

## L-038 · 2026-07-28 · v0.4 DEPLOYED LIVE at binjreel.com — served == merged, every live check green
`docker compose build` (app + web + tools — the tools image needed its own rebuild after the
first migrate ran stale; caught because the seed refused, fixed, re-run clean) · migration
`v0_4_admin_identity` applied to the live DB · admin seeded. Traefik: web at root (priority
10), API keeps `/v1/*` + `/health` by path rule (priority 100). **Live through the real edge:**
root serves the app shell (title "binjreel") · `/health` → 0.3.0 API · `/v1/app-config` →
rewards coming-soon · www → apex 301 · show page carries injected SEO (`<title>My Landlord Is
a Billionaire (Unfortunately) — binjreel</title>`, og:type video.tv_show) · /s/* carries the
static share card · **admin login live: 200, `mustChangePassword: true`, HttpOnly/Secure/
Strict cookie, token absent from body; ops 401 bare; login storm 401×4 → 429×3** · live served
bundle grep: 0 secret hits · **playback grant + re-mint minted live against the real zone**
(the GATE-3 test-clip episode, "Fired & Furious" ep 1 — FREE, so the Owner's cold browse can
play it) · Bunny callback path answers exactly as before (401 probe refusal) · share-moment
minted live. **DEFERRED-BY-ORDER (external accounts, per the standing rule — noted, not
blocked):** Apple/Google sign-in providers (UI + merge wired; provider config needs the
external accounts) · IAP/StoreKit/Play (v0.5 anyway) · ad network (v0.6). The temp admin
credential is UNTOUCHED for the Owner's own first login — the forced change is theirs to
perform at GATE-3. Served == merged: the running web container was built from this tree this
hour; commit + push follow as the phase gate.

## L-039 · 2026-07-28 · ENGAGEMENT CLOSED — both phases complete, pushed, STOPPED AT GATE-3
The combined Order is delivered: **v0.3 (`4ff70e1`) + v0.4 (`cd0d55e`) on `binjreel-dev`,
both pushed; remote `main` untouched at `ec2ef3d1` (W-002 parked).** Docs advanced: STATE ·
TDD roadmap (v0.3 + v0.4 marked built) · BUILD-REPORT (combined engagement) · SECURITY-REPORT
(v0.3 + v0.4 sections) · SECRETS.md (D-014 second destination; admin-identity divergence note)
· DEFECT-LEDGER (DL-B007 CLOSED FULLY) · /docker/PROJECTS.md registry rows. Ephemeral test
stack torn down; live stack: 5 containers healthy (app, web, postgres, redis, bridge).
Spend, estimated (no meter on the box): phase 1 ≈ $15 · phase 2 ≈ $20 · **engagement ≈ $35 of
the ≤ $150 ceiling** — logged per phase in BUILD-REPORT. Blocked: nothing. Deferred-by-order:
sign-in providers / IAP / ad network (external accounts). **STOP: v0.4 GATE-3 is the Owner's
live viewing — browse, play, swipe, wall, share, and the /admin forced password change, on
their own device.**

## L-040 · 2026-07-28 · DL-B008 FIXED — structural /admin screen gate, tested at the missing surface, live-verified
Order: defect fix, web face only. Reproduced first (headless Chromium, clean profile, against
the deployed site): fresh /admin DID render the login card — but the route fired a viewer
bootstrap (`POST /v1/auth/anonymous`) before any session resolved, the gate was one screen's
component state (convention, not structure), and NO test asserted what a browser renders at
/admin. Fix: `admin/_layout.tsx` gate over the whole route group (session resolves FIRST via
exactly one `GET /admin/api/me`; login-only / change-only / panel; change success revokes all
sessions → back through the front door; ops 401/403 mid-session re-enter the gate) · root
layout skips the viewer bootstrap on /admin · panel screen is now mount-only-past-the-gate.
NEW TEST CLASS: Playwright e2e (`web/e2e/admin-gate.spec.mjs`, runs the REAL bundle in a real
browser) — login-only + zero-other-calls (browser log AND stub-API log), change-only for
must-change, the full temp→forced-change→re-login→panel journey, eye-reveal. 4/4 · server
suite 9/9 still green. Deployed (web image rebuilt; container healthy). LIVE screen-level
verify, clean profile: login form and nothing else; network exactly `GET /admin/api/me`;
served == merged (bundle `entry-10e989a3…` identical container vs tree); viewer home
regression-checked (bootstrap + catalog render intact). DL-B008 logged with traceback
(API-level auth verified, screen-level never asserted — the DoD named "enforced" without
naming the surface); SECURITY-REPORT F-005 section added. No API change; no ledger touch;
no design work. Commit + push follow.

## L-041 · 2026-07-29 · RESTYLE ORDER opened — design package unpacked, inventoried, cubby cleared
Order: conform the live web face to the ratified design package. Scope app/web only (tokens,
styles, assets, component visuals); no IA, no API, no ledger, no features; budget ≤ $40.
Unpacked `Canvas doc with six screens-handoff.zip` from cubby →
`docs/design-package/canvas-doc-with-six-screens/` (permanent visual record); zip removed from
cubby. **Inventory:** `project/binjreel Handoff Package.dc.html` (v1.1, the visual source of
truth — §1 tokens · §2 mobile dark ×6 · §3 mobile light ×5 · §4 web 1440 (home dark/light,
player+rail, wall modal) · §5 admin backend ×5 · §6 component kit · §7 voice note · §8
rewards/profile · §9 search/onboarding) · `project/tokens.css` (v1.0 FINAL — complete: hex
for both themes, named fonts Gabarito/Plus Jakarta Sans/JetBrains Mono via Google Fonts, type
scale, 4pt space, radii, elevation, motion curves — nothing missing, nothing ambiguous, no
STOP) · `project/HANDOFF.md` (implementation notes: three-accent system, hard rules, layout
numbers, asset status — mockup stills are placeholders) · `project/uploads/` (10 placeholder
9:16 stills, hero bed `BRBGBlurredVing.mp4` + poster, webp still) · review-only variants
(`binjreel Design Package.html` base64 bundle · `Redesign.dc.html` earlier pass ·
`standalone-src.html`) · `support.js` (mockup runtime, not for port). Voice note (§7) to be
transcribed verbatim into DESIGN.md v2.

## L-042 · 2026-07-29 · RESTYLE BUILT — DESIGN.md v2 contract + full token/surface conformance, one stage-input defect found & fixed
**Contract:** `app/web/DESIGN.md` rewritten as v2 — faithful transcription of the package's
tokens.css (both themes' hex, Google-Fonts loading strategy, --t-* type scale, 4pt space,
radii, elevation, motion curves), the hard rules, per-screen conformance notes for the five
briefed surfaces, the §7 voice note verbatim, and the order-scope divergences recorded
explicitly (dark-only ship — no theme toggle exists and one is a feature; ha-stats absent
from the API — slots styled, never faked; five-tab IA kept at all widths per the order's
"no IA changes" over §4's web header; wall copy stays the server's verbatim; player episode
rail stays the restyled drawer). **Conform:** `src/lib/theme.ts` rewritten to the package
palette/type/radii/space/motion (legacy aliases keep unbriefed screens on-token);
Gabarito/Plus Jakarta Sans/JetBrains Mono loaded via Google Fonts preconnect+swap — Expo's
`output:"single"` ignores `+html.tsx`, so a post-export `scripts/inject-html.mjs` step (wired
into export:web + Dockerfile) injects the links + base stage. Surfaces: Home (scrim-hero'd
hero, EP 1–N FREE lime badge, Gabarito 900 display, Start free act CTA, glass chips,
--t-section shelves, gutter 20/40); PosterCard kit (art-or-fallback, free badges radius 5,
rank medals — 1 solid heat / 2 heat stroke 2.5 / 3+ disabled stroke, numeral −14/−20 overlap,
titles below card, mono meta); Show page (art hero, title 38/36, meta dots, kit episode grid
1:1 5-up 8gap radius 11 with FREE/✓ OWNED/coin-locked states, act-tint lock cells, cast
avatars); Player (glass chrome everywhere, EP overline under --scrim-header, act progress bar
+ mono timestamps + EPISODES n/N, drawer with mono state column, 2400ms auto-hide); Wall
(fixed door order restyled to kit — coins neutral w/ price+balance in coin, unlimited hero
door w/ act border + door-hero gradient + BEST VALUE, earned full free treatment w/ dashed
SOON — bottom sheet <1024 / 860px centred modal ≥1024, "Back to the show" ghost always
visible); Admin login + forced change (two-tone wordmark + ADMIN overline, auth-glow, overline
field labels, inset inputs w/ act focus ring, eye-reveal mono when revealed, mockup copy);
tab bar/chips/empty-states (kit copy on My List) conformed. **DEFECT FOUND UNDER THE RESTYLE
(DL-B009 candidate → fixed in place):** the raw `<video>` element is invisible to
react-native-web's responder system — presses targeting it never reached the stage Pressable,
so tap-chrome-toggle and tap-to-hold were DEAD in browsers since v0.4 (masked because chrome
never auto-hid). With the package's 2400ms auto-hide that would strand the viewer chrome-less.
Fixed: DOM pointer listeners own the stage on web (tap toggle · 220ms hold-pause · touch
swipe), Pressable handlers kept for the native v0.5 path behind Platform guards. Verified
headless: chrome shows → auto-hides at 2400ms → tap returns it (mouse AND touch), hold pauses.
**Proof: API 147/147 (ephemeral PG17+Redis7) · web server 9/9 · Playwright admin-gate 4/4 —
zero behavior drift; screen-level visual pass against live data (8 surfaces × mobile/web
widths) eyeballed conformant.** No API change, no route change, no ledger touch.

## L-043 · 2026-07-30 · RESTYLE DEPLOYED LIVE — served == merged, regression-checked, conformance pairs delivered, STOPPED AT GATE-3
Web image rebuilt (`docker compose build web && up -d web`; container healthy). **Live through
the real edge:** served bundle hash == this tree (`entry-5fafbd2ddbad090f3152b807d563e2c7.js`
both sides) · Google Fonts (Gabarito / Plus Jakarta Sans / JetBrains Mono, preconnect+swap)
live at root · served-bundle secret grep **0 hits** · `/health` 200 and `/v1/app-config`
intact (API untouched — 0 lines) · admin face regression-checked (ops bare 401 · bad login
401 · gate e2e 4/4 on the shipped bundle) · viewer face regression-checked (anonymous
bootstrap + live catalog render, show-page SEO title injection intact, www→apex 301).
**Conformance for the Owner:** `docs/design-package/conformance/01…08-*.side-by-side.png` —
each briefed surface shot from the DEPLOYED site beside its package mockup (08 forced-change
drives the hash-identical bundle via a local harness: reaching that screen live would spend
the Owner's one-time temp credential, which remains UNTOUCHED for their GATE-3 first login).
Suites at close: **API 147/147 · web server 9/9 · Playwright admin-gate 4/4** — zero behavior
drift under the restyle. Docs advanced: DESIGN.md v2 (the binding contract) · BUILD-REPORT
(this engagement) · STATE. Spend, estimated (no meter on the box): **≈ $25 of the ≤ $40
ceiling.** Blocked: nothing. Commit + push `binjreel-dev` close the loop. **STOP: GATE-3 —
the Owner's viewing is the test; taste is the acceptance.**

## L-044 · 2026-07-30 · GATE-3 REDIRECT — CONFORMANCE COMPLETION: assets wired, 1440 implemented, surfaces conformed, DL-B009 fixed, two render-crashes + one hard-rule violation + one destructive-test hazard found, STOPPED AT GATE-3
**Contract:** `docs/design-package/…/project/HANDOFF.md` **in full** (superseding the prior
order's surface list) plus its `.dc.html` and `tokens.css`. Scope `app/web` only. The prior
restyle applied tokens onto surfaces whose art slots were all NULL and left desktop as
stretched mobile; this closes both.

**1 · ASSETS (the largest miss).** Ten `uploads/*.jpg` stills + `hero-poster.png` +
`BRBGBlurredVing.mp4` copied into `app/web/art/` and served BY THE WEB CONTAINER at `/art/*`
— never hotlinked out of `docs/`. Deliberately outside `dist/`: Metro never sees them, so real
art swaps in as a URL change, not a rebuild. Covers on both series, thumbnails on all 14
episodes, wired in `prisma/seed.ts` (`still()`, deterministic 1–10 wrap) so the picture is
reproducible rather than hand-patched. **Two server defects this exposed:** the MIME map had
no `.jpg/.jpeg/.webp/.mp4` — the browser got `application/octet-stream` and every slot rendered
empty; and there was no Range support, so the hero `<video>`'s opening range probe left it
stalled on its poster. Both fixed. `og:image` now resolved to absolute (a `/art/…` path is
unfetchable for an off-origin crawler). Traversal re-checked: `%2e%2e`, `..%2f`, `....//`
all fall through to the SPA shell, no source leaked.

**2 · RESPONSIVE WEB (1440).** The 72px header at ≥1024 carrying **the same five
destinations** the 84px tab bar carries below it — mounted once in the root layout so it is
identical across every browse route, and excluded from the player and `/admin`. Web hero: a
real `<video>` bed (`autoplay muted loop playsinline preload=metadata` + `poster`,
`object-fit:cover`), `--scrim-hero-web-x` + `--scrim-hero`, 600px copy column, the featured
9:16 card (240×427, clamped to keep 9:16 inside a 452–560 hero), and the Up Next rail. Player
gains the docked episode rail at desktop, dropping to the swipe-up drawer below 1024. Shelves
bleed past the right gutter (`paddingLeft: contentInset`, `paddingRight: 0`); content centres
at 1360; breakpoints 390/768/1024/1440 in `theme.ts`.

**3 · SURFACES.** Search → §9's three states (idle trending / results grid / no-results with a
live way onward) — it was still on v0.4 legacy aliases with no font families. Rewards → the §8
hub in its coming-soon state, two-column at web, **balance renders an em dash, not a fake 248**.
Profile → §8 with the 260/720 web split reusing the admin sidebar pattern; money moved from
tangerine to **coin gold**. For You → given actual art (it was rendering a title on an empty
`--bg-raised` panel: a poster slot with no poster). `/admin` panel → §5's system (260 sidebar ·
64 topbar · 28 gutter · 40px rows · tabular-nums), with only the sections that EXIST as nav.

**4 · HARD RULES swept explicitly** (table in the conformance README). **One violation caught:**
the show page's Follow button turned `--heat-text` when active — heat is never a button. Now
`--act-text`. `grep -rn "colors.heat" src/` now returns exactly two lines, both rank 1–2 medals.

**5 · DL-B009 — FIXED, two defects with traceback.** (a) `PosterCard` is itself a `Pressable`;
nesting it inside the Continue Watching `Pressable` meant the inner one won the gesture on web,
so **every tap landed on the show page, never the player** (fixed by a `nested` prop that
renders a plain `View`). (b) The card navigated without `?t=`, AND the player's mount-time
`POST /progress` posted that `0` back — so opening an episode **destroyed its stored mark**.
The player now learns `seriesId` from the prefetched `/next` payload instead of writing to find
out, falling back to the first real progress tick on a series' final episode. Verified live at
**390 and 1440**: lands on `/watch/{id}?t=6`, seeks, autoplays, and the resume pointer survives.

**6 · TWO DESKTOP RENDER CRASHES found by the mechanical pass** (both blanked the entire page at
≥1024, and both would have been invisible to an eyeball check of mobile): expo-router's
`<Link asChild>` forwards react-native's ARRAY-shaped `style` into a raw DOM anchor, and a
comma-separated two-shadow `boxShadow` string is parsed into an array by RN 0.86 — react-dom
assigns either index-wise to the CSSStyleDeclaration and throws. Nav is now
`Pressable + router.push`; the hairline ring is a real border.

**7 · THE SUITE COULD DESTROY PRODUCTION — found the hard way, then guarded.** `resetDatabase()`
TRUNCATEs every table including `tenants`, and running it via the `app` service inherits the
**LIVE** `DATABASE_URL`. Doing exactly that mid-gate wiped the deployed catalog, the operator
account and every media ref; `/v1/auth/anonymous` then 500'd on a dangling tenant FK and the
site could not mint sessions. **Restored:** reseeded, app restarted to re-resolve the tenant,
and the one real Bunny clip (`b56ffcb4-…`) re-attached to *Fired & Furious* ep 1 with the
suite's placeholder `bunny_ref`s cleared so no episode advertises media that 404s. **Guarded:**
`test/helpers.ts` refuses to truncate any database not named like a test database (override
`ALLOW_DESTRUCTIVE_DB_RESET=1`), proven to fail closed against `binjreel`; a compose `test`
service runs the suite against `${POSTGRES_DB}_test`. **`docker compose run --rm test` is now
the only way the suite runs.**

**8 · MECHANICAL VERIFICATION.** `web/scripts/conformance.mjs` screenshots the package's own
`.dc.html` frames as the reference halves and pairs them with **deployed** captures at **390
and 1440** → `docs/design-package/conformance-v2/` (11 pairs + halves + `UNPAIRED.txt`).
`web/e2e/assets-gate.spec.mjs` is the automated gate: no `<img>`/`poster`/`/art` background
resolves empty on **Home · Show · For You · Search** at both widths, a surface with no art at
all fails, and the hero bed's attributes + reduced-motion fallback are asserted. Real Chrome,
not bundled Chromium — the latter has no H.264, so the stage and the bed would both capture as
stills and "did it resume" is unobservable.

**Divergences recorded** (conformance-v2 README D-1…D-6): hero ha-chip carries **no number**
(no laugh-count field exists anywhere in `schema.prisma`; "numbers are facts, never bait", so
it is omitted, not faked) · no coin balance / "Go unlimited" in the header (economy chrome, no
economy yet) · the 20 MB **animated** webp is not wired to any surface (`hero-poster.png` is
the specified reduced-motion fallback) · cast headshots render initials (HANDOFF lists them as
still-needed, so no placeholder exists; they emit no `<img>`, so the gate is not weakened) ·
9:16 stills lose ~25% in 2:3 slots as HANDOFF warns · light theme not shipped (Owner-ratified).
**Deferred debt logged, not built:** Onboarding §9 → **v0.5** · Coins & plans admin §5 →
**v0.7** · the §5 screens with no product behind them (Dashboard, Shows table, Show editor,
Viewers, Moderation, Audit log) → with their features.

**NEEDS THE OWNER'S CALL — two residual artifacts** left in the LIVE database by that
destructive run, not created by the seed: a duplicate shelf `slug='new'` (visible as a second
identical "New on binjreel" on Home) and a draft series `Unannounced Pilot` (invisible in every
viewer read). Deleting production rows was **not** performed unilaterally; the exact SQL is in
the conformance README.

**Proof at close: API 147/147 (isolated `binjreel_test`) · web server 9/9 · admin-gate 4/4 ·
assets-gate 4/4 (new) · DL-B009 resume 2/2 (new, 390 + 1440). Served == merged
(`entry-35e50d3553d20602731c87338badcbf6.js` both sides).** Commit `5c612eb`, pushed to
`binjreel-dev`. **STOP: GATE-3 — the Owner's eyes remain the acceptance.**

---

## L-045 · 2026-07-31 · PUNCH-LIST PASS (Owner review of conformance-v2, waves 1–4) — recorded retroactively
**Why this entry exists.** The pass shipped as commit `9ed56fe` ("conform: punch-list pass — coin
mark, lime discipline, full-bleed player, restored components", on `binjreel-dev`, present on
`origin/binjreel-dev`) and was written up inside
`docs/design-package/conformance-v2/README.md`, but it never got a BUILD-LEDGER entry. Logged now
so the as-built trail has no hole between L-044 and GATE-3 close. Content transcribed from that
README; nothing re-verified in this order beyond the data checks in Part 1 below.

**Note on the wave count.** The README heads the section *"waves 1–4"* but carries explicit
sections for **Wave 1, Wave 2, Wave 3** and a closing *"Not done — and why"*. There is no `## Wave
4` heading. Recorded as found rather than invented — if a fourth wave was distinct, its record is
the "Not done" list.

**Wave 1 — cross-cutting (0.1–0.12).** Coin glyph unified as `components/CoinGlyph.tsx`, a DRAWN
rotated diamond rather than a codepoint (the old `◉`/`*` varied by font), wired through episode
cells, paywall door 1, player rail + drawer, Rewards balance, Profile stat and both headers' coin
pills, in `--act`/`--act-text`; `--coin` gold retained for admin table money · **lime ≠ disabled**:
`SoonPill`/`SoonButton` in `components/Chrome.tsx` become the only way to say coming-soon, on
`--surface-2` + `--text-secondary`, so Rewards' three earn cards and the wall's earned door go
neutral and lime returns only when a door actually opens · mono pulled back off prose (paywall
sub-line, player `Ep 1 · Episode 1`, `Episodes 1 / 6`) and kept for counts, timecodes, table
figures, overlines · `useCollapsingTitle` makes the bar title the H1's collapsed state, killing
duplicate titles; Home drops its stack header · one two-tone `Wordmark` at every width · 2-line
card titles at 1.25 with space for BOTH lines reserved so meta baselines align across a row ·
`components/TabIcon.tsx` draws five icons AND their labels, because react-navigation's web label
collapsed to ~4px regardless of navigator heights — `tabBarShowLabel` off, the tab item is one box
we own · tab bar pinned to 84 (50 content + 34 safe area) · shelf rhythm 28 mobile / 48 web,
title-to-first-card 12 · **catalogue seeded from 2 shows to ten** in `prisma/seed.ts` (82 episodes,
all ten stills in play, `freeEpisodeCount` deliberately varied INCLUDING shows with none), cards
left at 116×174.
**Two left open at the time:** the duplicate shelf (0.11 — needed the Owner's approval; closed in
this order, see below) and `-:--` duration (0.12 — only one fixture episode has encoded media, a
12.8s clip, and duration is read off the element: data, not chrome. **Still open.**)

**Wave 2 — restored components.** Paywall: UP NEXT card with still/overline/meta, an icon tile per
door, real buttons (`Unlock` / `Start 7 days free` / `Earn`) rather than disabled text-links,
`$6.99 / month` at 24/900 in `--act-text`, `Cancel anytime`, footer `EP 1–2 ALWAYS FREE, FOREVER`,
duplicate balance line deleted, glass ✕ over the paused frame, and the paused frame itself now
renders (a cold deep link gets the locked episode's own still under `--scrim-hero` instead of flat
black); web wall becomes a 3-column 860px grid centred on the VIEWPORT over a full-viewport scrim ·
Player: **full-bleed** (`object-fit: cover` — `contain` was letterboxing into black bands), back
chevron + show title + `Ep N · title` + ⋮ overflow, 56px social-rail circles, playback settings as a
horizontal pill row (`CC ON` / `1×` / `Auto` / `Share this moment`), 4px scrubber with a 14px knob,
grab handle + centred count, bottom scrim so white-on-video reads; at ≥1024 the mobile rail goes,
the stage centres 9:16 and the rail becomes a 380w `--surface-1` panel with 72h cards, 44×62 thumbs
and keyboard hints · Profile: identity row, 3-up stat row, §8.3 Free-plan card, both settings groups
(`Sign out` in `--heat-text`, the one sanctioned heat-on-text), build stamp · Search: scope tabs
with `Shows N` live and Episodes/People/Tags disabled rather than showing a count no endpoint can
answer, real 3-up grid, magnifier, clear ✕, `Cancel` · Rewards: tangerine on the BUTTON not the
card, real balance figure, notice moved out, icon tiles, 7-segment streak · Home: floating glass
header over the hero (wordmark · coin pill · 44px round search — the search FIELD is gone, search is
a destination), `--scrim-hero` + 80px foot fade, full-width 56h CTA + 56×56 `+`, genre chip rail as
its own row, `All`/`View all` + pager circles, `● LIVE` on heat shelves, `NO COINS` on free shelves,
continue-watching progress bar + `Ep N · Ms left`, badges capped at one with priority HOT > FREE >
NEW and suppressed on free shelves, 90/110px rank numerals overlapping the card, 16×4 active dot.

**Wave 3 — geometry.** Web nav restored to the wider set (Home · Comedy · Blowing up ● · Free ·
Rewards · My List) — and **`/trending` + `/free` were built so nothing links nowhere**, both reusing
existing reads (`/v1/top10`, the server's own free-to-binge shelf). Header gutter 40, search
180w/44h, coin pill closes §2.14's gap with the REAL balance. Hero title wraps to 3 lines at a 620
measure and never ellipsizes; the featured card's title is gone (§4.1 carries badges only); the
badge row is access-only so it no longer duplicates the meta line. Web `My List` ghost button added.

**Not done — and why (carried forward as debt, not silently dropped).** §2.11 / §3.9 / §9.3 /
§1.13's ha-rate metas — every one needs a ha/laugh count field that does not exist (D-1); omitted
rather than fabricated · §3.7 caption/quote block, §3.8 hint pill, §10.7 admin error toast — P3,
unbuilt, no data or no failure path exercised at this gate · §4.9 rail range tabs — unreachable at
12 episodes, required before a 20+ episode show · §8.2 `HA'S GIVEN` replaced with `ON YOUR LIST`,
which is a real figure · §10 admin login deltas (top-anchored lockup, `Keep me in` row, glow anchor,
52h inputs, r20) — deferred; it was the closest surface to spec and the budget went to P1s.

**Suites at that gate (as recorded, not re-run here):** API 147/147 (isolated database) · web server
9/9 · admin-gate 4/4 · assets-gate 4/4 · DL-B009 resume 2/2 (390 + 1440). Served == merged.

---

## L-046 · 2026-07-31 · v0.4 GATE-3 CLOSED — OWNER-CERTIFIED. Approved data cleanup executed; records caught up
**The gate is closed by the Owner, not by report.** v0.4 — the web face, carrying the restyle
(L-041…L-043) and the conformance completion + punch-list pass (L-044, L-045) — is **Owner-certified**:
- **Design accepted at both widths — 390 and 1440.** The taste acceptance GATE-3 existed for.
- **The forced admin password change was performed.** The seeded temp credential is retired by
  use, exactly as `mustChangePassword` was built to force. (Separately, an operator sign-in
  refusal was diagnosed the same day and closed as not-a-defect — `DEFECT-LEDGER.md` DL-B010.)

**Part 1 of this order — approved destructive cleanup, EXECUTED.** The Owner approved the exact SQL
recorded under "Residual data artifacts" in `docs/design-package/conformance-v2/README.md`. Both
artifacts were residue of the mid-gate truncate (L-044 §7), neither created by `prisma/seed.ts`.
Ran verbatim, in one transaction, against the live database:

```sql
DELETE FROM shelf_items WHERE shelf_id IN (SELECT id FROM shelves WHERE slug = 'new');
DELETE FROM shelves WHERE slug = 'new';
DELETE FROM series  WHERE title = 'Unannounced Pilot';
```

**Checked before firing, because the SQL deletes production rows.** `shelves` carries **no unique
constraint on `slug`** (PK on `id` only), and two rows shared the title "New on binjreel" at
`order_index` 2 — slugs `new` and `new-on-binjreel`. Confirmed against `prisma/seed.ts:423` that the
canonical slug is **`new-on-binjreel`** and that `new` appears nowhere in the seed, so the statement
targets the artifact and not the real shelf. Every FK into `series` is `ON DELETE CASCADE`, so the
third statement's reach was measured first: 1 episode, 1 shelf_item, 0 tags/cast/resume/follows/
list_items. A targeted `pg_dump --data-only` of the nine affected tables was taken as a rollback
point at `/tmp/binjreel-preclean-20260731.sql` — `admin_users` deliberately excluded, so no
credential material is in that file.

| table | before | after | Δ |
|---|---|---|---|
| `shelves` | 7 | **6** | −1 |
| `shelf_items` | 32 | **30** | −2 |
| `series` | 12 | **11** | −1 |
| `episodes` | 83 | **82** | −1 (cascade) |
| `series_tags` / `series_cast` / `series_resume` | 22 / 13 / 13 | **22 / 13 / 13** | 0 |

Reported deletions: `DELETE 2`, `DELETE 1`, `DELETE 1`. The draft series' single `shelf_item` sat on
the `new` shelf, so it was already removed by statement 1 — which is why `shelf_items` fell by 2 and
not 3.

**Post-cleanup verification (live, through Traefik):**
- **Home returns exactly one "New on binjreel"** — `slug=new-on-binjreel`, the seeded one. Shelf
  slugs now `new-on-binjreel · top-10 · comedy`, with `hero` returned under its own key.
- **`POST /v1/auth/anonymous` still mints** — **201**, 64-char token. (This is the call that 500'd
  on a dangling tenant FK during the L-044 truncate, so it is the right canary.)
- **Fired & Furious ep 1 still resolves its media** — `GET /v1/episodes/{id}/playback` → **200**,
  `reason: "free"`, a Bunny URL on `vz-fc89839c-dac.b-cdn.net` ending `playlist.m3u8`, signed with
  the D-016 **directory token in the path** (`bcdn_token` + `expires` segments, not a query string).
  Fetched the manifest itself: **HTTP 200, 776 bytes, `#EXTM3U`** — real HLS, not just a URL that
  parses.

**Reconciliation noted while counting.** 11 series remain but the punch-list says ten shows / 82
episodes. The eleventh is `Another Tenant Show` on the **`other-tenant`** tenant — the cross-tenant
isolation fixture, correctly invisible in every binjreel read. The binjreel tenant holds exactly ten
shows and 82 episodes. No discrepancy.

**Records caught up in this same order:** L-045 (the missing punch-list entry) · `DEFECT-LEDGER.md`
DL-B009 (Continue-Watching tap, both halves) and DL-B011 (the suite could truncate production),
both transcribed from L-044 and marked CLOSED.

**Not touched, by order:** `STATE.md`, `TDD.md`, `DECISIONS.md`, `/docker/PROJECTS.md` — the
Architect advances those and delivers them through `cubby/`. No code, no features, no deploy in this
order.

---

## L-047 · 2026-07-31 · DOCUMENT PLACEMENT — the v0.5 drop completed: stage contract, D-017…D-023, STATE baton; TDD citations reconciled
**Document placement only.** No code, no features, no deploy, no build. The v0.5 build is a
separate order and was not opened.

**Why this entry has two halves.** The drop arrived in two parts. The **TDD advance landed
first** (five Architect-authored edits: target line v0.3→v0.5, §11(4) v0.4 row to
Owner-certified, §11(5) rewritten as the live target, §9.2 superseded by D-023, §8 gaining the
D-022 pricing block). That left `TDD.md` citing a stage contract and two decisions **that were
not yet on the box** — `docs/CONTEXT-v0.5.md`, D-022 and D-023 all dangled. The TDD edit was
therefore **deliberately held uncommitted** until the rest of the set arrived, so the record was
never published in a self-contradicting state. This entry closes that window: the citations now
resolve and the whole set is committed as one.

**Part 1 — the two dropped citations restored.** The §11(4) As-built line had been reduced to
`L-036…L-046 · conformance-v2 · DL-B009/B011 closed` when the new As-built replaced the old one
during the TDD merge, dropping the `BUILD-REPORT` and `SECURITY-REPORT v0.4` references. Both are
back:
`*As-built: L-036…L-046 · BUILD-REPORT · SECURITY-REPORT v0.4 · conformance-v2 · DL-B009/B011 closed.*`
Nothing else in the TDD was changed by this order.

**Part 2 — three files placed from `cubby/`.**
| From | To | Mode | Result |
|---|---|---|---|
| `cubby/CONTEXT-v0.5.md` | `docs/CONTEXT-v0.5.md` | new file | placed; **no existing CONTEXT touched** — `CONTEXT.md`, `-v0.1`, `-v0.2`, `-v0.4` all retain their original mtimes, v0.4's remaining the prior stage's as-ordered record |
| `cubby/DECISIONS-append-v0.5.md` | `docs/DECISIONS.md` | **append-only** | D-017…D-023 added |
| `cubby/STATE.md` | `docs/STATE.md` | overwrite | the baton rewritten to current truth |

**Append-only proven, not asserted.** `DECISIONS.md` was 246 lines / sha256 `911220416ecbe8f6`
before. After the append, the **first 246 lines hash to exactly that same value** — the prior
record is byte-identical, nothing rewritten, renumbered or reordered. File is now 339 lines.
Numbering verified whole: **D-001…D-023 each present exactly once, zero duplicates** (16/16
pre-existing intact, 7/7 new).

**`cp` + `rm`, never `mv` — and the inode proves it.** `mv` would replace the destination inode
and leave the file root-owned, reintroducing the DL-B004 cross-ownership failure where the
Architect cannot rewrite a file the Worker created. `docs/STATE.md` kept its **original inode**
through the overwrite. Ownership then set to **1000:1000 on every touched file** —
`CONTEXT-v0.5.md`, `DECISIONS.md`, `STATE.md`, `TDD.md`, `BUILD-LEDGER.md`. Two of those
(`STATE.md`, `TDD.md`) had been sitting `0:0` and would have been unwritable by the Architect;
they are not now. The three cubby copies were removed; the unrelated
`cubby/Rules for a video - Sheet1.csv` was left alone.

**Part 3 — reconciliation. Every TDD reference now resolves:** `docs/CONTEXT-v0.5.md` exists ·
D-022 present in `DECISIONS.md` · D-023 present in `DECISIONS.md` · the `HANDOFF.md` governing
visual contract resolves in the design package. The incoming `STATE.md` was **identity-checked
before it was allowed to overwrite** — STATE's own DL-010 warning records a bare `TDD.md` from
another project being uploaded over binjreel's by name collision, so this is not a formality. It
passed all four `PROJECTS.md` phrases: `binjreel` · `binjreel-dev` · `papamg/binjreel` ·
multi-tenant-ready SaaS at audit tier high. It also **resolves W-002** — credential scope
account-wide, proven read and write against a second `papamg` repo, with the per-repo lookup
caveat carried forward as backlog.

**Raised, not acted on (the Architect owns `/docker/PROJECTS.md`).** The new STATE describes the
product as *"a vertical micro-series platform for every genre, comedy first, including
micro-docs. Content by Journey Viral"*. That is a **third** descriptor: the registry says
*Short-Form Vertical Drama Platform* and carries a 2026-07-25 reconciliation note blessing that
phrase and *Short Form Video Delivery Platform*. Per that note a name mismatch alone is **not** a
WRONG-PROJECT signal and stopping on it would be a false stop — so placement proceeded. But the
registry's identity-check phrase list has now drifted from the STATE it is meant to validate, and
only the Architect can close that.

**Published.** The docs repo (`papamg/binjreel-docs`, established L-046 / the credential run) was
committed and pushed with the TDD advance and this placement **as a single consistent commit**.
