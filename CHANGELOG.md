# Changelog

All notable changes to Sprint Poker will be documented here.

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versioning: [SemVer](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- (nothing yet)

## [0.3.1] — 2026-04-24

### Added
- "Copy Join Link" button in QR modal: copies the full `?code=` join URL to clipboard; button text changes to "✓ Copied!" for 2 seconds then reverts

### Fixed
- Removed stale `console.log` debug statement left in `nextIssue()`

## [0.3.0] — 2026-04-24

### Added
- QR code for room joining: facilitator room-code badge opens a modal with a QR code encoding `https://ytap1.github.io/SprintPoker/?code=SP-XXXXXX`; scanning on mobile pre-fills the room code and focuses the name field
- `?code=` URL parameter support: opening the join URL directly pre-fills `#join-code` and focuses `#join-name` so mobile users only need to enter their name
- qrcode.js 1.0.0 via cdnjs (ADR-0007)

### Changed (mobile / responsive overhaul)
- Header: `#conn-label` and `#my-name-label` hidden on mobile; room code + phase pill remain — fits in one line at 390 px
- Lobby: `min-height` removed; hero `h1` scales to 24 px; panels full-width with 20 px padding
- All tap targets set to `min-height: 44px`; form inputs get `font-size: 16px` to prevent iOS auto-zoom on focus
- Game grid: sidebar (team + backlog) now renders **below** main on mobile (`order: 2`), so the issue card and voting deck are immediately visible after scrolling
- Vote cards: `52 × 74px`, `gap: 8px`, `touch-action: manipulation` for fast tap response
- Results: stat boxes stack vertically with label and value in a row; action buttons stack full-width

## [0.2.0] — 2026-04-24

### Added
- `PEER_CONFIG` object at the top of the script — one place to configure the signalling server and ICE/TURN servers; no hunting through code to switch providers
- `buildPeerOpts()` helper wires `PEER_CONFIG` into both `hostRoom()` and `joinRoom()`; hardcoded `{ debug: 0 }` replaced in both
- Self-hosted PeerJS signalling support: set `PEER_CONFIG.signalling` to `{ host, port, path, secure }` to point at a Railway/Render instance; `null` keeps the current PeerJS free-cloud default
- TURN relay config: `PEER_CONFIG.iceServers` ships Google STUN by default; commented examples for Metered and Cloudflare Calls TURN relays are ready to uncomment
- Export CSV: facilitator-only "↓ Export Results CSV" button in the backlog panel, visible once at least one issue is pointed; downloads `sprint-results-<roomcode>.csv` with columns `#`, `Title`, `Points`; values with commas/quotes are properly escaped
- ADR-0006: documents the configurable signalling/TURN decision

## [0.1.1] — 2026-04-24

### Fixed
- Facilitator peer reconnection: `peer.on('disconnected')` now calls `peer.reconnect()` to survive transient signalling drops; `peer.on('open')` guarded so re-open on reconnect doesn't re-enter the game
- Recoverable PeerJS error types (`network`, `server-error`) also trigger reconnect with a status indicator; fatal errors toast and re-enable the Create button
- `#btn-create` re-enabled and reset to "+ Create Room" on any fatal peer error (mirrors the Bug 9 fix for `#btn-join`)
- Responsive layout: `@media (max-width: 768px)` collapses `game-grid` to a single column, moves sidebar above main, stacks lobby panels vertically, shrinks vote cards to 52 × 74 px, and wraps the header cleanly

## [0.1.0] — 2026-04-24

Initial working release of Sprint Poker.

### Added
- Single-file app (`index.html`) — CSS, HTML, and JS inline, no build step
- Real-time multiplayer via PeerJS 1.5.4 WebRTC data channels
- Facilitator role: create room, manage backlog, start voting, reveal cards, accept points
- Participant role: join by room code, cast and retract votes, see live results
- Fibonacci deck (1, 2, 3, 5, 8, 13, 21, ?, ☕) and T-Shirt deck (XS, S, M, L, XL, XXL, ?, ☕)
- Configurable per-round countdown timer (30 s – 3 min)
- Backlog panel: add issues manually or import from CSV
- Reveal phase: average, consensus indicator, vote distribution bar chart
- Done phase: completion screen after last issue is estimated
- `resetSession()`: facilitator can restart the session without leaving the room
- Join Room 10-second timeout with clear error message and button recovery
- Connection status indicator (connecting / connected / disconnected) in header
- Room code click-to-copy
- Phase pill in header (waiting / voting / revealed / done)
- `HANDOVER.md`, `SPRINT_POKER_GUIDE.md`, `.ai/` documentation

### Fixed
- Blank game screen after creating a room (`display: ''` fell back to CSS `display: none` — fixed to `display: 'block'`)
- Results panel never visible after reveal (same root cause, same fix)
- Race condition: room code shown before PeerJS peer ID was registered, causing `peer-unavailable` for fast joiners — `enterGame()` restored inside `peer.on('open')`
- No done phase when backlog exhausted — `nextIssue()` else branch now sets `gamePhase = 'done'` before `renderAll()`
- Accept & Next button did nothing on the last issue — `JSON.stringify(suggestedPts)` inside a template literal produced `onclick="nextIssue("5")"`, breaking the HTML attribute; fixed using `addEventListener` with a closure
- Join button permanently stuck on "JOINING…" — all error paths (peer error, connection error, 10 s timeout) now re-enable the button and show a descriptive message
- PeerJS host errors after lobby was hidden were silently dropped — changed from `showError('create-error', …)` to `toast(…)`

[Unreleased]: https://github.com/ytap1/SprintPoker/compare/v0.3.1...HEAD
[0.3.1]: https://github.com/ytap1/SprintPoker/compare/v0.3.0...v0.3.1
[0.3.0]: https://github.com/ytap1/SprintPoker/compare/v0.2.0...v0.3.0
[0.2.0]: https://github.com/ytap1/SprintPoker/compare/v0.1.1...v0.2.0
[0.1.1]: https://github.com/ytap1/SprintPoker/compare/v0.1.0...v0.1.1
[0.1.0]: https://github.com/ytap1/SprintPoker/releases/tag/v0.1.0
