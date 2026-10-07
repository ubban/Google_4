# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Sky Fighter is a single-file HTML5 game — a Raiden-style vertical shoot-em-up with boss encounters, weapon progression, and procedurally varied difficulty. All code, styles, and assets are embedded in `index.html`. No build process, external dependencies, or tooling required.

## How to Develop

**Running the game:** Open `index.html` directly in any modern browser (or use Claude Code's `/run` skill to launch it). The game runs entirely client-side.

**Making changes:** Edit `index.html` directly. The game reloads on refresh; no compilation step needed.

**No testing framework:** This project has no test suite. Verify changes by playing the game and checking specific mechanics (e.g., weapon switching, boss patterns, difficulty balance).

## Code Architecture

The game is organized as one large self-invoking function (line 423–1666) to preserve scope isolation. Key sections:

### 1. **State & Configuration** (lines 463–527)
- `DIFFICULTIES`: Three difficulty presets (easy/normal/hard) that adjust spawning, enemy speed, shot patterns, and HP multipliers
- `THEMES`: Six sky/cloud/sun color schemes that rotate per stage
- `ENEMY_TYPES`: Grunt, Interceptor, Tank definitions with HP, speed, score, and visual properties
- `BOSSES`: Six named boss designs with unique color schemes, HP modifiers, and attack patterns
- `BOSS_PATTERNS`: Array of attack functions for each boss type; each boss rotates through 3 pattern slots (line 1372–1409)

Game state variables: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, `chain`/`multiplier`, `difficulty`, `quality` (adaptive quality setting).

### 2. **Audio** (lines 542–676)
Synthesized sound effects using Web Audio API (no external audio files).

- `ensureAudio()`: Initializes audio context lazily on first user interaction
- `beep(freq, dur, type, vol, opts)`: Square/sine/sawtooth oscillator with optional sweeps, filters, and panning
- `noiseBurst(dur, vol, opts)`: White noise with envelope and filtering
- `sfx` object: Pre-composed sound cues (shoot, laser, explode variants, bomb, boss telegraph, etc.)

Audio muting is persisted to localStorage.

### 3. **Rendering** (lines 680–1638)
All drawing uses 2D canvas context. Key functions:

- `drawPlane(x, y, w, h, color, darkColor, flame)`: Procedural fighter jet; reused for player and enemies
- `drawBossShip(b)`: Boss-specific hull, wings, core with telegraphing glow effect
- `draw()`: Main render loop; draws sky gradient + clouds + entities + particles; applies screen shake and flash effects
- `flashHit()` / `flashBomb()`: CSS animation triggers for visual feedback

The canvas is 480×720 (BASE_W, BASE_H); dynamically scaled via CSS to fit the viewport.

### 4. **Input Handling** (lines 767–861)
Three input methods, no conflicts:

- **Keyboard**: Arrow keys / WASD for movement, Space for fire, X/Shift for bomb, P/Esc for pause
- **Touch**: Left 55% of screen is a virtual joystick (magnitude/direction mapped to player velocity); right 45% has FIRE and BOMB buttons
- **Mouse**: Pointer position steers the player (low-priority; keyboard/joystick override)

Audio context resumes on first user interaction.

### 5. **Game Loop & Physics** (lines 1148–1302)
- `update(dt)`: Player movement (framerate-independent), weapon firing, collision detection (rectangles), enemy spawning, stage progression
- Spawn timer ramps down as frame count increases (line 1201), causing enemy density to rise per stage
- Stage progression: 18 kills → boss warning → boss spawns after 1.2s delay (line 1206)
- Enemy patterns: 85% of enemies spawn with "aimed" fire; 15% use spread patterns

### 6. **Combat & Collision** (lines 1087–1146)
- **Weapon balance**: Vulcan (default, rapid fire, spread at level 3) vs. Laser (lower fire rate, pierce enemies, areas at level 3+)
- **Multiplier system**: Chain counter increments on each kill; resets if 2.2s passes without a kill (line 1089). Multiplier caps at 1 + (chain / 5) = max 2.8x (9 chain)
- **Enemy bullets**: Radial, aimed-spread, and column patterns; boss patterns are complex (8-way, homing, expanding rings, etc.)
- **Power-ups**: 6% → life, 8% → bomb, 20% → weapon upgrade (dropped on enemy death)

### 7. **Boss System** (lines 1043–1454)
- Boss enters from top (y: -80) and moves horizontally, bouncing off edges
- Boss telegraphs before each attack: white core glow + audio cue (line 1421, 1433)
- Six bosses cycle by stage (mod 6); each has 3 attack patterns (defined in BOSS_PATTERNS)
- Boss HP scales with difficulty and boss design multiplier (base: 70 + stage * 35, line 1046)
- Boss destruction awards 500 points, triggers stage banner, and increments stage counter

### 8. **Accessibility & Preferences** (lines 520–540)
- Reduced motion: Honors `prefers-reduced-motion` media query; can be toggled in-game; disables animations if enabled (body class `.reduced-motion`, line 349–358)
- Mute button: Audio is entirely optional
- Screen reader support: `#srLive` announces game state changes (e.g., stage progression, lives remaining, new best score)

### 9. **Quality Adaptation** (lines 473–483)
Dynamically adjusts particle counts and shadow rendering based on frame time. Three levels:
- Quality 0: Full effects (shadows, glows, all particles)
- Quality 1: 60% particles, reduced shadows
- Quality 2: 35% particles, no shadows

Triggers if average frame time (45-frame window) exceeds 1/40 or drops below 1/56 (lines 480–482).

### 10. **Persistence** (lines 463–467)
Game state stored in localStorage:
- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

## Common Edits

**Adjusting difficulty parameters** (line 485–489):
- `spawnBase` / `spawnMin`: Frame interval between enemy spawns (lower = denser)
- `enemySpeed`: Scales all enemy movement by this factor
- `enemyShootChance`: Probability enemy will shoot when it enters screen
- `bossHpMul`: Multiplier on final boss HP

**Adding a new weapon or power-up type:**
1. Add type to ENEMY_TYPES or define new constant
2. Update `fireWeapon()` (line 1124) to handle new weapon branches
3. Update `applyPowerUp()` (line 1304) to handle new power-up logic
4. Add sound effect to `sfx` object if needed

**Tuning boss attack patterns** (line 1372–1409):
- Boss patterns are stage-indexed; each pattern is a function that fires bullets into `enemyBullets[]`
- Helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, etc.
- Telegraph delay is 22 frames (line 1421); attack timer is 85 frames (line 1429)

**Modifying visuals:**
- Theme colors defined in `THEMES` array (line 493–500)
- Enemy/boss colors in ENEMY_TYPES and BOSSES; drawing logic in `draw()` and `drawBossShip()`
- Particle colors set per explosion context (lines 1241, 1270, 1279, 1285, 1442)

**Changing canvas size:** Update `BASE_W` and `BASE_H` (line 753); aspect ratio and scaling handled automatically by `resize()` (line 754–762).

## Notes for Future Work

- **No external dependencies**: Game uses only browser APIs (Canvas 2D, Web Audio, localStorage, media queries)
- **Single file design**: Simplicity and portability take precedence over code splitting or modularity
- **Performance is important**: Quality adaptation and particle culling prevent slowdown on lower-end devices
- **Accessibility is built-in**: Screen reader announcements, reduced motion support, and high contrast cosmetics already wired
