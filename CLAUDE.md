# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Vanilla JS Tetris on HTML5 Canvas: `index.html` + `style.css` + `game.js`. No dependencies, no `package.json`, no build, no linter, no test suite. UI text and README are in Spanish.

## Running

Open `index.html` directly, or serve the folder statically (recommended):

```bash
python -m http.server 8000   # then http://localhost:8000
npx serve .
```

Verification is manual in the browser.

## Architecture (`game.js`)

Single script, `'use strict'`, all state in module-level `let` globals (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`, …). `init()` resets everything and is also the restart handler.

Key invariants:

- **Cell value = piece type = color index.** `PIECES[t]` matrices are filled with the value `t`, `board` cells store that value (0 = empty), and `COLORS[t]` renders it. Index 0 is `null` in both arrays. Adding/changing a piece must keep these three aligned; `randomPiece()` hardcodes `* 7`.
- **Pieces are square matrices** rotated with `rotateCW` (transpose + reverse). `tryRotate` applies simple horizontal kicks `[0, -1, 1, -2, 2]` — not SRS.
- **`collide(shape, x, y)`** is the single source of truth for legality (walls, floor, settled blocks; rows above the top, `y < 0`, are allowed). Movement, rotation, gravity, ghost (`ghostY`) and spawn game-over all go through it.
- **Piece lifecycle:** gravity/soft drop/hard drop → `lockPiece()` → `merge()` → `clearLines()` (updates lines, score, level, `dropInterval`) → `spawn()` (promotes `next`, calls `endGame()` if the new piece collides immediately).
- **Loop:** `requestAnimationFrame(loop)` accumulates `dt` into `dropAccum`; pause/game over stop it via `cancelAnimationFrame(animId)`. Resuming resets `lastTime` to avoid a huge `dt`.
- **Scoring:** `LINE_SCORES[cleared] * level`; hard drop +2/cell, soft drop +1/row. Level = `floor(lines/10)+1`; `dropInterval = max(100, 1000 - (level-1)*90)`.
- The overlay (`#overlay`) is shared by PAUSA and GAME OVER; its title/score text is set by `togglePause`/`endGame`.

## Coupling to watch

- Board canvas size in `index.html` (`300×600`) must equal `COLS*BLOCK × ROWS*BLOCK`.
- `drawNext` assumes a 4×4 preview grid at 30px (`#next-canvas` is `120×120`).
- DOM element IDs are looked up at top of `game.js`; renaming in HTML requires updating them.
- Controls are listed in three places: the `keydown` handler, the panel in `index.html`, and README.
