# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a self-contained HTML5 application. The entire game logic, rendering, and UI is in a single `index.html` file with embedded CSS and JavaScript. No external dependencies or assets are required.

## Running the Game

- **Local**: Open `index.html` in any modern web browser
- **Quick server**: `python3 -m http.server 8000` then visit `http://localhost:8000`
- The game is fully responsive and works on mobile/touch devices

## Code Architecture

The JavaScript is wrapped in an IIFE (Immediately Invoked Function Expression) to avoid global namespace pollution. The code is organized into logical sections:

### Core Game Loop
- `loop(now)` — requestAnimationFrame callback; calls `update()` then `draw()` each frame
- `update(dt)` — Game state updates (movement, collisions, spawning)
- `draw()` — Canvas rendering

### Game State
- `player` — Player ship object (x, y, w, h, speed, weapon, level, invuln)
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps` — Entity arrays
- `boss` — Current boss object (null when not fighting)
- `score`, `lives`, `kills`, `stage`, `chain`, `multiplier` — Game stats
- `running`, `paused`, `gameOver` — Game flags

### Input Systems
- **Keyboard/Gamepad**: Arrow keys or WASD for movement; Space for fire; X/Shift for bomb; P/Esc for pause
- **Mouse**: Hover to steer, click to fire (steering uses smooth interpolation)
- **Touch**: Left zone for joystick movement, right zone for action buttons; on-screen joystick base appears on touch
- Priority: keyboard/joystick > mouse steering (prevents fighting inputs)

### Entity Systems

**Player**: Single plane with weapon state (vulcan/laser), level (1–3), and invulnerability frames.

**Enemies**: Three types defined in `ENEMY_TYPES`:
- `grunt` — Fast, low HP, score 10
- `interceptor` — Very fast, weaving pattern, score 15
- `tank` — Slow, 4 HP, heavy, score 35 (appears stage 2+)

Each enemy has a shoot pattern (spread or aimed at player) and random spawn variations.

**Bosses**: Six designs in `BOSSES` array (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN). Each has unique attack patterns stored in `BOSS_PATTERNS`. Bosses telegraph attacks before firing via a 22-frame wind-up.

**Bullets**: Player bullets pierce-through on laser (reduced by level) and have straight or spread trajectory. Enemy bullets are radial patterns or aimed spreads.

**Power-ups**: Drop from enemy kills; restore life, add bombs, or upgrade weapons.

### Collision Detection
- `rectsOverlap(a, b)` — AABB collision check using entity half-widths
- Called for player↔enemy, player↔bullets, player↔powerups, bullets↔enemies/boss

### Difficulty System
Each difficulty tier (`DIFFICULTIES.easy/normal/hard`) adjusts spawn rate, enemy speed, enemy shoot chance, lives, bombs, and boss HP multiplier. Player can change difficulty before starting.

### Audio System
- Uses **WebAudio API** to generate all sounds via oscillators and noise buffers (no external audio files)
- `ensureAudio()` — Lazily initializes AudioContext (deferred until first input)
- `beep(freq, dur, type, vol, opts)` — Synth sound with sweep and filter
- `noiseBurst(dur, vol, opts)` — Noise with envelope
- `sfx` object — Named sound effect functions (shoot, laser, explode, bomb, etc.)
- Sounds are spatially panned based on X position using `panFor(x)`

### Rendering
- **Canvas setup**: Fixed 480×720 logical size, scaled to fit window (max 1.6×)
- **Themes**: Six visual themes with different sky gradients and colors (cycle per stage)
- **Shaders**: Screen shake applied via canvas translate
- **Particles**: Spawn on hits and explosions; fade out over life duration
- **Performance scaling**: `quality` variable (0=full, 1=medium, 2=low) adjusts particle count and glow effects based on frame time

### UI and Accessibility
- **HUD**: Score, best, multiplier, weapon level, hearts, bombs (top-center)
- **Boss bar**: Shows enemy HP; hidden outside boss fights
- **Screen reader live region** (`srLive`): Announces game events
- **Reduced motion**: Respects `prefers-reduced-motion` media query; disables animations if user toggles the 🚫 button
- **Pause overlay**: Shows game state and allows difficulty change before restart
- **Local storage**: Persists best score, mute state, difficulty, and reduced-motion preference

## Key Design Decisions

**Single File**: Simplifies distribution (one file to share) and eliminates build complexity.

**No External Assets**: All graphics are drawn via Canvas2D, all sounds generated via Web Audio. Game works offline and loads instantly.

**Adaptive Quality**: Frame time tracking adjusts particle count and glow to maintain 60 FPS on weak hardware.

**Strict Collision**: Uses simple AABB (axis-aligned bounding box) overlap; no physics engine. Collision is frame-perfect but fast.

**Boss Telegraph**: Visual 22-frame wind-up before attacks gives players reaction time and telegraphs danger.

**Weapon Progression**: Player starts weak (single-fire vulcan); upgrades unlock spread patterns and weapon switching. Laser is faster-firing but lower damage; vulcan is slower but more versatile.

**Chain Multiplier**: Killing enemies rapidly builds a multiplier (up to 10x) that decays after 2.2 seconds without a kill. Encourages aggressive play and rewards skill.

## Common Tasks

**Adjust difficulty balance**: Edit `DIFFICULTIES` object or `BOSS_PATTERNS` array.

**Add enemy type**: Add entry to `ENEMY_TYPES`, update spawn logic in `spawnEnemy()`.

**Change visual theme**: Modify `THEMES` array (sky gradient, cloud color, sun color).

**Add sound effect**: Write function in `sfx` object using `beep()` or `noiseBurst()`.

**Adjust player mechanics**: Modify `player` object properties, weapon fire patterns in `fireWeapon()`, or collision/invuln logic.

**Tweak performance**: Adjust `quality` scaling thresholds in `trackPerf()` or particle spawn counts in `spawnExplosion()`.

## Browser Compatibility

Requires: ES6+ JavaScript, Canvas2D, Web Audio API, LocalStorage, Touch Events (optional).
Tested on: Modern Chrome, Firefox, Safari, Edge.
