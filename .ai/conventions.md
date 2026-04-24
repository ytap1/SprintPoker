# Conventions: Sprint Poker

Rules Claude must follow when writing code in this repo. When a rule
conflicts with a default behaviour, log the exception in `.ai/decisions.md`.

## File Structure

- Everything lives in `index.html`. Do not create separate JS or CSS files.
- Documentation goes in `*.md` at repo root or `docs/` or `.ai/`.
- No `package.json`, no build config, no `node_modules`.

## JavaScript Style

- **`const` / `let` only.** Never `var`.
- **Descriptive names.** No single-letter variables except loop indices (`i`, `j`).
- **Functions over classes.** No ES6 classes needed in this codebase.
- **Arrow functions** for callbacks and short helpers; named `function` declarations for top-level functions (they hoist, which matters in a single-file app).

## State and Rendering

- **Mutate state first, render after.** Always: update `state.*` → call render function(s) → call `broadcastToClients(SYNC)` if facilitator.
- **`renderAll()` is the safe default.** Call targeted renders only for performance-sensitive paths (e.g. `renderDeckSelection()` on vote toggle).
- **Never read from the DOM** to determine app state — always read from `state`.

## DOM Manipulation

- **Never `element.style.display = ''`** to show an element that has `display: none` in CSS. This clears the inline style and falls back to the CSS rule, keeping it hidden. Always set an explicit value: `'block'`, `'flex'`, `'grid'`, etc.
- **Never inline dynamic values in `onclick` attributes.** Using `JSON.stringify` inside a template literal produces broken HTML when the value is a string (e.g. `onclick="fn("5")"`). Always use `addEventListener` with a closure:
  ```js
  btn.addEventListener('click', () => fn(value));
  ```
- **Use `esc()` for any user content injected into `innerHTML`** to prevent XSS.

## Networking / PeerJS

- **Facilitator is authoritative.** Only the facilitator mutates shared state. Clients send `JOIN` and `VOTE`; the facilitator broadcasts `SYNC` in response to every state change.
- **Always broadcast SYNC after any facilitator state change.** If you mutate `state` in a facilitator function and forget to call `broadcastToClients`, clients will desync.
- **Wrap `conn.send()` in try/catch.** `broadcastToClients` already does this — don't add bare sends elsewhere.
- **Don't call `renderAll()` before `enterGame()`.** Game DOM elements exist but are hidden; renders succeed silently, but the results are invisible.

## Error Handling

- **User-visible errors go to `showError(id, msg)` or `toast(msg)`.**
  - `showError`: use when the lobby is visible (pre-join errors).
  - `toast`: use when the game screen is active (post-join errors).
- **Always re-enable buttons on failure.** If a button is disabled while an async operation runs, every error path must re-enable it. Never leave the user stuck.
- **Don't catch bare errors silently.** `catch(e) {}` is only acceptable in `broadcastToClients` where one broken connection must not stop the others.

## Forbidden

Do not do these without first updating `.ai/decisions.md`:

- Adding a new CDN dependency (new `<script src="...">` tag).
- `console.log` in code considered stable (debug logs are fine during active work — remove before marking stable).
- Inline `onclick` with dynamic values (see DOM Manipulation above).
- `element.style.display = ''` to show elements (see DOM Manipulation above).
- CLI instructions in docs — the user has no terminal.

## When Adding New Code

1. Does it mutate state? Follow the mutate → render → broadcast order.
2. Does it touch the DOM? Check the display-value and inline-onclick rules above.
3. Does it add a failure path? Ensure the user gets feedback and any disabled buttons are re-enabled.
4. Is public-facing behaviour changing? Update `SPRINT_POKER_GUIDE.md` FAQ and `HANDOVER.md` if it fixes a known bug.
5. Diff minimal — no drive-by renames, no unrelated formatting.
