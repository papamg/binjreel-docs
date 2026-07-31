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
> filenames collide across the fleet.

**Identity check.** This is **binjreel** — a **vertical micro-series platform for every genre,
comedy first**, including micro-docs. Content by **Journey Viral**. App class **consumer content
SaaS, multi-tenant-ready (single-tenant in practice)**, audit tier **high**. Repo
`papamg/binjreel`, build branch `binjreel-dev`; record repo `papamg/binjreel-docs`.
*If you opened this expecting a different project, product, or version line: **STOP**.*

---

## WHERE WE ARE

**v0.4 — the web face — BUILT, DEPLOYED, AND OWNER-CERTIFIED AT GATE-3 (2026-07-31).** The design
is accepted at both phone and desktop widths and the Owner's forced admin password change is
complete. binjreel.com serves the full comedy-styled face: hero and shelves home, SEO show pages,
the vertical player with prefetch-fed swipe and mid-session re-mint, the three-door wall rendered
from the server offer verbatim, search, rewards hub in its coming-soon state, profile, My List,
share-a-moment, and a real admin identity behind a structural screen gate. Ten seeded shows, 82
episodes, real artwork on every slot, enforced by an automated assets gate. Suites at close: **API
147 · web server 9 · admin-gate 4 · assets-gate 4 · resume 2.** Served == merged.

**The record is no longer a single copy.** `/docs` is under version control for the first time —
130 files pushed to `papamg/binjreel-docs` (`01ffa02`). The app repo and the docs repo have
disjoint trees, so the §13 wall between source and record is structural, not conventional.

**The v0.5 stage contract is written and placed** (`docs/CONTEXT-v0.5.md`). Pricing is ratified
(D-022) and the reward-expiry contradiction between the TDD and the shipped Rewards copy is
resolved in favour of the shipped promise (D-023).

**Baseline history, in one line each.** v0.1 — the money-and-access spine, 60 tests, GATE-2
audited (`9c757af`), with D-011 (append-only in the database, no exception) and D-012 (row lock on
every wallet mutation) taken in code. v0.2 — media, GATE-3 complete, real paywalled video through
Bunny with directory-scoped token signing (`574275a`). v0.3 — the client contract, 14 scopes, 138
tests, deployed (`4ff70e1`). v0.4 — the Expo web face and real admin identity (`cd0d55e`), then
the restyle, the conformance-completion redirect, and the punch-list pass (`5c612eb`, `9ed56fe`).
**Money core diff across every engagement since v0.1: 0 lines.** Full as-built trail:
`BUILD-LEDGER.md` L-000…L-046.

---

## NEXT ACTION

**Worker — build v0.5 Phase A**, per `docs/CONTEXT-v0.5.md`. Phase A needs no store account and
does not wait: native shell and native player, onboarding to HANDOFF, the pricing config, the
rewards half with no ad network in it, the ledger-backup mechanism, and the engine's publish-state
touch (N-008). Budget ≤ $120 total, Phase A ≤ $80. Stop at the Phase A gate and report.

**Director — the two store enrollments are the clock.** Apple Developer (the D-U-N-S number is the
long pole, one to two weeks) and Play Console. **Phase B cannot open without them**, and Phase B
is the money proof.

---

## WAITING ON — Director

### W-002 · Credential scope — ✅ RESOLVED as account-scoped, with a routing caveat that stands
**Status: resolved 2026-07-31 · the routing finding is carried forward, not closed**

The account-level `papamg` claim is now **proven, not asserted**: the same credential authenticates
against a second repository (`papamg/binjreel-docs`), read **and** write, with the original
`papamg/binjreel` entry verified byte-identical before and after. The repo-scoped hypothesis is
disproven. Coverage of *every* `papamg` repo remains inferred from two data points rather than
exhaustively tested.

**The caveat, which is the useful part.** A broad token did **not** remove the routing problem.
`credential.useHttpPath` is still `true`, so git looks credentials up by host **+ path** —
`binjreel-docs` worked only because a label line was added for it. **Every future `papamg` repo
will still need its own copied label.** The token is account-wide; the lookup remains per-repo.
That is precisely the gap the Account-Aware Credential Routing work exists to close, now with a
clean demonstration behind it.

**Still genuinely open from this entry:** the two-`main` reconciliation — remote `papamg/binjreel`
`main` sits at `ec2ef3d1`, unrelated to this box's history, and `binjreel-dev` has been pushed
past it without touching it. Branch protection on `main` also remains unconfirmed. Both are
deliberate Director decisions and neither blocks v0.5.

---

## BACKLOG

| # | Item | Status | Where |
|---|---|---|---|
| 1 | v0.1 money-and-access spine · v0.2 media · v0.3 client contract · v0.4 web face | ✅ **ALL DONE** — v0.4 Owner-certified at GATE-3 2026-07-31 | `BUILD-LEDGER.md` L-011…L-046 |
| 2 | **v0.5 — native shell + money (launchable)** | **NEXT — contract written and placed; Phase A opens immediately, Phase B waits on store accounts** | `docs/CONTEXT-v0.5.md` |
| 3 | **Continuous ledger backup** — the last gate before real receipts | **scoped into v0.5 Phase A (A6)**; the off-box destination + credential remain the Director's | `SECURITY-REPORT.md` N-001 |
| 4 | Apple Developer + Play Console enrollment | **open — Director, IN PROGRESS, the clock on Phase B** | D-U-N-S lead time |
| 5 | Apple ASN + Google RTDN verification keys (declared, **fail closed** while empty) | open — Director, at real-receipt go-live | `docs/SECRETS.md` |
| 6 | Compliance set before store submission: auto-renew/click-to-cancel, digital-goods tax, age-gating, privacy policy matching actual telemetry, DMCA | open — Director, gates Phase B's close | `docs/TDD.md` §12 |
| 7 | Two-`main` reconciliation + branch protection on `main` | open — Director (W-002 residue) | `DEFECT-LEDGER.md` DL-B005 |
| 8 | **Per-repo credential labelling** — account-wide token, per-repo lookup; every new `papamg` repo needs its own label line | open — feeds Account-Aware Credential Routing | W-002 |
| 9 | Canon gaps DL-B001/B002/B003/B005 (empty app-class library · **no token budget in orders — has now recurred on every order this project has received** · no false-Director-gate rule · credential placement not required path-keyed/append) | open — route via Tone canon drop | `docs/DEFECT-LEDGER.md` |
| 10 | **5A scalability & tenancy audit has never run** — index/query-plan coverage under tenant-scoped load, pooling, backup/DR architecture unexamined. Named as a pre-real-traffic gate | open — recorded so its absence is a known gap, not an assumed pass | `SECURITY-REPORT.md` seam note |
| 11 | Remaining product config: per-genre free-episode counts, final playback TTL, launch catalogue size and cadence, web-top-up (D-020) activation window | open — Director; **none block v0.5** (episode price, packs, subscription now settled by D-022) | `docs/TDD.md` §12 |
| 12 | Deferred conformance debt: Coins & plans admin §5 → v0.7 · the §5 admin screens with no product behind them → with their features · light theme parked (dark-only ship stands) | queued | `conformance-v2/README.md` |
| 13 | IODE remote-write capability (retires all remote per-project bridges) | open — Architect routes via Tone canon drop | D-009 |
| 14 | Dependency advisory: `uuid` < 11.1.1 via Expo **build** tooling (runtime web image ships zero npm deps) — revisit at the v0.5 SDK touch | open — **v0.5 is that touch** | `SECURITY-REPORT.md` v0.4 |

---

## Reading order for the next session

1. `/docker/PROJECTS.md` — which project, where its records (read first).
2. `git -C /docker/v2s-canon pull`, then canon's `_v2s/` — doctrine.
3. This file — project state.
4. `docs/IDEAL-SCENE.md` — the destination.
5. `docs/TDD.md` v2 — the as-designed technical intent.
6. `docs/CONTEXT-v0.5.md` — the live stage contract (prior stages: `CONTEXT-v0.4.md`,
   `CONTEXT.md` (v0.3), `CONTEXT-v0.2.md`, `CONTEXT-v0.1.md`).
7. `docs/DECISIONS.md` D-001…D-023 — the settled forks.
8. `docs/DEFECT-LEDGER.md` DL-B001…DL-B011 — the kept defects. Next free number: **DL-B012**.
9. `docs/BUILD-LEDGER.md` L-000…L-046 — the as-built trail.
10. `docs/design-package/canvas-doc-with-six-screens/project/HANDOFF.md` — the governing visual
    contract, in full, plus `conformance-v2/README.md` for what conformed and what deferred.
