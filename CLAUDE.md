# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built entirely in vanilla HTML5/Canvas/JavaScript. The entire game logic and rendering is contained in a single `index.html` file with inline CSS and JavaScript. There are no external dependencies, build tools, or test frameworks.

**To play:** Open `index.html` in any modern web browser (desktop or mobile).

## Architecture & Key Concepts

### Game Loop & Delta Time
The game uses `requestAnimationFrame()` for a constant 60 FPS game loop with delta time (`dt`) scaling. All movement and timers are delta-time-aware to ensure frame-rate independence. The loop tracks `lastTime` to compute `dt`, capping it at 50ms to prevent spiral of death on lag.

- **Loop entry point:** Line 1640 (`loop(now)` function)
- **Update phase:** Lines 1148–1302 (`update(dt)` function)
- **Draw phase:** Lines 1520–1638 (`draw()` function)

### Canvas Rendering
- **Base canvas:** 480×720px (internal resolution)
- **Scaling:** Auto-scales up to 1.6× to fill window, maintaining aspect ratio
- **Resize handling:** Lines 754–765 watch for window resize/orientation change
- **Adaptive quality:** Lines 474–483 monitor frame times and downscale particle counts or shadow blur on lag

### Game State
Core state tracking (Lines 864–870):
- `player`: Position, weapon type/level, cooldown, invulnerability timer
- `enemies`, `bullets`, `enemyBullets`: Array pools for dynamic entities
- `score`, `lives`, `stage`, `kills`: Game progress
- `chain`, `multiplier`: Combo system; multiplier increases per 5 consecutive kills
- `boss`: Active boss entity or null
- `paused`, `gameOver`, `running`: Game phase flags

### Difficulty & Content
- **Difficulties** (Lines 485–489): Easy/Normal/Hard each adjust spawn rate, enemy speed, shoot chance, lives, bombs, boss HP
- **Enemy types** (Lines 502–506): Grunt (basic), Interceptor (fast, weaving), Tank (slow, tanky, 4 HP)
- **Bosses** (Lines 508–515): 6 named bosses cycle per stage, each with distinct visuals and attack patterns
- **Themes** (Lines 493–500): 6 parallax sky/cloud combinations cycle per stage
- **Boss patterns** (Lines 1372–1409): Each boss has 3 distinct attack types (radial burst, aimed spread, walls, teleport, etc.)

### Input Handling

**Keyboard:**
- Arrow keys / WASD: Move
- Space: Fire
- X / Shift: Bomb
- P / Escape: Pause

**Mouse:** Move cursor over canvas to steer toward it; click to fire. Keyboard/joystick input takes priority over mouse.

**Touch:** Left half of screen = movement joystick; right half = FIRE/BOMB buttons.

All inputs converge on movement vector (`mvx`, `mvy`) and firing state by line 1178 in `update()`.

### Audio (Web Audio Synthesis)
No external audio files. Sounds are generated on-the-fly using Web Audio API oscillators and noise bursts.

- **Initialization:** `ensureAudio()` (line 545) creates AudioContext, compressor, master gain on first user interaction
- **Sound library:** Lines 624–676 define SFX (shoot, laser, explosions, bomb, powerup, boss telegraph, etc.)
- **Panning:** Bullets/explosions pan left/right based on x position via `panFor()` (line 562)
- **Muting:** Toggled via button; persisted to localStorage

### Collision Detection
Simple AABB (axis-aligned bounding box) with `rectsOverlap()` (line 1069). All entities have `.x`, `.y`, `.w`, `.h`.

### Particle System
Explosion particles spawn via `spawnExplosion()` (line 1073). Count scales based on `quality`. Particles update position, decay lifetime, fade opacity.

### Score & Combo System
- Killing enemies increments `chain`; 5 chains = 1 multiplier tier (max 10×)
- `chainTimer` resets to 2.2s per kill; if it expires, combo breaks
- Score increases by `(baseScore * multiplier)`
- Combo pop message displays at intervals (every 10 kills)

### State Persistence
Four localStorage keys (Lines 463–466):
- `skyfighter.bestScore`: Best score ever
- `skyfighter.muted`: Audio mute flag
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Respects system prefers-reduced-motion; toggleable via button

### Accessibility
- Screen reader announcements via `announce()` (line 468), which updates a live region
- Respects `prefers-reduced-motion` and adds `reduced-motion` class to body (line 530)
- Reduced motion disables screen shake, flash effects, and animations

## Code Structure by Line Ranges

| Section | Lines | Purpose |
|---------|-------|---------|
| Head & CSS | 1–359 | HTML structure, responsive styling, animations (heart beat, combo pop, boss bar, etc.) |
| Overlay & Buttons | 361–420 | Menu card, difficulty selector, start button, pause UI |
| Script Preamble | 422–462 | DOM element references, storage key constants |
| Audio System | 542–676 | WebAudio context, oscillator/noise generation, SFX library |
| UI Rendering | 680–749 | Hearts, bombs, mute button, motion button, difficulty setting |
| Canvas Setup | 752–765 | BASE_W/BASE_H constants, resize() function, orientation listener |
| Input Handlers | 767–861 | Keyboard, touch (joystick), mouse movement, firing button |
| Game State | 863–870 | Global variables for entities, score, stage, etc. |
| Game Logic | 871–1467 | resetGame(), spawnEnemy(), spawnBoss(), update(), collision, boss AI, power-ups |
| Rendering | 1507–1638 | drawClouds(), draw() canvas compositing, entity drawing |
| Game Loop | 1640–1653 | Main requestAnimationFrame loop |
| Entry Point | 1655–1665 | Start button listener, initial requestAnimationFrame call |

## Working with the Code

### Adding a Feature
1. **New enemy type?** Add to `ENEMY_TYPES` (line 502), define spawn logic in `spawnEnemy()` (line 1022)
2. **New boss?** Add to `BOSSES` (line 508), add attack patterns to `BOSS_PATTERNS` (line 1372)
3. **New weapon?** Extend `fireWeapon()` (line 1124) and weapon-swap logic in `applyPowerUp()` (line 1304)
4. **New visual theme?** Add entry to `THEMES` (line 493); sky gradient, cloud color, sun color, name
5. **New sound effect?** Add function to `sfx` object (line 624) using `beep()` or `noiseBurst()`

### Tweaking Difficulty
Modify `DIFFICULTIES` (line 485):
- `spawnBase` / `spawnMin`: Higher = fewer enemies
- `enemySpeed`: Multiplier on enemy movement speed
- `enemyShootChance`: 0–1 probability enemy shoots
- `lives` / `bombs`: Starting count
- `bossHpMul`: Multiplier on all boss HP

### Debugging
- **Frame counter:** Global `frame` variable increments every update; used for animations, pattern selection
- **Pause:** Press P or click pause button to freeze and inspect state
- **Console:** Check console for any errors; audio errors often occur if muted before any sound plays
- **Performance:** Watch browser DevTools Performance tab. High `dt` values or low FPS indicate performance issues

### Common Pitfalls
- **Collision timing:** Collision checks happen *after* position updates, so fast bullets can skip thin objects. Consider subdividing movement if needed.
- **Audio context:** Audio context is lazy-initialized on first user interaction to comply with browser autoplay policies
- **Canvas size:** Internal canvas is always 480×720; CSS scaling is handled by `resize()`. Never hardcode canvas.width/height elsewhere.
- **Particle counts:** Particle spawns already scale down on low-quality mode (`quality` > 0). Never spawn massive bursts.

## Running & Testing

**No build step, no tests.** Simply:
1. Open `index.html` in a browser
2. Play through all three difficulty levels to verify balance
3. Test on mobile (iOS Safari, Android Chrome) for touch controls
4. Check console for errors (Ctrl+Shift+I in most browsers)

For mobile testing, use Chrome DevTools device emulation or test on an actual device.

## File Checklist Before Committing
- No console errors or warnings
- Audio mute/unmute works
- Pause works
- All three difficulties playable
- Boss attacks visible and challenging
- Touch controls work on mobile
- Reduced motion mode disables animations
- localStorage persists best score and settings across page reloads
