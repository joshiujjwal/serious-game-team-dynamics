# CLAUDE.md — serious-game-team-dynamics

AI agent context file. Keep this under 200 lines. Update it when you discover non-obvious truths about this codebase.

---

## What This Project Is

Browser-based facilitated workshop tool. A facilitator hosts a room; participants join via 6-char code. Activities run in real time over WebSockets. React + TypeScript frontend, Node.js + WebSocket backend.

---

## Commands

```bash
# Install all dependencies (run from root)
npm install                    # TODO: confirm once monorepo is wired

# Development (runs client + server concurrently)
npm run dev

# Client only (Vite, default http://localhost:5173)
npm run dev:client

# Server only (ts-node-dev, default ws://localhost:3001)
npm run dev:server

# All tests
npm test

# Client tests only (Vitest)
npm run test:client

# Server tests only (Jest)
npm run test:server

# Lint (ESLint + Prettier check)
npm run lint

# Type check (no emit)
npm run typecheck

# Build for production
npm run build
```

> **Always run `npm test` before starting work.** If tests were already failing, don't make them worse.

---

## Directory Map

```
src/client/
  components/   Reusable UI — no business logic here
  hooks/        Custom hooks: useWebSocket, useRoom, useGameState
  pages/        Route-level views: FacilitatorPage, ParticipantPage, ResultsPage
  types/        Client-side TypeScript interfaces (no runtime code)

src/server/
  handlers/     One file per WS message type (joinHandler, startActivityHandler, etc.)
  game/         Pure business logic — RoomManager, ActivityStateMachine, activities/
  types/        Server-side TypeScript interfaces shared with message protocol

tests/client/   Mirrors src/client/ — Vitest + React Testing Library
tests/server/   Mirrors src/server/ — Jest, no DOM
```

---

## Architecture Decisions

- **WebSocket only for game state** — REST is only for room creation and health check. All in-game events are WS messages.
- **Shared message types** — `src/server/types/messages.ts` is the single source of truth for the WS protocol. Client imports from there (or a shared `packages/shared` if monorepo).
- **Anonymous responses** — The server never sends a `participantId → response` mapping to clients. Only aggregated results are broadcast.
- **In-memory state first** — No database. Room state lives in a `Map` in `RoomManager`. Redis is a Phase 5 upgrade if needed.
- **Activity as a class** — Each activity (TrustSpectrum, RoleBlindspotBingo) is a class implementing `IActivity`. The state machine delegates to the current activity instance.

---

## Non-Obvious Conventions

- **Message handlers are pure functions** — they receive `(room: Room, participant: Participant, payload: unknown)` and return `ServerMessage[]` to broadcast. Side effects (writing to `RoomManager`) happen in the WS router, not in handlers.
- **Vitest for client, Jest for server** — don't mix them. Vitest config is in `vite.config.ts`; Jest config is in `jest.config.ts` at root or in `packages/server/`.
- **No `any` in message handlers** — validate incoming WS payloads with a runtime check (zod or manual) before casting. Never trust `ws.on('message')` data.
- **Component tests use React Testing Library** — no Enzyme, no snapshot tests. Test behaviour, not markup.

---

## Workflow

1. `git pull` — get latest
2. `npm test` — confirm baseline is green
3. Read `TODO.md` — find the next unchecked task in the current phase
4. Write failing tests (red)
5. Implement until tests pass (green)
6. `npm run lint && npm run typecheck` — must be clean
7. Review diff manually
8. `git commit -m "feat: <what you did>"`
9. If you discovered something non-obvious, update this file

---

## Things Claude Should NOT Do

- Do not refactor code that isn't directly related to the current task
- Do not remove or skip tests to make CI green
- Do not add `// @ts-ignore` or `as any` without a comment explaining why
- Do not introduce a new dependency without checking `package.json` first
- Do not add console.log statements to production code paths
