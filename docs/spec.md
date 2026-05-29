# Specification — serious-game-team-dynamics

## Overview

A browser-based **facilitated workshop tool** that uses serious game mechanics to surface and reflect on team dynamics. A facilitator controls the session; participants join from any device using a 6-character room code. Activities are timed, anonymous-by-default, and followed by structured debrief phases.

**Problem statement**: Teams rarely get structured opportunities to discuss trust, role clarity, communication patterns, and conflict styles. Off-the-shelf team-building activities are often low-fidelity or not psychologically safe. This tool provides a lightweight, facilitator-guided experience that generates real conversation from real (anonymised) team data.

---

## Functional Requirements

### Room & Session Management
- [ ] Facilitator creates a room; receives a unique 6-character room code
- [ ] Participants join via room code + display name (unique within room, ≤ 20 chars)
- [ ] Facilitator sees live participant roster; can remove participants
- [ ] Room expires after 4 hours of inactivity or explicit close by facilitator
- [ ] Participants who disconnect within 60 s can rejoin and restore their in-progress state

### Facilitator Controls
- [ ] Facilitator sees a dashboard: participant list, current activity phase, timer
- [ ] Facilitator selects which activity to run from a library
- [ ] Facilitator advances through phases: Briefing → Active → Debrief → (next activity or close)
- [ ] Facilitator can set a countdown timer per activity (default: 3 min)
- [ ] Facilitator can reveal/hide results before the debrief discussion

### Participant Experience
- [ ] Participant sees activity prompt, submits response before timer expires
- [ ] Submission is acknowledged; participant waits for others or for facilitator to advance
- [ ] During debrief, participant sees aggregated (anonymous) results
- [ ] Participant cannot see other individuals' raw responses

### Activities

#### Activity 1: Trust Spectrum
- Facilitator selects a statement (e.g. "I feel comfortable asking for help in this team")
- Each participant places themselves on a 1–5 scale anonymously
- After all submit (or timer expires), results shown as aggregated heatmap
- Debrief prompt shown to facilitator: "Where is the team clustered? What does the spread tell us?"

#### Activity 2: Role Blindspot Bingo
- Each participant receives a unique 5×5 bingo card of role-related behaviours (e.g. "Takes credit for shared work", "Avoids conflict")
- Participants mark cells that feel relevant to their team
- When a row/column/diagonal is complete, participant gets "bingo"
- Debrief: facilitator sees which cells were most frequently marked across the team

### Non-Functional Requirements
- [ ] Supports 30 concurrent participants per room without degradation
- [ ] WebSocket message round-trip p99 < 200 ms on a 10 Mbps connection
- [ ] All interactive elements keyboard-accessible (WCAG 2.1 AA)
- [ ] Works on Chrome, Firefox, Safari (latest 2 versions), iOS Safari, Android Chrome
- [ ] No participant PII stored — display names are ephemeral and not persisted after session ends

---

## Data Model

### Server-side (in-memory, optionally Redis-backed)

```typescript
interface Room {
  code: string;            // 6-char alphanumeric
  facilitatorId: string;   // WS connection ID
  participants: Map<string, Participant>;
  currentActivity: ActivityState | null;
  createdAt: Date;
  lastActiveAt: Date;
}

interface Participant {
  id: string;              // WS connection ID
  displayName: string;
  joinedAt: Date;
  connected: boolean;
}

interface ActivityState {
  type: ActivityType;      // 'TRUST_SPECTRUM' | 'ROLE_BLINDSPOT_BINGO'
  phase: ActivityPhase;    // 'BRIEFING' | 'ACTIVE' | 'DEBRIEF' | 'COMPLETE'
  config: ActivityConfig;
  responses: Map<string, unknown>; // participantId → response
  startedAt: Date;
  timerDuration: number;   // seconds
}
```

### WebSocket Message Protocol

```typescript
// Client → Server
type ClientMessage =
  | { type: 'JOIN'; roomCode: string; displayName: string }
  | { type: 'CREATE_ROOM' }
  | { type: 'START_ACTIVITY'; activityType: ActivityType; config: ActivityConfig }
  | { type: 'ADVANCE_PHASE' }
  | { type: 'SUBMIT_RESPONSE'; payload: unknown }
  | { type: 'KICK_PARTICIPANT'; participantId: string }

// Server → Client
type ServerMessage =
  | { type: 'ROOM_CREATED'; roomCode: string }
  | { type: 'JOIN_SUCCESS'; roomCode: string; participants: ParticipantSummary[] }
  | { type: 'JOIN_ERROR'; reason: string }
  | { type: 'PARTICIPANT_UPDATE'; participants: ParticipantSummary[] }
  | { type: 'ACTIVITY_STATE'; state: ActivityStateSummary }
  | { type: 'SUBMIT_ACK' }
  | { type: 'RESULTS'; data: AggregatedResults }
  | { type: 'ERROR'; message: string }
```

---

## API / Interface Design

### REST (setup only — main interaction is WebSocket)

| Method | Path         | Description                        |
|--------|--------------|------------------------------------|
| POST   | `/api/rooms` | Create room (returns room code)    |
| GET    | `/api/health` | Health check                      |

### WebSocket

- Endpoint: `ws://<host>/ws`
- All messages are JSON-serialised `ClientMessage` / `ServerMessage` objects
- Server validates every incoming message against the TypeScript type definitions

---

## Test Plan

### Unit Tests

| Module                  | Cases to cover                                                           |
|-------------------------|--------------------------------------------------------------------------|
| `RoomManager`           | Create room, join room, duplicate name, disconnect cleanup, room expiry  |
| `ActivityStateMachine`  | All phase transitions, invalid transitions rejected, timer expiry        |
| `TrustSpectrumActivity` | Response collection, partial submission + timer expiry, aggregation      |
| `RoleBlindspotActivity` | Card generation (uniqueness), bingo detection, aggregation               |
| `useWebSocket`          | Connect, send, receive, reconnect on drop                                |
| `useRoom`               | Join, participant list updates, error states                             |
| `TrustSpectrumCard`     | Render, slider interaction, submit, disabled after submit                |
| `BingoCard`             | Render grid, mark cell, bingo detection                                  |
| `ResultsHeatmap`        | Renders correct bar heights from aggregated data                         |

### Integration Tests

- Full WS round-trip: facilitator creates room → 2 participants join → facilitator starts activity → both submit → facilitator advances to debrief → results visible to all
- Reconnect: participant disconnects mid-activity, reconnects within 60 s, sees correct state

### Edge Cases

- Participant submits response after timer expires → rejected with `ERROR` message
- Facilitator advances phase with 0 submissions → allowed, show empty debrief
- 31 participants attempt to join a room capped at 30 → 31st gets `JOIN_ERROR`
- Two participants submit identical display names → second gets `JOIN_ERROR: name taken`

---

## Open Questions

- [ ] Should room state survive a server restart? (Redis persistence vs. accept data loss)
- [ ] Do we need a facilitator authentication flow, or is possession of the room code enough?
- [ ] Should activities be configurable via a YAML/JSON definition file to allow community contributions?
- [ ] What's the right anonymity model for very small teams (≤ 3 people) where aggregation doesn't hide individuals?
- [ ] Mobile: tap-based spectrum slider vs. numeric input — which is more accurate on small screens?
