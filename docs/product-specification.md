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
