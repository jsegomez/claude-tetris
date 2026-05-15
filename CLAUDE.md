# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Game

No build system — open `index.html` directly in a browser, or start a local static server:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

No dependencies, no npm, no compilation step.

## Architecture

Three files, no framework:

- **`index.html`** — DOM structure: a main `<canvas>` (300×600px) for the board and a smaller `<canvas>` (120×120px) for the next-piece preview, plus a side panel with score/lines/level displays.
- **`style.css`** — Dark retro theme. Flexbox layout with the board canvas left and panel right.
- **`game.js`** — All game logic (~300 lines, no modules).

### Game Loop

`requestAnimationFrame` drives `loop(ts)`, which accumulates delta time and triggers gravity drops at `dropInterval` ms. `animId` holds the current frame ID and is cancelled on pause/game-over to stop the loop.

### State

All state lives in module-level variables: `board` (2D array, 10×20), `current` and `next` (piece objects), `score/lines/level`, `paused/gameOver`, and timing variables (`lastTime`, `dropAccum`, `dropInterval`).

Piece objects: `{ type: 1–7, shape: 4×4 matrix, x, y }`. Board cells store `0` (empty) or a color index (1–7).

### Key Functions

| Function | Purpose |
|---|---|
| `collide(shape, ox, oy)` | Collision check against walls and occupied cells |
| `tryRotate()` | Rotation with 5-offset wall kick `[0, -1, +1, -2, +2]` |
| `clearLines()` | Splice completed rows, unshift empty rows at top |
| `ghostY()` | Scans downward to find landing row for ghost rendering |
| `hardDrop()` | Instant drop; score += 2× rows fallen |
| `spawn()` | Promotes `next` → `current`; triggers game-over if immediate collision |
| `draw()` | Renders board, ghost piece (`globalAlpha=0.2`), current piece, overlays |

### Scoring & Speed

- Line clears: 100 / 300 / 500 / 800 × level for 1–4 lines simultaneously.
- `level = floor(lines / 10) + 1`
- `dropInterval = max(100, 1000 − (level−1) × 90)` ms

### Customization

Constants at the top of `game.js` control board size (`COLS=10`, `ROWS=20`, `BLOCK=30`), piece colors (`COLORS`), and line scores (`LINE_SCORES`).
