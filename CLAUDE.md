# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step or dependencies. Open directly in a browser:

```bash
open index.html          # macOS
python3 -m http.server 8000  # then visit http://localhost:8000
```

## Architecture

Three files with no framework, no bundler, no transpiler:

- `index.html` — DOM structure: two `<canvas>` elements (`#board` 300×600px, `#next-canvas` 120×120px), a side panel with score/lines/level/next-piece display, and a single `#overlay` div reused for both PAUSE and GAME OVER states.
- `style.css` — Dark retro-arcade theme using flexbox and CSS variables.
- `game.js` — All game logic (~305 lines, `'use strict'`, no modules).

### game.js internals

**State** is held in module-level `let` variables: `board` (2D array `ROWS×COLS`, values 0 or color-index 1–7), `current`/`next` piece objects `{type, shape, x, y}`, and `score/lines/level/paused/gameOver/dropAccum/dropInterval/animId`.

**Key functions and their responsibilities:**

| Function | Role |
|---|---|
| `init()` | Resets all state, starts the `requestAnimationFrame` loop |
| `loop(ts)` | RAF callback — accumulates `dt`, triggers gravity drop or `lockPiece()`, calls `draw()` |
| `collide(shape, ox, oy)` | Bounds + board overlap check; used everywhere before moving/rotating |
| `tryRotate()` | Clockwise rotation with wall kicks: tries offsets `[0, -1, 1, -2, 2]` |
| `rotateCW(shape)` | Matrix transpose + row reversal |
| `lockPiece()` | `merge()` → `clearLines()` → `spawn()`; called when a piece can't move down |
| `ghostY()` | Projects piece down until collision; drives the ghost-piece rendering |
| `draw()` | Clears canvas, draws grid + board + ghost (alpha 0.2) + current piece |
| `drawNext()` | Renders the next piece centered in the 120×120 preview canvas |

**Speed formula:** `dropInterval = Math.max(100, 1000 − (level − 1) × 90)` ms. Level increases every 10 lines.

**Scoring:** `LINE_SCORES = [0, 100, 300, 500, 800]` × level. Soft drop +1 pt/row, hard drop +2 pts/cell fallen.

### Canvas coordinate system

Board cells are `BLOCK=30` px squares. `drawBlock(ctx, x, y, colorIndex, size, alpha)` renders at `(x*size+1, y*size+1)` with a 1px gap and a white highlight strip at the top of each block.

## Tuning constants

If you change `COLS`, `ROWS`, or `BLOCK`, also update the `<canvas id="board">` `width`/`height` attributes in `index.html` to match (`COLS×BLOCK` and `ROWS×BLOCK`).
