# Sprint Poker — Handover Summary

## 1. What the App Does

Sprint Poker is a serverless, real-time planning poker tool for agile teams. One user creates a room as the **facilitator** and receives a short room code (`SP-XXXXXX`). Other participants enter that code on their own devices to join the same live session. The facilitator manages a **backlog** of issues (typed manually or imported via CSV), starts timed voting rounds, reveals cards, accepts an estimated point value, and advances to the next issue. When all issues are estimated the session enters a **done** phase with a "Start new session" reset. All state is held in the facilitator's browser and broadcast to connected clients over WebRTC data channels — no server, no database, no sign-in.

---

## 2. File Structure and Tech Stack

| Layer | Detail |
|---|---|
| **Single file** | `index.html` — CSS, HTML, and JS all inline, no build step |
| **WebRTC signalling** | PeerJS 1.5.4 via CDN (`unpkg.com/peerjs@1.5.4`) |
| **ICE / STUN** | Google STUN by default; TURN credentials added via `PEER_CONFIG.iceServers` |
| **Signalling server** | PeerJS free cloud (`0.peerjs.com`) by default; swap via `PEER_CONFIG.signalling` in `index.html` |
| **Hosting** | GitHub Pages — auto-deploys on push to `main` |
| **Framework** | None — vanilla JS, CSS custom properties |

### Key JS globals

| Symbol | Role |
|---|---|
| `state` | Single source of truth for all app state |
| `peer` | PeerJS `Peer` instance (facilitator or client) |
| `facilitatorConn` | Client-side `DataConnection` to the facilitator |
| `clientConns` | Facilitator-side map of `{ name → DataConnection }` |

### Message protocol (facilitator ↔ clients)

| Type | Direction | Purpose |
|---|---|---|
| `JOIN` | client → facilitator | Register participant on join |
| `JOIN_ERROR` | facilitator → client | Reject join (e.g. duplicate name) |
| `VOTE` | client → facilitator | Submit or retract a vote |
| `SYNC` | facilitator → all clients | Full state snapshot after any change |

---

## 3. Current State (as of 2026-04-27)

App is stable at v0.3.1 + two post-release fixes (commit `73b588a`). All originally-scoped bugs are resolved. Infrastructure items (self-hosted PeerJS, TURN relay) were deliberately deferred — risks explained and accepted. Next work is likely a new feature or a reported bug.

### Files to look at first
- `index.html:774` — `PEER_CONFIG` (signalling + ICE config)
- `index.html:937` — `handleFacilitatorMessage` (JOIN/VOTE/JOIN_ERROR logic)
- `index.html:957` — `handleClientMessage` (SYNC/JOIN_ERROR handling)

---

## 4. Known Remaining Issues / Limitations

### Self-hosted PeerJS signalling (deferred)
`PEER_CONFIG.signalling` is `null` by default (uses `0.peerjs.com`). To switch to a self-hosted server:

**Railway (recommended):**
1. Fork or clone [peer-server](https://github.com/peers/peerjs-server) on GitHub.
2. In Railway dashboard → New Project → Deploy from GitHub repo.
3. Set the `PORT` env var (Railway injects `$PORT` automatically via the Procfile).
4. Copy the generated URL (e.g. `your-app.railway.app`).
5. In `PEER_CONFIG.signalling` in `index.html`: `{ host: 'your-app.railway.app', port: 443, path: '/', secure: true }`.
6. Push to `main`.

**When to act:** only if `0.peerjs.com` downtime becomes a real blocker. Fallback for now: Teams poll or reschedule.

### TURN relay (deferred — privacy tradeoff accepted)
`PEER_CONFIG.iceServers` ships Google STUN only. TURN routes traffic through a third-party relay (encrypted but still off-device). Decision: accept symmetric-NAT edge case failures rather than relay traffic externally. Revisit only if NAT failures are actively reported by the team.

### Name collision
~~`clientConns[name] = conn` overwrote the connection on duplicate names~~ ✓ Fixed `73b588a` — facilitator now sends `JOIN_ERROR` and client bounces back to lobby.

### Timer drift
Each peer runs its own `setInterval` countdown after the initial SYNC. No re-sync during a voting round, so timers can drift across devices over a long session.

### No session persistence
All state is in-memory. A page refresh ejects the user. No reconnection mechanism.

### Host tab must stay open
Closing or refreshing the facilitator's tab ends the session for everyone.

---

## 5. Recommended Next Steps

| Priority | Item | Notes |
|---|---|---|
| — | Self-hosted PeerJS | Deferred — act only if `0.peerjs.com` is a real problem |
| — | TURN relay | Deferred — privacy tradeoff accepted |
| Low | Fix timer drift | Each peer runs its own `setInterval`; re-sync timer on SYNC during active round |
| Low | No session persistence | Page refresh ejects user; no reconnection mechanism |

---

## 6. Reflection (2026-04-27)

### What went well
- `/review` ran at session start and caught the clipboard false-positive immediately — the process worked
- Both fixes were clean and minimal, no scope creep
- Decision-making was fast — deferred items were evaluated and closed without over-deliberating
- Step-by-step protocol held: fix → verify → commit → confirm remote → push
- Deployed URL test confirmed both fixes before moving on

### What to improve
- **Numbering confusion** — the session pending list used different numbers than the HANDOVER; caused "#3 vs #4" back-and-forth mid-session. Keep one canonical numbered list and don't renumber across sessions.
- **CHANGELOG gap** — two fixes shipped without updating `CHANGELOG.md` until handover; caught at the end. Rule: update CHANGELOG before committing, not after.
- **Review timing** — the clipboard fix was already committed in a prior session before `/review` caught it. `/review` should run before committing, not at the start of the next session.
