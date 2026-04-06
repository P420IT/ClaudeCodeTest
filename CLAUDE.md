# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A collection of self-contained browser games — each game is a **single HTML file** with embedded CSS and JavaScript. No build tools, no dependencies, no bundlers. Open the file in a browser to play.

## Running the Games

```bash
# Windows — open in default browser
start tictactoe.html
start spaceinvaders.html
```

There are no build, lint, or test commands. Development is: edit the file → refresh the browser.

## Git & GitHub

The remote is `https://github.com/P420IT/ClaudeCodeTest`. Always commit and push after changes:

```bash
git add <file>
git commit -m "descriptive message"
git push
```

Commit message convention: imperative mood, explain *why* not just *what* (e.g. `"fix alien grid boundary check to use actual alien positions"`).

## Architecture

### Single-file game structure

Each game file is divided into clearly labelled sections (`// ── SECTION N: NAME ──`). The canonical section order for `spaceinvaders.html`:

1. **Constants & Config** — canvas size, color palette, state names, entity dimensions, speeds
2. **Level Data** — `LEVELS[]` array; each entry fully describes one level (rows, cols, speed, fire rate, movement pattern, divebomb flag)
3. **InputHandler** — `keys{}` (held) + `justPressed{}` (single-frame); `flush()` must be called at the end of every game loop tick
4. **Player** — position, lives, shoot cooldown, invincibility frames; draws itself via canvas paths
5. **AlienGrid** — flat `aliens[]` array + grid offset; handles march, zigzag, and dive-bomb movement; speed scales as `baseSpeed * min(4, total/living)`
6. **BulletManager** — object pool; bullets are marked `active=false` rather than spliced out
7. **ParticleSystem** — explosion and trail particles; drawn with `globalAlpha` fade
8. **Collision** — pure AABB functions; hitboxes are intentionally 80% of visual size
9. **HUD** — drawn on canvas (not DOM); `'Courier New'` for numbers, `'Segoe UI'` for overlays
10. **Renderer** — all screen states (menu, win, lose, pause, transition); parallax starfield
11. **GameStateManager (GSM)** — owns current state, level index, total score; calls `_startLevel()` on each `PLAYING` transition
12. **Main Loop** — `update(dt) → render() → InputHandler.flush()`; `dt` is capped at `0.05s` to prevent teleportation on tab-switch

### Game loop invariants

- All movement uses **delta-time** (`velocity * dt`), never frame counts.
- Canvas is scaled for device pixel ratio at startup: `ctx.scale(dpr, dpr)`.
- Bootstrap uses a **two-step `requestAnimationFrame`** to avoid a giant first `dt`.
- Every draw function that changes `globalAlpha` or transforms wraps in `ctx.save()` / `ctx.restore()`.

### Color palette (shared across games)

| Token | Hex | Usage |
|-------|-----|-------|
| `bg` | `#0d0d1a` | Canvas background |
| `dark1–3` | `#1a1a2e` / `#16213e` / `#0f3460` | UI panels, borders |
| `red` | `#e94560` | Player X, enemy bullets, danger |
| `cyan` | `#a8dadc` | Player ship, player bullets, info text |
| `yellow` | `#f4d03f` | Top-row aliens, high score highlights |

### High score persistence

`localStorage.setItem('si_hiscore', score)` — written on WIN or LOSE, read on the menu screen.
