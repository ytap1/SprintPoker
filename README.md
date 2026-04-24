# Sprint Poker

> Real-time planning poker for agile sprint estimation — no account, no server, just share the code.

## What It Is

Sprint Poker is a serverless, browser-based planning poker tool. One person creates a room as facilitator and gets a short room code (`SP-XXXXXX`). The rest of the team joins on their own devices by entering that code. Everyone votes simultaneously on story point estimates; the facilitator reveals votes, accepts a value, and the session advances to the next issue. All communication is peer-to-peer via WebRTC — no backend required.

## Status

- **Phase:** alpha
- **Version:** 0.1.0
- **Last updated:** 2026-04-24
- **Deployed:** Yes — [GitHub Pages](https://ytap1.github.io/SprintPoker/)

## How to Use

See [SPRINT_POKER_GUIDE.md](./SPRINT_POKER_GUIDE.md) for the full guide, including:

- How to create and share a room
- How to import a backlog from CSV
- Voting, revealing, accepting points, and re-voting
- What happens at the end of the backlog
- FAQ

Quick version:

1. Open the app URL in any modern browser.
2. **Facilitator:** enter your name, pick a deck and timer, click **+ Create Room**. Share the code in the header with your team.
3. **Team:** open the same URL, enter your name and the room code, click **→ Join Room**.
4. Facilitator clicks an issue in the backlog → voting starts → team picks cards → facilitator reveals → accept or re-vote → repeat.

## Tech Stack

| Layer | Detail |
|---|---|
| App | Single `index.html` — CSS, HTML, and JS inline. No build step. |
| Multiplayer | PeerJS 1.5.4 (WebRTC data channels via CDN) |
| Signalling | PeerJS free cloud (`0.peerjs.com`) |
| Hosting | GitHub Pages — auto-deploys on push to `main` |

## How to Deploy Your Own Copy

1. Fork this repo on GitHub.
2. Go to **Settings → Pages** → set Source to `main` branch, root folder.
3. Push any change to `main` — GitHub Pages will build and serve `index.html` automatically.
4. Share the Pages URL with your team.

No config files, no environment variables, no build commands needed.

## How to Contribute

This project follows a no-terminal, no-build-step workflow. Changes are made via Claude Code or the GitHub web editor and pushed directly to `main`.

Before making changes, read:

1. [CLAUDE.md](./CLAUDE.md) — working model and coding rules
2. [.ai/context.md](./.ai/context.md) — current state and constraints
3. [.ai/conventions.md](./.ai/conventions.md) — specific rules for this codebase
4. [.ai/decisions.md](./.ai/decisions.md) — why key choices were made

Key rules:
- Everything lives in `index.html`. Do not create separate JS or CSS files.
- Never use `element.style.display = ''` to reveal elements — always set an explicit value.
- Never put dynamic JS values in inline `onclick` attributes — use `addEventListener` with a closure.
- State mutation always precedes rendering and broadcasting.

## Known Limitations

- Host tab must stay open — closing it ends the session for everyone.
- PeerJS free cloud broker can be unreliable under load or rate limits.
- No TURN server — connections may fail across strict corporate or carrier NATs.
- No session persistence — a page refresh ejects you from the session.

## Roadmap

See [.ai/context.md — Roadmap](./.ai/context.md) for the full list. Near-term priorities:

- Facilitator peer reconnection on signalling drop
- Responsive layout for phones
- Self-hosted PeerJS signalling server
- Export estimated backlog to CSV

## Documentation

| File | Purpose |
|---|---|
| [SPRINT_POKER_GUIDE.md](./SPRINT_POKER_GUIDE.md) | End-user guide and FAQ |
| [HANDOVER.md](./HANDOVER.md) | Bug log and technical handover notes |
| [CHANGELOG.md](./CHANGELOG.md) | Version history |
| [CLAUDE.md](./CLAUDE.md) | Claude Code instructions and coding conventions |
| [.ai/context.md](./.ai/context.md) | Project context for AI sessions |
| [.ai/decisions.md](./.ai/decisions.md) | Architectural decision records |
| [docs/architecture.md](./docs/architecture.md) | Data flow and component structure |
