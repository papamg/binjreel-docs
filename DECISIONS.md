# DECISIONS.md — binjreel

*Append-only. Every decision taken on this project, with the reasoning that settled it and
who settled it. Determinable decisions are taken, verified, and logged here closed-loop —
they are not escalated (`procedures/decision-routing.md`). Only the four triggers — spend ·
scope/behaviour change · irreducible risk · strategic fork — rise to the Director as a CSW
packet (`procedures/four-item-escalation.md`).*

*Doctrine is canon (`/docker/v2s-canon/_v2s/`), not this file. This is operational state.*

---

## D-001 · 2026-07-25 · Repo root at `app/`, docs as a sibling · Worker
**Decision.** The real git repo root is `/docker/binjreel/app/`; `/docker/binjreel/docs/` sits
outside it as a sibling.
**Why.** Blueprint §14's as-built fleet layout, and it is what makes the wall structural rather
than procedural: the Architect's bridge is scoped to `/docs`, and because `/docs` is not inside
the repo, no bridge misconfiguration can expose `app/` source. Confirmed:
`git rev-parse --show-toplevel` → `/docker/binjreel/app`.
**Consequence.** The load-bearing binding is the committed `app/CLAUDE.md`; the bare
`/docker/binjreel/CLAUDE.md` covers the `/docs` subtree the repo copy cannot reach.

## D-002 · 2026-07-25 · Provisional stack: Node 22 · TypeScript · Express · Worker
**Decision.** The standup scaffold is Node 22 + TypeScript + Express. Recorded as
**provisional**, not settled architecture.
**Why.** `PROJECTS.md` and a working Dockerfile both require *some* stack, but no `TDD.md` or
`IDEAL-SCENE.md` is staged, so no design authority exists yet for this choice. Node was chosen
for the least-commitment reason available: it is the same toolchain the MCP bridge needs, so the
box carries one runtime instead of two at standup.
**Explicitly reversible.** The Architect's first real `CONTEXT.md` settles the product stack. A
short-form video delivery platform may well want a different runtime for transcode/streaming
paths; nothing in this standup should be read as having pre-empted that. Replacing the scaffold
is expected, not a regression.

## D-003 · 2026-07-25 · Tenant subdomains not routed yet — HTTP-01 cannot issue wildcards · Worker
**Decision.** The app router matches the apex and `www` only. `*.binjreel.com` is deliberately
**not** routed.
**Why.** `/docker/traefik/traefik.yml` configures the `letsencrypt` resolver with an
`httpChallenge`. An HTTP-01 challenge proves control of one hostname at a time and **cannot
issue a wildcard certificate**; Let's Encrypt requires DNS-01 for `*.domain`. Routing
`*.binjreel.com` now would attach a router to a certificate that can never be issued, producing
TLS failures on the exact per-tenant hostnames a multi-tenant SaaS depends on.
**Consequence — flagged for the Architect, not silently deferred.** If tenants are to be
addressed as `tenant.binjreel.com`, Traefik needs a **DNS-01** resolver (registrar API
credentials — a Director secret act) before the first tenant ships. If tenants are addressed by
path or by header instead, no change is needed. That is a design fork the TDD should decide
knowingly; it is recorded here so it cannot arrive as a surprise at deploy. Not escalated as a
CSW packet — nothing is blocked at standup and the decision belongs with the design, not ahead
of it.

## D-004 · 2026-07-25 · MCP bridge: own runtime, `/docs` only, no host port · Worker
**Decision.** The bridge is a small first-party Node service in `/docker/binjreel/mcp-bridge/`,
mounting **only** `/docker/binjreel/docs` (read-write) and nothing else. It publishes no host
port and is reachable only across `traefik-public`.
**Why.** The wall (Blueprint §13) is enforced by the *mount*, not by application logic — the
container has no filesystem path to `app/`, to another project, or to
`/docker/traefik/acme.json`, so no bug or prompt can reach them. Path containment inside the
service is a second layer for symlink/traversal attempts within `/docs` itself, not the primary
control. A generic filesystem MCP server pointed at a parent path was rejected: §13 forbids
consolidating bridges to a parent path, because the narrow scope *is* the wall.

## D-005 · 2026-07-25 · `github-pat` declared as a consumed slot, not placed per-project · Worker
**Decision.** `SECRETS.md` declares `github-pat` as a **box-level** slot this project
*consumes*. The Worker does not place a per-project git credential.
**Why.** `procedures/secret-manifest.md`: a `git-credential` on a host-only-keyed box is shared,
and a per-project placement silently overwrites the box credential, breaking every other repo.
This box does have `credential.useHttpPath=true` set globally, and the canon clone at
`/docker/v2s-canon` fetches successfully under it — which implies the stored credential is
already path-keyed, so a path-keyed `papamg/binjreel` entry would not in fact collide. The
manifest still declares the conservative form: the safety of adding an entry is the Director's
to confirm at placement, and the failure mode of guessing wrong is fleet-wide.

## D-006 · 2026-07-25 · Domain `binjreel.com` · Director
**Decision.** App on `binjreel.com` (+ `www`), MCP bridge on `docs.binjreel.com`.
**Why.** Director-supplied at standup. Held as a single `DOMAIN` variable in each service's
`.env` rather than hardcoded in compose, so the hostname is one edit in one place.
**Open.** The DNS records themselves are a Director act and are not yet pointed.

## D-007 · 2026-07-25 · Canon's "Owner" and this fleet's "Director" reconciled explicitly · Worker
**Decision.** The binding carries canon's template text verbatim (which says *Owner*) and adds
one line naming the Director as the same seat.
**Why.** Canon's live template says "Owner's merge / Owner's gates"; IODE's breadcrumb and this
project's `CONTEXT.md` say "Director". Same person, two labels. Silently rewriting canon's
template would put the project out of step with the source it is supposed to track; leaving the
mismatch unexplained reads as two different authorities. Naming the equivalence keeps the
template faithful and the meaning unambiguous.

## D-008 · 2026-07-25 · Bridge writes are atomic (temp-file + rename), not in-place · Worker
**Decision.** `write_doc` and `append_doc` write to a temp file in the target directory and
`rename()` it over the target, rather than opening the target for writing.
**Why.** Forced by DL-B004: the bridge runs as uid 1000 and the Worker writes `/docs` as root, so
the bridge can never open a Worker-created file for writing — which would have broken the
Architect's required session-close rewrite of `STATE.md`. `rename()` needs permission on the
*directory* (owned by uid 1000), not the file, so it works across both writers with no `chown`
ritual to forget. Rejected alternatives: running the bridge as root (throws away the hardening to
fix a permissions bug), and chown-after-every-write (a ritual whose failure is silent until an
Architect session close).
**Bonus, and the reason to keep it even if ownership changed.** `rename()` is atomic, so a crash
or a full disk mid-write cannot leave a half-truncated record — and these files *are* the
project's record.
**Invariant this creates:** `/docker/binjreel/docs` must remain owned by (or writable by) uid
1000. That is one directory-level fact set once, not a per-file obligation.

## D-009 · 2026-07-25 · MCP bridge kept provisional; IODE remote-write is the real fix · Director (CSW-approved)
**Decision.** binjreel's per-project MCP bridge stays running as the provisional chat-side write
path to this box's docs. It is **not** torn out now. The permanent fix is a separate IODE
capability — remote write/dispatch to fleet boxes — which, once built, retires binjreel's bridge,
JVScope's, and every future remote bridge in one move.
**Why.** IODE's write actuators (`write_file`, `run_in_claude_code`) resolve to the box IODE runs
on (Tone); `box_read_file` reads the whole fleet but there is no remote write. binjreel is on
Journey 2 (remote), so from the cockpit IODE can read it but not write it. Projects local to IODE
(e.g. StoryMachine on Tone) correctly need no bridge; remote-box projects (JVScope on Journey)
currently do. Killing binjreel's bridge today would strand the Architect's write path while leaving
the identical JVScope bridge standing anyway — a cosmetic change, not the one-cockpit end state.
**Consequence.** Until IODE remote-write ships, Architect-authored docs reach this box by
Director-run `scp` (verified working 2026-07-25). The IODE remote-write request and the three
canon-gap items (DL-B001/B002/B003) route through the Tone canon drop → PR → Director ratify.

## D-010 · 2026-07-25 · Multi-tenant-ready from the first commit, single-tenant in practice · Director
**Decision.** binjreel is built **tenant-aware from day one** — every table carries `tenant_id`,
every content/wallet query is tenant-scoped — but **runs as a single tenant (us) in v0.1**. No
tenant-management UI, no client onboarding, no per-tenant billing until a real second client
exists. One tenant is seeded; `tenant_id` resolves from a single configured default.
**Why.** Director confirmed the platform will "ultimately" carry multiple content clients, "but it
will be us for now — build it out for future use." Threading `tenant_id` through the schema is
nearly free at the foundation and painful to retrofit; the machinery that sits on it (onboarding,
per-tenant billing, tenant UI) costs real work and has nothing to justify it yet, so it waits.
This confirms the direction the TDD already designed for; nothing in the design changes.
**Consequence.** v0.1 CONTEXT builds the column and the query-scoping now; the second-tenant
machinery is explicitly out of scope until earned. D-003's path/header tenant addressing (not
subdomain) is the mechanism, needing no Director secret acts.

## D-011 · 2026-07-25 · The ledger is append-only in the DATABASE, with no exception · Worker
**Decision.** `wallet_entries` carries triggers that reject `UPDATE` and `DELETE` outright. There
is no privileged bypass, no session flag, no "internal only" path. The anonymous→real account
merge — the one operation that legitimately moves a wallet between accounts — is expressed as
**paired compensating entries**: a debit on the source and a matching credit on the destination,
one pair per (coin class, expiry) bucket.

**Why.** TDD §4.2 calls the ledger append-only. In application code that is a promise every
current and future call site has to keep, and it is one bug, one migration script, or one
hand-run `psql` session away from being broken silently. In the database it is enforced once.

**The alternative that was written and then rejected.** The first merge implementation did
`UPDATE wallet_entries SET user_id = ...`, which the trigger blocks, so it carried a
`set_config('binjreel.allow_ledger_merge')` escape hatch the trigger would honour. That was cut
before it ever ran. An append-only ledger with a privileged bypass is not append-only — it is an
append-only ledger plus a documented way around it, and a documented way around it is precisely
what a future caller finds, reuses for something plausible, and thereby normalises. Compensating
entries need no exception, keep both accounts' histories literally true, and are how a ledger is
supposed to express a movement in the first place.

**Bucket-by-(class, expiry), not a single lump sum.** Moving one net balance would launder a
reward coin's expiry into a permanent coin. v0.1 has no reward-earning surface, so this is
currently theoretical — which is exactly why it was worth getting right now rather than
discovering it when v0.5 turns rewards on.

**Consequence.** Correcting a balance is only ever done by appending a compensating entry. Test
suites reset with `TRUNCATE`, which does not fire row-level triggers, so the guarantee costs the
tests nothing. Conservation across a merge is asserted in the test suite.

## D-012 · 2026-07-25 · Idempotency keys do not prevent double-spend; a row lock does · Worker
**Decision.** Every wallet mutation takes `SELECT … FOR UPDATE` on the user row before reading
the balance it depends on, and re-checks the idempotency key **inside** the lock.

**Why.** These are two different failure modes and the first one's fix does nothing for the
second. An idempotency key makes a **replay** of one act harmless — the same key collides. It
says nothing about two **different** acts interleaving: without a lock, five concurrent 30-coin
spends against a 100-coin balance all read a sufficient balance and all five commit. The account
goes to -50 and every individual operation looks correct in isolation.

The key is also re-checked after the lock is granted, not only before: another transaction can
insert that key between the first read and the lock, which is a TOCTOU window on money.

**Why this is written down.** The bug is invisible to single-threaded testing — the ordinary
suite passes perfectly with the lock removed. It is proven by a dedicated concurrency test that
fires five simultaneous spends and asserts both that at most three succeed and that the final
balance agrees exactly with the number that did. A future refactor that "simplifies away" the
lock will pass every other test in the suite.

**Consequence.** All wallet writes serialise per account, which is the correct granularity — two
different viewers never contend. Merge locks both accounts in sorted id order so two merges
touching the same pair cannot deadlock.

## D-013 · 2026-07-27 · Master storage speaks the S3 API directly — SigV4 in ~200 audited lines, no vendor SDK · Worker
**Decision.** The master-storage client (`src/media/storage.ts`) implements AWS Signature V4
itself against any S3-compatible endpoint, rather than pulling in a vendor SDK.
**Why.** Three reasons, in weight order. (1) TDD §2 makes vendor-swappability the point of owning
the masters — the S3 API is the de-facto portable surface (AWS, B2, Wasabi, R2, Hetzner, MinIO all
speak it), so coding to the protocol rather than a vendor's SDK keeps the swap a config change.
(2) The fleet's secret posture wants ONE auditable module per external credential; an SDK is a
large opaque dependency sitting exactly on the credential path (and this build's `npm audit`
cleanliness — 0 across 6 prod deps — is worth defending). (3) The client needs exactly two
operations (signed HEAD, presigned GET); SigV4 for two operations is small, stable, documented.
**Consequence.** The client is deliberately narrow: there is NO method that returns an unsigned
URL, and presign TTL clamps at 3600s in code. If a future version needs multipart upload or
listing, revisit — that is the point at which an SDK earns its weight.

## D-014 · 2026-07-27 · The media-admin surface is gated by a box-held operator token, not by viewer sessions · Worker
**Decision.** Master registration and media-state reads require `x-media-admin-token`, compared
constant-time against the `media-admin-token` slot; empty slot = the surface answers a uniform
401 to everything.
**Why.** CONTEXT v0.2 orders "a path to register a master" but names no principal for it, and
v0.1 has exactly one authentication class: viewer sessions. Registration triggers outbound vendor
calls and rewrites media state — an operator act. The three options were: ship it open
(indefensible), hang it on viewer sessions (every anonymous signup could re-point an episode's
master), or gate it on a dedicated box-held secret. The third is the minimal honest gate that
does not build v0.4's role system early.
**Consequence.** Logged as ADDED (built beyond CONTEXT's letter, within its intent). The token is
an interim: v0.4's studio-admin roles replace it, at which point the slot is retired. The uniform
401 (empty slot, missing header, wrong token — one identical body) is load-bearing: it keeps the
surface unprobeable, and a test pins it.

## D-015 · 2026-07-27 · The Bunny callback authenticates by a query-string secret — a vendor constraint, taken with mitigations · Worker
**Decision.** `POST /v1/media/bunny/callback?token=<bunny-webhook-secret>`, constant-time
compare, fail-closed when the slot is empty.
**Why.** Bunny Stream webhooks can carry no custom header and are not signed, so a URL-borne
shared secret is the strongest authentication the vendor offers. The alternative — accepting
unauthenticated callbacks and "verifying" by re-querying Bunny's API — would make our media
state machine drivable by anyone who can name a GUID, rate-limited only.
**Consequence & mitigations.** A URL-borne secret transits request logs, so: (1) every URL that
reaches a log line passes `redactSecretParams()` first — request serializer AND the
access-refused audit path (tested); (2) the secret authenticates ONLY media-state transitions —
never money, never entitlements; a holder can at worst flip episode media states (integrity/
availability, bounded); (3) library id is checked even with a valid token; (4) rotation is free.
Recorded as SECURITY-REPORT F-003 (fixed: log redaction) + N-005 (the residual, accepted).

## D-016 · 2026-07-27 · Playback signing is the DIRECTORY-scoped HS256 token per Bunny's reference implementation · Worker
**Decision.** `signPlaybackUrl` signs `token_path = /{videoGuid}/` (never a single file) with
Bunny's current scheme — `"HS256-" + base64url(HMAC-SHA256(tokenAuthKey, token_path ‖ expires ‖
signingData))` where `signingData` includes `token_path` — emitted in the path-based
`/bcdn_token=…` URL form. **Supersedes** the v0.2 signer's original single-file
`sha256(key+path+expires)` construction AND that signer's comment asserting directory scope came
for free (DL-B006: it did not; HLS died at the edge).
**Why.** (1) HLS is a tree of files under `/{guid}/`; only a directory token covers what the
manifest references. (2) The edge requires `token_path` inside the signed data — proven by live
contrast (with: 200; without: 403). (3) Between the two accepted token forms (legacy sha-concat
and HS256 HMAC — both verified live), the HMAC form is the vendor's current documented scheme
and the reference implementation's output; the legacy form is what the docs now mark superseded.
(4) The path-based URL form means relative segment resolution inherits auth with zero player
cooperation — no query string to propagate.
**Consequence.** The signed scope is one video's directory for seconds — strictly wider than one
file, still uselessly narrow to an attacker without a fresh mint. Unit tests recompute the
reference algorithm independently; the live edge proof is the regression oracle at every future
GATE-3. URLs are not IP-locked (mobile clients roam networks mid-session) — revisit only with
data. Open on the Director: the zone's empty-referer 403 (DL-B006 discovery) before v0.3.
