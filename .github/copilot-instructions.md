# GitHub Copilot Instructions — serious-game-team-dynamics

## Project Context

Real-time facilitated workshop tool for team dynamics. Facilitator creates a game room; participants join via a 6-character code. Activities (Trust Spectrum, Role Blindspot Bingo) run over WebSockets with anonymous responses and shared debriefs.

**Stack**: React 18 + TypeScript (Vite) · Node.js + Express + `ws` · Zustand (client state) · Vitest (client tests) · Jest (server tests)

---

## TypeScript Conventions

- `strict: true` everywhere — never suggest `any` or `@ts-ignore` without a `// reason:` comment
- Use `interface` for object shapes, `type` for unions
- Explicit return types on all exported functions and React components
- Validate incoming WebSocket messages with runtime type guards before using them
- Use `unknown` for untyped external data; narrow with `if` checks or zod

## React Conventions

- Functional components only; one per file
- Props interface named `<Component>Props` in the same file
- Custom hooks in `src/client/hooks/`, always named `use<Name>.ts`
- No business logic in components — derive state in hooks, render in components
- No inline styles; use CSS Modules or a utility class library

## Server Conventions

- WS message handlers must be pure functions: `(room, participant, payload) => ServerMessage[]`
- All state lives in `RoomManager`; handlers do not mutate state directly
- `src/server/types/messages.ts` is the single source of truth for the WS protocol — always use those types
- Use `async/await`; no `.then()` chains

## Testing Conventions

- **Write tests before implementation** (red → green)
- Client: Vitest + React Testing Library — `screen.getByRole()`, no snapshots, no `querySelector`
- Server: Jest — unit test game logic without WS, integration test with a real WS server in `beforeAll`
- Mock the WebSocket in client hook tests; do not make real network calls in unit tests

## Boundaries — Do NOT

- Do not refactor code outside the scope of the current task
- Do not remove or comment out tests
- Do not introduce new npm dependencies without noting them in your PR description
- Do not add `console.log` to production code paths
- Do not suggest storing participant PII — display names are ephemeral
- Do not suggest breaking the anonymity model (no participant-to-response mapping exposed to clients)

## File Naming

| Type              | Convention         |
|-------------------|--------------------|
| React components  | `kebab-case.tsx`   |
| Hooks             | `use-name.ts`      |
| Server handlers   | `name-handler.ts`  |
| Types files       | `types.ts` or `name.types.ts` |
| Test files        | `*.test.ts` / `*.test.tsx` |
