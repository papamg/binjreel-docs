# CONTEXT.md — binjreel · v0.3 (the client contract — catalog, engagement, rewards-ready seams)

*The stage contract for the current build. This is the Order the Worker executes against: what to
build, the boundaries, and the definition of done. Scoped tightly — this is the whole blast
radius for v0.3. Serves TDD.md v2 §6, §9.1–9.3, §11(3). Advances with the baton at GATE.*

*Department: **4B Build** (server; no client code in this Order). Doctrine is canon; this is
project state. Where this is more specific than the TDD, that specificity is the Worker's
starting point, overridable by better construction at or above the IDEAL-SCENE floor.*

*Supersedes the v0.2 CONTEXT as the live stage contract; archive the prior file as
`CONTEXT-v0.2.md` at placement (do not delete — it is the v0.2 as-ordered record).*

---

## Target

**v0.3 — the client contract.** Everything a client (web face v0.4, native v0.5) must be able to
ask the server, none of which exists today: what shows are there, what's in this one, what plays
next, where did I stop, what do I own, what does this locked episode cost, and what did this
shared link point at. Plus the mid-session re-mint that closes DL-B007 properly. **Reads and
engagement writes only — the money core diff for this entire version must be 0 lines.**

Why this slice now: the running service has nine doors and no catalog; the app cannot exist
until the server can be browsed. This is pure server work, testable exactly like v0.1/v0.2,
with **no Director dependency anywhere in it** — it runs while the store-enrollment clock ticks.

## Ratified frame (from TDD v2 — do not re-litigate here)

- **The entitlement engine is untouched (TDD §4.3).** `mayPlay` remains the only URL gate. The
  offer payload, prefetch, and re-mint all *consume* it; none re-implements it. No second
  source of truth for "may this account play this."
- **Ledger off-limits (D-011/D-012).** v0.3 writes no wallet entries. The offer payload *reads*
  balance via the existing derivation. If any path seems to need a ledger write, BLOCK.
- **Media refs never leak.** No catalog, show-page, tag, search, next-episode, offer, or
  deep-link response may carry `bunny_ref`/`master_ref` or any playable URL to an unentitled
  caller. Unpublished content is *absent* from viewer reads, not 403'd.
- **Genre first-class (D-018).** `series.genre` + `nonfiction` flag + per-genre free-count
  default enter the schema in this version (migration), with tags many-to-many.
- **Three-door offer shape (D-017).** The offer payload carries the ad door *as data*
  (`adUnlockAvailable`, capped state, config) even though the ad economy builds at v0.6 —
  the shape ships now so the client renders one contract forever. With no ad network
  configured it is `false`/fail-closed.
- **Audit tier high (TDD §10).** Dedicated security pass on the new surfaces before complete;
  share-link tokens non-enumerable; every new list/read rate-limited; tenant + user scoping on
  every query.

## Build scope — what v0.3 IS

1. **Schema migration:** genre/nonfiction on series (backfill: both launch series → Comedy),
   `series_tags`, `shelves`+`shelf_items` (incl. hero slots), `follows`, `list_items`,
   `watch_progress` (+ series resume pointer), `watch_history`, `share_links`,
   `episode_schedule`, `captions`, `cast_members`+`series_cast`. Progress/engagement tables
   isolated from money tables (no FKs into wallet; separate write paths).
2. **Catalog reads (viewer-facing):** series list w/ cover, synopsis, genre, tags, episode
   count, free count, publish-state honoured; episode list w/ order, duration, thumbnail,
   free/locked/owned state *for the caller*, unlock price; genre hubs; tag pages; search
   (title + tags, trending-searches read); Top-10 (config-driven window, computed from
   history/unlock counts).
3. **Shelf composition:** home payload = hero slots + ordered shelves resolved from
   merchandising data; Continue Watching shelf composed per-caller; "Free to Binge" resolved
   by free-count. Shelf CRUD sits behind the interim media-admin token (D-014) until v0.7.
4. **Show-page composition:** one call returning the full §TDD 9.1 show page (series, tags,
   cast block, episode grid with per-caller lock states, resume point, related-by-tag/genre).
5. **Engagement writes:** follow/unfollow, list add/remove, progress write (cheap, idempotent
   under rapid repetition), history append; resume read; all caller-scoped.
6. **Next-episode + prefetch:** one call resolving next-in-series → if entitled, include a
   signed playback grant (existing mint, unchanged); if not, include the offer payload;
   end-of-series explicit. Prefetch cannot mint for unentitled episodes.
7. **Three-door offer payload:** price, caller balance, sufficiency, coin packs (config),
   subscription option (config), `adUnlock` state (fail-closed false). Same shape anonymous or
   signed-in. No media refs. Copy fields server-side.
8. **Entitlement summary:** owned episodes per series + subscription status for the caller;
   same source of truth as `mayPlay` (shared code path, not a re-implementation).
9. **Mid-session re-mint:** renew the playback grant for the same episode without interrupting
   playback; full entitlement re-check every mint; **raise the 30s code default TTL in
   `config.ts`** (DL-B007 code item — pick a sane default, e.g. 900s, env still overrides).
10. **Deep-link + share-moment:** mint (operator/system + a caller-scoped "share this moment"
    mint), opaque non-enumerable tokens carrying series/episode/position + origin tag; resolve
    returns target + entitlement state + (if entitled) a grant, else the offer; unavailable
    targets resolve to a clean state, never an error; resolution rate-limited against
    enumeration; origin tags persisted for the future JVScope join.
11. **Caption manifests:** per-episode caption-track read (storage seam like masters; empty is
    a valid state).
12. **Episode scheduling:** `publish_at` honoured by all viewer reads (scheduled = absent until
    time); the follow-fanout *hook* recorded (actual push lands v0.5 — log-only notifier now).
13. **Fixtures + tests:** seed a comedy-shaped catalog (2 series, tags, shelves, cast, captions
    stub); every DoD item proven by test against real Postgres/Redis per house pattern.
14. **Dedicated security pass** on all new surfaces (1C pattern), SECURITY-REPORT updated.

## Out of scope — what v0.3 is NOT (log ADDED if tempted)

- No client/UI of any kind (v0.4). No push delivery (v0.5 — hook only). No ad network, no SSV
  endpoint, no check-in/task/referral system (v0.5/v0.6) — only the offer *shape*.
- No recommendation engine — For You at v0.4 consumes trending/genre reads from this contract.
- No Stripe/web top-up (D-020, later). No studio-admin UI (v0.7) — shelf/schedule CRUD is
  API-behind-D-014-token only. No `events` sink (v0.8). No localization beyond caption seams.
- No change to `mayPlay`, the ledger, receipts, or the media pipeline beyond the re-mint route
  and the TTL code default. No new secrets (nothing to declare; SECRETS.md unchanged).

## Definition of done (GATE 2 / GATE 3 verification points)

- [ ] Migration applied; both launch series carry genre Comedy; fixtures seeded.
- [ ] Catalog/hub/tag/search/Top-10 reads live, tenant-scoped; unpublished + scheduled content
      absent; **no media ref or playable URL in any viewer read for an unentitled caller** —
      proven by test.
- [ ] Show-page composition returns the full contract in one call incl. per-caller lock states
      and resume point.
- [ ] Follows/lists/progress/history: caller-scoped; cross-user and cross-tenant reads return
      nothing; progress write idempotent-safe under rapid repetition; resume survives
      sign-out/sign-in AND the anonymous→real merge — proven by test.
- [ ] Next-episode returns grant (entitled) or offer (not) — never a bare refusal; end-of-series
      explicit; prefetch cannot mint for unentitled episodes.
- [ ] Offer payload complete (three-door shape, adUnlock fail-closed false), identical shape
      anonymous vs signed-in, zero media refs, all values from config.
- [ ] Entitlement summary provably agrees with `mayPlay` (shared path; a test that breaks if
      they diverge).
- [ ] Re-mint renews without a new unlock, re-checks entitlement every mint, and the 30s code
      default is raised (DL-B007 closed fully — note closure in DEFECT-LEDGER).
- [ ] Share/deep links: non-enumerable, origin tags persisted, unavailable → clean state,
      resolution rate-limited; a guessing test demonstrates enumeration resistance.
- [ ] **Money core diff = 0 lines; full v0.1+v0.2 suite (87) still green** plus the new suite.
- [ ] Security pass run on all new surfaces; SECURITY-REPORT updated; no high/critical open.
- [ ] BUILD-LEDGER as-you-go; BUILD-REPORT closes the loop; STATE + TDD v2 roadmap advanced at
      close (v0.3 marked built).

## Token / cost budget (Blueprint §08 — closes DL-B002 for this project's Orders)

Comparable scale to the v0.1 build (schema + many read surfaces, no novel external
integrations). Budget: **one Worker engagement ≤ $50 API spend**, logged as a stat in the
BUILD-REPORT. If projected to exceed at any point, pause and report the projection with the
cause — do not silently continue. Cheap-and-correct beats expensive-and-correct; the budget is
the least resource this result truly requires, not a ceiling fighting quality.

## Build discipline (from the binding)

- Commit to `binjreel-dev` only, never `main`. Append to `BUILD-LEDGER.md` after each step and
  before any pause. Log DIVERGENCE (naming the stale section) or ADDED as they occur.
- Owner-level questions → BLOCKED in the ledger, question to STATE "Waiting On," pause. Do not
  improvise around a blocker.
- Run the security pass after the build, before marking complete.

## Director acts this build waits on

**None.** This Order is deliberately Director-free. In parallel (not gating this build): the
store enrollments, W-002 push/two-`main`/branch protection, /docs under version control, and
the ledger backup remain on the Director's desk per STATE and TDD v2 §12.
