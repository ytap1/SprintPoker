# Sprint Poker — Quick Start Guide

## What is this?
A real-time planning poker tool for sprint estimation. Your team votes on story points simultaneously — no one influences anyone else until cards are revealed.

---

## How a Session Works

```
Facilitator creates room → shares code → team joins → vote per issue → reveal → repeat
```

---

## Facilitator (Team Lead / Scrum Master)

### Before the meeting
1. Open the app URL in your browser
2. Under **Create Room**:
   - Enter your name
   - Pick a deck: **Fibonacci** (1, 2, 3, 5, 8, 13, 21) or **T-Shirt** (XS–XXL)
   - Set a timer per voting round (default: 1 min)
3. Click **+ Create Room**
4. A **6-character room code** appears in the top header (e.g. `SP-A1B2C3`)
5. Share this code with your team via Slack/Teams/chat

### Optional: Prepare your backlog
- Type issues one by one in the backlog and press Enter
- **Or** drop a CSV file into the import zone — one issue title per line

### Running the session
1. Click an issue in the backlog → voting starts automatically
2. Watch the team's vote indicators fill up (cards are hidden until revealed)
3. When ready (or timer runs out), click **Reveal Cards**
4. Review results:
   - ✓ **Consensus** = everyone voted the same → accept and move on
   - ✗ **No consensus** = discuss briefly → click **Re-vote** or **Accept & Next**
5. Points are saved to the backlog automatically

---

## Team Members

### Joining a session
1. Open the app URL
2. Under **Join Room**:
   - Enter your name
   - Type the room code shared by your facilitator
3. Click **→ Join Room**

### Voting
- Select your card when the facilitator starts a round
- Your vote is **hidden** from others until the facilitator reveals
- You can change your vote any time before reveal
- `?` = unsure / need more info
- `☕` = need a break

---

## Card Decks

| Fibonacci | Use for... |
|-----------|------------|
| 1, 2, 3   | Small tasks (hours to a day) |
| 5, 8      | Medium tasks (a few days) |
| 13, 21    | Large tasks (consider breaking down) |

| T-Shirt | Rough equivalent |
|---------|-----------------|
| XS / S  | 1–3 points |
| M / L   | 5–8 points |
| XL / XXL| 13–21 points |

---

## CSV Import Format

Create a `.csv` file with one issue per line:

```
title
Implement user login endpoint
Fix null pointer on checkout flow
Migrate DB schema for users table
Add unit tests for payment service
Integrate ERPNext with Boomi adapter
```

- First row is skipped if it says `title` or `issue`
- Only the title column is used — extra columns are ignored
- Drag and drop onto the import zone, or click to browse

---

## FAQ

**Q: What happens if the facilitator closes the tab?**
The room ends. Facilitator should keep their tab open for the whole session.

**Q: Can two teams use this at the same time?**
Yes. Each room has a unique code — sessions are fully isolated.

**Q: Do I need an account or login?**
No. Just the URL and the room code.

**Q: The room code says "not found" — what do I check?**
- Make sure you're typing the full code exactly (e.g. `SP-A1B2C3`)
- The facilitator must have their tab open — the room only exists while they're connected
- Try refreshing and joining again

**Q: Can I re-estimate an issue?**
Yes — facilitator can click any completed issue in the backlog and restart voting.

---

## Tips for Better Estimation Meetings

- Keep rounds short — use the 60s timer to maintain pace
- If one person votes very differently, discuss *why* before re-voting
- Break down any issue estimated at 13+ points
- Use `?` to flag issues that need more clarification before estimating
- Export your pointed backlog by screenshotting or copying from the sidebar
