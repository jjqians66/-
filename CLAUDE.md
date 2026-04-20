# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a single-file HTML5 canvas game — a Pac-Man-style love chase game (爱心追逐) where a player character navigates a 17×17 maze collecting hearts and rings while avoiding chasers. The entire application lives in `index.html` with no external dependencies, no build step, and no package manager.

## Development

**Running the game**: Open `index.html` directly in a browser (file:// works fine — no server required).

**No build, lint, or test commands exist.** All development is done by editing `index.html` directly and refreshing the browser.

## Architecture

The entire codebase is `index.html` (~730 lines), structured as:

- **Lines 7–84**: Inline CSS — canvas sizing, D-pad button layout, overlay modal, touch-action suppression
- **Lines 108–120**: HTML D-pad buttons wired via `ontouchstart`/`onmousedown` calling `setDir(dx, dy)`
- **Lines 122–729**: Inline `<script>` containing all game logic

### Game Loop & State

Global state variables (declared at the top of the script): `maze`, `dots`, `score`, `lives`, `level`, `gameState`, `player`, `chasers`, `powerTimer`, `frightTimer`, `inputDir`, `nextDir`.

The main loop is `gameLoop()` (called via `requestAnimationFrame`). Each frame:
1. `updatePlayer()` — moves player, checks dot/ring collection, applies next direction at grid alignment
2. `updateChasers()` — moves each chaser, handles frightened AI vs. normal pathfinding
3. Drawing functions: `drawMaze()`, `drawPlayer()`, `drawChaser()`, `drawPowerOverlay()`

### Maze

`mazeTemplate` is a 17×17 array of integers:
- `0` = open path, `1` = wall, `2` = heart collectible, `3` = ring power-up, `4` = respawn safe zone

`resetMaze()` copies the template into `maze[]` and counts total dots.

### Entity System

Player and chasers are plain objects: `{ x, y, dx, dy, speed }`. Position is in pixel space; `CELL = 20`. Grid alignment is checked by `isAligned()` (within 3px of a cell center). `canMove(x, y, dx, dy)` / `canMoveAt(cx, cy)` handle wall collision.

### Chaser AI

At grid alignment, each chaser picks the open direction that minimizes Euclidean distance to the player (normal mode) or a random open direction (frightened mode, 60% speed). After being eaten, chasers respawn after 120 frames from a fixed spawn point.

### Input

Three simultaneous input paths all call `setDir(dx, dy)`:
- Keyboard arrow keys / WASD (`keydown` listener)
- Swipe detection on canvas (`touchstart`/`touchend`)
- HTML D-pad buttons (`ontouchstart`/`onmousedown`)

`setDir` sets `nextDir`; the player applies it only when grid-aligned, enabling queued turns.

### Audio

Web Audio API only — no audio files. `playBeep(freq, duration)` creates a short oscillator burst. Called inline at collection/collision events.

### Scaling

Canvas scales to the viewport on load using `devicePixelRatio` and `window.innerWidth`/`innerHeight`. All game coordinates use the logical `CELL`-based grid; the canvas context is scaled once at init.
