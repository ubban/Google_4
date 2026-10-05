# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 canvas application. The game features multiple difficulty levels, bosses with distinct attack patterns, weapon/bomb power-ups, and stage progression.

## Running the Game

- **Direct**: Open `index.html` in any web browser
- **Local server**: `python3 -m http.server 8000` (or similar) and navigate to `http://localhost:8000`
- **No build process**: The entire game is self-contained in `index.html`

## Architecture & Key Systems

### Single-File Structure
All code lives in `index.html` with embedded CSS and a single `<script>` block containing the game engine. Key architectural sections:

**Game State** (lines ~463–527):
- Player object: position, health, weapons, level, invulnerability timer
- Enemies array: spawned with type-specific properties (grunt, interceptor, tank)
- Boss object: enters after 18 enemy kills per stage, cycles through 6 designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Bullets, enemy bullets, power-ups, particles: managed as arrays
- Difficulty config, themes, enemy types, boss designs: lookup tables at top of script

**Core Loop** (lines ~1640–1665):
- `loop(now)`: requestAnimationFrame callback at ~60fps, gate-keeps update/draw behind `running && !paused`
- `update(dt)`: processes input, spawns enemies/bosses, updates physics, handles collisions, updates UI
- `draw()`: renders sky/clouds (parallax), player, enemies, bullets, explosions, HUD elements
- Frame-time tracking for adaptive quality (see Adaptive Performance, below)

**Key Update Functions**:
- `update(dt)` (lines ~1148–1302): main tick — moves objects, spawns enemies/bosses, collision detection, score/life management
- `updateBoss(dt)` (lines ~1411–1439): boss movement, telegraph/attack timing, pattern execution
- `draw()` (lines ~1520–1637): canvas rendering — backgrounds, all game objects, HUD overlays

**Spawning & Waves**:
- Enemies spawn on a timer that tightens as the stage progresses (spawnBase → spawnMin as frame count increases)
- After 18 kills, boss spawns after a 1.2s delay (spawnBoss → updateBoss loop)
- Boss patterns fire on a ~85-frame attack cycle with a 22-frame telegraph (red pulse effect)

**Weapon System** (lines ~1124–1146):
- Player switches between Vulcan (spread pattern, 3 levels expand spread width) and Laser (piercing beams, up to 3 lanes at level 3)
- Each weapon has distinct projectile size, speed, and glow color

**Boss Patterns** (lines ~1350–1409):
- 6 boss designs × 3 unique patterns each = 18 attack patterns total
- Patterns fire from boss.x/y at designated speeds; some are radial, some aimed at player, some form columns or spirals
- Each pattern is a function stored in BOSS_PATTERNS, called by boss.patternIndex % patterns.length

### Adaptive Performance (lines ~473–483)
Monitors average frame time over 45 frames and adjusts quality tier (0=full, 1=medium, 2=low):
- Full quality: particle counts at 100%, glow shadows enabled, blur filters
- Medium: particle counts at 60%
- Low: particle counts at 35%, shadow blur disabled
Steps up when avg frame time < 1/56s, down when > 1/40s — keeps 60fps on lower-end devices.

### Audio System (lines ~542–750)
- Web Audio API synth engine (no external audio files)
- Master gain → compressor node → per-sound biquad filters and gain
- Spatial panning driven by event x position
- Envelope system for pitch sweeps and layered effects (multi-tone explosions, telegraph cues, fanfare)
- Synth `sfx` object with methods: `shoot()`, `laser()`, `bomb()`, `hit()`, `explode()`, `tankExplode()`, `interceptorExplode()`, `powerup()`, `bossTelegraph()`, `bossHit()`, `bossDown()`, `stageClear()`, `gameover()`, `uiClick()`, `mute()`

### Themes & Visuals (lines ~493–500, ~1507–1544)
- 6 background themes (Daylight, Sunset, Night, Storm, Arctic, Volcanic) cycling by (stage - 1) % 6
- Each theme has: sky gradient (3 stops), cloud color, sun color for glow effect
- Parallax clouds at two speeds (0.5x and 1.2x) render over the sky
- Player and enemy planes are procedurally drawn with shading, cockpit, wings, and flame effects

### Enemy & Boss Designs
- **Enemy Types** (ENEMY_TYPES object):
  - Grunt: 34×34, red, 1 HP, normal speed, always shoots aimed
  - Interceptor: 26×26, orange, 1 HP, 1.7x speed, weaves left-right (sine curve), fires aimed
  - Tank: 46×46, purple, 4 HP, 0.55x speed, slow, shows health pip, appears stage 2+
- **Bosses** (BOSSES array, 6 designs): each has hull colors, core color, and hpMul (ranging 0.8–1.7) for balance

### Accessibility Features
- **Reduced Motion Toggle** (lines ~529–540): Disables screen shake, vignette pulse, flash animations for photosensitive users; respects OS `prefers-reduced-motion`
- **aria-live Region** (srLive element): Screen readers announce life loss, boss defeats, stage progression, game over results
- **Mute Button**: Silences all audio without affecting game state; persisted to localStorage

### Persistent Storage (lines ~463–466, ~517–527)
- `skyfighter.bestScore`: best score across sessions
- `skyfighter.muted`: audio mute state (1/0)
- `skyfighter.difficulty`: last difficulty selected (easy/normal/hard)
- `skyfighter.reducedMotion`: reduced motion preference (1/0)

### Input Handling
- **Keyboard**: Arrow keys and WASD move, Space/X/Shift fire/bomb, P/Esc pause
- **Mouse**: Hover to steer toward cursor, click to fire; disabled when keyboard/touch active
- **Touch**: Left joystick zone for movement, right buttons for fire and bomb
- **Mobile Responsiveness**: All controls scale with viewport, HUD uses clamp() for font sizing

## Editing & Common Tasks

**Add a New Enemy Type**:
1. Add entry to ENEMY_TYPES (e.g., `scout: { w: 20, h: 20, hp: 1, speedMul: 2.5, color: '#00ff00', dark: '#00aa00', score: 20, minStage: 3 }`)
2. Update spawnEnemy() logic to include it in the choices filter and spawn probability
3. Add draw logic if visual differs from default plane (drawPlane already handles color)

**Add a New Boss Design**:
1. Add entry to BOSSES array with hull/wing/core colors and hpMul
2. Create a new pattern array in BOSS_PATTERNS (3 functions firing enemy bullets with fireRadial/fireAimedSpread/etc.)
3. Patterns are cycled by patternIndex % patterns.length automatically

**Adjust Difficulty**:
- Edit DIFFICULTIES object (spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul)
- Changes take effect when difficulty is selected on start screen

**Tweak Weapon Balance**:
- `fireWeapon()` (lines ~1124–1146): adjust projectile speed, pierce count, spread angles
- Boss HP damage (line ~1259): currently 3 per hit; change to rebalance

**Add Screen Shake or Flash Effects**:
- `screenShake(magnitude, duration)` at key moments (boss attack, bomb, collision)
- `flashHit()` and `flashBomb()` trigger CSS animations on #flash element (respects reduced motion)

## Testing

- **No automated test framework**: verify gameplay manually by opening in browser
- **Cross-browser**: test on desktop Chrome/Firefox/Safari and mobile (iOS Safari, Chrome Android)
- **Performance**: use DevTools Performance tab; watch for frame rate dips, especially on lower-end mobile
- **Accessibility**: use screen reader (NVDA, JAWS) to verify aria-live announcements; test reduced-motion toggle with DevTools emulation

## Notes for Future Work

- All game logic is imperative and procedural; no class abstractions — straightforward to edit but monolithic
- Collision detection is AABB (axis-aligned bounding box); works well at this scale but would need quadtree for many objects
- Audio uses Web Audio API synth; no fallback for browsers that don't support it (rare but possible)
- Boss patterns are deterministic; no procedural generation or randomization of attacks
- No level editor or data-driven design — themes, enemies, bosses are all hardcoded lookup tables
