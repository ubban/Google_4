# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game implemented as a single-file HTML/CSS/JavaScript web application. It runs in any modern browser without build tools or external dependencies. The game features multiple difficulty levels, 6 stage themes, 6 unique bosses, diverse enemy types, power-ups, and synthesized sound effects using the Web Audio API.

## Running the Game

- **Play locally:** Open `index.html` in a web browser
- **No build step required** — the game is a standalone HTML file with embedded CSS and JavaScript

## Architecture Overview

The entire game logic is contained in a single 1,669-line `index.html` file with three main sections:

### 1. **DOM & UI Styling** (lines 1-358)
- Responsive canvas setup with viewport scaling
- HUD elements: score, multiplier, weapon info, hearts, bombs, boss health bar
- Overlay system for title screen, pause, and game-over screens
- Touch control UI for mobile
- CSS animations for UI elements (combos, stage banners, flash effects, vignette)
- Accessibility features: reduced-motion media query, screen reader live region

### 2. **Game State & Core Systems** (lines 422-895)

**Difficulty System** — Three presets (`DIFFICULTIES` object, line 485) adjust spawn rates, enemy speed, shoot chance, and starting resources. Persists to localStorage.

**Game Objects:** Defined as plain objects with properties for position, size, health, etc. Core arrays:
- `player` — Single player aircraft with weapon level, cooldown, invulnerability timer
- `bullets` — Player projectiles (Vulcan spread shots or Laser piercing beams)
- `enemies` — Array of enemy ships with AI patterns
- `enemyBullets` — Projectile array for enemy fire
- `boss` — Single boss object when active (null between stages)
- `powerUps` — Temporary drop items (life, bomb, weapon upgrade)
- `particles` — Explosion effects

**Weapon System** (lines 1124-1146):
- **Vulcan:** Spread shot pattern, fires continuously, lower per-bullet damage
- **Laser:** Piercing beams from center or sides (based on level), higher fire delay

**Enemy AI** (lines 1022-1040, 1210-1226):
- Three enemy types (grunt, interceptor, tank) with speed/HP variations
- AI patterns: "spread" (radial fire) or "aimed" (tracking shots)
- Weaving interceptors follow sine curves
- Difficulty ramp increases spawn rate every 300 frames

### 3. **Game Loop & Rendering** (lines 1148-1653)

**Update Phase** (`update()`, line 1148):
- Player movement (keyboard/joystick/mouse steering)
- Weapon firing with cooldown
- Collision detection (bullets-vs-enemies, bullets-vs-boss, projectiles-vs-player, items)
- Enemy spawning (waves until 18 kills trigger boss)
- Boss updates with attack patterns
- Chain/combo multiplier system (resets if no kills for 2.2 seconds)
- Score calculation based on multiplier

**Render Phase** (`draw()`, line 1520):
- Animated sky gradient from theme
- Cloud parallax layers
- Player aircraft (with laser aura when equipped)
- Bullets and enemy projectiles
- Enemies with HP bars (tanks only)
- Boss ship with glowing core
- Power-up pickups
- Particle effects
- Screen shake applied via canvas translation
- Adaptive glow effects based on performance quality

**Performance Adaptation** (lines 473-483):
- Tracks frame times and adjusts `quality` level (0=full, 1=medium, 2=low)
- Reduces particle counts and glow effects at lower quality

### 4. **Boss System** (lines 508-1409)

Six boss types (`BOSSES` array, line 508), each with:
- Unique visual design (gradient hull, wing color, glowing core)
- Six distinct firing patterns (line 1372, `BOSS_PATTERNS`)
- Health scaling based on stage and difficulty

**Boss Patterns** (examples):
- Radial bursts, aimed spreads, columns
- Double rings, spirals, 8-way crosses
- Teleport + barrage sequences
- Homing missile volleys

Boss behavior:
- Enters screen over 1.2 seconds, then bounces horizontally
- Attacks with telegraph flash (22 frames warning), then fires pattern
- Cycle through 3 patterns per boss, then repeat
- Defeated after HP reaches 0; triggers stage clear and progression

### 5. **Audio System** (lines 542-676)

Web Audio synthesis (no external audio files):
- **Oscillator-based sounds:** `beep()` function for tonal effects with frequency sweeps
- **Noise bursts:** `noiseBurst()` for percussive hits
- **Sound FX object** (`sfx`, line 624): Pre-composed calls for each action
  - Shoot/laser sounds with frequency sweeps
  - Explosion types (grunt, interceptor, tank)
  - Hit, bomb, powerup, boss-specific cues
  - UI click, stage clear, game over

Features:
- Dynamic panning based on screen position
- Filter envelopes (lowpass/highpass)
- Gain envelopes for attack/release
- Mute toggle + localStorage persistence
- Safe initialization on first user interaction

### 6. **Input System** (lines 767-861)

**Keyboard:** Arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)

**Mouse:** Move to steer player (glides toward pointer), click to fire

**Touch:** 
- Left zone for joystick movement (visual base + stick)
- Right zone for FIRE/BOMB buttons
- Passive event listeners to prevent scrolling

**Priority:** Keyboard/joystick input takes precedence over mouse steering to prevent fighting inputs.

### 7. **Persistence & Settings** (line 463)

LocalStorage keys:
- `skyfighter.bestScore` — High score
- `skyfighter.muted` — Audio state
- `skyfighter.difficulty` — Selected difficulty
- `skyfighter.reducedMotion` — User accessibility preference (respects system prefers-reduced-motion)

### 8. **Themes & Visual Progression** (line 493)

Six environment themes cycle per stage:
- Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge
- Each has sky gradient colors, cloud colors, sun glow color
- Theme changes every stage and persists across difficulty changes

## Common Development Tasks

### Add a New Enemy Type
1. Define in `ENEMY_TYPES` object (around line 502): add `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Add spawning logic to `spawnEnemy()` (line 1022): update random selection
3. Add explosion SFX to `EXPLODE_SFX` object (line 678) if unique sound needed
4. Add drawing logic in `draw()` (around line 1603) if custom rendering required

### Add a Boss Attack Pattern
1. Add new function (e.g., `fireCustomPattern(b)`) that calls `enemyBullets.push()` with projectile objects
2. Projectile object format: `{ x, y, w, h, radial: true, dx, dy }` for boss bullets
3. Add pattern function to appropriate boss's pattern array in `BOSS_PATTERNS` (line 1372)

### Add a Sound Effect
1. Add function call in `sfx` object (line 624) that invokes `beep()` or `noiseBurst()`
2. `beep(freq, dur, type, vol, opts)` — synthesizes tones with optional frequency sweep, filter, panning
3. `noiseBurst(dur, vol, opts)` — white noise with filter envelope
4. Call `ensureAudio()` first to initialize AudioContext on user interaction

### Adjust Difficulty Balance
1. Modify values in `DIFFICULTIES` object (line 485):
   - `spawnBase`/`spawnMin` — Enemy spawn rate (lower = faster)
   - `enemySpeed` — Multiplier for enemy movement speed
   - `enemyShootChance` — Probability enemy fires (0-1)
   - `bossHpMul` — Boss health multiplier
2. Retest with various play sessions to ensure pacing feels right

### Change Visual Theme or Colors
1. **Sky gradients:** Modify `THEMES` array (line 493) colors
2. **UI colors:** Search CSS `#hud`, `.stat`, button classes (lines 34-285)
3. **Player/enemy colors:** Modify arguments to `drawPlane()` calls, or color properties in type definitions
4. **Glow effects:** Adjust `ctx.shadowBlur` and `ctx.shadowColor` in draw functions (quality-dependent, lines 1579-1636)

## Code Style & Patterns

- **State management:** All game state in closure variables at module scope
- **No frameworks:** Vanilla JavaScript with canvas 2D
- **Immediate mode rendering:** `draw()` clears canvas and redraws all objects every frame
- **Object-based collisions:** `rectsOverlap()` function (line 1069) for AABB checks
- **DeltaTime-based physics:** All movement multiplied by `dt` parameter for frame-rate independence
- **LocalStorage for persistence:** Settings survive page reloads

## Performance Considerations

- **Canvas size:** 480×720 (scaled to fit window, max 1.6× upscaling)
- **Particle limits:** Adaptive based on frame time — quality level reduces particle counts
- **Glow effects:** Disabled below certain quality levels to maintain 60 FPS on lower-end devices
- **Frame time tracking:** Rolling 45-frame average triggers quality adjustments

## Accessibility Features

- **Reduced motion:** Respects `prefers-reduced-motion` system setting and provides toggle
- **Screen reader support:** Live region (`#srLive`) announces game state changes
- **High contrast HUD:** Semi-transparent backgrounds with text shadows for legibility
- **Safe area insets:** Respects iOS notches and status bars via `env(safe-area-inset-*)`

## Known Limitations

- Single HTML file (no modularization) — updates require searching/replacing within file
- Canvas-only rendering (no DOM-based objects) — limits accessibility/customization
- Web Audio synthesis only (no actual audio files) — sound quality depends on browser
- Touch controls fixed to left/right split (not customizable layouts)

