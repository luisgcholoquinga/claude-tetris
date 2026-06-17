# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step required. Open `index.html` directly in a browser or serve it with any static server:

```powershell
# Python
python -m http.server 8000

# Node.js
npx serve .
```

Then open `http://localhost:8000`.

## Architecture

Three files, no dependencies:

- **`index.html`** — DOM structure: `<canvas id="board">` (300×600 px) for the main board, `<canvas id="next-canvas">` (120×120 px) for piece preview, a side panel (`<aside class="panel">`) with score/lines/level HUD, and `#overlay` for pause/game-over states.
- **`style.css`** — Dark retro-arcade theme. Uses CSS variables, flexbox, and `backdrop-filter` for overlay blur.
- **`game.js`** — All game logic (~300 lines, `'use strict'`, no classes).

### Key data structures in `game.js`

- **`board`**: `ROWS × COLS` (20×10) matrix; `0` = empty, `1–7` = piece color index.
- **`current` / `next`**: piece objects `{ type, shape, x, y }` where `shape` is a 2D matrix of color indices.
- `PIECES` array (index 1–7) and `COLORS` array (index 1–7) are parallel — piece type maps directly to color.

### Core function flow

```
init() → spawn() → requestAnimationFrame(loop)
loop(ts): accumulates dt, drops piece or calls lockPiece()
lockPiece(): merge() → clearLines() → spawn()
```

Rotation uses `rotateCW()` (transpose + reverse rows) with wall-kick offsets `[0, -1, 1, -2, 2]` in `tryRotate()`.

Ghost piece is computed each frame in `ghostY()` by projecting downward until collision.

### Tunable constants (top of `game.js`)

| Constant | Default | Notes |
|---|---|---|
| `COLS` / `ROWS` | 10 / 20 | Must match canvas `width`/`height` in `index.html` (`COLS×BLOCK` × `ROWS×BLOCK`) |
| `BLOCK` | 30 | Pixel size per cell |
| `LINE_SCORES` | `[0,100,300,500,800]` | Points for 1–4 line clears (multiplied by level) |
| `dropInterval` | 1000 ms | Initial fall speed; recalculated as `max(100, 1000 − (level−1) × 90)` |
