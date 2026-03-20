# CLAUDE.md

## Project Overview

Classic Pac-Man arcade game implemented as a single-file HTML5 Canvas application with vanilla JavaScript. No build tools, no dependencies, no external assets — just open `index.html` in a browser.

## Repository Structure

```
pac-man/
├── CLAUDE.md      # This file — AI assistant guide
├── README.md      # Game description and controls (Japanese)
└── index.html     # Complete game (HTML + CSS + JS, ~400 lines)
```

## Tech Stack

- **HTML5 Canvas** for rendering
- **Vanilla JavaScript** (no frameworks, no bundler)
- **Google Fonts** (Orbitron, Press Start 2P) loaded via CDN
- No package.json, no npm, no build step

## Architecture (index.html)

All code lives in a single `<script>` tag. Key sections:

| Section | Lines | Description |
|---------|-------|-------------|
| HTML/CSS | 1–49 | Layout, HUD, retro arcade styling |
| Constants | 52–89 | `CELL`, `LAYOUT` (maze grid), ghost config |
| Game state | 86 | `maze`, `pac`, `ghosts`, `score`, `lives`, `level`, `state`, `powerTimer`, `frame` |
| `initGame()` | 91–105 | Reset all state, place entities |
| Movement | 107–184 | `isWall()`, `wrapX()`, `movePac()`, `moveGhost()` |
| Collisions | 186–210 | `checkCollisions()`, `checkWin()` |
| Game loop | 212–235 | `tick()` — update logic at ~10 FPS effective for ghosts |
| Rendering | 237–351 | `draw()` — canvas rendering (maze, pac, ghosts, overlays) |
| HUD | 353–357 | `updateHUD()` — score/lives/level DOM updates |
| Input | 365–397 | Keyboard (arrows + WASD), click, and touch/swipe |
| Startup | 399–400 | `initGame()` + `gameLoop()` |

## Game Mechanics

- **Maze**: 22×19 grid. `1`=wall, `0`=dot (10 pts), `2`=power pellet (50 pts), `3`=empty
- **Pac-Man**: Grid-based movement with input buffering (`nextDx`/`nextDy`)
- **Ghosts**: 4 ghosts (Blinky/Pinky/Inky/Clyde) with Manhattan-distance chase AI; random movement when frightened
- **Power-ups**: 7-second duration (70 frames), ghosts flash white before expiring
- **Levels**: All dots eaten → next level; ghosts speed up with level progression
- **Lives**: 3 lives; game over when all lost
- **Tunnels**: Horizontal wrapping at maze edges

## Game States

- `start` — Title screen overlay; click or keypress to begin
- `playing` — Active gameplay
- `gameover` — Score display; click or keypress to restart

## Development Workflow

### Running the game
```bash
# Just open index.html in any browser
open index.html          # macOS
xdg-open index.html      # Linux
start index.html         # Windows
```

### No build/test/lint commands
This project has no build system, no test framework, and no linter configuration. Changes are tested manually by playing the game in a browser.

## Conventions

- All game code is in a single file — keep it that way for simplicity
- Japanese language for user-facing text (README, in-game messages)
- Global scope with `let`/`const` for game state
- Functions are standalone (no classes)
- Canvas rendering in `draw()`, game logic in `tick()`, input handling via event listeners
- `requestAnimationFrame` drives the game loop

## Key Modification Points

- **Maze layout**: Edit the `LAYOUT` 2D array (line 57–79)
- **Game speed**: Adjust ghost frame-skip formula in `tick()` (line 225)
- **Power-up duration**: Change `powerTimer = 7 * 10` in `movePac()` (line 142)
- **Ghost AI**: Modify `moveGhost()` (line 159–184)
- **Visual style**: CSS at top of file + canvas drawing in `draw()`
- **Controls**: Key mappings in `keydown` handler (line 365–378)
