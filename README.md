# serious-game-team-dynamics

> 🚧 **Status: Early Development**

A **browser-based facilitated workshop tool** that uses serious game mechanics to surface, reflect on, and improve team dynamics. A facilitator runs structured activities in real time; participants join from any device via a shared room code.

---

## Tech Stack

| Layer      | Technology                            |
|------------|---------------------------------------|
| Frontend   | React 18 + TypeScript + Vite          |
| Realtime   | WebSockets (ws / Socket.IO)           |
| Backend    | Node.js + Express + TypeScript        |
| State      | Zustand (client) / in-memory + Redis (server) |
| Testing    | Vitest (client) + Jest (server)       |
| Lint/Format| ESLint + Prettier                     |
| CI         | GitHub Actions                        |

---

## Getting Started

```bash
# 1. Clone
git clone https://github.com/joshiujjwal/serious-game-team-dynamics.git
cd serious-game-team-dynamics

# 2. Install dependencies
npm install          # root (workspace)
# TODO: confirm monorepo vs separate package.json per package

# 3. Run in development
npm run dev          # starts both client (Vite) and server (ts-node-dev)

# 4. Run tests
npm test             # all tests
npm run test:client  # Vitest only
npm run test:server  # Jest only

# 5. Lint
npm run lint
```

---

## Project Structure

```
serious-game-team-dynamics/
├── src/
│   ├── client/              # React + TypeScript SPA
│   │   ├── components/      # Reusable UI components
│   │   ├── hooks/           # Custom React hooks (useRoom, useGame, useWS)
│   │   ├── pages/           # Route-level pages (Lobby, Activity, Results)
│   │   └── types/           # Shared client-side TypeScript types
│   └── server/              # Node.js WebSocket + REST server
│       ├── handlers/        # WS message handlers (join, start, vote, etc.)
│       ├── game/            # Game logic: state machine, scoring, activities
│       └── types/           # Shared server-side TypeScript types
├── tests/
│   ├── client/              # Vitest unit + component tests
│   └── server/              # Jest unit + integration tests
├── docs/
│   ├── spec.md              # Feature specification
│   └── adr/                 # Architecture Decision Records
├── .github/
│   └── copilot-instructions.md
├── CLAUDE.md                # AI agent context
├── AGENTS.md                # OpenAI-style agent instructions
└── TODO.md                  # Evidence-gated task breakdown
```

---

## Core Concepts

| Concept       | Description                                                              |
|---------------|--------------------------------------------------------------------------|
| **Room**      | A session created by the facilitator; participants join via 6-char code  |
| **Activity**  | A timed game unit (e.g. "Trust Spectrum", "Role Blindspot Bingo")        |
| **Round**     | One iteration within an activity                                         |
| **Debrief**   | Post-activity reflection phase with visible aggregated results           |

---

## Contributing

- **Test first, always**: write failing tests before implementation (red → green)
- **Small PRs**: one logical change per PR
- **Evidence required**: PRs must include test output or screenshot proving the feature works
- **Review AI descriptions**: read the diff, don't just approve what Copilot wrote
- **No dead code**: if you remove a feature, remove its tests too
