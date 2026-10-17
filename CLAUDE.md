# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a vertical shoot-'em-up game (Raiden-style) built as a single HTML5 Canvas application. It's a complete, self-contained game with no external dependencies or build process—everything (game logic, audio synthesis, UI) is in `index.html`.

## Game Architecture

The entire game is contained in a single-file HTML application (~1669 lines). The structure is organized in layers within the JavaScript IIFE:

### Core Game Loop & Rendering
- **Canvas & Context**: Fixed 800x600 canvas rendered to fit the viewport
- **Frame Loop**: Runs at ~60fps with adaptive performance management (quality levels 0-2)
- **Rendering Pipeline**: 
  - Background layers (gradient sky, clouds, animated sun/shadow)
  - Game entities (player, enemies, bullets, effects)
  - HUD overlay (score, lives, weapon info, boss health bar)
  - UI overlays (menus, pause screen, game-over screen)

### Entity Management
Entities are tracked in simple arrays:
- `player` (single object)
- `enemies` (array of enemy objects)
- `bullets` (projectiles from player and enemies)
- `enemyBullets` (projectiles fired by enemies)
- `effects` (visual effects like explosions)
- `allEntities` (convenience array for iteration)

Each entity has `x`, `y`, `w` (width), `h` (height), `vx`/`vy` (velocity), `update()`, and `draw()` methods.

### Game State Management
- **Phases**: `'menu'` → `'playing'` → `'wave'` → `'boss'` → `'gameOver'` → back to `'menu'`
- **Difficulty**: Three levels (easy, normal, hard) stored in localStorage and affecting spawn rates, enemy behavior, lives/bombs
- **Progression**: Stages progress after defeating `KILLS_PER_STAGE` (18) enemies; boss spawns every 6 stages

### Weapon System
Weapons are defined in `WEAPON_DATA` with properties like:
- `cycle` (fire rate in frames)
- `bps` (bullets per shot)
- `scatter` (spread angle)
- `vel` (projectile velocity)
- `color`, `dark` (visual properties)

Each weapon has a level (upgraded by pickups), affecting fire rate and spread.

### Audio
Synthesis-based audio using Web Audio API:
- No external files—all sounds generated procedurally
- `beep(freq, dur, type, vol, opts)` is the core sound function
- Effects include filters, frequency sweeps, panning, delays
- Can be muted via `localStorage.getItem(STORAGE_MUTE)`

### Input Handling
- **Keyboard**: Arrow keys + Space (fire) / X (bomb) / P (pause) / M (mute)
- **Touch**: Virtual joystick on left, fire/bomb buttons on right
- **Responsive**: Touch controls hidden on desktop, shown on mobile/tablet

### Visual Themes
Six selectable environment themes (`THEMES` array) with custom sky gradients, cloud colors, and sun colors.

### Accessibility Features
- ARIA live region for announcements (`srLive`)
- Keyboard navigation for all menus
- Support for `prefers-reduced-motion` media query
- Semantic HTML for screen reader support

## Common Development Tasks

### Running the Game
No build or development server is needed. Simply:
1. Open `index.html` in a web browser (file:// protocol works fine)
2. Or, from a terminal: `python3 -m http.server 8000` and visit `http://localhost:8000`

### Testing Specific Features
- **Difficulty Balance**: Change the `DIFFICULTIES` object (lines ~486-489) and test with different `spawnBase`, `enemySpeed`, `enemyShootChance`
- **Enemy Behavior**: Enemy types and their properties are in `ENEMY_TYPES` (lines ~502-506); adjust `hp`, `speedMul`, or `score`
- **Boss Difficulty**: Modify `BOSSES` array (lines ~508-515) to adjust `hpMul` (HP multiplier)
- **Weapon Balance**: Edit `WEAPON_DATA` for fire rate, spread, and damage behavior
- **Audio**: The `beep()` function creates all sounds; adjust frequency, duration, and type to test new sound effects

### Debugging
- **Console Logging**: Add `console.log()` statements; the game doesn't suppress console output
- **Performance**: Frame timing is tracked in `frameTimes` array; check avg frame time to identify bottlenecks
- **Canvas Rendering**: Inspect canvas draw calls in DevTools—all rendering is 2D Canvas API (no WebGL)

### Making Changes
- **Safe Small Changes**: Tweak numbers in config objects (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`)
- **Medium Changes**: Add new enemy types by extending `ENEMY_TYPES`; add new bosses by extending `BOSSES`
- **Risky Changes**: Modifying the game loop logic (`update()` / `draw()`), collision detection, or weapon fire mechanics—test thoroughly

## Key Constants & Configuration

| Constant | Purpose | Location |
|----------|---------|----------|
| `KILLS_PER_STAGE` | Enemies to defeat before advancing | ~491 |
| `DIFFICULTIES` | Per-difficulty spawn/behavior tuning | ~485 |
| `ENEMY_TYPES` | Enemy definitions (grunt, interceptor, tank) | ~502 |
| `BOSSES` | Boss definitions (6 unique bosses) | ~508 |
| `THEMES` | Visual themes (6 variants) | ~493 |
| `WEAPON_DATA` | Weapon fire behavior | (searched in file) |
| `STORAGE_*` | LocalStorage keys for persistence | ~463 |

## Browser Compatibility

- Requires modern browser with:
  - HTML5 Canvas 2D context
  - Web Audio API (for audio synthesis)
  - Touch Events API (for mobile)
  - localStorage (for best score / settings)
- Tested on Chrome, Firefox, Safari, Edge
- Mobile/tablet support via touch controls

## Known Limitations & Potential Improvements

- Single-file design trades modularity for simplicity; refactoring into modules would require a build step
- Audio synthesis CPU cost may impact low-end devices (adaptive quality helps mitigate)
- No network features (no online multiplayer or high-score leaderboard)
- Screen shake and vignette effects disabled under `prefers-reduced-motion`
