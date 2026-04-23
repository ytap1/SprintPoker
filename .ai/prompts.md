# Prompts: {{PROJECT_NAME}}

Copy-paste templates for common Claude sessions. Fill the `<...>` slots
before pasting. Edit as the project matures.

## Starting a Chat (any concern)

```
Concern for this chat: <App Dev | Prompt Refinement | Docs | other>.
Stay in this lane unless I say otherwise.

Before changing anything, read in order:
1. CLAUDE.md
2. .ai/context.md
3. .ai/conventions.md
4. .ai/decisions.md

Confirm in 3 bullets:
- current phase + anything known broken
- the working-model rules that affect today's task
- two things "Forbidden" prohibits

Do not write code until I approve. Today's goal: <goal>.
```

## Adding a Feature

```
Feature: <one-sentence description>

User story:
  As a <role>, I want <capability>, so that <outcome>.

Acceptance criteria:
  - <testable statement>
  - <testable statement>
  - <error/edge case>

Constraints (from .ai/context.md and .ai/conventions.md):
  - <project-specific>

Deliver in this order, pause for review after each:
  1. Plan: files to create/modify, public surface.
  2. Implementation.
  3. Doc updates (README / context.md / architecture.md as needed).
  4. Verification: what to check on the deploy preview after push.
```

## Debugging

```
Bug: <one-line symptom>

Repro:
  1. <step>
  2. <observed>

Expected: <what should happen>
Where seen: <deploy URL / browser / device>

Relevant files (start here, expand as needed):
  - <path>

Logs / errors:
```
<paste>
```

Do this:
  1. State a hypothesis before reading more code.
  2. Identify the smallest change that confirms or rejects it.
  3. If it survives, propose the fix and how to verify it on the deploy
     preview.
  4. Do not "fix" by widening a try/except or silencing the symptom.
```

## Refactoring

```
Refactor: <module or pattern>
Why now: <specific pain — not "feels messy">
Out of scope: <what must NOT change>

Rules:
  - Behavior must not change.
  - One concern per commit.
  - If a bug surfaces mid-refactor, note it and keep going — fix separately.

Deliver:
  1. "Before" sketch (5–10 lines).
  2. "After" sketch.
  3. The diff.
  4. What to spot-check on the deploy preview.
```

## Code Review

```
Review the diff below against .ai/conventions.md and .ai/context.md.

Give me, in order:
  1. Blocking issues (correctness, security, broken invariants).
  2. Convention violations with file:line references.
  3. Suggestions (non-blocking, marked "nit:").
  4. What's good — at least one thing.

Don't rewrite the code. Point at the line and describe the change in one
sentence. If a convention is ambiguous, say so and recommend an ADR.

Diff:
```
<paste diff or PR link>
```
```

## Update Context

```
We just finished: <what shipped>.

Update .ai/context.md so a fresh chat tomorrow has accurate ground truth:
  - "Current State" — phase, version, deployed, broken
  - "Entry Points" — add/remove any new files
  - "Key Constraints" — any new hard rules from this change
  - "Last updated" date at the top

If a significant choice was made (new dep, reversed earlier call), add
an entry to .ai/decisions.md.

Show me the diff before saving.
```
