# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single HTML file with embedded CSS and JavaScript. The game runs entirely in the browser using the Canvas API and LocalStorage for persistence.

## Running the Game

- **Local Testing**: Open `index.html` directly in a browser (file:// protocol or via a simple HTTP server)
- **No Build Step**: The project requires no compilation or bundling—it's ready to run as-is
- **Browser Compatibility**: Requires modern browser with Canvas and Touch Events support

## Code Architecture

The entire game logic is in a single `<script>` tag in `index.html`, organized as an IIFE (Immediately Invoked Function Expression). The structure follows this pattern:

### Core Systems

1. **Game State & Config**
   - `DIFFICULTIES`: Three difficulty presets (easy/normal/hard) controlling spawn rates, enemy speed, lives, bombs, and boss HP
   - `KILLS_PER_STAGE`: Tracks enemy kill count to advance stages
   - `THEMES`: Six visual themes with sky gradients, cloud colors, and sun tints for stage variation
   - `STORAGE_*` constants: localStorage keys for best score, mute, difficulty, reduced motion

2. **Performance Adaptation**
   - `trackPerf()` and `quality` variable: Dynamically downgrades rendering quality if FPS drops below 40 (switches to lower particle counts, less frequent updates)
   - Maintains rolling average of frame times to detect sustained performance issues

3. **Entity System** (inferred from code structure)
   - Player: tracked separately with position, health, weapons, bombs
   - Enemies: spawned with configurable rates, movement speed, and shoot probability
   - Bullets: player projectiles and enemy fire
   - Particles: visual effects for explosions, impacts, stage transitions
   - Boss: special enemy type with health bar and multi-phase behavior

4. **Input Handling**
   - **Desktop**: Mouse (movement + click), Keyboard (arrows/WASD for move, Space/X/Shift for actions)
   - **Mobile**: Touch zones (left 55% for movement, right 45% for actions); joystick base shown on drag
   - Input state maintained in key maps or direction vectors; processed each frame

5. **Audio & Visual Feedback**
   - Mute state stored in localStorage; buttons toggle state
   - Screen shake (vignette animation), white flash on hit, bomb explosion flash
   - Reduced motion mode disables animations for accessibility; respects prefers-reduced-motion

6. **UI & HUD**
   - Real-time score, best score, multiplier, weapon name/level, health hearts, bomb count
   - Boss health bar (appears when boss active)
   - Combo popup animation when combos trigger
   - Stage banner announces theme name
   - Overlay screens for start menu, game over, difficulty selection

7. **Data Persistence**
   - Best score saved to `skyfighter.bestScore`
   - Difficulty preference to `skyfighter.difficulty`
   - Mute/reduced-motion toggles persist across sessions

### Key Game Loop Sections

- **Initialize**: Load saved state, attach event listeners, set up canvas
- **Render**: Clear canvas, draw background gradient (theme-based), render entities, update HUD
- **Update**: Move player/enemies, check collisions, spawn new enemies based on difficulty and kill count, handle weapon upgrades, track multiplier decay
- **Input Processing**: Poll keyboard state, handle mouse movement/clicks, process touch events
- **Collision Detection**: Player bullets vs enemies, enemy bullets vs player, pickups vs player
- **Stage/Boss Transitions**: Track kills, spawn boss when threshold met, show banner and theme changes

## Development Workflow

- **Edit the game**: Modify the `<script>` section in `index.html`
- **Test immediately**: Save and refresh the browser; no build step required
- **Check both input modes**: Test desktop (mouse + keyboard) and mobile (touch controls) when adding input features
- **Respect performance**: Use the adaptive quality system; avoid expensive operations in the render loop without profiling impact
- **Accessibility**: Announce significant events via `announce()` for screen readers; respect reduced-motion preferences

## Important Design Notes

- **Single Entity**: Player is tracked with explicit variables (health, weapon, bombs, position); enemies/bullets stored in arrays
- **Canvas Rendering**: All drawing happens via `ctx` (2D context); no DOM manipulation for game objects
- **LocalStorage Keys**: Always use the `STORAGE_*` constants; do not hard-code strings
- **Difficulty Balancing**: DIFFICULTIES object is the single source of truth for spawn and enemy behavior tuning
- **Stage Progression**: Stages advance every `KILLS_PER_STAGE` kills; themes cycle through THEMES array
