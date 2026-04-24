# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Classic Pac-Man arcade game implemented as a single-file HTML5 Canvas application with vanilla JavaScript. No build tools, no dependencies, no bundler — the entire game is `index.html` (~660 lines of HTML + CSS + JS). User-facing text is Japanese.

## Running the game

```bash
open index.html          # macOS
xdg-open index.html      # Linux
start index.html         # Windows
```

There is no build, no test suite, and no linter. Changes are verified by opening the file in a browser and playing.

## Architecture

All code is in one `<script>` block in `index.html`. The runtime has three layers that cooperate via shared module-level `let`/`const` state — there are no classes.

1. **State** (declared at `index.html:87-95`): `maze`, `pac`, `ghosts`, `score`, `lives`, `level`, `state`, `powerTimer`, `frame`, plus `hiScore`, `readyTimer`, `fruit`, `fruitTimer`, `dotsEaten`, `totalDots`, `ghostEatCombo`, `floatingScores`. `LAYOUT` (line 58) is the immutable source-of-truth maze; `maze` is a per-level mutable copy produced by `resetLevel()`.
2. **Game loop** (`gameLoop` → `tick` → `draw`, driven by `requestAnimationFrame`): `tick()` only advances when `state === 'playing'`. Pac-Man moves every frame; ghosts move every `Math.max(1, 3 - Math.floor(level / 3))` frames so they speed up with level. Collisions run after each tick, then `checkWin()` triggers `resetLevel()` and increments `level`.
3. **Rendering** (`draw()`): pure function of current state — maze cells, fruit, Pac-Man (arc with animated mouth), ghosts (body + eyes, or frightened blue/white flash), floating scores, and the appropriate overlay for `state` (`start` / `playing` + READY / `gameover`).

### Maze encoding
`LAYOUT` is a 22-row × 19-col grid where `1`=wall, `0`=dot (10 pts), `2`=power pellet (50 pts), `3`=empty. `isWall()` treats out-of-bounds as wall; `wrapX()` implements horizontal tunnel wrapping (cells marked `3` on the edges of rows 8, 10, 12).

### Ghost AI (`getGhostTarget`, `index.html:318`)
Each ghost has a distinct target, mimicking the arcade original. Ghosts pick the valid neighbor (no reversing) with the smallest Euclidean distance to the target:
- **Blinky (red, idx 0)** — target is Pac-Man's current tile.
- **Pinky (pink, idx 1)** — target is 4 tiles ahead of Pac-Man.
- **Inky (cyan, idx 2)** — target is the reflection of Blinky's position through a point 2 tiles ahead of Pac-Man.
- **Clyde (orange, idx 3)** — chases Pac-Man when distance > 8; otherwise scatters to bottom-left `(0, ROWS-1)`.

When `frightened`, ghosts pick a random valid direction. When eaten, they respawn at `homeX/homeY` after `respawnTimer = 40` ticks.

### Power-ups and scoring
`powerTimer = 7 * 10` on power pellet pickup (70 ticks ≈ 7 seconds of frightened state). Ghost-eat score escalates via `ghostEatCombo`: `200 * 2^(combo-1)` → 200/400/800/1600. Frightened ghosts flash white when `powerTimer < 30`. `floatingScores` are rising ephemeral labels drawn for 40 ticks after eating a ghost or fruit.

### Fruit
Spawns once at `(9, 13)` when `dotsEaten` hits 70 or 170; the fruit type/points come from `FRUITS[min(level-1, 6)]` (🍒100 → 🔑3000). Fruit despawns after `fruitTimer = 100` ticks if not eaten.

### Audio
`playSound(type)` lazily creates a single `AudioContext` on first use and synthesizes each effect (`chomp`, `power`, `eatghost`, `death`, `fruit`, `levelup`, `start`) from an `OscillatorNode` + `GainNode`. No audio files.

### High score
Persisted in `localStorage` under key `pacman-hiscore`. Written only on game-over when `score > hiScore`.

### Input
`keydown` handler maps arrows and WASD (both cases) to `[pac.nextDx, pac.nextDy]` for input buffering; the same handler, plus click and touchend, starts or restarts the game from `start`/`gameover` states. Touch uses start/end coordinates to infer a swipe direction.

## Conventions

- Keep everything in `index.html`. Don't split into modules, add a bundler, or introduce a framework.
- Japanese for user-facing strings (overlays, controls hint, README).
- Module-level `let`/`const` state, standalone functions. No classes.
- Changes to layout/AI/timing are made by editing the relevant constant or function in place — there is no config file.

## Key modification points

- **Maze layout** — `LAYOUT` at `index.html:58`
- **Ghost AI targeting** — `getGhostTarget()` at `index.html:318`
- **Ghost movement / frightened behavior** — `moveGhost()` at `index.html:340`
- **Ghost speed curve** — frame-skip formula in `tick()` at `index.html:441`
- **Power pellet duration** — `powerTimer = 7 * 10` in `movePac()` at `index.html:281`
- **Fruit spawn thresholds and points** — `FRUITS` table at `index.html:100` and spawn check at `index.html:289`
- **Sound design** — `playSound()` switch at `index.html:118`
- **Controls** — `keydown` map at `index.html:618`
