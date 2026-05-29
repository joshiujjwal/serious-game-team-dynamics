# serious-game-team-dynamics — Task Breakdown

## How to Use This File

Work one task at a time. For every task:
1. **Write tests FIRST** — make them fail (red phase)
2. **Implement** until tests pass (green phase)
3. **Review the diff** manually before committing
4. **Commit** with a descriptive message
5. **Update CLAUDE.md / AGENTS.md** if you learned something worth preserving (compound loop)

Do not proceed to the next phase until all tasks in the current phase have passing tests and a human has reviewed the diff.

---

## Phase 0: Foundation ⬜

- [ ] Initialise monorepo workspace (npm workspaces or turborepo) with `packages/client` and `packages/server`
- [ ] Configure TypeScript (`tsconfig.json`) for both client and server with strict mode
- [ ] Set up Vite for client with React 18 + TypeScript template
- [ ] Set up ts-node-dev (or tsx) for server hot-reload
- [ ] Add ESLint + Prettier config shared across packages
- [ ] Add Vitest for client, Jest for server — write a smoke test in each that asserts `1 + 1 === 2`
- [ ] GitHub Actions CI: lint + test on push to `main` and on all PRs
- [ ] Review all AI-generated config files (CLAUDE.md, AGENTS.md, this file) and adjust to reality

**Evidence gate**: CI is green, smoke tests pass, lint reports zero errors.

---

## Phase 1: WebSocket Backbone ⬜

- [ ] Define shared TypeScript message types in `src/server/types/messages.ts` (ClientMessage, ServerMessage unions)
- [ ] Write server tests for: room creation, participant join, duplicate name rejection, disconnect cleanup
- [ ] Implement `RoomManager` — creates rooms with 6-char codes, tracks participants, handles disconnects
- [ ] Implement WS server in Express (`/ws` upgrade path) with message router dispatching to handlers
- [ ] Write client tests for `useWebSocket` hook: connects, sends messages, receives messages, reconnects on drop
- [ ] Implement `useWebSocket` hook — manages WS lifecycle, exposes `send()` and `onMessage` callback
- [ ] Write client tests for `useRoom` hook: join room, see participant list update in real time
- [ ] Implement `useRoom` hook — wraps `useWebSocket`, handles `JOIN` / `PARTICIPANT_UPDATE` messages

**Evidence gate**: Server tests pass. Client hook tests pass. Manual test: two browser tabs can join the same room and both see each other's names appear.

---

## Phase 2: Facilitator Flow ⬜

- [ ] Write tests for `FacilitatorPage`: renders room code, shows participant list, start-activity button disabled until ≥ 2 participants
- [ ] Implement `FacilitatorPage` — create room on mount, display room code as large QR-friendly text, participant roster
- [ ] Write tests for `ParticipantPage`: join form, name validation (non-empty, ≤ 20 chars, unique), waiting state
- [ ] Implement `ParticipantPage` — join form → waiting lobby → activity view
- [ ] Write tests for activity state machine: idle → briefing → active → debrief → idle
- [ ] Implement `ActivityStateMachine` on server — facilitator triggers transitions, broadcasts state to all participants
- [ ] Write tests for `useGameState` hook: reflects server activity state, handles all transitions
- [ ] Implement `useGameState` hook on client — subscribes to state broadcast, exposes current phase

**Evidence gate**: Facilitator can create room, participants can join, facilitator can advance through activity phases and all clients reflect the correct phase in real time.

---

## Phase 3: First Activity — "Trust Spectrum" ⬜

- [ ] Document "Trust Spectrum" rules in `docs/spec.md` under Activities section
- [ ] Write server tests for `TrustSpectrumActivity`: prompt delivery, vote collection, reveal timing, scoring
- [ ] Implement `TrustSpectrumActivity` — facilitator picks a statement; each participant places themselves on a 1–5 spectrum anonymously; results revealed after all submit or timer expires
- [ ] Write client tests for `TrustSpectrumCard` component: renders prompt, slider input, submit button, disabled after submit
- [ ] Implement `TrustSpectrumCard` — shows prompt + interactive spectrum slider + submit
- [ ] Write client tests for `ResultsHeatmap` component: renders aggregated spectrum positions
- [ ] Implement `ResultsHeatmap` — visual bar/heatmap of where the team clustered
- [ ] Write client tests for `DebriefView`: shows results, facilitator-only "next" button
- [ ] Implement `DebriefView` — results + facilitator controls

**Evidence gate**: Full end-to-end run of Trust Spectrum with 3+ participants. Screenshot of results heatmap included in PR.

---

## Phase 4: Second Activity — "Role Blindspot Bingo" ⬜

- [ ] Document activity rules in `docs/spec.md`
- [ ] Write server tests for `RoleBlindspotActivity`: bingo card generation (unique per participant), submission, completion detection
- [ ] Implement `RoleBlindspotBingoActivity` on server
- [ ] Write client tests for `BingoCard` component: renders 5×5 grid, marks cells, detects bingo
- [ ] Implement `BingoCard` component
- [ ] Implement debrief for this activity — show which blindspots were most commonly marked

**Evidence gate**: Activity runs end-to-end with 2+ participants. Tests pass.

---

## Phase 5: Polish & Harden ⬜

- [ ] Add reconnect logic: participant rejoins within 60 s and their state is restored
- [ ] Add facilitator kick-participant capability
- [ ] Add timer component shown to all participants during active phase
- [ ] Add mobile-responsive CSS (no horizontal scroll on 375px viewport)
- [ ] Add rate limiting on WS message handling (max 10 msg/s per connection)
- [ ] Write load test: 30 concurrent participants in one room, all submitting within 5 s — p99 < 200 ms
- [ ] Accessibility audit: all interactive elements keyboard-accessible and have ARIA labels

**Evidence gate**: Load test passes. Lighthouse accessibility score ≥ 90.

---

## Phase 6: Ship ⬜

- [ ] Write `docker-compose.yml` for local full-stack run
- [ ] Write `Dockerfile` for server
- [ ] Add deployment docs to `docs/deploy.md` (Railway / Render / Fly.io recommended)
- [ ] Confirm all environment variables documented in `.env.example`
- [ ] Final end-to-end test with real users (not just localhost)
- [ ] Tag `v0.1.0` release

**Evidence gate**: Someone outside the dev team runs a full workshop session using the deployed URL.

---

## Parking Lot 🅿️

- Third activity ideas: "Decision Driver Cards", "Conflict Style Sorter", "Psychological Safety Pulse"
- Persistent session history (so teams can compare results over time)
- Facilitator guide PDF export per session
- Auth layer (magic link email) for returning facilitators
- Spectator mode (read-only view for observers)

---

## Lessons Learned 📝

_Update this section whenever you hit a non-obvious problem or find a better pattern._

| Date | Lesson |
|------|--------|
| —    | (none yet) |
