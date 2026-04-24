# Metrics: Sprint Poker

Track AI-assisted work so later you can tell what actually helped.
Add a row per meaningful session ("meaningful" = code shipped, not "asked a quick question").

## Columns

- **Date:** YYYY-MM-DD the session ended.
- **Feature / Fix:** short tag matching a commit or HANDOVER bug entry.
- **Sessions:** distinct chats it took to finish.
- **Bugs Found:** discovered during/because of the session.
- **Notes:** one line. What worked, what didn't, what to try next.

## Log

| Date       | Feature / Fix                        | Sessions | Bugs Found | Notes                                                              |
|------------|--------------------------------------|----------|------------|--------------------------------------------------------------------|
| 2026-04-24 | Initial Sprint Poker app             | 1        | 0          | Full app built in one session — single HTML file, PeerJS WebRTC   |
| 2026-04-24 | Blank screen after create (Bug 1–2)  | 1        | 2          | `display:''` fallback to CSS; also caught results panel same bug   |
| 2026-04-24 | Race condition on join (Bug 3)       | 1        | 1          | enterGame() before Peer() caused peer-unavailable for fast joiners |
| 2026-04-24 | Done phase + resetSession (Bug 4–6)  | 1        | 0          | Three related features shipped in one pass                         |
| 2026-04-24 | Accept & Next broken (Bug 10)        | 1        | 1          | JSON.stringify in onclick broke HTML attribute — caught by testing  |
| 2026-04-24 | Join button stuck on JOINING (Bug 9) | 2        | 1          | Needed investigation pass before fix; timeout + button recovery    |

## Rollups

- **Total sessions to v0.1.0:** ~8
- **Bugs introduced and fixed same session:** 2 (race condition, display fallback)
- **Biggest time sink:** join-room debugging (no console access on iPad — required code-only investigation)
- **Most effective pattern:** "investigate thoroughly before touching code" prompt reduced back-and-forth on the join bug

## Prompts Worth Saving

- Asking Claude to trace through the full flow manually before writing code (caught the onclick quote-conflict bug before it needed a second fix).
- "Investigate thoroughly before updating code" — critical when console logs aren't available.
