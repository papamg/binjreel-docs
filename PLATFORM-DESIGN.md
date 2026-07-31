# PLATFORM-DESIGN.md — binjreel v2 (the full platform & app design)

*The comprehensive design: a scalable platform and native mobile app that matches the leaders
(ReelShort, DramaBox, NetShort, GoodShort) leg for leg and exceeds them where they are weak.
Grounded in MARKET-LANDSCAPE.md; supersedes nothing yet — it is the ratification candidate that,
once approved, drives TDD v2, the story-map rewrite, and the v0.3 stage contract.*

*Owner decisions already taken and honored throughout: full rewards + ad economy IN the design;
content supply = our studio first, open creator/studio onboarding later; launch catalog = two
Journey Viral comedy shows (in production); UI skew = ReelShort/NetShort; all genres including
micro-docs, comedy first.*

---

## 1 — Positioning

**binjreel is the vertical streaming platform for every genre — starting where the giants
aren't: comedy.** The incumbents built a $3B/yr category on one flavor (romance melodrama) for
one demographic. binjreel launches comedy-first with micro-docs on the roadmap, on top of a
money-and-access core that is already built, tested, and audited — and with a funnel-science
engine (JVScope) the incumbents replace with raw ad spend.

Three sentences an investor should retain:
1. The category is proven ($3B/yr, 115% growth) and its next frontier is *genre diversification*
   — exactly where binjreel starts.
2. The leader loses money because acquisition is brute force; binjreel's sister platform
   measures which hook produces payers, per clip, scientifically.
3. The platform is multi-tenant and audit-grade from the first line of code: today a studio
   with an app, tomorrow infrastructure other studios pay to stand on.

## 2 — The economy (three legs, one ledger)

All three legs settle into the **existing append-only wallet** (D-011/D-012 unchanged). Nothing
below touches ledger or entitlement logic; everything below is a new *credit source* or a new
*surface*. The wallet's two coin classes were designed for this: `paid` (no expiry) and
`reward` (expiring). Spend order: reward before paid, oldest expiry first.

### 2.1 Leg 1 — Coins (built)
Per-episode unlock at a config-driven coin price; coin packs via IAP with bonus coins on larger
tiers; idempotent, replay-proof, account-bound. **Add:** a **web top-up lane** (Stripe, on the
web face) at a small bonus-coin advantage — the GoodShort pattern; the wallet is already
source-agnostic, so this is a new credit source, zero entitlement change. Store-policy care:
the app never links to the web store (Apple rules); the web face simply offers it to web
visitors.

### 2.2 Leg 2 — Subscription (built, perks added)
Unlimited access tiers (weekly / monthly / annual). **VIP perks beyond access** (the category
standard): ad-free experience, 1080p, offline downloads, early-release episodes, a monthly
coin allowance for any à-la-carte edge cases. Perks are entitlement flags, not new money paths.

### 2.3 Leg 3 — The free-earning economy (NEW)
The engagement engine, monetizing the majority who never pay and building the daily habit:

- **Daily check-in** — small reward-coin grant, escalating on consecutive-day streaks; a missed
  day resets. Values and curve are config.
- **Rewarded ads for coins** — watch a 5–30s ad, earn reward coins; daily cap (config). 
- **Rewarded-ad episode unlock** — at the wall, a third door: watch one ad → *this* episode
  unlocks (a time-boxed or permanent entitlement, config). The wall becomes coins / unlimited /
  one ad.
- **Task & bonus center** — one-time bonuses (link email, enable notifications, first share),
  watch-time milestones, and a **referral program** (both sides earn reward coins on the
  referee's first qualifying action).
- **Streak-keeper mechanics** — comedy skews casual; a "streak freeze" earned by watch-time
  keeps the habit forgiving rather than punishing.

**Integrity requirements (the part clone kits get wrong, and we won't):** every ad reward is
credited **only on the ad network's server-side verification callback** (SSV), signature-
verified, idempotent on the network's event id — exactly the store-receipt pattern already
built. Client claims credit nothing. Caps enforced server-side. Fail-closed with the ad-network
slots empty. Reward credits are ordinary ledger entries; the books stay penny-true.

**Ad stack:** one mediation layer (AdMob or AppLovin MAX — a named decision at build time)
behind a single client module + one SSV endpoint, mirroring the Bunny seam pattern. Interstitial
/banner formats are deliberately EXCLUDED at launch (they are the #1 review complaint across
competitors); rewarded video only. Programmatic display is a later, separate decision.

## 3 — The app (native, Expo/React Native; web face first per the standing plan)

### 3.1 Structure — five tabs
**Home · For You · Rewards · My List · Profile**

### 3.2 Home (the ReelShort pattern, comedy-first)
- **Hero banner carousel** — editorial slots (launch: the two Journey Viral shows).
- **Shelves**, horizontally scrolling vertical-poster cards with badges (New / Hot / Trending /
  Free / rank medals):
  1. Continue Watching (resume cards with progress bars) — first when it exists
  2. New Releases
  3. **Top 10** (rank-badged, the ReelShort signature)
  4. Genre hubs: **Comedy** first, then Micro-Docs, Drama, and hubs as catalog grows
  5. Trope/tag shelves ("Workplace Chaos", "Cringe Kings", "True Story") — data-driven
  6. binjreel Originals
  7. Free to Binge (fully-free series — the top-of-funnel shelf)
- Shelves are **merchandising data** (studio-arranged rows), not code — the admin arranges the
  home the way a grocer faces shelves.

### 3.3 For You (the swipe feed)
Full-screen vertical discovery: episode-1 hooks and marked highlight moments, swipe to sample,
tap-through to the show. Starts editorial/heuristic (trending + genre affinity); the
recommendation brain grows on the events data (§6). This surface doubles as the **web teaser
feed** for logged-out visitors.

### 3.4 Show pages (every series, its own address — Owner requirement #7)
Cover art and trailer/hook loop · title, season/episode count, genre + tag chips (tappable into
tag pages) · synopsis · **Follow** (new-episode notifications) and **Add to My List** · share
(moment-precise deep links, §5) · cast block (name cards; the fandom seam) · **episode grid**
with lock states (free / owned / coins / ad-unlockable) and per-episode thumbnails · Start
Watching / Resume button · related shows. Show pages render on the web with full SEO metadata —
every series is a landing page for its own clips.

### 3.5 Tag & genre pages
Every tag chip resolves to a browsable page (the GoodShort pattern); genre hubs are curated
tag pages with editorial headers. **Search** with trending searches and tag suggestions.

### 3.6 Rewards (the earning center)
Check-in calendar with streak state · task list with claim buttons · watch-ad-for-coins entry
(with today's remaining cap visible — honesty as UI) · referral card · balance header showing
paid vs expiring reward coins **separately, with expiry dates** (the anti-complaint design: no
viewer should ever be surprised by an expiry).

### 3.7 My List
Following (with new-episode dots) · saved shows · full watch history · downloads (VIP).

### 3.8 The player (the destination surface)
Everything already designed stands: vertical, gapless, swipe up/down, tap-to-hold, prefetch of
the next entitled episode, mid-session link re-mint, no referer dependency. **Added from the
category standard:** episode drawer (jump within a series) · playback speed (0.75–2×) ·
quality selector (540/720/1080; 1080 = VIP) · captions (day one; required for sound-off social
natives and for localization later) · share-this-moment (§5) · "coming up" nudge at the last
free episode so the wall is anticipated, never an ambush.

### 3.9 The wall (three doors now)
Server-driven offer payload, rendered at the swipe into a locked episode: **coins for this
episode · unlimited · watch one ad.** Balance shown; one-tap purchase; instant return to the
exact frame. Copy and prices are server data. The ad door respects the daily cap; when capped,
it shows *when* it returns rather than disappearing silently.

### 3.10 Profile
Account/sign-in state, subscription management, coin history (the ledger, readable), settings,
notification preferences, help.

## 4 — Content architecture (all genres, comedy first — Owner requirements #6, #8)

- **Genre is a first-class dimension** of series (not a tag): Comedy, Micro-Doc, Drama, Thriller,
  Romance, Reality, … Genres carry their own hub pages, their own default free-episode counts,
  and their own pacing norms (a comedy episode lands a punchline; a micro-doc episode lands a
  revelation; a drama episode lands a cliff — same wall mechanics, different beat).
- **Micro-docs** are series like any other, with a `nonfiction` flag driving presentation
  (chapter language, source/credits block on the show page).
- **Launch catalog**: the two Journey Viral comedy shows, each on its own show page, both featured
  in the hero carousel; the Free to Binge shelf seeded from their opening runs. Catalog grows
  first-party, then by the creator/studio path (§7).
- **Cadence discipline** (the category's habit engine): scheduled episode drops with Follow
  notifications; the admin schedules releases rather than bulk-publishing.

## 5 — The differentiators (Owner requirement #9 — beyond the two pay systems)

1. **Comedy-first, all-genre platform** — the wedge nobody owns, launching where MicroCo/Crisp
   are still raising money to go.
2. **Funnel science instead of funnel spend.** Every social clip carries an origin tag; JVScope
   attributes clip → install → wall → payer. The growth loop the leader replaces with a
   nine-figure ad budget, run as measurement.
3. **Share-the-moment (the comedy engine).** Any viewer can share a timestamped moment; the
   link opens the app (or web face) at that exact beat, then rides the free run into the wall.
   Comedy is the most shareable genre ever made — every laugh becomes a distribution unit, and
   every share is a *tagged, measured* funnel entry. Drama platforms can't copy the culture
   even if they copy the button.
4. **The honest wall.** Never charged twice, never charged for what you own, purchases follow
   the account forever, reward expiries visible before they bite — provable properties of an
   append-only audited ledger, positioned openly against the category's loudest complaints.
5. **Platform-as-infrastructure (the creator/studio path, §7).** Multi-tenant with to-the-penny
   books: the structural play no incumbent studio-app can bolt on.
6. **Web face as funnel + store.** Instant playback for clip-link clickers with no install
   cliff; SEO'd show pages; web top-up at bonus value. The install becomes a step the product
   earns, not a tollgate before the first laugh.
7. **The fandom seam (later, cheap, proven).** Cast pages and behind-the-scenes shelves
   (ReelShort's ReelTalk proves the pattern); comedy casts are natural social creators — the
   talent becomes the marketing.

## 6 — Platform architecture additions (control plane)

Everything below extends the built core; the money core diff for all of it is **zero lines**.

- **Catalog & browse services**: series/episode/genre/tag reads, shelf merchandising data,
  search, show-page composition. Tenant-scoped, publish-state-honoring, media-ref-leak-proof.
- **Engagement services**: follows, lists, progress (already scoped), notifications
  (push + new-episode fanout), share-link mint/resolve with origin tags (already scoped).
- **Rewards services**: check-in/streak state, task registry (config-driven), referral graph,
  the ad-SSV endpoint, daily-cap enforcement. Credits = ledger entries, coin_class `reward`.
- **Recommendation seam**: For You starts heuristic (trending, genre affinity, follow graph);
  the `events` sink (already designed, v-later) is its training substrate. Named seam, not a
  launch build.
- **Events & the JVScope join** (already designed §9 TDD): unchanged, now with share-moment
  origin tags flowing from day one of sharing.
- **Studio admin** (already designed, extended): scheduling, shelf merchandising, tag
  management, hero-carousel slots, rewards-config panel.
- **Localization seam**: caption tracks per episode day one; UI strings externalized; dubbing
  and multi-language catalog = a later stage, not a rebuild.
- **Scaling stages** (TDD §7) unchanged; the events firehose and recommendation compute land at
  Stage 3 exactly as designed.

## 7 — Content supply roadmap (Owner decision: ours + open creator/studio onboarding later)

- **Phase 1 — Our studio.** Journey Viral produces; the platform is single-tenant in practice.
- **Phase 2 — Invited studios.** Selected producers onboard as tenants or labeled catalogs;
  revenue share computed from the same append-only ledger — statements that reconcile to the
  penny, by construction. (The audited wallet becomes the *trust product* that convinces a
  studio to bring its catalog.)
- **Phase 3 — The studio portal.** Self-serve upload/schedule/analytics for partners; the v0.x
  studio admin generalized. The marketplace question (open vs curated) is deliberately deferred
  — **curated** is the default posture; open-creator UGC is explicitly NOT the model.

## 8 — Strengths / weaknesses (Owner requirement #5 — exterior and interior view)

**Exterior — strengths:** validated category with room at the genre frontier; US home-market
advantage; differentiation an incumbent can't cheaply copy (funnel science, multi-tenant books,
comedy culture). **Exterior — weaknesses/risks:** two shows is a thin catalog against
hundred-series libraries — mitigated by comedy's rewatch/share dynamics, Free-to-Binge
positioning, and honest expectation-setting (launch as "the comedy vertical app," not "the
everything app"); UA costs are rising category-wide — mitigated by the funnel-science model
being *precisely* the answer to that; giants may move into comedy — speed and culture are the
moat, and the multi-tenant play means we can host would-be rivals.

**Interior — strengths:** the money core is real, tested, audited — most entrants fake this
part first and pay later; two-plane design keeps delivery costs rented and swappable; one
person + agents operating model keeps burn near zero. **Interior — weaknesses:** single-copy
risk (unpushed code, unversioned docs — already flagged, Director-gated); the rewards economy
adds the platform's first *adversarial* surface (reward farming) — met with SSV-only crediting,
server caps, and expiring coins; the app is the largest unbuilt surface and the only one gated
on external accounts (store enrollments — start now); design/brand for comedy is a genuinely
different visual voice from the drama incumbents — treat brand design as a real workstream,
not a skin.

**Adjustment taken from this review:** launch messaging narrows to the comedy wedge (own a
niche loudly rather than claim breadth thinly); rewarded video only at launch (no interstitials
— the category's self-inflicted wound); reward-coin expiry always visible (turn the category's
dark pattern into our trust signal).
