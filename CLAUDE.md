# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A playable Tetris clone in vanilla JavaScript (ES6+), HTML5 Canvas, and CSS. No build step, no package manager, no external dependencies. Three files: `index.html`, `style.css`, `game.js`.

## Running the game

Open `index.html` directly in a browser, or serve it statically:

```bash
python3 -m http.server 8000
# or
npx serve .
```

There is no build, lint, test, or install step — there is no `package.json`. Changes to `game.js`/`style.css`/`index.html` take effect on browser reload.

## Architecture

All game logic lives in `game.js` as top-level functions operating on module-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) — there are no classes and no modules.

- **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1–7` identifying which piece type locked there.
- **Pieces**: the 7 standard tetrominoes are defined as square matrices in `PIECES`. Rotation (`rotateCW`) is a transpose + row-reverse, not per-piece rotation tables.
- **Collision** (`collide`): checks board bounds and existing locked cells for a given shape/offset.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` and takes the first that doesn't collide.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time in `dropAccum` and advances the piece one row once `dropInterval` is exceeded. `dropInterval` shrinks as `level` increases (`max(100, 1000 - (level-1)*90)`).
- **Locking a piece** (`lockPiece`): `merge()` writes the piece into `board`, `clearLines()` removes full rows (scores via `LINE_SCORES` × `level`, bumps `level` every 10 lines), then `spawn()` promotes `next` to `current` and generates a new `next`. If the new `current` immediately collides, `endGame()` fires.
- **Ghost piece** (`ghostY`): projects `current` straight down to its landing row; drawn at low alpha.
- **Rendering**: `draw()` clears and redraws the whole board canvas each frame (grid, locked cells, ghost, current piece); `drawNext()` renders the preview piece on a separate small canvas (`#next-canvas`).

State is initialized/reset entirely in `init()` (called on load and on restart-button click), so adding new persistent state means updating `init()` too.

## Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK` (px per cell), `COLORS` (palette per piece type), `LINE_SCORES`, initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `<canvas id="board">` `width`/`height` in `index.html` to match (`COLS×BLOCK` by `ROWS×BLOCK`).

## Conventions

- Comments and UI copy (labels, overlay text) are in Spanish; keep new user-facing strings consistent with that.
- No semicolon-less style — code uses semicolons and `'use strict'`.
