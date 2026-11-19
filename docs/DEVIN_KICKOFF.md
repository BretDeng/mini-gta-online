# Devin Kickoff Prompt — mini-gta-online (Milestone 1: Authoritative Multiplayer Core)

## Objective
Bootstrap the `mini-gta-online` monorepo and implement a verifiable, load-tested authoritative multiplayer game server core in Go with a minimal TypeScript debug client.

## Repository Structure
- `/proto`: Protobuf schemas (`game.proto`)
- `/server`: Go 1.23+ authoritative game server
- `/client`: Minimal Vite + TypeScript HTML5 Canvas debug client
- `/loadtest`: Headless Go bot simulator
- `docker-compose.yml` & `Makefile` at root

## Technical Requirements
1. **Protobuf Protocol (`/proto/game.proto`)**
   - Define `ClientInput` (seq, tick, movement vector, action flags), `EntityState` (id, x, y, vx, vy, rotation), `DeltaSnapshot`, `FullSnapshot`, `Heartbeat`, and `ReconnectRequest` (session_token, last_ack_tick).
   - Provide a `Makefile` target `make proto` to compile Go and TS bindings.

2. **Hybrid Transport & 20Hz Authoritative Tick Loop (`/server`)**
   - WebSockets (`/ws`) for WebRTC SDP/ICE signaling, session auth, and automatic fallback.
   - Pion WebRTC (`github.com/pion/webrtc/v4`) unordered/unreliable DataChannel for high-frequency `ClientInput` and `DeltaSnapshot` sync.
   - Fixed 20Hz (50ms) server tick loop running authoritative movement & collision bounds.

3. **9-Grid AOI (Area of Interest)**
   - Partition the open map into a uniform spatial grid.
   - Broadcast `DeltaSnapshot` updates ONLY to clients within the 9-cell neighborhood (current cell + 8 adjacent cells).
   - Include unit tests covering entity cell transitions (enter/leave AOI events).

4. **Disconnect Grace Period & Redis State Recovery**
   - On heartbeat timeout (>5s), keep the player entity in a 30s grace period and persist `(session_token, entity_state, last_ack_tick)` to Redis.
   - On `ReconnectRequest` with valid token within 30s, rebind the transport and push a `FullSnapshot` of the player's current 9-grid AOI.

5. **Load Testing & GitHub Actions CI (`.github/workflows/ci.yml`)**
   - Build a headless bot tool (`/loadtest`) that spawns 100 concurrent bots performing random movement plus 15% simulated abrupt disconnects and reconnects.
   - CI MUST fail unless:
     - All unit & reconnect integration tests pass (`go test -race ./...`)
     - 100-bot load test completes with Server Tick P99 < 15ms, 100% reconnect recovery rate, and zero goroutine leaks.

## Deliverable
Open a single draft PR for Task 1 first (scaffolding + `game.proto` + `docker-compose.yml`), wait for CI green, then proceed sequentially through Tasks 2–4.
