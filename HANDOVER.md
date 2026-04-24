# Sprint Poker — Handover Summary

## 1. What the App Does

Sprint Poker is a serverless, real-time planning poker tool for agile teams. One user creates a room as the **facilitator** and receives a short room code (`SP-XXXXXX`). Other participants enter that code on their own devices to join the same live session. The facilitator manages a **backlog** of issues (typed manually or imported via CSV), starts timed voting rounds, reveals cards, accepts an estimated point value, and advances to the next issue. When all issues are estimated the session enters a **done** phase with a "Start new session" reset. All state is held in the facilitator's browser and broadcast to connected clients over WebRTC data channels — no server, no database, no sign-in.

---

## 2. File Structure and Tech Stack

| Layer | Detail |
|---|---|
| **Single file** | `index.html` — CSS, HTML, and JS all inline, no build step |
| **WebRTC signalling** | PeerJS 1.5.4 via CDN (`unpkg.com/peerjs@1.5.4`) |
| **ICE / STUN** | PeerJS default (Google STUN `stun.l.google.com:19302`) |
| **Signalling server** | PeerJS free cloud (`0.peerjs.com`) |
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
| `VOTE` | client → facilitator | Submit or retract a vote |
| `SYNC` | facilitator → all clients | Full state snapshot after any change |

---

## 3. Bugs Fixed This Session

### Bug 1 — Blank game screen after creating a room
**Functions:** `enterGame()`  
`document.getElementById('game').style.display = ''` cleared the inline style, falling back to the CSS rule `#game { display: none }`. The game section was never shown. Fixed by setting `display = 'block'` explicitly.

### Bug 2 — Results panel never visible after reveal
**Functions:** `renderResults()`  
Same root cause: `panel.style.display = ''` fell back to `#results-panel { display: none }`. Fixed to `display = 'block'`.

### Bug 3 — Race condition: clients got "peer-unavailable" on join
**Functions:** `hostRoom()`  
An intermediate attempt to fix Bug 1 moved `enterGame()` to before `new Peer()`. This displayed the room code in the header before the facilitator's peer ID was registered with the PeerJS signalling server. Joiners who acted quickly got a `peer-unavailable` error. Reverted: `enterGame()` is back inside `peer.on('open', ...)` so the room code only appears once the room is truly live.

### Bug 4 — No done phase when backlog is exhausted
**Functions:** `nextIssue()`  
The `else` branch (all issues estimated) called `renderAll()` while `state.gamePhase` was still `'waiting'`. Fixed by setting `state.gamePhase = 'done'` before `renderAll()` and broadcasting SYNC.

### Bug 5 — No completion UI for the done phase
**Functions:** `renderResults()`  
`renderResults()` had no handling for `'done'`. Added a block (checked before the `'revealed'` check) that hides the voting deck, shows "✓ All issues estimated!", and renders a "Start new session" button for the facilitator, or "Session complete" for clients.

### Bug 6 — Missing `resetSession()` function
**Functions:** `resetSession()` (new)  
Added function: sets `gamePhase = 'waiting'`, clears `votes`, resets `currentIssueIdx = -1`, clears all issue `points`, stops the timer, calls `renderAll()`, and broadcasts a full SYNC so all clients reset simultaneously.

### Bug 7 — `suggestedPts` silently read wrong value
**Functions:** `renderResults()`  
`votes[0]` in the `suggestedPts` expression referred to the first element of the local `votes` array — technically correct but fragile. Replaced with `Object.values(state.votes).filter(Boolean)[0]` to make the intent unambiguous and independent of the local variable.

### Bug 8 — Facilitator PeerJS errors shown in hidden lobby
**Functions:** `hostRoom()`  
After `enterGame()`, the lobby `#create-error` element is hidden. The original `showError('create-error', ...)` in `peer.on('error', ...)` silently did nothing. Changed to `toast()` which renders over the game screen.

### Bug 10 — Accept & Next button did nothing on the last issue
**Functions:** `renderResults()`  
`JSON.stringify(suggestedPts)` inside a template literal produced `onclick="nextIssue("5")"` when `suggestedPts` was a string (consensus vote or average). The inner double-quotes closed the HTML attribute early; the browser parsed an invalid handler and silently ignored the click. Fixed by rendering the buttons with plain IDs and attaching handlers via `addEventListener`, passing `suggestedPts` as a real JS closure value with no serialisation.

### Bug 9 — Join button permanently stuck on "JOINING…"
**Functions:** `joinRoom()`, `resetJoinBtn()` (new)  
Any join failure — `peer-unavailable`, WebRTC/ICE error, or signalling timeout — left `#btn-join` disabled with "Joining…" and no recovery path. Three fixes applied:  
- **10-second timeout**: if `facilitatorConn.on('open')` never fires, destroys the peer, shows "Room not found or host disconnected. Check the code and try again.", and re-enables the button.  
- **`peer.on('error')`**: all error types now call `resetJoinBtn()` and show a specific message.  
- **`facilitatorConn.on('error')`**: now calls `resetJoinBtn()` and shows the error instead of silently setting status.  
- **`resetJoinBtn()`** extracted as a shared helper.

---

## 4. Known Remaining Issues / Limitations

### No TURN server
PeerJS uses Google STUN only. Connections across symmetric NATs (corporate networks, some mobile carriers) can fail silently. WebRTC ICE will stall indefinitely with no error message if STUN can't negotiate a path. A TURN relay is required for reliable cross-network connections.

### PeerJS free cloud reliability
`0.peerjs.com` is rate-limited and occasionally unavailable. There is no fallback signalling server configured. Either peer can fail to connect without a clear error surfaced to the user.

### ~~Facilitator peer not reconnected on signalling drop~~ ✓ Fixed in v0.1.1
`peer.on('disconnected')` now calls `peer.reconnect()`. Recoverable error types (`network`, `server-error`) also trigger reconnect. `peer.on('open')` is guarded so a reconnect re-open doesn't re-enter the game.

### ~~Create button also gets stuck~~ ✓ Fixed in v0.1.1
`#btn-create` is now re-enabled and reset to "+ Create Room" on any fatal peer error. Mirrors the Bug 9 fix for `#btn-join`.

### ~~No mobile-responsive game layout~~ ✓ Fixed in v0.1.1
`@media (max-width: 768px)` added: `game-grid` collapses to 1 column, sidebar reorders above main, lobby panels stack vertically, vote cards shrink to 52 × 74 px, header wraps.

### Name collision
`clientConns[name] = conn` in `handleFacilitatorMessage` overwrites the connection reference if two participants join with the same name. The first person's connection is lost.

### Timer drift
Each peer runs its own `setInterval` countdown after the initial SYNC. No re-sync on subsequent SYNCs during a voting round, so timers can drift across devices over a long session.

### No session persistence
All state is in-memory. A page refresh ejects the user from the session. There is no reconnection mechanism.

---

## 5. Recommended Next Steps

~~1. Add facilitator peer reconnection~~ ✓ Done (v0.1.1)

~~2. Fix the Create button~~ ✓ Done (v0.1.1)

~~3. Add responsive layout~~ ✓ Done (v0.1.1)

4. **Use a reliable signalling / TURN setup** — Options (all free tier available): self-hosted `peer-server` on Railway/Render, Metered TURN (free quota), or Cloudflare Calls for TURN relay. Configure via the `config.iceServers` option in the `Peer` constructor.

5. **Guard against name collisions** — Reject a `JOIN` message if the name already exists in `state.participants`, and send an error back to the client before `enterGame()` is called.
