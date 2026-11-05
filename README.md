# Mao — a table of consequences

A runnable, single-process Node / Socket.IO web card game with a vanilla JavaScript front end, shared browser/server engine, editable rule library, SQLite persistence and cached offline hotseat mode.

## Run

```sh
npm ci
npm run dev
# http://localhost:3000
```

Development editor: `/dev`, default passphrase **local-mao**. Set `DEV_PASSPHRASE` before making it public. No accounts or cloud credentials are required locally. See [DEPLOY.md](DEPLOY.md) for Docker, Northflank and Render.

## What works

- Home, join/create lobby, private invitation links, public board (including FULL rooms), waiting room and table routes.
- 2-player start minimum; 6 seats with 54 cards / 12 with 108; clear refusal at capacity and blocked downward deck changes.
- Chairman controls: start, kick, transfer, difficulty, deck and visibility. Role reassignment and 90-second disconnect grace.
- HMAC-signed sessions in localStorage; refreshing restores the authoritative hand. Opponent cards and deck order are omitted from normal snapshots.
- Shared seeded shuffle, matching, poker shape validation, maximum four of a rank, enabled-combo filtering, basic card effects, draw/death stacking, consumable counters, poison deadlines, timed shields, cap checks after every drawn card, exhausted-deck reshuffle, last-standing / empty-hand win checks.
- Mao declaration and server-serialised miss calls; chat phrase requirements and automatic +2 penalties; point of order; Chairman +2 penalty console with reason.
- Blind one-card trade intents and a basic UI. Both players select a card before completion; pending trade snapshots conceal the offered card. Decline intent exists but the UI is not complete.
- Position-based server-side deck reorder intent and basic UI: never sends deck order to online players. This is deliberately more restrictive than a card-aware rigging UI.
- Rule 17 six removal and timed, announced developer visibility. Authenticated developer swap intent exists; full swap UI is not implemented.
- Oval felt table, real card proportions and indexes, opponent backs capped at five, counts/lives/status rail, discard history long-press, responsive overlapping hand calculations, keyboard card selection, chat/journal and mobile tappable seat dots.
- Single master rule library; six ID-only selections with configuration overrides; Library / Rule sets / Offline test tabs; import/export; nesting warnings and fix; bulk seed/copy/reset.
- Rules persisted to SQLite using sql.js. Data/config writes flush to disk. Persist `DATA_DIR` on a mounted volume for restart durability.
- PWA cache after an initial online visit. Offline cached `/dev` reload and hotseat launch tested with browser network disabled. Hotseat hides hands behind a pass-to-player interstitial.
- HumanDriver / BotDriver interface scaffolding; Bot option disabled, BotDriver throws `not implemented`. Driver integration still needs refinement before AI can be added without turn-loop changes.
- Node 22 slim non-root Docker image, Render blueprint, health endpoint, configuration example and deployment instructions.

## Important: not the full Definition of Done

This implementation is a playable foundation, **not a claim of complete enforcement of every rule in the supplied specification**. The following are remaining work:

1. Full-house effects are represented in the library/rules page, but are not fully enforced. The exact 99JJ and 44555 special effects are not enforced.
2. Forbidden City separate deck, emperor protections and slavery ownership/redirection are not implemented. Their descriptions are present.
3. Civil war is WIP, default off, and not implemented as a playable faction mode. The prompt refers to four open contradictions but does not enumerate them; no rulings have been invented.
4. Rule 14 ordered hand-reveal sequence, trips-four life theft/heart trap, pair-six recurring draws, dictated-suit durations, communist redistribution/removal of kings, challenge windows and all full-house condition/card-event duration types need implementation. The current status engine supports timed deadlines and basic condition-based status only.
5. Damage resolution is partial. It is not yet a full ordered annihilation → poison → life-loss pipeline with every specified reflection, ownership redirect and immunity edge case.
6. Pair-ace self-draw/previous discard and several non-additive specials are incomplete. Full-house and longer poker hands currently reuse grouped rank effects rather than their complete dedicated resolution.
7. Rigging UI accepts hidden position permutations rather than a card-aware ordering workflow; quads-eight discard rebuild and Forbidden City dual-deck rigging are incomplete.
8. Developer visibility is implemented under Rule 17; complete table-wide swap controls are not. Rule 17 in two-deck mode removes all but one six rather than defining a new eight-six interpretation.
9. Match win totals are tracked and a third win announced; host end-session and match-completion lockout are not implemented.
10. Offline turn timers/phrase deadlines are not driven while the handoff interstitial is open. The shared engine has a `tick` function but the offline UI does not yet schedule it. Disconnect timers are an online feature.
11. Some advanced intents are intentionally rejected as unavailable. Card selection has keyboard support, but there is not yet a full keyboard-only seat/target chooser. Touch selection is implemented; long-press hand magnification needs more testing.
12. The landing page is a designed hero rather than the strictly three-element centred layout in the brief. The log is collapsible on mobile, always open on desktop.
13. No external cloud deployment has been performed. The live preview is the development sandbox, not a provisioned Northflank service. Render free storage is ephemeral, and Northflank volume pricing must be checked before promising a zero-cost persistent deployment.

Two-deck cap: the source gives 25 (§9.1), 26 (§7.7), and “over 26” elsewhere. The seed explicitly chooses **26**, following the mechanics section. It is editable through configuration.

## Rule authoring

A rule is inert prose unless its configuration uses a supported engine primitive. Rules are never executable code; no `eval` or imported scripts. A custom definition can participate in existing rank/count triggers without an engine edit, for example:

```json
{
  "id": "custom-eight",
  "name": "Eight makes you wait",
  "description": "A single eight makes the next player draw two.",
  "tags": ["extra"],
  "defaultConfig": {
    "trigger": {"rank": "8", "count": 1},
    "effect": {"draw": 2}
  }
}
```

Include its ID in a selection and save. New rounds resolve the live library without restart. Available effect keys are documented in the editor. Entirely new mechanical concepts still require a new interpreter primitive: arbitrary prose cannot be automatically enforced. Multiple same-trigger effect configs currently merge in library order, with later keys overriding earlier ones.

## Tests actually run

```sh
npm test
node checks/integration.mjs # requires running server
npx playwright install --with-deps chromium
node checks/browser.mjs     # requires running server
node checks/offline.mjs     # requires running server
```

- Unit checks: deck arithmetic, private snapshots, deterministic opening shuffle, rank ceiling, forged cards, turn authority, Mao call race, cap spare-life trim, empty piles.
- Fuzz: **200 games completed**, 2–6 players, Easy/Medium, single-card strategy. Phrase queues are cleared in this harness to isolate turn/effect resolution. This does not validate advanced-rule compliance or production balance.
- Live transport: 54/108 seat limits, one-player start refusal, downward-size refusal, duplicate names, private snapshots, session rejoin with identical hand.
- Chromium: two independent browser profiles create/join/start, render six-card hands and restore after refresh; zero page errors; no horizontal body overflow at 390px.
- Chromium offline: after cache installation and editor unlock, disable network, reload `/dev`, launch hotseat, pass interstitial and render hand; zero page errors.

Still needed: two physical machines finishing a full round, all 12-seat layouts, a dedicated 25-card mobile reachability assertion, full advanced-mechanic tests, security/rate-limit review and deployment smoke tests.

## Files

- `server.js`: HTTP, WebSocket intents, room/session lifecycle, SQLite settings.
- `shared/engine.js`: shared rules/shuffle/intents/private snapshot engine.
- `public/app.js`, `public/style.css`: vanilla client and card/table rendering.
- `public/sw.js`: offline cache.
- `data/seed.json`: authoritative default library and selections.
- `data/make-seed.cjs`: seed generation utility (not needed at runtime).
- `tests/`, `checks/`: unit, fuzz and live/browser checks.

Single process only. Rooms intentionally do not survive restart. Do not horizontally scale without adding room-code routing and an appropriate storage/coordination design.
