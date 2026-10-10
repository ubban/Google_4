# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single HTML file (index.html) with embedded CSS and JavaScript. No build system, dependencies, or external assets are needed — the entire game runs standalone in a browser.

## Architecture

The game code is organized into logical sections within a single IIFE (Immediately Invoked Function Expression) starting at line 422:

### 1. Canvas Setup & UI Elements (lines 426–461)
Canvas initialization and DOM element references. The game uses a fixed 480×720 logical resolution that scales to fit the viewport.

### 2. Configuration & Constants (lines 463–515)
- **STORAGE_** constants: localStorage keys for best score, mute state, difficulty, reduced motion
- **DIFFICULTIES**: Three difficulty modes (easy/normal/hard) with enemy spawn rates, speed, HP multipliers
- **THEMES**: Six visual themes with sky gradients, cloud colors, sun colors that cycle per stage
- **ENEMY_TYPES**: Three enemy variants (grunt, interceptor, tank) with stats
- **BOSSES**: Six boss designs with unique names, colors, and HP multipliers
- **BOSS_PATTERNS**: Attack patterns for each boss (radial bursts, aimed spreads, etc.)

### 3. Persistent State (lines 517–527)
localStorage integration for best score, mute setting, difficulty, reduced motion preference.

### 4. UI Updates & Event Handlers (lines 529–750)
Mute button, motion button, difficulty selection, pause. Many toggle a class and update visual/audio state.

### 5. Audio Synthesis (lines 542–676)
WebAudio API with procedural synth. `ensureAudio()` creates the audio context on first user interaction. `beep()` generates tones with frequency sweep and filters. `noiseBurst()` generates noise. `sfx` object maps game events to specific sound effects (shoot, laser, explosions, powerups, boss hits, etc.).

### 6. Canvas Sizing & Input (lines 752–861)
- `resize()`: Scales canvas to fit viewport while maintaining 480×720 aspect ratio
- Keyboard input: arrow keys, WASD, Space, X, Shift, P, Escape
- Mouse input: movement = steering toward pointer; click = fire
- Touch input: left zone = joystick (move), right zone = FIRE/BOMB buttons

### 7. Game State Variables (lines 864–869)
`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, plus scalars for score, lives, multiplier, chain counter, stage, etc.

### 8. Core Game Loop (lines 1640–1653)
`loop()` is called via requestAnimationFrame. Each frame:
1. Compute dt (capped at 50ms)
2. Track performance (adaptive quality)
3. If running and not paused: `update(dt)` then `draw()`
4. Queue next frame

### 9. Update Logic (lines 1148–1302)
`update(dt)` handles:
- Player movement (keyboard/joystick/mouse steering)
- Weapon firing (vulcan or laser, with cooldown)
- Bullet movement & cleanup
- Enemy spawning, movement, AI shooting
- Boss spawning & mechanics
- Collision detection (bullets vs enemies, bullets vs boss, enemies vs player, bombs)
- Chain/combo counter
- Stage progression (18 kills → boss warning → boss spawns)

### 10. Rendering (lines 1520–1638)
`draw()` renders the scene in order:
- Sky gradient + sun glow + animated clouds (parallax effect)
- Player plane
- Bullets (with glow if quality is high)
- Enemy bullets (circles)
- Enemies (plane-styled)
- Boss (if present)
- Power-ups (rotating squares)
- Particles (fading circles)
- Screen shake applied via canvas translate

### 11. Drawing Primitives (lines 913–1020)
- `drawPlane()`: Renders player/enemy planes with wings, nose, canopy, flame (if firing)
- `drawBossShip()`: Boss with rotating thrusters, glowing core, telegraph effect

### 12. Boss Attack Patterns (lines 1324–1409)
Helper functions for boss bullet patterns:
- `fireRadial()`: Bullets in a circle
- `fireAimedSpread()`: Cone aimed at player
- `fireColumns()`: Straight lines downward
- `fireSpiralBurst()`: Rotating spiral
- `fireDoubleRing()`: Two concentric rings
- `fireHomingSeeded()`: Aimed with noise for unpredictability
- `fireCross()`: 8-way cross

Each boss uses a unique sequence of these patterns that cycles as the fight progresses.

## Key Game Mechanics

**Weapon System:**
- Vulcan (default): Fast spray, narrow spread that widens with level
- Laser: Single-shot per button press, slower but pierces multiple enemies
- Level up from power-ups; at level 3, next power-up switches weapon back to level 1

**Chain Multiplier:**
- Each kill increments the chain
- Chain decays if you don't kill for 2.2 seconds
- Multiplier = 1 + floor(chain / 5), capped at 10×

**Bomb (smart bomb):**
- Destroys all onscreen enemies and bullets
- Clears boss projectiles (not boss HP directly, but heavy damage)
- Limited count per difficulty (2–3)

**Difficulty Scaling:**
- Enemy spawn rate, speed, shoot chance, and boss HP all scale
- Reduces on Easy; increases on Hard

**Stage Progression:**
- 18 kills per stage
- On 18th kill, 1.2s later boss spawns
- Boss defeated → next stage, visual/audio feedback
- Each new stage gets a new theme and potentially new enemy types

## Adaptive Performance

Lines 473–483 track frame times and adjust `quality` (0 = full, 2 = low). Effects toggled by quality:
- Particle count (`spawnExplosion()`)
- Glow/shadow effects (draw functions)
- Cloud detail

This helps the game stay 60fps even on weaker devices.

## Accessibility Features

- **Reduced motion mode**: Disables animations (screen shake, pulse, flash) for users with `prefers-reduced-motion`
- **Screen reader support**: `srLive` element announces events (game over, stage clear, lives remaining)
- **Keyboard controls**: Full game playable with keyboard alone
- **Touch controls**: Large buttons and joystick for mobile

## Editing Patterns

When modifying the game:

1. **New enemy type**: Add to `ENEMY_TYPES` constant, define `spawnEnemy()` logic, add explosion SFX name to `EXPLODE_SFX`
2. **New weapon**: Add case in `fireWeapon()`, define weapon name in `applyPowerUp()`, add fire SFX
3. **New boss**: Add entry to `BOSSES`, add pattern array to `BOSS_PATTERNS`, boss will auto-rotate through themes
4. **New visual theme**: Add to `THEMES` (sky colors, cloud color, sun color)
5. **New sound effect**: Call `beep()` or `noiseBurst()` from the `sfx` object or inline in event handlers
6. **Difficulty tuning**: Adjust `DIFFICULTIES` constants
7. **Game balance**: Tweak `KILLS_PER_STAGE`, enemy/bullet speeds, spawn intervals, weapon cooldowns

## Common Tasks

**Test a specific difficulty:**
Open DevTools console and run:
```javascript
localStorage.setItem('skyfighter.difficulty', 'hard');
location.reload();
```

**Disable audio for testing:**
Set `muted = true` in console or click the mute button in-game.

**Clear all settings and scores:**
```javascript
localStorage.clear();
location.reload();
```

**Inspect current game state:**
Pause the game and inspect `player`, `enemies`, `bullets`, `boss`, `score`, `stage` in console.

## Notes for Future Work

- Single HTML file keeps deployment simple but makes the code large. Consider extracting to modules if it exceeds ~2000 lines
- WebAudio synthesis is elegant but CPU-intensive on audio-heavy effects — consider pre-rendering SFX or using OfflineAudioContext if performance becomes an issue
- Boss patterns are procedurally fired; visual feedback (telegraph) gives the player 22 frames (~367ms) warning
- Collision uses AABB (axis-aligned bounding box) rectangles — fast but imprecise for rotated planes; fine for this style
