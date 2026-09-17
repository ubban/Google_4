# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a browser-based Raiden-style vertical shoot-'em-up game. It's a single-page HTML5 application with no build process or external dependencies—the entire game (visual, audio, logic) is self-contained in `index.html`.

## Running the Game

- **Direct browser**: Open `index.html` in any modern web browser (Chrome, Firefox, Safari, Edge)
- **Local server** (optional): For best compatibility, serve via HTTP: `python -m http.server` or `npx http-server`
- **No build step required**: The HTML file runs as-is

## Core Game Architecture

The game is organized as a single JavaScript IIFE (Immediately Invoked Function Expression) within `<script>` tags in `index.html`. Key systems:

### 1. **Game State & Variables**
- **Canvas & rendering**: `canvas`, `ctx` (2D context)
- **UI elements**: Score, health (hearts), bomb count, weapon display, boss health bar
- **Game loop**: `raf()` function drives animation at ~60 FPS
- **Difficulty settings**: `DIFFICULTIES` object defines enemy spawn rates, speed, shoot chance, lives, bombs by difficulty
- **Storage**: LocalStorage persists best score, mute state, difficulty, and motion preferences

### 2. **Entity System**
Three main entity types:
- **Player**: Single instance, position-based movement, fires bullets, drops bombs
- **Enemies**: Multiple instances (`grunt`, `interceptor`, `tank`) with varying HP, speed, and AI
- **Bosses**: 6 unique boss designs (`VANGUARD`, `BEHEMOTH`, `SPECTER`, `DREADNOUGHT`, `WRAITH`, `TITAN`) with custom rendering and attack patterns

All entities stored in arrays (`enemies`, `bullets`, `playerBullets`, etc.) and updated each frame.

### 3. **Weapon System**
- **Vulcan**: Default, rapid-fire spread pattern
- **Laser**: High damage, narrow beam, upgradeable levels
- **Missile**: Homing projectiles with area damage
- **Mortar**: Arc trajectory with slow-moving shells
Each weapon has distinct bullet behavior, fire rate, and damage.

### 4. **Combat & Scoring**
- **Weapon multiplier**: Bonus damage and score for consecutive hits without taking damage (multiplier resets on hit)
- **Enemy scoring**: Points vary by enemy type and stage
- **Boss health bar**: Displayed dynamically when boss is active
- **Hit feedback**: Screen flash, red vignette pulse, and screen shake on player damage (all accessibility-respectful via `prefers-reduced-motion`)

### 5. **Rendering Pipeline**
- **Backgrounds**: Parallax sky gradient with animated clouds and sun
- **HUD**: Static overlay showing score, best, weapon, lives, bombs
- **Entities**: Canvas-rendered with custom shapes (no sprites):
  - Player: Fighter jet silhouette with wings
  - Enemies: Colored geometric shapes
  - Bosses: Detailed multi-colored hull with wings and core
- **Effects**: Flash overlays, vignette (damage indicator), combo pop text, stage banners
- **Accessibility**: Uses `prefers-reduced-motion` media query to disable animations for users who need it; screen reader live region for announcements

### 6. **Input System**
- **Desktop**: Mouse (move + click), Arrow keys / WASD (movement), Space (fire), X / Shift (bomb), P / Esc (pause)
- **Mobile**: Touch drag left zone to move, right zone has FIRE and BOMB buttons
- **Joystick visualization**: Shows on touch input; base appears on first touch

### 7. **Audio Synthesis**
- **WebAudio API**: All sounds generated via synth (square waves, noise bursts, frequency sweeps)
- **No external assets**: Completely procedural
- **Sound categories**: Fire, impact, bomb, enemy spawn, boss defeat, level complete
- **Panning**: Spatial audio based on entity x-position
- **Mute toggle**: Persistent via localStorage

### 8. **Game Flow**
1. **Menu state**: Display overlay with difficulty selection, controls hint
2. **Play state**: Entities update, player input processed, collisions checked, score updated
3. **Stage progression**: After ~18 enemy kills, advance stage (spawn boss)
4. **Boss fight**: Boss appears, special attack patterns, defeat triggers next stage
5. **Game over**: On player health = 0, show stats overlay with best score comparison
6. **Pause**: Freezes game loop, displays overlay, resumes on input

### 9. **Difficulty Levels**
Three presets adjust spawn rate, enemy speed, shoot chance, available lives, bombs, and boss HP multiplier. Difficulty selection on menu persists in localStorage.

### 10. **Theme System**
Six visual themes cycle per stage, each with custom sky gradient colors, cloud color, and sun color. No explicit stage theming—themes rotate automatically.

## Key Development Notes

- **Single-file constraint**: All changes go into `index.html`. Keep inline styles and scripts organized for readability.
- **Performance optimization**: Adaptive quality system (`trackPerf()`) dynamically reduces particle/effect density on low-end devices.
- **Collision detection**: Simple AABB (axis-aligned bounding box) for all entity-entity checks.
- **Entity cleanup**: Dead entities are filtered out each frame; no memory leaks expected.
- **Canvas resolution**: Responsive; canvas size matches window size, elements scale via CSS `clamp()` for mobile/desktop.
- **Accessibility**: Respects `prefers-reduced-motion`, includes screen reader live region for game announcements, high-contrast colors, keyboard and touch controls.

## Common Tasks

**Add a new weapon**:
1. Define weapon object in code (fire rate, bullet behavior, damage)
2. Add bullet rendering in the render loop
3. Add weapon switch logic in input handler

**Add a new enemy type**:
1. Define type in `ENEMY_TYPES` with properties (size, HP, speed, color, score, min stage)
2. Create spawner function to instantiate
3. Implement AI/movement in entity update loop
4. Add rendering code in main draw function

**Add a new boss**:
1. Define boss in `BOSSES` array with colors and HP multiplier
2. Implement boss constructor and update logic (attack patterns)
3. Add boss-specific rendering
4. Hook into stage progression logic

**Modify difficulty balance**:
- Adjust values in `DIFFICULTIES` object (spawn rates, enemy speed, lives, bombs)
- Test across all three difficulties in-game

**Debug collisions**:
- Add temporary canvas debug rectangles to visualize bounding boxes during render
- Check AABB overlap logic in collision detection functions

## Testing

Run the game in a browser and test:
- All three difficulty levels
- All weapon types and upgrades
- Mobile touch controls on an actual device (or use browser DevTools device emulation)
- Boss fights on each stage
- Game over and best score persistence
- Reduced motion toggle (verify animations disable when enabled)
- Audio mute toggle

No automated tests; testing is manual via browser playthrough.
