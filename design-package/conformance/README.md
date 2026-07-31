# Conformance — RESTYLE ORDER (2026-07-29/30)

Each briefed surface, screenshot from the **deployed** app at https://binjreel.com beside its
package mockup. `NN-name.side-by-side.png` is the pair in one image; `.mockup.png` /
`.live.png` are the halves.

| Pair | Mockup (package §) | Deployed shot |
|---|---|---|
| 01-home-mobile | §2.1 Home / dark | live 390w, full home (hero + shelves + Top-10 medals) |
| 02-home-web | §4.1 Home / dark 1440 | live 1440w |
| 03-show-mobile | §2.2 Show page / dark | live 390w |
| 04-player-mobile | §2.3 Player overlay | live 390w, chrome woken after the 2400ms auto-hide |
| 05-wall-mobile | §2.4 The Wall | live 390w, real locked episode, server offer verbatim |
| 06-wall-web | §4.4 Wall 860px modal | live 1440w |
| 07-admin-login-mobile | §2.5 Admin login | live 390w, clean profile |
| 08-admin-change-mobile | §2.6 Forced change | **same-bundle local drive** — the live forced-change screen can only be reached by spending the Owner's one-time temp credential, which is theirs to perform at GATE-3. Served bundle hash == local bundle hash (`entry-5fafbd2d…`). |

Known, recorded divergences (DESIGN.md v2 §9): mockup art/ha-stats are placeholders — the
API has no ha-count field and production key art is still owed, so those slots render their
fallback treatment rather than faked numbers; the five-tab IA stays at all widths per the
order (the §4 web header is a later pass); purchase doors carry the web "Get the app —
coming soon" state (behavior unchanged); wall copy is the server's, verbatim.
