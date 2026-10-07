# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single HTML5 file with Canvas rendering and Web Audio API for sound synthesis.

- **Entry point:** `index.html` (all code contained within)
- **Architecture:** Vanilla JavaScript with no external dependencies
- **Canvas base resolution:** 480×720 (scales to fill viewport up to 1.6x)
- **Audio:** Procedurally generated sound effects using Web Audio API oscillators

## How to Run

The game runs directly in any modern browser:
- Open `index.html` in a browser to play
- Serves as a complete, self-contained game with no build step required
- Local storage persists best score and settings (difficulty, mute, reduced motion)

## Core Architecture

### Game Loop & Timing (Line ~1400+)
- `requestAnimationFrame` drives the game at native refresh rate
- Delta time (`dt`) tracked for frame-rate independent physics
- Adaptive quality scaling: monitors frame times and reduces particle/shadow effects if FPS drops
- Pause/unpause support with separate logic branch

### Main Game Objects

**Player** (Line 877, 875+)
- Single controllable aircraft with position `(x, y)`, dimensions `(w, h)` 
- Properties: `speed`, `cooldown` (shoot rate), `rapid` (sustained fire counter), `weapon` (vulcan/laser), `level` (weapon upgrade 1-3), `invuln` (post-hit iframe)
- Movement: keyboard (arrow/WASD), mouse hover (glides toward pointer), or touch joystick (left side of screen)
- Firing: left-click, space key, or touch fire button (right side)
- Bomb: X, Shift, or touch bomb button (limited to `maxBombs` per stage)

**Enemies** (Line 1022, 502-506)
- **Types:** `grunt` (weak, common), `interceptor` (fast weaver), `tank` (durable, strong)
- Spawn based on stage and difficulty; higher stages unlock tougher types
- Movement: downward at `speed`, some types weave horizontally
- Shooting: probabilistic with `pattern` (aimed or spread shot) and `shootTimer`
- Collision: `rectsOverlap()` (axis-aligned bounding boxes) for bullets, bombs, and player
- Death: spawns particles, score bonus (scaled by multiplier), occasional power-up

**Boss** (Line 1043, 508-515)
- Spawns after killing `KILLS_PER_STAGE` (18) enemies per stage
- Distinct designs per stage (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Mechanics: entering phase, attack patterns, telegraph warning (white glow), phase transitions
- Health bar UI shown while fighting
- Defeat progresses to next stage

**Projectiles**
- Player bullets: fast, straight, track across screen boundary
- Enemy bullets: slower, from each enemy shooting position
- Smart bomb: circle expanding outward, detects collisions with projectiles and enemies

**Particles** (Line 1073)
- Short-lived visual feedback for explosions; count scales with quality setting
- Properties: position, velocity, lifetime; rendered as colored circles with fade

**Power-ups** (Line 1058)
- Spawn randomly from defeated enemies (~6% life, ~8% bomb, ~20% weapon)
- Float downward at fixed `speed`; player collision triggers pickup effect

### Difficulty Levels (Line 485-489)
- **Easy:** Slower enemy spawn, lower shoot chance, 4 lives, 3 bombs, reduced boss health
- **Normal (default):** Balanced spawn/shoot rates, 3 lives, 2 bombs
- **Hard:** Fast spawn, high shoot chance, harder bosses

### Scoring & Progression
- **Score multiplier:** Increases with consecutive kills (chain), resets if no kills for `chainTimer` seconds
- **Stages:** Progress after reaching `KILLS_PER_STAGE` kills; enemy types and boss difficulty scale
- **Weapons:** Vulcan (spread) and Laser (narrow beam), upgraded via power-ups
- **Lives & bombs:** Limited resources; game over if lives = 0; bomb uses reset per stage

### Visual & Audio Systems

**Rendering (Line ~1350+)**
- Sky gradient themes (6 variants, one per stage cycling)
- Canvas draws: background, boss bar, enemies, player, bullets, particles
- Screen shake effect (controlled by reduced-motion setting)
- Post-hit flash and bomb flash animations
- Vignette pulse when critical health (1 life left)

**Sound Effects (Line 624-676)**
- No audio files; all sounds synthesized in real-time via Web Audio API
- Oscillators: sine, triangle, square, sawtooth waves
- Techniques: frequency sweep, filter modulation, noise bursts, panning (stereo spatial audio)
- Mutable via top-right button; mute state persists in localStorage

**Accessibility (Line 520-540)**
- Reduced motion preference: disables animations (screen shake, flash, pulse) for users with motion sensitivity
- Screen-reader announcements via `aria-live` region
- Touch controls with large hit zones for mobile

### Input Handling (Line 768-861)
- **Keyboard:** Arrow/WASD (move), Space (fire), X/Shift (bomb), P/Esc (pause)
- **Mouse:** Move to steer (plane glides toward pointer), click to fire
- **Touch:** Left zone for joystick (movement), right zone for fire/bomb buttons
- Input priority: keyboard/joystick > mouse (mouse doesn't override when other input active)

### State Management (Line 863-868)
- Game state variables: `running`, `paused`, `gameOver`, `lives`, `score`, `stage`, `chain`, `multiplier`
- Resetting done in `resetGame()` before each play session
- Local storage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`

## Key Functions & Patterns

### Collision Detection
- **`rectsOverlap(a, b)`** (Line 1069): Simple AABB (axis-aligned bounding box) check; used for all collision types

### Animation Timing
- Frame-based animations use `frame` counter (increments each tick) for deterministic timing
- CSS animations handle UI overlays (menu, game-over screen)
- `offsetWidth` force-reflow pattern triggers animation restart

### Audio Initialization
- **`ensureAudio()`** (Line 545): Lazy-loads AudioContext on first user input (required by browser autoplay policy)
- Oscillator/gain connections happen at synthesis time, not during setup

### Stage Progression
- After `KILLS_PER_STAGE` kills, boss spawns and `entering` flag prevents damage until on-screen
- Boss defeat triggers stage increment and enemy respawn

### Quality Scaling
- Tracks frame times in `frameTimes` array (45-frame window)
- Adjusts `quality` (0=full, 1=medium, 2=low) based on average frame duration
- Affects particle counts, shadow blur radius, and animation smoothness

## Important Implementation Details

1. **Canvas coordinate system:** Origin at top-left; y increases downward (typical screen coordinates)
2. **Weapon spread:** Vulcan fires 3 bullets in arc; Laser fires 1 narrow beam
3. **Boss attack cycles:** Each phase has attack pattern (projectile waves, speeds, angles)
4. **Bomb timing:** Explosions freeze enemies briefly; combo multiplier preservation prevents score loss
5. **Enemy weaving:** Only `interceptor` type applies sine-wave pattern to X position
6. **Power-up falloff:** Items disappear if they scroll off-screen; player must move to intercept

## Common Editing Tasks

**Adjust difficulty curves:**
- `DIFFICULTIES` object (Line 485-489): modify spawn rates, speeds, health multipliers

**Add new enemy types:**
- Extend `ENEMY_TYPES` (Line 502-506) with new properties
- Update spawn logic in `spawnEnemy()` (Line 1022) to include new type based on stage
- Add explosion animation in `EXPLODE_SFX` (Line 678)

**New boss design:**
- Add entry to `BOSSES` array (Line 508-515): color scheme, health multiplier, name
- Boss uses stage-based cycling; no need to adjust spawn logic

**Tweak visual themes:**
- `THEMES` array (Line 493-500): sky gradient colors, cloud/sun tints
- Background rendered in `drawFrame()` using theme colors

**Sound effects tuning:**
- Modify frequency, duration, envelope shape in `sfx` object (Line 624-676)
- Most effects use `beep()` (tone) or `noiseBurst()` (white noise) primitives

**UI/Game-over screen:**
- HTML overlay (`id="overlay"`) shown/hidden via JavaScript
- Statistics populated from game state before final render
