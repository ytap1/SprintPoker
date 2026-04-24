# Context: Sprint Poker

> Single source of truth Claude reads at the start of every session.
> Keep under ~150 lines. Link out instead of inlining.

Last updated: 2026-04-24

## What This Is

Sprint Poker is a serverless, real-time planning poker tool for agile sprint estimation. Facilitator creates a room; team joins by 6-character code; everyone votes simultaneously; facilitator reveals and accepts points; repeat per issue until backlog is done.

- **Problem:** Estimation meetings run long when votes are anchored by whoever speaks first. Planning poker forces simultaneous blind votes.
- **User:** Scrum teams / agile squads doing sprint planning, typically 3–10 people.
- **Success:** Full backlog estimated in one session with no account setup and no server to maintain.

## Current State

- **Phase:** alpha
- **Version:** 0.1.0
- **Deployed:** Yes — GitHub Pages (auto-deploys on push to `main`)
- **Users:** Internal / test use only
- **Known broken:**
  - No TURN server — connections across symmetric NATs can silently fail
  - Facilitator peer not reconnected on signalling drop (room goes unjoinable silently)
  - Create button not re-enabled on PeerJS error (mirrors the bug fixed for Join)
  - No mobile-responsive layout for phones narrower than ~420 px
  - Timer drifts across devices (each peer runs its own `setInterval`)

## Stack

| Layer | Detail |
|---|---|
| **App** | Single `index.html` — CSS + HTML + JS inline, no build step |
| **WebRTC signalling** | PeerJS 1.5.4 via CDN (`unpkg.com/peerjs@1.5.4`) |
| **ICE / STUN** | PeerJS default — Google `stun.l.google.com:19302` |
| **Signalling server** | PeerJS free cloud (`0.peerjs.com`) |
| **Hosting** | GitHub Pages — push to `main` triggers deploy |
| **Framework** | None — vanilla JS, CSS custom properties |

## Working Model

- **No local execution.** User works from iPad/iPhone via Safari + Claude Code. No terminal, no `npm`, no `curl localhost`.
- **No CLI instructions in docs.** Steps must run in CI or Claude Code's sandbox.
- **Push directly to `main`** by default. Branch only for destructive changes.
- **Deploy is automatic** on push to `main`. Verify on the deployed preview.
- **Multi-chat workflow.** Separate Claude chats per concern. Stay in the lane.

## Key Constraints

- Single HTML file — no build step, no bundler, no package.json.
- No new CDN dependencies without an ADR.
- Never use `element.style.display = ''` to reveal elements that have CSS `display: none` — always set an explicit value.
- Never put dynamic values in inline `onclick` attributes — use `addEventListener` with a closure.
- Secrets: none in this project. No env vars needed.

## Entry Points

- **Main file:** `index.html` (all CSS, HTML, JS)
- **User guide:** `SPRINT_POKER_GUIDE.md`
- **Handover / bug log:** `HANDOVER.md`
- **Deploy config:** GitHub Pages — no config file needed (root `index.html` auto-served)
- **Tests:** none

## Roadmap

Near-term (no external deps required):
- Add facilitator peer reconnection: `peer.on('disconnected', () => peer.reconnect())`
- Fix Create button stuck on error (mirror Bug 9 fix for Join)
- Responsive layout: one `@media (max-width: 640px)` breakpoint

Medium-term (new deps, needs ADR):
- Self-hosted PeerJS server on Railway/Render — eliminates free-tier unreliability
- Metered or Cloudflare TURN relay — fixes symmetric NAT failures
- Export estimated backlog to CSV

Long-term:
- Firebase or Supabase for session persistence and reconnection
- Jira / ERPNext integration — push accepted point values to the ticket
- Spectator mode (join without voting)

## See Also

- `.ai/conventions.md` — coding rules
- `.ai/decisions.md` — ADRs
- `docs/workflow.md` — full working model
- `docs/architecture.md` — structure + data flow
- `HANDOVER.md` — full bug log from initial build session
