# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single HTML file (`index.html`). The entire game — styling, logic, and rendering — is self-contained with no external dependencies.

### Key Features
- Canvas-based rendering with procedural graphics
- Procedural audio synthesis (Web Audio API, no external audio files)
- Three difficulty levels (Easy, Normal, Hard) with persistent settings
- Touch/joystick controls for mobile devices
- Keyboard controls for desktop
- Progressive difficulty with 6 unique boss encounters
- Score multiplier system based on consecutive kills (chain mechanic)
- Adaptive performance tuning (quality scaling based on frame times)
- Accessibility: reduced-motion support, screen-reader announcements

## Architecture & Game Systems

### Core Game Loop & State
The game runs in an IIFE at the bottom of the file. Key game state variables:
- `running`, `paused`: Game state flags
- `player`: Object with `{x, y, w, h, level, speed}`
- `enemies`: Array of active enemy objects (grunt, interceptor, tank)
- `boss`: Current boss object (only one at a time)
- `bullets`: Array of projectiles from player
- `enemyBullets`: Array of enemy projectiles
- `powerUps`: Array of collectibles (life, bomb, weapon)
- `particles`: Array of explosion particle effects
- `frame`: Current frame counter (increments each render)
- `stage`: Current level (1–6 based on kills)
- Game vars: `score`, `multiplier`, `chain`, `lives`, `bombCount`

The main loop (`gameLoop()`) updates all entities, checks collisions, renders, and schedules the next frame via `requestAnimationFrame`.

### Difficulty System
Difficulty constants in `DIFFICULTIES` object define spawn rates, enemy behavior, lives, and boss HP scaling. Difficulty is persisted to localStorage and selected at game start.

### Enemy & Boss Design
- **Enemy types** (ENEMY_TYPES): grunt, interceptor, tank — each with distinct speed, health, scoring, and behavior (weaving patterns, spread vs. aimed fire)
- **Bosses** (BOSSES array): 6 unique boss designs with distinct visuals and HP multipliers; boss selection cycles based on stage
- Both drawn procedurally using canvas paths and gradients

### Weapon & Multiplier System
- **Weapons**: VULCAN (rapid-fire), LASER (single powerful shot), MISSILE (arc trajectory)
- Weapon level increments on pickup; max level is 6
- Kill chain builds multiplier: `1 + floor(chain / 5)`, max 10x
- Chain resets if player takes damage

### Audio System
Web Audio API synthesis with no external files:
- `beep()`: Oscillator-based tones with optional sweep, filter, panning
- `noiseBurst()`: Procedurally generated noise for explosion sounds
- `sfx` object: Pre-tuned sound functions (shoot, laser, explode, bomb, powerup, etc.)
- Master gain and dynamics compressor for balanced mixing
- Mute state persisted to localStorage; can resume suspended audio context

### Input Handling
- **Keyboard**: WASD or Arrows for movement, Space/X/Shift for bomb
- **Mouse**: Move to steer, click to fire
- **Touch**: Virtual joystick on left side, FIRE and BOMB buttons on right
- `keys` object tracks held keys; `firing` and bomb state updated each frame
- Joystick input calculated from touch position with velocity clamping

### Canvas & Rendering
- Base canvas: 480×720px (fitted to window with max 1.6× scale)
- Background: animated sky gradient per theme (6 themes rotate by stage)
- Rendering order: sky, clouds, enemies, boss, player, bullets, powerups, particles, HUD, UI
- Adaptive rendering: quality variable (0–2) reduces particle count and shadow blur on low FPS
- Screen shake, flash, and vignette effects controlled by `reducedMotion` preference

### UI & HUD
- Top bar: Mute, reduced-motion toggle, pause buttons
- Center HUD: Score, best score, multiplier, weapon name/level, hearts, bombs
- Boss bar: Shows health during boss phase (hidden otherwise)
- Overlays: Start menu (with difficulty buttons), game-over/stage-clear screens
- Stage banner animation on stage transition
- Combo pop animation on 5-kill milestones (shows multiplier)

### Persistent State
All saved to localStorage with `skyfighter.` prefix:
- `bestScore`: High score
- `muted`: Audio mute state
- `difficulty`: Last selected difficulty
- `reducedMotion`: User motion preference override

## Development Workflow

### Quick Start
1. Open `index.html` in a web browser (no server required; works with `file://` protocol)
2. Edit any code directly in the file
3. Reload the page to test changes

### Common Tasks

**Adjust Difficulty Balancing**
- Edit `DIFFICULTIES` object (line ~486) to change spawn rates, enemy speed, lives, bomb count, boss HP scaling

**Add/Modify Sound Effects**
- Create a function in the `sfx` object using `beep()` and `noiseBurst()` helpers
- Call `ensureAudio()` before generating sound to initialize Web Audio context
- Parameters: frequency (Hz), duration (s), waveform type, volume, optional sweep/filter/pan

**Change Game Visuals**
- Enemy/boss colors: Edit `ENEMY_TYPES` and `BOSSES` color properties
- Player/enemy drawing: Edit `drawPlane()` and `drawBossShip()` functions
- Sky themes: Edit `THEMES` array (gradient colors and names)
- Canvas size: Change `BASE_W` and `BASE_H` constants (note: adjust UI layouts if resizing significantly)

**Modify Weapon Behavior**
- Weapon data: `WEAPONS` object has speed, spread, reload, damage properties
- Fire pattern: `updatePlayerBullets()` function handles trajectory and spawning
- Visual appearance: Bullet rendering in main render loop (search for `ctx.arc` near `bullets` loop)

**Add New Enemy Type**
- Add entry to `ENEMY_TYPES` with w, h, hp, speedMul, color, dark, score, minStage
- Update `spawnEnemy()` selection logic to include new type in `choices` filter
- Draw function is generic (all enemies use `drawPlane()`)

**Tweak Game Balance**
- Enemy spawn timing: `DIFFICULTIES` → `spawnBase` and `spawnMin`
- Kill threshold per stage: `KILLS_PER_STAGE` constant (line ~491)
- Weapon progression: `WEAPONS` array power values
- Collision detection: `rectsOverlap()` function uses AABB; hitbox sizes in entity objects

### Testing
- Open the game, select a difficulty, and play through a stage
- Browser DevTools console (`F12`) shows no errors on clean code
- Test touch controls on mobile or use browser device emulation (Ctrl+Shift+M in Firefox/Chrome)
- Use mute and reduced-motion toggles to verify persistence across reload
- Check FPS in top-right corner (visual feedback of performance quality)

### Accessibility Considerations
- `prefers-reduced-motion` media query respected on load; can be overridden
- Screen reader announcements via `srLive` element with `aria-live="polite"`
- Touch controls auto-enabled on touch device detection
- High-contrast UI with text shadows for readability on sky background

## File Structure
- Single file: `index.html` (1669 lines)
  - Lines 1–359: HTML structure + CSS styling
  - Lines 360–421: DOM elements setup
  - Lines 422+: JavaScript game logic in IIFE

No build step, no dependencies, no separate asset files.
