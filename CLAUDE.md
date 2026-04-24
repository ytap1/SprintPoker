# CLAUDE.md

Read this first. Then `.ai/context.md`, then `.ai/conventions.md`.

## What This Project Is

**Sprint Poker** is a serverless, real-time planning poker tool for agile sprint estimation. One user creates a room as facilitator; others join by room code. All state lives in the facilitator's browser and syncs to clients over WebRTC data channels via PeerJS. No server, no database, no login.

**Single file:** everything is in `index.html` — CSS, HTML, and vanilla JS inline. No build step, no dependencies to install.

**Stack:** Vanilla HTML/CSS/JS · PeerJS 1.5.4 (WebRTC) · GitHub Pages (auto-deploy on push to `main`)

## Working Model (non-negotiable)

- **iPad/iPhone + Safari only.** No terminal, no laptop.
- **No package managers, no localhost, no CLI instructions in docs.**
- **Push directly to `main`** for routine changes. Branch only for destructive/high-risk work (mass renames, deletions, structural rewrites).
- **Deploy auto-triggers on push to `main`** via GitHub Pages. Verify on the deployed URL, never localhost.
- **Multi-chat workflow.** Each chat covers one concern (App Dev, Docs, etc.). Stay in the lane named in the current chat.

## Known Limitations

- **Host tab must stay open.** All state is in the facilitator's browser. Closing or refreshing ejects everyone.
- **PeerJS free broker.** `0.peerjs.com` is rate-limited and occasionally unreliable. No fallback server configured.
- **No TURN server.** PeerJS uses Google STUN only. Connections across symmetric NATs (corporate WiFi, some mobile carriers) can fail silently.
- **Facilitator peer not reconnected on signalling drop.** If the WebSocket to the PeerJS signalling server times out, the room becomes unjoinable with no warning.
- **No session persistence.** A page refresh ejects the user. No reconnection mechanism.
- **Name collisions.** Two participants with the same name overwrite each other in `clientConns`.

## Coding Conventions

- **Never use `element.style.display = ''`** to show an element that has `display: none` in CSS — it falls back to the CSS rule and stays hidden. Always set an explicit value (`'block'`, `'flex'`, etc.).
- **Never put dynamic JS values in inline `onclick` attributes** via template literals. String values serialised with `JSON.stringify` break the HTML attribute (e.g. `onclick="fn("5")"`). Use `addEventListener` with a closure instead.
- **State is mutated then rendered.** Always update `state.*` first, then call `renderAll()` or the specific render function, then `broadcastToClients(SYNC)`. Never render before mutating.
- **Facilitator is authoritative.** Only the facilitator mutates shared state. Clients only send `VOTE` and `JOIN` messages; all other state changes come back via `SYNC`.
- **`console.log` is acceptable for debugging** but should be removed before calling a feature stable.
- **No new top-level dependencies** without an ADR in `.ai/decisions.md`.

## Token Discipline

- Short, accurate prose beats long, hedged prose.
- No restating obvious context back to the user.
- No CLI walkthroughs (the user can't run them).
- Trim every doc you touch.

## Read in Order

1. [.ai/context.md](./.ai/context.md) — what this is, current state, stack
2. [.ai/conventions.md](./.ai/conventions.md) — coding rules
3. [.ai/decisions.md](./.ai/decisions.md) — ADRs
4. [docs/workflow.md](./docs/workflow.md) — full working model
5. [docs/architecture.md](./docs/architecture.md) — structure + data flow
6. [.ai/prompts.md](./.ai/prompts.md) — session templates
