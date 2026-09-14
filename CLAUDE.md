# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A classic Tetris implementation in vanilla JavaScript using HTML5 Canvas. No dependencies, no build process, no package.json — just open `index.html` and play.

## Running the game

There is no build/lint/test tooling in this repo. To run:

```bash
start index.html       # Windows: open directly in browser
```

Or serve it locally (needed if you add features requiring `fetch`/modules with CORS restrictions):

```bash
python3 -m http.server 8000
npx serve .
```

Then visit `http://localhost:8000`.

## Architecture

Three cooperating files, all at the repo root:

- **`index.html`** — DOM structure: a `300×600` `<canvas id="board">` for the game grid, a `120×120` `<canvas id="next-canvas">` for the next-piece preview, the HUD (score/lines/level), and a pause/game-over `#overlay`.
- **`style.css`** — dark/retro arcade visual theme (flexbox layout, monospace HUD, `backdrop-filter` overlay).
- **`game.js`** — all game logic, in one file, no modules/classes. Key pieces:
  - **Board model**: `ROWS × COLS` matrix where each cell is `0` (empty) or a color index `1–7` identifying the locked piece.
  - **Pieces**: `PIECES` array of square matrices. Rotation (`rotateCW`) is a transpose + row-reverse, not per-piece rotation tables.
  - **Collision** (`collide`): checks board bounds and overlap with locked cells.
  - **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` columns before giving up on the rotation.
  - **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time and advances the piece one row once `dropAccum >= dropInterval`.
  - **Line clearing** (`clearLines`): scans bottom-to-top, splices full rows out and unshifts empty rows at the top.
  - **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/cell dropped, soft drop adds 1 pt/row.
  - **Level/speed**: level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
  - **Ghost piece** (`ghostY`): projects the current piece straight down to its landing row, drawn at `globalAlpha = 0.2`.
  - Module-level `let` state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, ...) is mutated directly by these functions rather than passed around — this is a deliberate single-file, no-framework design, not an oversight.

Control flow: `init()` builds the board and starts the loop → `loop()` ticks gravity and calls `draw()` each frame → `keydown` handler dispatches move/rotate/soft-drop/hard-drop/pause → `lockPiece()` merges the piece into the board, clears lines, and spawns the next one. If a freshly spawned piece immediately collides, `endGame()` fires and the Game Over overlay is shown.

### Tunable constants (in `game.js`)

`COLS`, `ROWS`, `BLOCK` (cell size in px), `COLORS`, `LINE_SCORES`, `dropInterval`. If you change `COLS`/`ROWS`/`BLOCK`, also update the `width`/`height` attributes of `<canvas id="board">` in `index.html` to match (`COLS × BLOCK` by `ROWS × BLOCK`).
