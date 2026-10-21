# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Vanilla JavaScript Tetris on HTML5 Canvas. No dependencies, no `package.json`, no build step, no test suite, no linter. The whole app is three files: `index.html`, `style.css`, `game.js`.

## Running

Open `index.html` directly in a browser, or serve the directory statically (preferred):

```bash
python3 -m http.server 8000   # then open http://localhost:8000
npx serve .
```

Verification is manual: load the page and play.

## Architecture (`game.js`)

- **Single script, global mutable state.** `'use strict'` script (not an ES module) loaded at the end of `<body>`. All game state lives in module-level `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`, ...). Functions read and mutate these directly; `init()` resets everything and is also the restart handler.
- **One integer encodes piece type, color and cell contents.** Piece type `1..8` (the 7 standard tetrominoes plus a 1x1 piece) indexes both `PIECES` and `COLORS` (index 0 is `null`). Shape matrices store the type number in filled cells, and `merge()` copies those values into `board`, so a board cell is `0` (empty) or the color index of the piece that filled it. Adding or reordering pieces requires keeping `PIECES`, `COLORS` and `LIGHT_COLORS` aligned; `randomPiece()` derives the count from `PIECES.length`.
- **Collision is the single source of truth for movement.** `collide(shape, ox, oy)` is used for moves, rotation kicks (`tryRotate`, offsets `[0,-1,1,-2,2]`), gravity, ghost projection (`ghostY`) and game-over detection in `spawn()`. Rows with `y < 0` are allowed (above the board).
- **Loop.** `requestAnimationFrame(loop)` accumulates `dt` into `dropAccum` and drops one row when it exceeds `dropInterval`; `draw()` redraws the full board every frame. Pause and game over stop the loop via `cancelAnimationFrame(animId)`; unpausing restarts it by calling `loop()` with a fresh `lastTime`.
- **Lock sequence:** `lockPiece()` = `merge()` then `clearLines()` (updates score/lines/level/speed) then `spawn()` (promotes `next`, draws preview, may call `endGame()`).
- **Bucket (hold):** `holdPiece()` (Shift) stores `current.type` in `bucket`, or swaps with it via `createPiece(type)` (fresh spawn rotation/position). `canHold` allows one hold per piece; `spawn()` resets it. `drawBucket()` dims the preview while `canHold` is false.
- **Scoring/speed:** `LINE_SCORES[cleared] * level`; soft drop +1/row, hard drop +2/row; level = `floor(lines/10)+1`; `dropInterval = max(100, 1000 - (level-1)*90)`.

## Coupled values

- `<canvas id="board">` size in `index.html` must equal `COLS*BLOCK` x `ROWS*BLOCK` (currently 300x600).
- `<canvas id="next-canvas">` and `<canvas id="bucket-canvas">` are 120x120, which assumes the 4x4 preview grid with `NB = 30` in `drawPreview()`.
- DOM element IDs used by `game.js`: `board`, `next-canvas`, `bucket-canvas`, `theme-btn`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`. The overlay is shown/hidden by toggling the `hidden` class.

## Conventions

- All user-facing text (HTML labels, overlay messages, button text, aria labels) is in English, as are code identifiers and comments.
- Keyboard input uses `e.code` (`ArrowLeft`, `KeyP`, `Space`, ...), not `e.key`.
