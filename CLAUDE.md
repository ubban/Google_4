# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The entire game—HTML structure, CSS styling, and JavaScript gameplay logic—is contained in `index.html` (1669 lines). The game runs directly in a web browser without build tools, external dependencies, or asset files.

## Development & Running

### No build process required
The game runs directly as an HTML file. Simply open `index.html` in a modern web browser or serve it via any HTTP server.

**To test locally:**
```bash
# Option 1: Direct file open
open index.html

# Option 2: Simple HTTP server for better compatibility
python3 -m http.server 8000
# Then visit http://localhost:8000
```

### Key development constraints
- **Single-file architecture**: All code is in `index.html`. Changes to styling, structure, or gameplay logic all go in this file.
- **No dependencies**: No npm packages, no build tools, no external libraries. WebAudio API and Canvas API are the only web APIs relied upon.
- **No external assets**: All visuals are procedurally generated (canvas drawing). All audio is synthesized via WebAudio (no sound files).
- **LocalStorage only**: Game state persists using browser localStorage with keys prefixed `skyfighter.*`.

## Architecture & Key Systems

### Game Structure (lines 422–1669)

The JavaScript is wrapped in an IIFE (Immediately Invoked Function Expression) to avoid global namespace pollution. The file uses a procedural architecture with the following major systems:

#### 1. **Game State & Lifecycle**
- `gameRunning`, `gamePaused`, `gameOver` flags control game flow
- Difficulty settings (easy/normal/hard) scale enemy counts, speed, and AI behavior
- Score, best score, lives, and bombs are tracked globally and persisted to localStorage

#### 2. **Canvas Rendering**
- 2D canvas-based rendering at a fixed 800×600 game area (responsive scaling to fit window)
- Adaptive performance detection (lines 473–483): tracks frame times and adjusts `quality` level (0=full, 1=medium, 2=low) to maintain 60 FPS
  - If avg frame time > 1/40s, reduce quality (skip some particle/effect rendering)
  - If avg frame time < 1/56s, increase quality

#### 3. **Difficulty & Spawning System**
- Three difficulty tiers defined in `DIFFICULTIES` object (line 485)
- Enemy spawn rate scales with difficulty; harder difficulties spawn faster and more frequently
- `KILLS_PER_STAGE` = 18: player must kill 18 enemies to progress to the next stage
- Boss appears after each stage

#### 4. **Enemy & Boss System**
- **Enemy Types** (`ENEMY_TYPES`, line 502): `grunt` (fast, weak), `interceptor` (very fast, rare), `tank` (slow, tanky)
- **Boss Lineup** (`BOSSES`, line 508): 6 named bosses with unique color schemes and HP multipliers that cycle through stages
- Enemy AI includes:
  - Pathfinding with randomized weaving (interceptors only)
  - Probabilistic shooting based on difficulty
  - Speed variations per enemy type

#### 5. **Player Mechanics**
- **Movement**: Arrow keys, WASD, or mouse drag (desktop) / touch drag (mobile)
- **Weapons**: Vulcan (starting), Laser, Spread, Homing—each upgradeable via power-ups
- **Bombs**: Limited smart bombs that damage nearby enemies (X key / Shift / mobile button)
- **Lives & Damage**: Player has 3 or 4 lives depending on difficulty; each enemy hit costs 1 life

#### 6. **Audio System**
- WebAudio synthesis (lines 542–595): no audio files, all sound effects are generated procedurally
- `beep()` for tones with optional frequency sweep and filtering
- `noiseBurst()` for noise-based effects
- Master gain and compressor for audio balance
- Mute toggle stored in localStorage

#### 7. **Input Handling**
- **Desktop**: Keyboard (arrows/WASD, Space, X/Shift, P/Esc), mouse movement and click
- **Mobile**: Touch zones divided into movement (left 55%) and action buttons (right 45%, with FIRE and BOMB buttons)
- Touch joystick base appears on first touch in move zone

#### 8. **Visual & UX Systems**
- **Themes** (line 493): 6 distinct day/night/weather themes rotate by stage, affecting sky gradient and colors
- **HUD**: Score, best score, weapon level, multiplier, hearts, bomb count displayed persistently
- **Overlays**: Main menu, pause screen, game-over screen with replay stats
- **Accessibility**: Reduced motion support (respects `prefers-reduced-motion` CSS media query), screen reader announcements via `aria-live` region
- **Visual feedback**: Damage vignette flash, combo pop animations, stage banner, boss health bar

## Code Patterns & Key Functions

### Main Game Loop
The game runs via `requestAnimationFrame`. Each frame:
1. Clears the canvas
2. Updates all entity positions and timers
3. Handles collision detection
4. Renders all entities
5. Updates HUD

### Collision & Damage
- Bullets check bounding-box collision against enemies/bosses
- Enemy bullets check collision against player
- Bombs have a radial damage zone

### Weapon System
- Each weapon type has unique behavior (fire rate, spread angle, homing logic)
- Weapon levels increase damage or fire rate
- Power-ups dropped by enemies upgrade weapons or add lives/bombs

### State Persistence
- Best score, mute setting, difficulty choice, reduced-motion preference all saved to localStorage
- Keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`

## Common Development Tasks

### Adding a New Weapon Type
1. Define the weapon in the weapon constants (around line 1000–1200)
2. Add firing logic in the main update loop's weapon section
3. Update weapon upgrade progression

### Changing Difficulty Balancing
Edit the `DIFFICULTIES` object (line 485). Tweak:
- `spawnBase` / `spawnMin`: enemy spawn frequency
- `enemySpeed`: speed multiplier for all enemies
- `enemyShootChance`: how often enemies fire
- `lives` / `bombs`: player resources
- `bossHpMul`: boss health scaling

### Adding a New Boss or Enemy Type
- Add entry to `BOSSES` array (line 508) or `ENEMY_TYPES` object (line 502)
- Update `minStage` in enemy types if spawning should be limited to later stages

### Tweaking Visual Themes
Edit `THEMES` array (line 493). Each theme defines sky gradient colors, cloud color, and sun color. The theme cycles per stage.

### Audio Tuning
Modify parameters in `beep()` and `noiseBurst()` calls throughout the code, or adjust master gain (`masterGain.gain.value` line 556).

## Testing Notes

- Test on both desktop and mobile to ensure input handling works as expected
- Verify accessibility: use a screen reader or test with reduced-motion enabled
- Check frame rate stability using browser DevTools performance profiler—watch for dips below 60 FPS
- Verify localStorage is working (open DevTools, Application tab, LocalStorage)

## Git Workflow

- Develop on branch `claude/inspiring-planck-4cxr3y`
- Commit changes with clear messages describing gameplay or mechanic changes
- Push to origin when ready for review
