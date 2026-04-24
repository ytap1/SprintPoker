# Prompts: Sprint Poker

Copy-paste templates for common Claude sessions. Fill `<...>` slots before pasting.

## Starting a Session

```
Concern for this chat: <App Dev | Docs | Debugging>.
Stay in this lane unless I say otherwise.

Before changing anything, read in order:
1. CLAUDE.md
2. .ai/context.md
3. .ai/conventions.md

Confirm in 3 bullets:
- Current phase and anything known broken
- Working-model rules that affect today's task
- The two inline-onclick and display-style rules from conventions

Do not write code until I approve. Today's goal: <goal>.
```

## Adding a Feature

```
Feature: <one-sentence description>

User story:
  As a <facilitator | participant>, I want <capability>, so that <outcome>.

Acceptance criteria:
  - <testable statement>
  - <testable statement>
  - <edge case / error case>

Constraints:
  - Single index.html — no new files
  - No new CDN dependencies without an ADR
  - State mutation → renderAll() → broadcastToClients(SYNC) order
  - No inline onclick with dynamic values (use addEventListener)
  - No element.style.display = '' (use explicit value)

Deliver in this order, pause for review after each:
  1. Plan: which functions to add/modify, state changes needed.
  2. Implementation.
  3. Doc updates (SPRINT_POKER_GUIDE.md FAQ, HANDOVER.md if it fixes a bug).
  4. What to verify on the deployed preview after push.
```

## Debugging

```
Bug: <one-line symptom>

Repro:
  1. <step>
  2. <observed>

Expected: <what should happen>
Where seen: <device / browser>

Start with index.html. Investigate thoroughly before touching code.

Do this:
  1. State a hypothesis before reading more code.
  2. Identify the smallest change that confirms or rejects it.
  3. If it survives, propose the fix and how to verify on the deployed preview.
  4. Do not fix by silencing the symptom (no empty catch, no hidden errors).
  5. If buttons are disabled as part of the broken flow, ensure every error path re-enables them.
```

## Updating Docs After a Fix

```
We just fixed: <what broke and what the fix was>.

Update these docs to match current state:
  - HANDOVER.md: add a "Bug N" entry with function name, root cause, fix
  - SPRINT_POKER_GUIDE.md: add/update FAQ entry if the fix changes user-visible behaviour
  - .ai/context.md: update "Known broken" list (remove fixed item, add any new ones found)
  - CHANGELOG.md: add entry under [Unreleased] → Fixed

Commit with: "docs: <short description>"
```

## Architecture / Refactor Discussion

```
Topic: <what to change and why>

Current behaviour: <one paragraph>
Problem with it: <specific pain — not "feels messy">
Out of scope: <what must NOT change>

Constraints:
  - No build step
  - No new top-level CDN dependencies without an ADR
  - Facilitator-authoritative state model must be preserved
  - All changes verifiable on deployed preview

Give me:
  1. The proposal (what changes, what stays the same).
  2. The ADR entry to add to .ai/decisions.md.
  3. The minimal diff.
  4. What to verify after deploy.

Do not implement until I approve the proposal.
```
