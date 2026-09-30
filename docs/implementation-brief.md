# Transfer Scam Football — Claude implementation handoff

**Goal:** Build a complete, locally runnable football bluffing game with single-player, private multiplayer, ranked matchmaking, and a ladder.

**Architecture:** A TypeScript monorepo with a React/Vite client, a server-authoritative Fastify application, authenticated WebSocket state updates, and PostgreSQL persistence. Game rules live in pure server-side domain functions; persistence and delivery are adapters around them.

**Tech stack:** Node.js 24 LTS, npm workspaces, TypeScript, React, Vite, Fastify, @fastify/websocket, PostgreSQL, Drizzle, Better Auth, Zod, Vitest, Playwright, Docker Compose. Pin compatible stable package versions and commit the npm lockfile. No paid services are required locally.

This single attachment contains the engineering handoff followed by the full product specification. The appended specification explains the product and research; the handoff fixes engineering defaults and directs execution. Explicit user requirements take priority over technical defaults. These documents are ready for implementation, not another brainstorming interview.

## Copyable instruction for Claude

> Implement Transfer Scam Football from the attached product specification and this handoff. The workspace currently has planning documents and no application. Inspect applicable repository instructions, then implement the application directly at this repository root, preserving README.md, CLAUDE.md, and docs/. Build the milestones in order, starting with actual sourced content and the full single-player loop, then private multiplayer, then accounts, ranked matchmaking and ladder. Continue through the whole local MVP; completing only a mockup or solo mode is not completion. Use the chosen defaults without asking routine design questions again. Verify game rules, hidden-answer boundaries, concurrent actions, browser flows, and ratings with meaningful tests. Maintain a progress checklist. Do not invent football facts, present test fixtures as verified careers, install paid services, or publish automatically. If content cannot be verified, mark it unavailable and explain the exact remaining content work while continuing independent engineering. Deliver source, setup instructions, verified test results, known limitations, and a runnable local preview.

## 1. Fixed product rules

The footballer is known. Every distinct real club from their complete qualifying career appears, plus exactly one false club, shuffled. Qualifying means at least one senior first-team competitive appearance, including loans. Exclude youth/reserves, national teams, trials, unofficial friendlies, and signings without appearances. Count returning clubs once. Never shorten the real-club list.

Multiplayer: five rounds. In each round assign different footballers to the two builders; each builds the OTHER participant's puzzle. Nobody solves a player whose true career they saw while constructing in the same match. Ten distinct footballers per match. Each builder sees the real career and five optional false-club suggestions, and can choose ANY recognized eligible club through searchable typing, not just those suggestions. Require a final confirm action. An invalid choice does not lock the puzzle.

Solve: four or more cards depending on the full career. One selection, locked immediately. No reveal or opponent answer leakage until both answers lock or the timer ends. Correct solve earns one point; opponent's wrong answer or solve timeout earns one fooling point. A correct/correct round gives each player one point; correct/wrong gives 2–0; wrong/wrong gives 1–1. Totals tie = draw. No speed bonus.

Solo: five different footballers per run, unlimited runs, untimed, no account needed. Server picks a reviewed fake; player selects once, sees the answer/explanation, then chooses Next. Score out of five. No builder, bot opponent, Elo update, or solo ladder in this release.

## 2. Constants and phase behavior

| Setting | Value |
|---|---|
| Multiplayer rounds | 5 |
| Suggestions per builder | 5 |
| Construction deadline | 25 seconds |
| Solve deadline | 20 seconds for max real-club count ≤6; 30 for 7–12; 40 above 12 |
| Reveal duration | 7 seconds |
| Solo puzzles | 5; untimed |
| Initial eligible career | At least 3 distinct qualifying clubs |
| Room capacity | Exactly 2 |
| Disconnect grace | 30 seconds; phase clocks continue |
| Guest/session persistence | 30 days; extend on activity |
| Empty private lobby expiry | 30 minutes |
| Solo run expiry | 7 days without activity |
| WebSocket heartbeat | Ping every 10 seconds; declare stale after 20 seconds without pong |
| Initial load-test target | 50 matches / 100 active players |

Construction may end early once both builders confirm. If the deadline expires, server chooses a prevalidated fallback from the assigned suggestions for an unconfirmed participant. Solve starts for both at the same server time, and uses the longer of the two careers to choose one shared timer. Solve ends early only when both lock; otherwise wait until the deadline. Then reveal both answers, wait seven seconds, and start the next construction. After round five's reveal, complete the match.

Accept an action only while server receipt time is strictly before phaseEndsAt, under the match's transaction lock. The client cannot supply a trusted clock, phase, role, score, or rating. Display countdown based on serverNow/phaseEndsAt and resync on snapshots.

Explicit leave after start = forfeit. One player disconnected beyond grace = forfeit; both beyond grace = void, without ratings. Reconnecting within grace restores the existing phase, answers, and deadlines. A browser refresh does not create another participant. Disconnects do not grant more time. A server restart voids unfinished multiplayer matches without Elo changes; persist enough data for a clear "match interrupted" screen. Solo sessions survive restart.

## 3. Presentation

Working name: Transfer Scam. English UI, responsive mobile-first design. Use a playful football trading-card feel: dark navy background, off-white cards, lime action accent, and a distinct coral reveal accent. Use system fonts and text club cards initially. No dependency on scraped badges, player photos, paid fonts, or generated assets. Use the player's name and optional nationality text, not fabricated photos.

Home actions: Single Player, Play With Friend, Play Ranked. Ranked requires an account and is feature-gated until implemented. No nonfunctional buttons masquerading as completed features.

Builder title: "Sneak in a fake club". Show real clubs in a compact grid, a "Type a club…" searchable field, and five suggestion buttons. Choosing a suggestion fills the same pending selection as typing; confirm with "Send the scam". Once confirmed, show a locked state.

Solver title: "Which club is the scam?". All club cards look identical; use server-shuffled order independent of true/fake status. Selecting a card locks it and shows "Answer locked". During waiting do not show correct/incorrect colors or opponent choices. Reveal identifies the fake, the chosen answer, points, and the full real career with source links.

Support 360px-wide screens, long club names, long careers with accessible scrolling, keyboard activation, visible focus, labels, 44px touch targets, and reduced motion. Never hide or truncate away a career club. Keep the timer and confirmation accessible without trapping focus. Solo uses the same solver/reveal components but no timer.

Results: match result, totals, five-round recap, ranked delta where relevant, Play Again/Rematch. Private rematch needs consent from both and creates a new match. Ranked Play Again rejoins the general queue rather than ranking a requested private rematch. Sharing includes result/score and a site link, not a secret answer or unplayed future puzzle.

## 4. Content and validation

Seed at least 30 independently sourced complete real careers, with five reviewed suggestion clubs each and an initial catalog of at least 150 real clubs with aliases. These are delivery targets requiring research, not facts already supplied. Use retired players where practical and only verified active-player snapshots. The eligible corpus must contain at least ten distinct players for a multiplayer match.

Local development may use clearly named fictional fixtures to test game mechanics, stored outside the production seed. A playable football MVP requires sourced real content. Do not count fictional fixtures toward the seed target or give invented claims real source links.

Wikidata supplies candidate identities/careers, not automatic complete appearance evidence. Keep source URL, access date, evidence notes, senior appearance verification, career-completeness review status, and reviewer. Use official club/player archives and credible source records for manual validation. Do not silently scrape a paid source or assume image rights come with statistics.

Catalog search normalizes casing, diacritics, and common aliases. Return canonical IDs/names, not arbitrary user strings. Ambiguous terms require an exact selection. A typed club is accepted when the career is complete/reviewed, the selected canonical club is not in that full career, and the pair is not in ambiguity exclusions. It need NOT be a suggested club. Do not validate using "missing from an incomplete database = false".

Exclude confusing false choices with known youth/reserve/trial/friendly associations from the initial pool. Maintain a blocked-pair registry so these do not accidentally pass custom validation. Normalize club renames to the same identity, while treating genuinely distinct clubs as distinct records with notes.

Random assignments favor comparable recognition tiers and real-club counts within one club. With insufficient pairs, minimize count difference, use the shared longer timer, and flag the pairing for later difficulty analysis. Randomize which participant receives which career. Avoid repeats within a match/run; reduce the participant's last 20 seen players across sessions where the corpus allows it, without failing when the corpus is small.

Publish content versions immutably. Ongoing matches retain their version. A disputed pair is quarantined while reviewed; correcting content cannot silently change a locked answer. Public ranked eligibility requires two independent reviews. The software should implement that metadata even though another human content review happens before public release.

## 5. Repository map

Create the following boundaries; extra small files are fine, but keep unrelated responsibilities apart:

```text
transfer-scam/
  package.json, package-lock.json, .nvmrc, .env.example, README.md
  compose.yaml, Dockerfile, .gitignore
  apps/web/src/
    app.tsx, styles.css, api.ts
    components/ClubCard.tsx, CareerGrid.tsx, Countdown.tsx, ClubPicker.tsx
    features/solo/, lobby/, match/, ranked/, account/, admin/
  apps/server/src/
    app.ts, server.ts, config.ts, auth.ts
    db/schema.ts, db/client.ts
    content/{repository,validation,assignment,views}.ts
    solo/{service,routes}.ts
    matches/{engine,service,routes,views,recovery}.ts
    realtime/{gateway,subscriptions}.ts
    matchmaking/{queue,service,routes}.ts
    ratings/{elo,ledger,rebuild}.ts
    admin/{routes,service}.ts
    reports/{routes,service}.ts
  packages/contracts/src/{schemas,events}.ts
  content/{clubs,players,careers,suggestions,exclusions,sources}.json
  scripts/{import-content,validate-content,grant-admin,rebuild-ratings}.ts
  tests/{unit,integration,e2e,load}/
  docs/{content-policy,operations,verification}.md
```

Production football content and full career records are server-only. Do not place content JSON in web/public, bundle it into React, expose it through source maps, or export the complete corpus through a public API.

## 6. Persistence model

Use UUID primary IDs except canonical club/player external IDs may be recorded separately. Store timestamps in UTC and display appropriately. Add migrations and database constraints.

- Better Auth tables for users/accounts/sessions/verification. Add public handle, role, rating (1000), ranked count, and wins/draws/losses.
- Guest principal: opaque server-issued session, display name, created/last-active/expiry. Never merge somebody's guest history based on display-name equality.
- Clubs: canonical name, aliases, country, identity notes; unique aliases can point to one club, ambiguous aliases can point to several.
- Players/content versions: identity, aliases, recognition tier, verified career IDs, evidence, completeness status, verification time, review metadata, ranked eligibility.
- Suggestion pairs and exclusions: player/version, club, status/reason/evidence. Five eligible suggestions are required to generate a builder board.
- Solo sessions/puzzles: owner principal, immutable content references, shuffled card IDs, secret fake, locked answer, correctness, current index, expiry.
- Rooms: code, two participant slots, ready flags, expiry, rematch state. Use eight uppercase characters from an unambiguous alphabet and collision-check.
- Matches: mode, players/principals, status, round index, phase deadline, disconnection state, scores, completion reason, immutable assigned content references, revision counter.
- Round puzzles: builder, solver, real IDs, suggestions, selected fake/fallback, shuffled solve cards, answer, answer/lock timestamps.
- Match events: event ID, match, revision, type, server timestamp, safe action metadata. Sensitive answers are not public events before reveal.
- Queue tickets: user, created time, heartbeat/expiry, active status. Unique active ticket per user and no queue ticket while in a live ranked match.
- Rating ledger: unique match ID, both before/after ratings, delta, outcome, void flag, correction audit.
- Reports: match or solo puzzle, disputed player/club/version, reason, review status, admin actions.

Use transaction/row locking for match actions, queue pairing, and result application. A unique match ledger key and processed-action key guarantee one rating update and idempotent actions. Do not keep the only match copy in RAM.

## 7. HTTP and realtime contracts

All custom mutating endpoints accept an Idempotency-Key UUID. Use credentialed same-origin sessions and authorize the principal on every request. Better Auth owns /api/auth/*; don't override its API conventions.

| Endpoint | Request/action | Response |
|---|---|---|
| POST /api/guest | displayName, 2–24 characters | Creates guest cookie; public principal |
| GET /api/me | Session | Principal, public profile, active room/match/run |
| GET /api/clubs?q= | 2–80 character search; up to 10 results | Canonical IDs/names/country; no career data |
| POST /api/solo | Guest/account session | Run ID and first solver view |
| GET /api/solo/:id | Owner only | Current safe view or recap |
| POST /api/solo/:id/answer | puzzleId, clubId | Locked answer/reveal; repeats return same result |
| POST /api/solo/:id/next | revealed puzzleId | Next puzzle or final recap |
| POST /api/rooms | Session | Room code/link and lobby view |
| POST /api/rooms/join | code | Join existing available room |
| POST /api/rooms/:id/ready | ready boolean | Lobby snapshot; creates one match when both ready |
| POST /api/rooms/:id/rematch | consent boolean | Rematch lobby/new match once both agree |
| GET /api/matches/:id | Participant only | Recipient-specific safe snapshot |
| POST /api/matches/:id/construct | roundId, puzzleId, clubId | Confirmed fake choice for its builder only |
| POST /api/matches/:id/answer | roundId, puzzleId, clubId | Own answer-locked acknowledgement; no correctness |
| POST /api/matches/:id/leave | Participant | Forfeit or lobby exit acknowledgement |
| POST /api/queue | Registered account | Unique ticket and queue state |
| DELETE /api/queue/:id | Owner | Cancellation, or already-matched destination |
| GET /api/queue/:id | Owner | Status, elapsed time, assigned match if any |
| GET /api/ladder?cursor= | Public | 50 entries: handle, rank, rating, record; next cursor |
| GET /api/profiles/:handle | Public | Public stats only, never email/session details |
| POST /api/reports | Owned puzzle reference, clubId, reason ≤500 characters | Report ID |
| /api/admin/* | Administrator only | Content/review/report management |

Use HTTP 401 for no principal, 403 for wrong owner/role, 404 for inaccessible/missing IDs, 409 for stale phase/locked conflict/full room/already active, 422 for invalid fake club, and 429 for rate limits. Return stable machine codes and clear inline UI messages. Identical retries return the original acknowledgement; a reused key with different payload returns conflict.

WebSocket /ws checks session and exact allowed Origin. After connecting, client sends subscribe with a room, queue ticket, or match ID; authorize membership before registering. Emit full safe snapshots containing resourceId, revision, serverNow, phase, phaseEndsAt, own role/view, public score, and permitted opponent status. Client discards snapshots older than its latest revision and fetches a fresh snapshot after reconnect. Heartbeat is transport-only, not a client-controlled game clock.

Builder view: its assigned player, full true club cards, five suggestions, own selection, deadline. Solver view: its assigned player, shuffled cards, own lock status, deadline; never include fake flag, career membership, builder suggestions, or opponent selection. Reveal view adds truth and answers. Inspect serialized payloads and browser network traffic to verify this, not just UI rendering.

## 8. Accounts, queue and ladder

Better Auth email/password sessions with database storage; no custom password hashing. Use guest sessions for solo/private rooms and registered accounts for ranked. Local email verification/reset uses Mailpit in Docker Compose and a configured SMTP adapter. Public ranked requires verified email; no external email vendor is needed locally. Seed local verified test accounts for browser tests. Handle reset/expired verification clearly.

Public handle: 3–20 letters/numbers/underscores, case-insensitive unique. Display names never expose email. Assign admin role only with the local grant-admin script or an existing authorized admin, never client profile input.

Elo starts at 1000, K=24, denominator 400, result 1/0.5/0. First ten valid matches are provisional. Compute one rounded delta and apply its negative to the other player; do not independently round two inconsistent numbers. Use ratings at finalization under locked user rows, record them in the ledger, and prohibit concurrent ranked matches per user. Voids do not affect stats or ratings.

One global queue. Rating tolerance: 100 initially; 200 after 15 seconds; 350 after 30; 600 after 60. Both tickets must accept the pair under their own waiting-time tolerance. Prioritize oldest eligible waiting ticket, then closest rating. Avoid last-ten-minute rematches while alternatives exist; allow after 60 seconds of waiting. Queue heartbeat every ten seconds; expire an abandoned ticket after 30. At two minutes show quiet-queue options without inventing opponents or silently auto-canceling.

Ladder includes accounts with ten valid ranked matches, sorted rating descending, then ranked wins descending, then stable user ID ascending. Use the same order for pagination and position. Profiles show provisional status before eligibility. Private matches/solo never update Elo. Keep chat out of this release.

## 9. Security, recovery and operations

Use secure HttpOnly session cookies in production, restrictive origins, Better Auth's CSRF/origin handling, authorization on every game action, and WebSocket Origin checks. Game mutations require JSON and a valid same-origin session. No secrets in frontend env variables.

Initial rate limits per principal/IP as appropriate: club search 60/minute, room creation 10/hour, join attempts 20/minute, game mutations 60/minute, reports 5/hour, with separate sane auth limits. Return retry information; don't rate-limit WebSocket transport pongs as game actions. Limit JSON payloads to 16KB and client socket frames to 8KB.

Structured logs include request/match IDs and state transitions, not passwords, cookies, auth tokens, or answer secrets in user-visible logs. /health/live checks process; /health/ready verifies DB and migrations. Scheduled maintenance prevents new ranked matches, drains existing ones, and preserves or voids interrupted results explicitly.

Confirmed content mistakes: quarantine immediately; void affected ranked matches and rebuild the chronological rating/stat ledger without them under a maintenance lock. Never double-credit a correction. Admin report actions and content publication have an audit trail.

Serve compiled frontend and API from the same server in production. Vite proxies API/WebSockets in local development. Docker Compose includes PostgreSQL and Mailpit. Prepare Docker deployment instructions for one Render web-service instance and a Postgres database, but do not provision/pay/deploy. Multiple server instances require distributed ownership/queues and are explicitly outside this MVP.

## 10. Ordered implementation milestones

- [ ] **Foundation/content:** Create workspaces, strict TypeScript config, scripts, database migrations, environment schema, pure domain types, club search, content importer/validator, and sourced seed records. Test aliases, full careers, fake validation, distinct cards, and safe views. Deliver documented local install/seed/start commands.
- [ ] **Single-player:** Implement guest identity, persisted five-puzzle sessions, game-picked fake, responsive solver/reveal/recap, Next, replay, refresh/resume, and reports. Verify one real browser session completes all five without an account; ensure no answers leak before submission and no ratings change.
- [ ] **Private multiplayer:** Implement two-person lobby/readiness, five-round assignment, free typed fake plus suggestions, transaction-safe phase transitions, realtime snapshots, shared timers, scoring, reveal, recap, rematch, disconnect/forfeit, and restart voiding. Verify two independent browser contexts complete and rematch a game.
- [ ] **Accounts/ranked:** Implement Better Auth integration, local verification/reset mail, queue matching/cancel/expiry, provisional Elo ledger, ladder/profile, and ranked-result finalization. Verify queue → match → five rounds → one rating update → accurate ladder/profile. Verify tied results, forfeits, canceled tickets, and duplicate completion events.
- [ ] **Administration:** Add role-protected content/review/report screens, pair quarantine, immutable publication, review metadata, and tested rating rebuild. Verify users/guests cannot access admin actions, and a void correction changes ledger/stat totals consistently.
- [ ] **Verification/delivery:** Run formatting/lint, typecheck, meaningful unit/integration tests against disposable Postgres, Playwright flows, mobile/accessibility checks, and the 100-player workload. Fix failures and document precise limitations. Build the production bundle and Docker image. Deliver source, local preview, screenshots, operations notes, content provenance, and a checklist of what is complete.

Do not claim a milestone is complete based on placeholder screens, synthetic production careers, or in-memory-only persistence. Do not stop after the first milestone when the full local MVP is still feasible. If a service/environment blocker appears, preserve runnable work and continue independent milestones while clearly reporting the blocker.

## 11. Required acceptance cases

1. A career with N distinct real clubs renders exactly N+1 distinct solver cards, even for long careers.
2. A valid typed fake not in suggestions is accepted; a true club, ambiguous alias without selection, unknown name, excluded association, or stale round is rejected without locking.
3. Shuffling and styling do not reveal the fake's position or how it was chosen.
4. Constructing and solving never exposes the same footballer to the same participant within a match.
5. A guess before cutoff is accepted once; at/after cutoff it is rejected; retries cannot change the answer. Submission/timeout races produce one outcome.
6. Both correct = 1–1; one correct = 2–0; both wrong = 1–1. Solo receives only solve points.
7. Reload/reconnect preserves selected fake, locked answer, server deadline and score; disconnect does not pause clocks.
8. Empty construction uses a validated fallback; absent solve is wrong; post-start leave forfeits; server restart voids active matches without ratings.
9. Stranger sessions cannot view/modify rooms, matches, queues, or solo runs. Full corpus/answers are absent from frontend assets and pre-reveal network payloads.
10. Two concurrent pairing attempts cannot assign the same ticket twice. Cancel/pair races have one consistent final state.
11. A completed ranked match updates both users/ledger exactly once and preserves total rating. Solo, private and voided matches produce no ranked changes.
12. Local signup → email verification → ranked queue works; reset mail and expired-token handling work; admin roles cannot be self-assigned.
13. Content reports quarantine pairs, edits do not mutate active puzzle versions, and rating rebuild skips voided matches deterministically.
14. Five-puzzle solo recap and five-round multiplayer recap agree with stored answers/events. Ten-match ladder eligibility and pagination order are consistent.
15. A 360px mobile viewport supports typing, suggestions, full lists, locked states and reveals; keyboard-only users can complete both modes.

Provide standard root commands: npm run dev, build, lint, typecheck, test, test:integration, test:e2e, test:load, db:migrate, db:seed, content:validate. Make these scripts real and document prerequisites; do not present expected success as executed evidence.

## 12. Explicit boundaries

No ads/payments/pay-to-win, tournaments, party rooms over two players, chat, undisclosed bots, automated paid data subscriptions, badge/photo scraping, native mobile apps, or multiple server instances. These are future possibilities and do not block the specified MVP.

Public content review, actual audience retention, hosting capacity/costs, and commercial/name availability are not established by this plan. Implement the release controls and local functionality; do not assert those external outcomes have been achieved.

## 13. Official reference links

- [Vite React/TypeScript setup and runtime requirements](https://vite.dev/guide/)
- [Node.js release policy](https://nodejs.org/en/about/previous-releases)
- [Fastify WebSocket plugin](https://github.com/fastify/fastify-websocket)
- [Better Auth Fastify integration](https://better-auth.com/docs/integrations/fastify)
- [Better Auth email/password usage](https://better-auth.com/docs/basic-usage)
- [Render WebSocket and reconnect behavior](https://render.com/docs/websocket)
- [Render web services](https://render.com/docs/web-services)
- [Wikidata licence](https://www.wikidata.org/wiki/Wikidata:Licensing) and [team membership property](https://www.wikidata.org/wiki/Property:P54)

Check current official APIs when coding, choose compatible stable dependencies, and freeze the tested versions in the lockfile. These sources support platform/data capabilities, not the originality or commercial prospects of the game.


---

# Appendix: full product specification

# Transfer Scam Football — product specification and delivery plan

Status: implementation handoff, 30 September 2026. The user authorized filling the remaining gaps for Claude to begin implementation. Explicit user preferences remain requirements; remaining defaults have been selected below and in the accompanying implementation-brief.md. This is a specification, not evidence that software or football datasets have already been built. If engineering details differ, the handoff's exact constants/contracts take precedence.

Confirmed preferences: show every real club in the player's career plus one fake; let multiplayer builders type their own scam club or tap suggestions; include single-player with a game-supplied fake; count senior first-team competitive appearances including loans and exclude youth/reserve teams; keep the beta free while testing retention. Selected defaults: simultaneous five-round multiplayer, five suggestions, untimed five-puzzle solo runs, English mobile-first browser UI, and a local-first TypeScript implementation.

## 1. Product goal

Build a football browser game that is easy to explain, amusing to play, and suitable for short competitive matches. Two players create false club-career clues for each other and try to spot the other's fake. The public beta includes matchmaking and a ranked ladder. The longer-term goal is an audience and possible revenue.

The signature experience is choosing a believable lie and discovering whether the opponent believed it. The quiz mechanism has precedents; absolute originality is not established. Player-created traps are the proposed distinction.

## 2. Core match

- Two human players; five simultaneous rounds per match.
- Each round assigns two distinct footballers of broadly comparable recognition and career complexity. Each participant receives one footballer for constructing the opponent's puzzle.
- A footballer is not reused within the match. Players cannot construct and solve a puzzle about the same person in that match: constructing it reveals the real career.
- Construction lasts 25 seconds to allow typing. The builder sees every verified real club in the assigned footballer's qualifying career and can type a real club name or tap one of five valid suggestions. Both paths use the same validation and confirmation.
- The solver sees the footballer's name and all distinct real club cards plus exactly one fake, shuffled. A career with N distinct qualifying clubs produces N+1 cards. No dates or indication of which card was inserted. Do not truncate a long career to a subset of clubs.
- Solving lasts 20 seconds if the longer of the two real careers has at most six distinct clubs; 30 seconds for seven to twelve; 40 seconds above twelve. Both players receive the same round deadline. One selection, locked immediately. A wrong answer cannot be retried.
- Solvers cannot see the opponent's answer, success, or chosen card before both have locked or the deadline passes.
- Reveal lasts approximately seven seconds, with the inserted club, chosen answer, explanation, both scores, and a source link for the real career. An answer-report action is available.
- One point for identifying the fake; one point for fooling the opponent. Each player can earn zero, one, or two points in a round. Maximum ten per match.
- Equal totals produce a draw. Speed is not a scoring factor in the first experiment. Adding sudden death is a later rule decision, not necessary for ratings to work.
- Typical duration: approximately 4–5 minutes including introductions and results; long-career matches can take about six minutes.

Implement these initial settings as server-side configuration with defaults in the handoff. Tune them after play-tests without changing core game semantics or active matches.

Because full careers vary in length, pair puzzles with similar club counts and editorial recognition levels, then comparable observed difficulty as data accumulates. The timer tier depends on the longer career and applies equally to both. Use an accessible responsive grid and scrolling when necessary; every club remains available and long careers are never truncated.

## 3. Typed scam clubs and optional suggestions

Typing is a first-class choice: the builder can select a real club beyond the suggestions. Search a club catalog using canonical names and aliases, resolve the entry to one club ID, then confirm it. Show five optional suggestions. Suggestions are not mandatory.

Validate both paths on the server: the club must be recognized, distinct from every qualifying real career club, and eligible under the verified career's content policy. If an alias is ambiguous, require selection of the exact club. Unrecognized input receives a brief explanation and keeps suggestions available; arbitrary invented club names are not accepted. Selecting a real career club receives an inline explanation, and the builder can try again before the deadline. A valid suggestion fills the same pending selection and uses the same confirmation as typing.

Do not require custom choices to appear on the shortlist. For a complete, reviewed career, the server can validate a recognized custom club against the full career and any recorded ambiguity exclusions. Where the career cannot support a confident decision, mark the custom pair as unverified and keep it out of the round rather than declaring absence proof. Track rejected custom choices to improve catalog coverage and content verification.

The builder is deliberately shown the true information: deception should be the challenge during construction. The solver sees identical card styling regardless of whether the fake was typed or suggested, with no input-method indicator.

The first version lets the builder choose only the fake club, not the true clubs, placement, dates, typography, or player. Otherwise players could create unfairly obscure boards or leak the answer through presentation.

Randomized card positions must be independent of which card is false. Player names and club names use consistent rendering. Start with text cards; add badges and photographs only when suitable asset sources and usage terms are established.

## 4. Football content policy

Initial player pool: recognisable footballers whose careers intersect approximately 2000 onward, initially centered on major European clubs. A career must include earlier and later qualifying clubs too; 2000 is a player-selection preference, not a cutoff that deletes real clubs.

Confirmed rule: a real club counts if the player made a senior first-team competitive appearance, including a loan appearance. Exclude youth teams, reserve teams, national teams, trials, unofficial friendlies, and signings without a qualifying appearance. Count a club once even if the player returned. Document club renames and successor-team ambiguities explicitly.

This strict appearance rule requires evidence beyond a list of teams. Curate records manually where structured sources do not support the definition reliably. Keep the agreed definition and exclude unverified players rather than substituting membership for appearances.

Use retired players first where practical to reduce changes, alongside selected active players with a clear verification date. Pause affected content after a new transfer until checked. A transfer announcement alone does not establish a competitive appearance.

## 5. Data preparation and quality

Seed at least 30 manually checked players for the local MVP, with five reviewed fake suggestions each and at least 150 real clubs in the searchable catalog. Expand toward 100–200 players after play-testing. These are content-production targets, not already verified datasets or a guarantee of sufficient repeat-play variety.

For every player record, retain identity, display names and aliases, source references, verification date, career clubs, qualifying appearance evidence, and confidence/status. Club records include stable IDs, display names, aliases, country, and identity notes.

Store reviewed suggestion pairs and ambiguous/disallowed pairs separately. Typed fakes need not be preselected suggestions: validate their club identity against the complete reviewed career and ambiguity policy. An absent entry in an incomplete database is not proof that a player never appeared for a club. Verify career completeness and likely confusing cases, including loans and similarly named clubs, before enabling these puzzles.

The initial eligibility rule is at least three distinct qualifying clubs. Generate puzzles only from approved complete-career records. Validate that all presented cards have distinct club IDs, the real-card set equals the entire verified qualifying career, exactly one additional card is fake, and the chosen typed or suggested fake passes server-side validation. Freeze the content version used by each match so later edits do not alter an ongoing result.

Candidate fake clubs should be plausible: comparable era, geography, or football context. Editors should assess the shortlist; country or league similarity alone does not guarantee a convincing lie. Avoid controversial examples until the rules and data can resolve them.

A report includes the match, round, player, disputed club, content version, and reason. Reviewers can suspend a puzzle and correct its source record. Repeated substantiated reports trigger a review of the related dataset, rather than one isolated patch.

Require a second content review before a puzzle enters public ranked play. For a confirmed ranked content error, suspend the pair immediately, mark affected matches void, and rebuild the rating ledger chronologically excluding them. Preserve original and corrected results in the audit log. Do not merely subtract an old rating delta from the current rating.

### Source research

[Wikidata's structured data is CC0](https://www.wikidata.org/wiki/Wikidata:Licensing), making it a useful starting point for identities and career candidates. However, [P54 describes sports-team membership](https://www.wikidata.org/wiki/Property:P54); it is not, by itself, proof of a senior competitive appearance. References and qualifiers are part of [the statement model](https://www.wikidata.org/wiki/Help:Statements), and coverage needs auditing. Do not assume photos, badges, or article prose share the structured-data licence.

[Sportmonks documents player career histories](https://www.sportmonks.com/blogs/creating-player-profiles-with-sportmonks-football-api-and-wordpress/) and may be a paid enrichment source. Evaluate historical coverage and the exact intended use against [its current terms](https://www.sportmonks.com/terms-of-service/) before selecting it. Its [image guidance](https://www.sportmonks.com/integrity-support/) treats image rights separately from statistical data.

[API-Football's guide](https://www.api-football.com/news/post/how-to-get-started-with-api-football-the-complete-beginners-guide) documents player and match statistics, but [its terms](https://www.api-football.com/terms) expressly do not supply a publication licence. Do not assume purchasing API access establishes the permissions needed for a public commercial game.

Official archives can verify individual real-club claims; for example, [Barcelona's Ronaldo archive](https://players.fcbarcelona.com/en/player/764-ronaldo-ronaldo-luis-nazario-da-lima) distinguishes official and unofficial appearances. This is useful evidence, not a universal complete-career database or permission to reuse visual assets.

No paid API responses or systematic career-completeness benchmark were tested for this specification. Use Wikidata for club identities and career candidates, then manually sourced appearance evidence and complete-career review for the seed corpus. No paid provider is needed to begin implementation; paid enrichment requires a later coverage and cost decision.

## 6. Modes and launch scope

Initial playable prototype: single-player and private friend matches with the same verified content policy. Friend matches include guest names, invite links/codes, construction, solving, reveals, results, rematch, and content reports.

Public beta: registered identities, private rooms, a single ranked matchmaking queue, rating updates, leaderboard, placement experience, reconnect handling, and moderation tools. Ranked matchmaking and the ladder are part of the intended public product, not permanently deferred extras.

Single-player is a confirmed initial mode, not a later optional feature. The game selects a footballer and a verified fake club and presents all real clubs plus that fake, shuffled. There is no construction phase or simulated opponent. The player selects one club, locks the answer, sees the correct fake and career explanation, and continues to another puzzle.

Solo format: five puzzles per run, unlimited runs, no required account, and no timer. Score one point per correct selection, maximum five. Keep single-player scores separate from the multiplayer rating and ladder. Show correct and incorrect answers in the run recap and provide Play Again and Play With a Friend actions.

Generate solo puzzles on the server from the same approved career corpus and reviewed fake suggestions used by multiplayer. Do not send the answer before the selection locks. A solo run has a persistent session ID and each puzzle a fixed content version; refreshing resumes the current puzzle and cannot change an already locked answer. Avoid repeated footballers within a run and avoid recent repeats across runs when sufficient content exists. Solo solves and rating updates use separate completion paths so practice can never change a ranked rating.

Defer multiple regional queues, chat, tournaments, spectator mode, user-created footballers, currencies, and purchased power-ups. Split queues only when actual concurrency supports them.

## 7. Player flows and screens

1. Home: one-sentence explanation, quick example, Single Player, Play With Friend, and Play Ranked when the ranked beta is available.
2. Entry: immediate guest access to single-player, guest entry for friend rooms, and account creation/sign-in for ranked.
3. Lobby: opponent, ready state, match format, and compact explanation of what counts as a club.
4. Queue: elapsed wait, cancellation, and honest availability feedback.
5. Construction: assigned player, all true career-club cards, searchable club-name input, five suggestion buttons, inline validation, countdown, and confirmed selection.
6. Solve: opponent's assigned player, all true career-club cards plus one fake shuffled together, countdown, single locked selection.
7. Reveal: fake club, selected club, points, short verified career explanation, report action.
8. Results: multiplayer win/loss/draw, round recap, rating change if ranked, rematch, and queue again. Solo has a correct-answer score, puzzle recap, Play Again, and Play With a Friend.
9. Ladder/profile: rating, games, wins/draws/losses, and separate solve/fool rates. Do not imply accuracy alone equals rank.
10. Content administration: records, references, approved fakes, reports, suspensions, and version history.

Design for phones first, keyboard access on desktop, readable timers, visible focus, reduced-motion support, and feedback that does not depend only on colors. Do not build a time-sensitive game around long scrolling lists.

## 8. Technical boundaries

Selected stack: Node.js 24 LTS, TypeScript in an npm-workspace monorepo, React with Vite for the client, Fastify with its WebSocket plugin for the server, PostgreSQL with Drizzle for persistence, Better Auth for accounts, Vitest for unit/integration tests, and Playwright for browser verification. Use PostgreSQL in Docker Compose locally. Commit a lockfile and pin tested package versions at implementation rather than installing unverified prereleases. The architecture has these boundaries:

- Browser client renders the authoritative match state and sends actions.
- Match service owns phases, deadlines, assignments, choices, validation, and scoring.
- Content service retrieves approved, versioned puzzles and separate builder/solver views.
- Matchmaking service manages queue tickets, ratings, and opponent pairing.
- Persistence stores content, users, match events/results, ratings, and reports.
- Administrative interface manages content approval and disputed answers.
- Monitoring records failures, timing, queue availability, and product events.

Use authenticated WebSocket subscriptions for state updates and HTTP endpoints for idempotent actions. Server timestamps determine all deadlines. Client countdowns are displays, not enforcement. Start with a single server instance; persist state and events in PostgreSQL. A server restart voids unfinished multiplayer matches without rating changes and preserves solo runs. Redis and multiple match servers are outside the initial scope.

State progression: lobby → construction → solve → reveal, repeated five times → completed. Canceled and forfeited are explicit terminal states. Persist enough state/events to reconnect and audit results; a page refresh cannot restart a round.

Solo progression: session creation → solve → reveal, repeated five times → recap. Reuse puzzle validation, answer checking, and reveal components while keeping solo progression independent of the multiplayer match lifecycle, matchmaking, and ratings.

Do not send the correct answer, complete career, or construction choices for a player's solver puzzle to that client before reveal. Separate response schemas are essential. Builder access is limited to the puzzle assigned for constructing the opponent's challenge.

## 9. Reliability and edge cases

- A missing construction choice receives a predefined server-selected fallback when the deadline passes. It cannot extend the round or expose future content. Repeated stalling is recorded.
- Missing solve submissions are wrong answers. Lock the first valid submission; retries cannot change it.
- A brief disconnection does not pause shared clocks. Reconnection restores the current state and remaining server time.
- Disconnection has a 30-second grace period, without pausing phase clocks. If only one participant remains disconnected beyond the grace period, they forfeit. Both disconnected yields a void match. Explicit leave during a started match forfeits immediately. Show this policy before entering ranked.
- If the server fails or the match becomes inconsistent, void the match and do not update ratings.
- A canceled queue ticket cannot still create a match unnoticed. A user may have at most one active ranked match or queue ticket.
- Replayed requests, duplicate event delivery, simultaneous timeout/submission, and duplicate completion events must produce one result and one rating update.
- Escape display names, rate-limit relevant actions, and use access checks for every room/match action. Private room codes need protection against guessing.

## 10. Fairness and ratings

Begin at rating 1000. Use Elo with K=24 for both players, expectation 1/(1+10^((opponentRating-rating)/400)), and results 1 for win, 0.5 for draw, 0 for loss. Round the winner/first player's delta to the nearest integer and apply its exact opposite to the other player, preserving total rating. First ten valid ranked matches are provisional; players appear on the main ladder after ten. Persist both updates and the unique match ledger entry atomically.

Use one global queue: tolerance ±100 for the first 15 seconds, ±200 until 30 seconds, ±350 until 60 seconds, then ±600. Both tickets' current tolerances must accept the pair. Within eligible pairs prioritize oldest waiting tickets, then closest ratings. Ignore opponents from the last ten minutes while alternatives exist; relax that avoidance after 60 seconds. At 120 seconds show that the queue is quiet and offer solo/private play or continued waiting, without canceling automatically or inserting an undisclosed bot.

Rematches in friend rooms are unranked. Avoid repeatedly matching the same pair in ranked where alternatives exist; record repeated pairs and suspicious outcomes for review.

Assignments must avoid systematic player-pool advantages. Stratify puzzles by recognition, real-club count, and observed difficulty; test balance empirically. An attacker learning the true career before constructing their puzzle is intentional, but learning the opponent's answer is not.

Do not rely on a speed tiebreaker to compensate for unequal puzzle difficulty. Browser lookup cannot be completely prevented; keep answer data private, keep rounds short, and review implausible behavior rather than claiming cheat-proof play.

## 11. Verification plan

Content checks: qualifying-club definitions, loans, repeated spells, aliases, renamed clubs, suggestion pairs, typed-club validation, ambiguity exclusions, source completeness, and active-player changes. Verify that valid custom clubs outside the shortlist are accepted, true clubs are rejected, ambiguous aliases need resolution, unknown clubs do not become invented answers, and both input methods produce identical solver cards.

Game-engine tests: each legal phase/action; scoring all four multiplayer success/failure combinations; deadlines; locked answers; fallback construction; five-round termination; draw; and round data visibility. Solo tests cover exactly one supplied fake, the entire real career, one locked selection, one point per correct answer, five-puzzle completion, no repeated footballer within a run, refresh/resume, hidden answers before reveal, and no ranked-rating changes.

Concurrency tests: submission exactly at cutoff, duplicate submission, disconnected players, simultaneous readiness, queue cancellation racing pairing, and duplicate result processing.

End-to-end checks: a guest opens single-player, solves five puzzles, sees a correct recap, and starts another run; refreshing preserves state. Two separate browser sessions join a room, construct different puzzles, solve, reveal, finish, and rematch. For ranked, verify queue → match → result → exactly one rating update → ladder display.

Device checks: a narrow phone screen, desktop keyboard-only play, touch input, slow connection, refresh/reconnect, long club names, long complete careers without truncated clubs, and reduced motion.

Security checks: a player cannot access another puzzle's answer before reveal, join an unauthorized room, submit after a deadline, edit a locked answer, or alter scores/ratings from the client.

Initial load-test target: 50 simultaneous multiplayer matches (100 connected players) on the single server, including constructions, submissions, reveals, and results. Record actual hardware, latency percentiles, errors, and memory; this target is a validation workload, not a production capacity claim.

## 12. Delivery stages and acceptance gates

Stage A — foundation and content. Implement validation and prepare sourced real/fake career boards. Manual play-tests can run alongside engineering to assess rule comprehension, trap choices, and rematch interest. These observations guide later tuning; they are not an additional approval gate before the authorized local implementation.

Stage B — playable solo and friends prototype. Deliver the single-player solve/reveal/run loop first, then the entire five-round private-match loop using the same initial verified dataset. Verify guest solo runs and separate-device friend games, timers, reveals, reconnects, and reports before building on them. Record qualitative feedback without making another user interview a prerequisite for the ranked implementation. Single-player is useful even before two people are online and does not wait for matchmaking infrastructure.

Stage C — ranked beta. Add accounts, queue, ratings, ladder, abandonment policy, administration, and monitoring. Advance only after full ranked-flow verification, fairness review, and scheduled sessions demonstrate enough participants to get actual matches.

Stage D — audience experiment. Launch to a small football community, arrange a shared play window, invite feedback, and test challenge sharing. Avoid broad promotion before onboarding and rematches are dependable.

Stage E — commercial experiment. The beta is confirmed free while validating retention. Test revenue only after repeat play exists. Choose the later commercial model explicitly; do not assume an audience or income from the existence of competitors.

Implement the ordered milestones in implementation-brief.md: foundation/content, solo, private realtime matches, accounts/ranked, administration/verification. Public deployment follows separately; do not create paid resources or publish automatically. The entire local feature set remains the implementation objective, even though solo/private flows should be completed first.

## 13. Product measurement

Track anonymous or consented aggregate events consistent with the chosen privacy approach: solo run start/completion/replay, entering a room/queue, match start, completed construction, solve submission, reveal, match completion, rematch request, share, and return session. Label each event by mode so solo retention and multiplayer retention are measured separately.

Evaluate rule comprehension, construction choice distribution, solve/fool rates, round abandonment, match completion, rematch rate, queue wait percentiles, disconnects, and next-day/next-week returns. Define denominators; a rematch request is distinct from a completed rematch.

Review whether the same fake club wins too often, whether builders choose randomly, and whether unfamiliar true clubs make every choice feel arbitrary. Small play-tests provide observations, not statistical proof. Set numeric go/no-go targets after a baseline pilot rather than inventing universal success thresholds.

## 14. Audience and monetization proposals

Start with one language and one reachable football audience. The first outreach message should demonstrate a believable trap and invite a friend to beat it. Share results without exposing unrevealed puzzles. Scheduled beta sessions can concentrate players enough to test matchmaking.

Keep ranked rules, clue access, and scoring equal for all participants. The beta will be free while validating retention. Proposed later revenue options: optional cosmetics, supporter features outside competitive advantage, or ads outside timed rounds. Select one after measuring repeat play and operating costs; the later revenue model remains an open decision.

Operating cost planning should include hosting, realtime concurrency, database/authentication, monitoring, asset rights, and editorial verification time. Actual service choices and costs require current research once budget and stack are selected.

## 15. Implementation decisions and release boundaries

The remaining defaults have been selected for implementation: English, mobile-first web, the TypeScript stack above, five-round simultaneous matches, five suggestions plus typed choices, 25-second construction, shared tiered solve timers, seven-second reveals, untimed five-puzzle solo runs, the Elo and queue rules above, and 30-second disconnection grace.

Implementation begins locally without paid external services. Public hosting target: a Dockerized Render web service with PostgreSQL, one server instance and one origin for client/API/WebSocket traffic. Hosting purchase, domain choice, and public deployment are later operational actions, not blockers for local development. No speculative fixed delivery date or spending amount is implied.

All MVP screens, schema/API contracts, security boundaries, task order, and required checks are specified in implementation-brief.md. Start implementation with those documents; ask only if a genuine environment limitation or conflict prevents progress. Later product tuning, branding availability, revenue model, and additional content are outside the first implementation and do not require repeated design approval before coding.
