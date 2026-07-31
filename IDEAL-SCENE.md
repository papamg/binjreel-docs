# IDEAL-SCENE.md — binjreel

*What "done right" looks like from the outside — the viewer's experience and the operator's, at
the abundance standard: every interaction returns more than it cost. This is the north star the
TDD serves. It describes the destination, not the mechanism.*

*Doctrine is canon; this is project state. Companion to TDD.md (how) and the two scoping docs
(why). Authored by the Architect, Phase 1.2.*

---

## The viewer

**She never waits, never thinks, and pays at the peak.**

She meets binjreel as a 30-second cut on her phone's social feed — a scene that ends mid-breath,
right at the turn. One tap installs the app, and the app opens *on that exact story, at that exact
moment*, already playing. No home screen, no signup, no genre survey standing between her and the
next scene. The video is sharp and it starts instantly, whether she's on strong wifi or one bar of
cell.

She watches the way she scrolls — thumb only. Swipe up, next episode, gapless. Swipe down, back.
Tap to hold. Episodes are a minute or two each and roll one into the next with no seam, so ten
minutes vanish before she notices. The free run carries her deep enough to care about who these
people are.

Then the turn. The last free episode ends on a cliff, she swipes up hungry for the next — and the
wall meets her exactly there, at the peak of wanting. It doesn't scold or interrupt; it offers.
*Keep watching.* Two doors: a few coins for this episode, or unlimited for a flat price. She taps,
her phone recognizes her face, and she's back in the story in about four seconds — no card to
type, no page to leave. The wall landed where the craving was, and clearing it felt like part of
the show.

After that first purchase, money disappears from her experience. Coins spend silently as she
binges; if she's on unlimited, she never sees a price again. She's warned she's low *before* she
runs dry mid-scene, never after. She closes the app on episode 14 and a day later a nudge brings
her back to episode 14, precisely. Her progress, her purchases, and her unlimited status follow
her — new phone, tablet, doesn't matter; they belong to *her*, not her device.

Nothing she paid for can be taken, lost, or double-charged. She is never billed twice for the same
tap, never charged for a scene she already owns, and never made to feel that a glitch cost her
money. The one thing the system guards most fiercely is the truth of what she paid for.

**The abundance test:** she came for one more scene and left having spent less attention getting
what she wanted than she expected to. The product got out of her way.

## The operator (the studio side)

**One person, plus agents, runs a platform that behaves like it has a staff of fifty.**

The studio uploads a finished master and moves on. The system takes it from there — grinds it into
every size a phone might need, files it, and makes it ready, showing plainly where each episode is
in that process and flagging the rare failure loudly instead of hiding it. Setting a series live,
choosing how many episodes open free, pricing an unlock, arranging the home shelves — each is a
few plain choices, not an engineering task.

The operator sees the truth of the business without asking for it: which hooks pull viewers in,
where they hit the wall, whether they chose coins or unlimited, where they drift away. The clips on
social and the behavior in the app read as one story, so the question "which hook produces viewers
who actually pay?" has an answer. That intelligence flows into the same content engine the studio
already trusts.

The books reconcile to the penny, always. Every coin bought, earned, and spent traces to a record
that cannot be quietly rewritten, so the operator can answer any "what did this person pay for"
with certainty, and a lost server never means a lost ledger.

**The abundance test for the operator:** the platform returns more than it costs to run — in
revenue, yes, but first in *attention saved*. The Director spends judgment on stories and growth,
never on keeping the lights on.

## What must never happen

- A viewer waits on a spinner between episodes, or hits a wall that feels like a scold instead of
  an offer.
- A viewer is charged twice, charged for something owned, or loses a purchase to a glitch or a new
  phone.
- A paid stream leaks to someone who didn't pay.
- The operator has to read plumbing to understand the business, or becomes the infrastructure
  engineer the framework exists to prevent.
- The record of who paid for what is ever in doubt.

## The standard this sets for the TDD

Every technical choice in TDD.md is answerable to one of these outcomes. Instant, gapless playback
is why media never touches the origin box. The paywall-at-the-peak is why entitlement is checked
server-side before a signed URL is minted. "Never charged twice" is why the ledger is append-only
and every money act is idempotent. "Purchases follow the person" is why entitlement binds to the
account, not the device. If a design decision doesn't serve a line in this scene, it's ceremony,
and it's cut.
