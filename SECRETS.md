# SECRETS.md — binjreel (the secret manifest)

*Per `/docker/v2s-canon/_v2s/procedures/secret-manifest.md`. This file **declares** which
secrets this project needs and where each one lives on the box. It **never holds a value.***

**Three roles, one artifact.** The **Architect** authors and maintains this manifest (it is
operational state in `/docs`). The **Director** places the values through the dedicated secret
path. The **Worker** reads it to know what the project expects. *Authored by the Worker at
Phase 1.3 standup because no Architect session has opened on this project yet; it passes to the
Architect at the first CONTEXT.*

**Invariants.**
- **No value, ever.** No secret value appears here, anywhere else in `/docs`, or in git. This
  file is read over the read-write bridge; the values never cross it.
- **Destination is one of the closed set** — `git-credential` · `env` · `service-env` ·
  `ssh-key`. Never an arbitrary path.
- **Status lifecycle:** `empty` → `filled` (Director placed it) → `verified` (a read-only auth
  check passed with **no value printed** — e.g. `git ls-remote`, an API identity call).
  A slot reaches `verified` only by observing that the secret *works*, never by reading it back.

**Origin remote this manifest binds to:** `https://github.com/papamg/binjreel.git` — wired at
standup (`git remote -v` confirms), per Impl. Guide 1.3 and DL-001. Wiring the remote is standup
infra, not a first-deploy step, precisely so secret intake has a defined slot to bind to.

---

## Slots

### `github-pat` — **BOX-LEVEL SHARED · consumed, not placed by this project**
| | |
|---|---|
| **Destination** | `git-credential` (git's secure credential store, keyed to the origin remote host) |
| **Location on box** | git credential store, keyed `github.com` (+ path, see note) |
| **Placed by** | Director |
| **Status** | **`verified` for `papamg/binjreel` — 2026-07-26, by observing it work.** Push+delete of a throwaway ref both returned exit 0 (`* [new branch]` / `- [deleted]`), remote left byte-identical at `main` `ec2ef3d1`. A companion **box-level read path to `papamg007/v2s-canon`** is likewise verified — `ls-remote` forced a network answer matching local HEAD `834fc98`. Evidence: `BUILD-LEDGER.md` **L-022**. *(Prior state, 2026-07-25 18:19 → 2026-07-26: `filled`, NOT `verified`, non-functional — a single **host-only** entry on a `usehttppath=true` box, matching nothing and taking canon read down with it. Diagnosis `DEFECT-LEDGER.md` **DL-B005**; corrected by the Director to the path-keyed model.)* **Scope caveat — this slot is verified per-repo, not per-account:** under `useHttpPath=true` the entry matches `papamg/binjreel` only, and no sibling `papamg` repo has been tested, so the "account-level" framing is **not yet observed**. Tracked as `STATE.md` **W-002** |
| **Needed for** | `git push` of `binjreel-dev` and `main`; the deploy pipeline; the Director's IODE portal binding to this project |

**Scope warning (secret-manifest.md, "Slot scope").** This is a **shared** slot. This project
must **not** place its own `github-pat`: on a host-only-keyed box that overwrites the box
credential and breaks every other repo on it. binjreel *declares and consumes* the shared slot.
*Observed nuance:* this box sets `credential.useHttpPath=true` globally and the canon clone
fetches successfully under it, which implies the stored credential is already path-keyed — so a
path-keyed entry for `papamg/binjreel` would not collide in practice. The Director confirms that
at placement; the manifest keeps the conservative declaration because the blast radius of
guessing wrong is fleet-wide (DECISIONS.md D-005).

**Blocked on:** the `papamg/binjreel` repo does not exist yet. Create the repo, then place the
PAT, then `verified` is reachable.

### `mcp-bridge-token`
| | |
|---|---|
| **Destination** | `env` — `BRIDGE_TOKEN` in `/docker/binjreel/mcp-bridge/.env` |
| **Location on box** | `/docker/binjreel/mcp-bridge/.env` (mode 600, outside the git repo) |
| **Placed by** | Worker at standup (see the note below) · **rotatable by the Director at any time** |
| **Status** | **`verified`** — a 32-byte random value is in place, and the bridge was observed to *accept* it and to *reject* both a missing header and a wrong token (401). No value was read back or printed |
| **Needed for** | Authenticating the Architect's connector to the read-write docs bridge. `Authorization: Bearer <token>` |

**Why the Worker placed this one, when the manifest says values are Director-placed.** This is
not a credential to any external authority — it is a locally-generated shared secret with
exactly one purpose: preventing an unauthenticated read-write bridge to the project record
layer from being reachable on the public internet. Leaving it empty until a Director session
could place it would mean either a bridge that serves nothing (so the ordered health check
could not be honestly confirmed) or, far worse, a bridge with the gate switched off. So it was
generated on the box, at mode 600, outside `/docs` and outside git. **The bridge is
fail-closed:** with `BRIDGE_TOKEN` empty it answers every `/mcp` request `503` rather than
serving the docs layer open. Rotate with `openssl rand -hex 32` and restart the bridge; nothing
else depends on the value.

### `anthropic-api-key`
| | |
|---|---|
| **Destination** | `env` — `ANTHROPIC_API_KEY` in `/docker/binjreel/app/.env` |
| **Location on box** | `/docker/binjreel/app/.env` (mode 600, gitignored) |
| **Placed by** | Director |
| **Status** | `empty` — declared, slot present and empty. Correct at standup |
| **Needed for** | Any Claude-using path in the product. Baseline slot for a Claude-using app per the Customization Pass; **not yet required** — nothing in the standup scaffold calls Claude |

---

### `postgres-credentials` — **placed by the Worker at v0.1 build**
| | |
|---|---|
| **Destination** | `env` — `POSTGRES_USER` / `POSTGRES_PASSWORD` / `POSTGRES_DB` in `/docker/binjreel/app/.env` |
| **Location on box** | `/docker/binjreel/app/.env` (mode 600, gitignored, outside `/docs`) |
| **Placed by** | Worker at build · **rotatable by the Director at any time** (`openssl rand -hex 24`, update `.env`, `docker compose up -d`) |
| **Status** | **`verified`** — a 24-byte random value is in place and the app was observed to connect, migrate, and serve `/health` through it. No value was read back or printed |
| **Needed for** | The control plane's database — users, the coin ledger, entitlements, subscriptions |

**Why the Worker placed this one, on the same reasoning as `mcp-bridge-token`.** It is not a
credential to any external authority: it is a locally-generated secret for a Postgres instance
that publishes **no host port** and sits on a Docker network marked `internal: true`, reachable
only by the app container. The alternative — leaving it empty until a Director session — is a
stack that cannot start, so the ordered health check could not be honestly confirmed. Generated
on the box, mode 600, outside `/docs` and outside git.

### `apple-asn-public-key`
| | |
|---|---|
| **Destination** | `env` — `APPLE_ASN_PUBLIC_KEY` in `/docker/binjreel/app/.env` |
| **Location on box** | `/docker/binjreel/app/.env` (mode 600, gitignored) |
| **Placed by** | Director, when the Apple integration goes live (v0.2) |
| **Status** | `empty` — declared, slot present and empty. **Correct today** |
| **Needed for** | Verifying App Store Server Notifications V2 (ES256 JWS). PEM SPKI format |

**EMPTY IS SAFE AND IS FAIL-CLOSED.** With this slot empty the endpoint exists and **refuses
every notification** — it does not accept unverified money. Proven by test (SECURITY-REPORT,
Auth & Access). v0.1 verifies against a configured public key; Apple production signs with an
`x5c` certificate chain validated to Apple's root CA, which is the v0.2 upgrade behind the same
interface.

### `google-rtdn-public-key`
| | |
|---|---|
| **Destination** | `env` — `GOOGLE_RTDN_PUBLIC_KEY` in `/docker/binjreel/app/.env` |
| **Location on box** | `/docker/binjreel/app/.env` (mode 600, gitignored) |
| **Placed by** | Director, when the Google Play integration goes live (v0.2) |
| **Status** | `empty` — declared, slot present and empty. **Correct today** |
| **Needed for** | Verifying the Pub/Sub push OIDC token on Google Play RTDN. PEM SPKI format |

**Same fail-closed posture as Apple.** The RTDN message body is not signed, so the push token
*is* the authentication — with no key configured, every notification is refused.

---

## Slots NOT needed, recorded so their absence is deliberate

- **Session/token signing secret — not needed.** Sessions are opaque 32-byte CSPRNG tokens stored
  server-side in Redis (as SHA-256 digests), not signed JWTs. There is nothing to sign, so there
  is no signing key to hold, rotate, or leak. Recorded because the Customization Pass anticipated
  one and its absence is a design consequence, not an oversight.
- **Per-tenant encryption keys / KMS — not needed in v0.1.** Isolation is enforced by row scoping
  and server-side tenant resolution, not by per-tenant cryptography. Revisit if a tenant ever
  requires key separation at rest; nothing here forecloses it.
- **Traefik DNS-01 registrar credentials — not needed.** TDD §3 settled D-003 as path/header
  tenancy, not `tenant.binjreel.com`, so no wildcard certificate is required and no registrar API
  credential is a Director act.

## v0.2 media slots — declared at build, **every one fail-closed while empty**

*The v0.1 "class slots expected at v0.2" placeholder is superseded by the concrete declarations
below. All six are `env` slots in `/docker/binjreel/app/.env` (mode 600, gitignored). The
`.env.example` template documents each key. Behaviour with the slot empty is PROVEN by test
(SECURITY-REPORT v0.2 pass), not asserted: playback answers `503 media_not_configured` and mints
nothing; registration refuses; the callback rejects everything. **These are the GATE-3 keys — the
one new spend v0.2 introduces (a Bunny account) plus our own bucket.***

### `bunny-stream-api-key`
| | |
|---|---|
| **Destination** | `env` — `BUNNY_STREAM_API_KEY` |
| **Placed by** | Director, at GATE-3 (requires the Bunny account — a spend) |
| **Status** | `empty` — declared, slot present and empty. **Correct today** |
| **Needed for** | Ingest/transcode calls (`AccessKey` header). Pairs with the non-secret `BUNNY_STREAM_LIBRARY_ID` |

### `bunny-token-auth-key`
| | |
|---|---|
| **Destination** | `env` — `BUNNY_STREAM_TOKEN_AUTH_KEY` |
| **Placed by** | Director, at GATE-3 |
| **Status** | `empty` — declared, slot present and empty. **Correct today** |
| **Needed for** | Signing playback URLs (CDN token authentication). **THE media credential:** whoever holds it can mint working URLs for any video in the pull zone. Pairs with the non-secret `BUNNY_STREAM_CDN_HOST` |

### `bunny-webhook-secret`
| | |
|---|---|
| **Destination** | `env` — `BUNNY_WEBHOOK_SECRET` |
| **Placed by** | Director at GATE-3 (`openssl rand -hex 32`; also set in the Bunny webhook URL: `https://binjreel.com/v1/media/bunny/callback?token=<value>`) |
| **Status** | `empty` — declared, slot present and empty. **Correct today** (callback endpoint refuses everything) |
| **Needed for** | Authenticating the transcode-completion callback. Compared constant-time; the token is REDACTED from request logs (F-003). Rotate freely: update `.env` + the Bunny webhook URL, restart |

### `media-admin-token`
| | |
|---|---|
| **Destination** | `env` — `MEDIA_ADMIN_TOKEN` · **two containers as of v0.4:** the API (the gate itself) and the WEB container (the /admin ops proxy attaches it server-side). It never reaches a browser — proven by test (bundle scan) and live (served-bundle grep) |
| **Placed by** | Director at GATE-3 (`openssl rand -hex 32`) — local shared secret, no external authority |
| **Status** | `placed` (v0.2 GATE-3). Web container reads the same `.env` slot; empty = the /admin ops proxy answers 503 (fail-closed) |
| **Needed for** | The operator gate on master registration, media-state reads, and the v0.3 merchandising/schedule/tag writes (header `x-media-admin-token`). A viewer session can never open it; an ADMIN session reaches it only THROUGH the web proxy's allow-list. Retired at v0.7's full roles |

### Admin identity (v0.4) — recorded, not a slot
The v0.4 Order said to declare an "admin-session signing secret" slot. **DIVERGENCE, recorded
here and in the BUILD-REPORT:** admin sessions are server-side opaque 256-bit tokens in Redis
(SHA-256-stored, TTL'd, revoked wholesale on password change) — the viewer-session pattern.
There is nothing to sign and therefore no signing secret to declare; declaring an unused slot
would be a standing lie in this manifest. The only new secret material is the seeded admin
password **hash** (bcrypt cost 12, in the seed; the plaintext temp value exists nowhere in the
repo and is invalidated by the forced first-login change). The browser holds the session token
only inside an `HttpOnly; Secure; SameSite=Strict; Path=/admin` cookie.

### `master-storage-access-key` / `master-storage-secret-key`
| | |
|---|---|
| **Destination** | `env` — `MASTER_STORAGE_ACCESS_KEY` / `MASTER_STORAGE_SECRET_KEY` |
| **Placed by** | Director, at GATE-3 (any S3-compatible provider; pairs with non-secret `MASTER_STORAGE_ENDPOINT`/`REGION`/`BUCKET`) |
| **Status** | `empty` — declared, slots present and empty. **Correct today** |
| **Needed for** | The master bucket WE OWN (TDD §2). The app only ever HEADs (signed) and mints ≤1h presigned GETs for Bunny's fetch; the secret never appears in any URL. **Director note at placement: the bucket itself must be private** — the client is proven never to construct an unsigned URL, but bucket policy is the other half of "masters not publicly reachable" and is proven live at GATE-3 |

---

## Standing checks

- `.env` is gitignored and mode 600; `.env.example` is the committed template and holds no
  values. Verified at standup: `.env` was confirmed absent from the first commit's staged tree.
- No secret value in `/docs`. This manifest declares slots and statuses only.
- The bridge cannot reach `/docker/traefik/letsencrypt` / `acme.json` — SSL private keys are
  outside its mount entirely (Blueprint §13; DECISIONS.md D-004).
