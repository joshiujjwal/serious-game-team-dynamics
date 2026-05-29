# AGENTS.md — serious-game-team-dynamics

## Setup

```bash
# From repo root
npm install

# Verify environment
npm run typecheck   # must exit 0
npm test            # must exit 0 (or document known failures)
npm run lint        # must exit 0
```

## Project Overview

Real-time facilitated workshop tool. Facilitator creates a room; participants join via 6-char code. Game activities run over WebSockets. Stack: React 18 + TypeScript (Vite) on the client, Node.js + Express + ws on the server.

---

## Code Style

### TypeScript
- `strict: true` in all tsconfig files — no exceptions
- Prefer `interface` over `type` for object shapes; use `type` for unions and aliases
- No `any` — use `unknown` and narrow explicitly. Add a comment if you cast
- Explicit return types on all exported functions
- Use `readonly` on function parameters that should not be mutated

### React
- Functional components only — no class components
- One component per file; filename matches component name (PascalCase)
- Props interfaces named `<ComponentName>Props`
- Hooks in `src/client/hooks/` — named `use<Name>.ts`
- No inline styles — use CSS modules or a utility-class library (TBD in Phase 0)
- Do not fetch data inside components — use hooks, keep components dumb

### Node.js / Server
- All async functions use `async/await` — no raw Promise chains
- WS message handlers are pure: `(room, participant, payload) => ServerMessage[]`
- Validate every incoming WS message before processing — use zod or manual type guards
- Errors are typed — throw `AppError` instances, not raw `Error` with string parsing

### Naming
| Thing           | Convention         | Example                        |
|-----------------|--------------------|--------------------------------|
| Files (client)  | kebab-case         | `trust-spectrum-card.tsx`      |
| Components      | PascalCase         | `TrustSpectrumCard`            |
| Hooks           | camelCase + `use`  | `useGameState`                 |
| Server handlers | camelCase + Handler | `joinHandler`                 |
| Types/Interfaces | PascalCase        | `ActivityState`, `ClientMessage` |
| Constants       | UPPER_SNAKE_CASE   | `MAX_PARTICIPANTS`             |

---

## Testing

### Philosophy
- **Red first**: Write failing tests before writing implementation code
- **Green fast**: Implement the simplest thing that makes the test pass
- **Refactor clean**: Clean up only after tests are green

### Client (Vitest + React Testing Library)
```bash
npm run test:client          # run once
npm run test:client -- --watch  # watch mode
```
- Test files: `tests/client/**/*.test.tsx`
- Test behaviour, not implementation: use `screen.getByRole`, not `querySelector`
- Mock WebSocket in hook tests using `vi.mock` or a fake WS implementation
- Do not use snapshot tests

### Server (Jest)
```bash
npm run test:server          # run once
npm run test:server -- --watch  # watch mode
```
- Test files: `tests/server/**/*.test.ts`
- Unit test `RoomManager` and activity classes without a live WS connection
- Integration tests use `ws` client connecting to a real `http.Server` spun up in `beforeAll`
- Clean up rooms in `afterEach` to prevent test pollution

### Coverage expectation
- All game logic in `src/server/game/` must have ≥ 80% branch coverage
- Client hooks must have ≥ 70% branch coverage

---

## Pull Request Instructions

1. **Title format**: `<type>: <short description>` — types: `feat`, `fix`, `test`, `refactor`, `docs`, `chore`
2. **Evidence required**: include one of:
   - Test output (`npm test` result)
   - Screenshot (for UI changes)
   - Load test result (for performance claims)
3. **Review the diff yourself** before marking ready for review — read every line
4. **One logical change per PR** — don't bundle unrelated fixes
5. **No PRs that only add tests** are bad — but no PRs that only add features without tests are worse
6. Keep PRs under 400 lines of diff where possible; split larger changes into a stack

---

## Key Files to Understand First

| File                            | Why it matters                              |
|---------------------------------|---------------------------------------------|
| `src/server/types/messages.ts`  | WS protocol — source of truth for all messages |
| `src/server/game/RoomManager.ts` | Core server state — room lifecycle         |
| `src/server/game/ActivityStateMachine.ts` | Controls all phase transitions    |
| `src/client/hooks/useWebSocket.ts` | Client WS lifecycle                      |
| `src/client/hooks/useRoom.ts`   | Client room state derived from WS events    |
| `docs/spec.md`                  | Full feature spec and data model            |
| `TODO.md`                       | Current task breakdown with evidence gates  |
