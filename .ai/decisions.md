# Decisions: Sprint Poker

Append-only log of architectural and technical decisions.

- **Never rewrite history.** Reversed decisions get a new entry that supersedes the old one.
- **Keep entries short.** Link out if it needs more than a page.
- **Number sequentially** (ADR-0001, ADR-0002, ...).

---

## ADR-0001: Push directly to `main`; branch only for destructive changes

- **Date:** 2026-04-24
- **Status:** Accepted

### Context

Solo developer working from iPad/Safari + Claude Code. PR review overhead adds friction with no second reviewer. Auto-deploy on push to `main` is the desired feedback loop.

### Decision

Push directly to `main` for routine changes. Create a branch only when the change is destructive or high-risk: schema rewrites, mass renames, deletions.

### Alternatives Considered

- **PR-per-change:** standard team practice but adds friction with no reviewer and slows the iPad/web workflow.

### Consequences

- **Positive:** fastest possible iteration; deploy preview = production.
- **Negative:** no review gate. Mitigation: `git revert` is the rollback path.

---

## ADR-0002: Single HTML file — no build step, no bundler

- **Date:** 2026-04-24
- **Status:** Accepted

### Context

The user works exclusively from iPad/Safari with no terminal, no Node.js, no local dev server. Any build step would require a CI pipeline or a laptop, neither of which is available.

### Decision

All CSS, HTML, and JS live inline in `index.html`. No `package.json`, no bundler, no separate files.

### Alternatives Considered

- **Vite + separate files:** clean DX but requires a build step — not viable without a terminal.
- **GitHub Actions build:** possible but adds CI complexity for a project that doesn't need it yet.

### Consequences

- **Positive:** zero toolchain; deploy is `git push`; any editor (including GitHub web) can make changes.
- **Negative:** file grows large; no tree-shaking; no TypeScript. Acceptable at this scale.
- **Follow-ups:** if the file exceeds ~2000 lines of JS, revisit with a CI-based build (Vite + GH Actions).

---

## ADR-0003: PeerJS 1.5.4 via CDN for WebRTC signalling

- **Date:** 2026-04-24
- **Status:** Accepted

### Context

The app needs real-time bidirectional data between browser tabs with no server. WebRTC data channels are the right primitive; PeerJS wraps the signalling complexity.

### Decision

Use PeerJS 1.5.4 loaded from `unpkg.com`. Use the PeerJS free cloud server (`0.peerjs.com`) for signalling. Accept the free-tier limitations for now.

### Alternatives Considered

- **Raw WebRTC:** much more implementation work with no gain at this scale.
- **WebSockets via a server:** requires a server process (Node, Deno, etc.) — violates the no-server constraint.
- **Socket.io / Ably / Pusher:** server-dependent or paid; overkill for a single-room tool.

### Consequences

- **Positive:** zero server cost; peer-to-peer after handshake; works on any modern browser.
- **Negative:** free `0.peerjs.com` is rate-limited and occasionally unreliable; no TURN server means connections can fail across symmetric NATs.
- **Follow-ups:** self-host `peer-server` on Railway/Render once reliability becomes a blocker. Add a TURN relay (Metered free tier or Cloudflare Calls) for NAT traversal.

---

## ADR-0004: Facilitator-authoritative state model

- **Date:** 2026-04-24
- **Status:** Accepted

### Context

In a multi-peer WebRTC mesh, conflicts arise when multiple peers can mutate shared state. A tie-breaking authority is needed.

### Decision

The facilitator's browser is the single source of truth. Clients send `JOIN` and `VOTE` messages to the facilitator; the facilitator applies changes to `state` and broadcasts a full `SYNC` snapshot to all clients after every mutation. Clients never mutate shared state directly.

### Alternatives Considered

- **CRDTs:** correct for conflict-free eventual consistency but far too complex for this use case.
- **Symmetric mesh (each peer broadcasts):** requires conflict resolution and ordering guarantees — unnecessary when one user (the facilitator) is always in control.

### Consequences

- **Positive:** simple protocol; no conflicts; easy to reason about; rollback is just re-broadcasting previous state.
- **Negative:** facilitator is a single point of failure — if their tab closes, the session ends. Acceptable given the product design (facilitator-led sessions).
- **Follow-ups:** if persistence is added (Firebase/Supabase), the facilitator role can be transferred to the server, enabling reconnection.

---

## ADR-0006: Configurable PeerJS signalling server and TURN relay

- **Date:** 2026-04-24
- **Status:** Accepted

### Context

The PeerJS free cloud (`0.peerjs.com`) is rate-limited and occasionally unavailable. Additionally, PeerJS defaults to Google STUN only, which fails silently on symmetric NATs (corporate WiFi, some mobile carriers). Both issues block reliable use in a team setting.

### Decision

Add a top-of-file `PEER_CONFIG` object that centralises signalling host settings and the `iceServers` array. A `buildPeerOpts()` helper converts `PEER_CONFIG` into the options object passed to the `Peer` constructor, replacing the hardcoded `{ debug: 0 }` in both `hostRoom()` and `joinRoom()`.

- **Signalling**: `PEER_CONFIG.signalling` is `null` by default (uses PeerJS free cloud). Set it to `{ host, port, path, secure }` to point at a self-hosted server (Railway or Render free tier — see HANDOVER.md §4 for deployment steps).
- **TURN**: `PEER_CONFIG.iceServers` ships with Google STUN only. TURN credentials (Metered free tier or Cloudflare Calls) are plugged in here — see HANDOVER.md §5 for how to wire them in.

### Alternatives Considered

- **Hardcode the self-hosted URL:** simpler but requires a find-and-replace if the server moves.
- **Environment variable at build time:** not viable — there is no build step (ADR-0002).

### Consequences

- **Positive:** server URL and ICE config are in one place; no hunting through the code; switching from free cloud to self-hosted is a 3-line change.
- **Negative:** none. Behaviour is identical when `PEER_CONFIG.signalling` is `null`.
- **Follow-ups:** deploy a self-hosted `peer-server` on Railway/Render and populate `PEER_CONFIG.signalling`. Add TURN credentials to `PEER_CONFIG.iceServers` for symmetric-NAT support.

---

## ADR-0005: addEventListener over inline onclick for dynamic values

- **Date:** 2026-04-24
- **Status:** Accepted

### Context

Several render functions build button HTML via `innerHTML` template literals and pass computed values as inline `onclick` attributes (e.g. `onclick="nextIssue(${JSON.stringify(pts)})"`). When `pts` is a string, `JSON.stringify` wraps it in double quotes, which break the HTML attribute parser — the click handler is silently ignored.

### Decision

Never embed dynamic JS values in inline `onclick` attributes. Instead, render the button HTML with a plain ID and attach the handler via `addEventListener` with a closure, so the value is passed as a real JS variable.

### Alternatives Considered

- **Single-quote wrapping (`onclick="fn('${val}')"`**): works for simple strings but breaks on values containing apostrophes; fragile.
- **data-* attributes + delegated listener:** clean but adds indirection; overkill for this scale.

### Consequences

- **Positive:** no HTML-injection fragility; values of any type (string, number, null, object) work correctly.
- **Negative:** slightly more code per dynamic button. Acceptable.
