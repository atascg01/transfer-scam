# Claude implementation instructions

Implement Transfer Scam Football using [docs/implementation-brief.md](docs/implementation-brief.md) and [docs/product-specification.md](docs/product-specification.md).

The implementation brief fixes the remaining defaults and contains the full product specification as an appendix. Its exact engineering constants and contracts take precedence over general wording in the product specification. Explicit user requirements take priority.

## Authorized scope

Build the complete locally runnable MVP: single-player, private realtime multiplayer, accounts, ranked matchmaking, Elo ratings, ladder, and the specified content/report administration.

Implement directly at repository root, preserving the planning documents. Inspect applicable repository instructions, use the selected stack, and work through the ordered milestones. Do not restart brainstorming or ask about routine choices already settled in the brief.

## Required behavior

- Present all distinct qualifying real career clubs plus exactly one fake.
- Allow multiplayer builders to type their scam club or use five suggestions.
- Count senior first-team competitive appearances, including loans; exclude youth/reserves, trials and signings without appearances.
- Keep answers server-side until reveal; enforce phases, deadlines, ownership and scoring on the server.
- Keep solo and private play separate from ranked ratings.
- Research and verify real football content. Clearly separate fictional test fixtures from production seeds.
- Test full browser flows, concurrency, reconnects, hidden-answer boundaries and atomic rating updates.
- Keep a milestone checklist and report evidence of completed checks and remaining limitations.

## Delivery

Deliver source, actual setup scripts, migrations, sourced content/provenance, tests, a working local preview, and operations documentation. Continue through the full local MVP; a mockup or single-player-only build does not satisfy the scope.

Do not provision paid services or publish automatically. If a concrete environment/content blocker prevents one task, report it clearly and continue independent work.
