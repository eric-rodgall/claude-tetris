# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A vanilla-JS Tetris. No build, no dependencies, no `package.json`. Three source files: `index.html`, `style.css`, `game.js`.

## Running

Open `index.html` directly, or serve statically (`python3 -m http.server 8000`). There are no tests, linter, or build step.

## Architecture (`game.js`)

Single-file, module-pattern (no classes). All mutable game state lives in module-level `let` bindings (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, drop timing vars); `init()` resets them all and is also the restart handler.

Key model details:
- `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1–7` that also indexes both `COLORS` and `PIECES`.
- Pieces are square matrices. Rotation (`rotateCW`) is transpose + row-reverse; `tryRotate` applies wall kicks by testing x-offsets `[0,-1,1,-2,2]`.
- `collide(shape, x, y)` is the single source of truth for legality — used for movement, rotation, ghost projection, spawn/game-over, and lock detection.
- Game loop is `requestAnimationFrame`-driven (`loop`), accumulating `dt` into `dropAccum` and stepping down when it exceeds `dropInterval`. Pause cancels the frame; `togglePause` restarts it.
- `lockPiece` → `merge` → `clearLines` → `spawn`. Game over is detected in `spawn` when the new piece collides immediately.
- Scoring: `LINE_SCORES` table × `level`; soft drop +1/row, hard drop +2/cell. Level = `floor(lines/10)+1`; `dropInterval = max(100, 1000 - (level-1)*90)`.

## Gotchas

- Canvas pixel dimensions in `index.html` (`board` = 300×600, `next-canvas` = 120×120) must stay in sync with `COLS`, `ROWS`, `BLOCK` in `game.js`.
- HUD text is Spanish; keep new user-facing strings in Spanish.
- `updateHUD` must be called after any state change that isn't already covered by the loop (the keydown handler calls it at the end).
