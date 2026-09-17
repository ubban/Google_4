# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application with:
- Canvas-based 2D rendering
- WebAudio API for procedural sound effects (no external audio files)
- LocalStorage for persistent state (high score, difficulty, preferences)
- Responsive design supporting desktop, tablet, and mobile (touch + keyboard controls)
- Adaptive quality scaling to maintain ~60 FPS on low-end devices

## Architecture

The entire game runs in a single `index.html` file (~1669 lines) structured as:

1. **HTML head** (lines 1-359): Responsive CSS with:
   - Dark-themed UI using CSS custom properties
   - HUD elements for score, lives, bombs, weapon info
   - Boss health bar, combo popup, stage banner animations
   - Touch control zones (joystick + buttons)
   - Accessibility: `prefers-reduced-motion` support, ARIA live region for announcements

2. **HTML body** (lines 361-420): Semantic elements:
   - Game canvas
   - Visual effects overlays (vignette, flash)
   - HUD stats display
   - Main menu/overlay
   - Touch control buttons

3. **JavaScript** (lines 422+): Game logic inside an IIFE, organized into sections:
   - **Audio system**: WebAudio synth generating dynamic SFX (no pre-recorded files)
   - **Game state**: Player, enemies, bullets, power-ups, stages, bosses
   - **Rendering**: Canvas drawing functions for entities, effects, HUD
   - **Input handling**: Keyboard + mouse on desktop, touch + joystick on mobile
   - **Game loop**: requestAnimationFrame with delta-time physics
   - **Difficulty system**: Three modes (Easy/Normal/Hard) affecting spawn rates, enemy behavior, lives, bombs
   - **Progression**: 6 stages with themed environments, scaling enemy difficulty, boss encounters

### Key Game Mechanics

- **Player**: Center-bottom of screen, 3-4 lives (difficulty-dependent), controlled via mouse/keyboard/touch
- **Weapons**: 3 tiers (Vulcan, Laser, Inferno) with increasing fire rate and damage
- **Enemies**: 3 types (Grunt, Interceptor, Tank) with different speeds, HP, and patterns
- **Power-ups**: Weapon upgrades and bonus lives
- **Bombs**: Screen-clearing smart bombs with limited uses (2-3 per life)
- **Bosses**: 6 unique boss designs, one per stage, with distinct attack patterns and phases
- **Scoring**: Base points per kill + multiplier for consecutive hits without taking damage
- **Combo system**: Multiplier resets on hit; visual feedback with combo popups

### Difficulty & Progression

**DIFFICULTIES** constant (line 485-489) defines per-difficulty tuning:
- `spawnBase/spawnMin`: Enemy spawn timing (lower = faster)
- `enemySpeed`: Multiplier for enemy movement speed
- `enemyShootChance`: Probability each enemy fires per tick
- `lives/bombs`: Starting values
- `bossHpMul`: Health multiplier for bosses

**THEMES** (lines 493-500): 6 visual themes (gradients, colors, names) cycling through stages

**BOSSES** (lines 508-515): 6 boss definitions with unique coloring, phases, and health scaling

**ENEMY_TYPES** (lines 502-506): Enemy stats (size, HP, speed, color, score value, minimum stage)

## Development Guidelines

### Running the Game

**Desktop**: Open `index.html` in a modern browser (Chromium, Firefox, Safari)
- No build step, no server needed
- Fully playable offline

**Testing Input**: 
- Keyboard: Arrow keys / WASD to move, Space to fire, X / Shift to bomb, P to pause
- Mouse: Move to position, click to fire
- Touch: Drag in left zone to move, tap FIRE and BOMB buttons

### Making Changes

The single-file structure prioritizes portability but makes edits less compartmentalized. When modifying:

1. **Game balance** (difficulty, enemy stats, boss HP, scoring): Update `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES` constants
2. **Visuals** (colors, animations, UI): Edit CSS in `<style>` or canvas drawing code (search for `ctx.` calls)
3. **Audio** (SFX design, volume): Modify `sfx` object and `beep()`/`noiseBurst()` calls
4. **Game mechanics** (player behavior, enemy AI, spawning): Find the relevant section in the game loop or update functions
5. **Controls**: Input handlers (keyboard, mouse, touch joystick) start around line 768

### Key Functions & Concepts

- **Game loop** (`rafLoop`): Handles physics, rendering, and game state updates at 60 FPS
- **Adaptive quality** (`trackPerf`, `quality` var): Reduces particle/effect density if FPS drops below 40
- **Screen shake**: Non-zero `shakeTime`/`shakeMag` offsets canvas context, respects `reducedMotion`
- **Player rendering**: `renderPlayer()` draws an abstract aircraft shape
- **Enemy AI**: Each enemy type has a `move()` behavior and random `shoot()` logic
- **Boss patterns**: Each boss has 4 attack phases; phases advance at health thresholds
- **Collision**: Simple AABB (axis-aligned bounding box) checks via `rectsOverlap()`
- **LocalStorage**: Persistence for best score, mute state, difficulty, motion preferences (lines 517-526)

### Touch & Mobile

The game auto-enables touch controls on devices with `ontouchstart` or `maxTouchPoints > 0`. Visual joystick appears when player starts dragging in the left half of the lower screen. Two large buttons (FIRE, BOMB) are always visible in the right half. This ensures comfortable play on phones/tablets without forcing a specific screen size.

### Accessibility

- Color-based UI (hearts for lives, bombs for remaining) also include emoji
- `prefers-reduced-motion` is respected: animations, screen shake, and flash are disabled
- Game screen announces state changes via ARIA live region (score multiplier, stage clear, etc.)
- All buttons have `title` attributes
- Semantic HTML with proper ARIA roles

### Performance Notes

- **Canvas size**: Fixed 480×720 (game coordinates), scaled with CSS for responsive display
- **Rendering**: Only entities on screen are drawn; off-screen cleanup is built-in
- **Audio**: WebAudio context suspended until first user input; resumed on demand
- **Quality scaling**: If avg frame time exceeds 25ms, particle density and certain effects are reduced

### Common Edits

**Increase difficulty**:
- Lower `spawnBase`/`spawnMin` in `DIFFICULTIES` to spawn enemies faster
- Raise `enemyShootChance` or `bossHpMul`

**Add a new enemy type**:
- Add entry to `ENEMY_TYPES` object (line 502)
- Update `possibleEnemies` array in stage logic
- Add drawing code in `renderEnemy()` and explosion SFX mapping in `EXPLODE_SFX`

**Add a new boss**:
- Add entry to `BOSSES` array (line 508)
- Define attack phases in the boss update logic (search for `boss.phase`)
- Tune `hpMul` to balance difficulty

**Tune audio**:
- Modify `beep()` frequency, duration, or volume parameters
- Adjust filter type/frequency for tonal color
- Increase `noiseBurst()` duration for longer, deeper explosions

## Testing Checklist

- [ ] Game starts and accepts keyboard input (arrows, space, X/shift for bomb)
- [ ] Mouse aiming and click-to-fire works (desktop)
- [ ] Touch joystick appears and responds (mobile)
- [ ] Enemy spawning increases in difficulty as stages progress
- [ ] Bosses appear at stage end and have visible attack patterns
- [ ] Bomb clears screen and plays sound
- [ ] Score multiplier increments and resets correctly
- [ ] Lives and bombs display updates
- [ ] Game over triggers when lives reach zero
- [ ] Best score persists across sessions
- [ ] Difficulty selection affects enemy behavior visibly
- [ ] Audio mute button works (localStorage remembered)
- [ ] Reduced motion toggle removes animations as expected
- [ ] All visual effects respect `prefers-reduced-motion` preference
