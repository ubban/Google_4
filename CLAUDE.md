# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game implemented as a single-file HTML5 application. The entire game (HTML, CSS, and JavaScript) is contained in `index.html` (~1,700 lines).

## Architecture

### Single-File Design
- All game logic, styling, and markup are in `index.html`
- No external dependencies (pure Canvas API, native Web Audio)
- Rendered directly in any modern browser

### Core Game Structure
The JavaScript is organized in a single IIFE (Immediately Invoked Function Expression) with the following major sections:

1. **DOM & Storage Management** — UI elements, canvas reference, localStorage for best score/settings
2. **Adaptive Performance** — Frame time tracking to adjust quality dynamically (3 levels: 0=full, 1=medium, 2=low)
3. **Game Configuration**
   - `DIFFICULTIES` — spawn rates, enemy speed, lives, bombs per difficulty (easy/normal/hard)
   - `THEMES` — 6 visual themes with sky/cloud/sun color palettes
   - `ENEMY_TYPES` — grunt, interceptor, tank with stats (HP, speed, score, appearance)
   - `BOSSES` — 6 boss variants with names and color schemes
4. **Audio Engine** — WebAudio synthesis (no external files); beep/noise generators
5. **Game State** — player, enemies, bullets, power-ups, boss, scoring
6. **Update & Render Loop** — requestAnimationFrame-driven, ~60 FPS target
7. **Input Handling** — keyboard (WASD/Arrows), mouse (move + click), touch (drag + buttons)
8. **Canvas Rendering** — 2D drawing for player, enemies, bullets, effects, UI
9. **UI/Overlay** — menu screen, pause, game-over stats, difficulty selection

### Game Flow
1. Start screen with difficulty selection and controls hint
2. Stage banner displays (`STAGE 1`, etc.)
3. Enemies spawn at increasing rates; player destroys 18 enemies to complete a stage
4. Collect weapon power-ups to level up gun (changes from VULCAN → LASER → SPREAD)
5. Boss appears after stage clear; defeat it to progress
6. Smart bombs (limited count) clear all bullets on-screen
7. Lose 1 life on hit; game over at 0 lives
8. Final screen shows score, best score, and difficulty completed

## Key Code Sections

### Player Ship
- Attributes: position (x, y), velocity, health, weapon level, bomb count
- Movement: mouse X position or keyboard input (constrained to canvas bounds)
- Firing: weapon system with rate-limiting; spreads/lasers based on power-up level

### Enemy System
- Spawned at intervals (configurable per difficulty); spawn rate increases over time
- Three types: grunt (1 HP, red), interceptor (1 HP, fast, red-orange), tank (4 HP, purple)
- Movement: vertical descent with optional weaving (interceptor)
- Firing: random chance to shoot; bullets are circles that descend
- Destroyed when hit; drop weapon power-ups or score pickups

### Boss System
- One boss per stage; appears after KILLS_PER_STAGE (18) enemy kills
- Boss sprite: stylized plane with hull colors, wings, core glow
- Spawn pattern: wave-based, circles player, fires bursts
- Health bar displayed at top while active
- Drops score/weapon power-ups on defeat

### Weapon System
- Levels: 1 (VULCAN—narrow spread), 2 (LASER—tighter), 3 (SPREAD—wide triple shot)
- Power-up frequency scales with stage difficulty and combo multiplier
- Weapon fires at fixed rate; bullets are rectangles with optional glow

### Scoring & Multiplier
- Base score: enemy type determines (grunt 10, interceptor 15, tank 35)
- Multiplier increases when destroying enemies fast (combo chain)
- Multiplier decays over time if player isn't hitting enemies
- Best score persisted in localStorage

### Smart Bombs
- Player starts with 2–3 bombs (difficulty-dependent)
- Activation: Space/X key or mobile BOMB button
- Effect: 0.5s flash, vignette pulse, all enemy bullets cleared; no damage to enemies
- Audio feedback on activation

## Important Constants & Configuration

### Difficulty Presets
Located near line 485:
```javascript
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
};
```
Tweaking spawn rates and enemy stats here alters game difficulty.

### Visual Themes
Located near line 493. Each theme has sky gradient colors (3-stop), cloud color, sun color, and a name. Themes cycle per stage.

### Enemy Stats
Located near line 502. Modifying `w`/`h` (size), `hp` (health), `speedMul` (movement speed), `color`/`dark` (appearance) affects gameplay feel.

### Boss Configuration
Located near line 508. Each boss has color scheme (hullTop, hullBottom, wing, core), name, and `hpMul` (health multiplier).

## Development Workflow

### Running the Game
- Open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge)
- No build step, no server required
- Works offline

### Making Changes
1. Edit `index.html` directly
2. Reload browser tab to test changes
3. Use browser DevTools (F12) to inspect canvas and debug JavaScript

### Testing Key Features
- **Gameplay**: Play through a stage, collect power-ups, defeat a boss
- **Difficulty**: Test all three difficulty presets; verify spawn/speed/lives match constants
- **Controls**: Keyboard (WASD, Space, X, P), mouse, touch/mobile
- **Audio**: Toggle mute button; verify beeps play without error
- **Performance**: Monitor frame rate; ensure adaptive quality doesn't cause visual glitches
- **Settings Persistence**: Change difficulty and mute state; refresh page and verify they persist
- **Reduced Motion**: Toggle the effects button; verify animations disable and no crashes occur

### Debugging Tips
- Open DevTools Console to check for JavaScript errors
- Use `console.log()` in the game loop (update section) to trace state
- Canvas rendering happens in `renderFrame()`; modify paint calls there
- Game state variables (player, enemies, boss, score, etc.) are declared at the top of the IIFE
- Use browser pausing (DevTools > Sources) to step through update/render logic

## Common Modifications

### Adjust Game Difficulty
Modify the `DIFFICULTIES` object or tweak `enemySpeed`, `spawnBase`, `enemyShootChance` inline.

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` with stats and appearance
2. Add spawn logic in the enemy-spawning section
3. Add drawing code in `renderFrame()`

### Change Visual Theme
Modify `THEMES` array or edit CSS color variables at the top of `<style>`.

### Tweak Weapon Behavior
Edit the weapon firing code (canvas drawing of bullets, movement, collision) and rate constants.

### Add Sound Effects
Extend the `beep()` and `noiseBurst()` functions; call them from relevant game events (hit, power-up, boss spawn).

## Storage Keys
Game state is persisted to localStorage:
- `skyfighter.bestScore` — player's best score
- `skyfighter.muted` — audio mute state (1 or 0)
- `skyfighter.difficulty` — last selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion` — reduced-motion preference (1 or 0)

## Accessibility
- Screen reader support via ARIA live region (`#srLive`)
- Keyboard controls fully supported (arrow keys, space, X, P)
- Reduced-motion toggle respects user OS preference and disables animations
- Mobile touch controls with visual feedback
- High contrast palette for visibility
