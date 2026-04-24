# Changelog

All notable changes to Sprint Poker will be documented here.

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versioning: [SemVer](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- (nothing yet)

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

[Unreleased]: https://github.com/ytap1/SprintPoker/compare/v0.1.1...HEAD
[0.1.1]: https://github.com/ytap1/SprintPoker/compare/v0.1.0...v0.1.1
[0.1.0]: https://github.com/ytap1/SprintPoker/releases/tag/v0.1.0
