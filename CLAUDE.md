# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 web application. It features adaptive difficulty, multiple weapon types, boss fights with unique attack patterns, and persistent high scores via localStorage.

## Running the Game

- Open `index.html` in any modern web browser (Chrome, Firefox, Safari, Edge)
- No build process or dependencies required
- All assets (sounds, graphics) are generated programmatically in-engine

### Testing During Development

1. Open the file in a browser dev server or directly (file:// protocol works)
2. Test across different screen sizes and orientations (responsive design uses `clamp()` and safe-area-inset)
3. Check mobile/touch controls on actual devices or use Chrome DevTools device emulation
4. Use browser DevTools console to monitor performance and check for errors
5. Verify localStorage persistence by opening DevTools > Application > LocalStorage

## Architecture

The entire game is implemented within a single IIFE (Immediately Invoked Function Expression) starting at line 423. Key systems:

### Game Loop (lines 1640–1665)
- Runs via `requestAnimationFrame` for 60 FPS targeting
- Calls `update(dt)` for logic and collision detection
- Calls `draw()` for rendering
- Tracks performance and adjusts quality level for adaptive rendering

### Game State (lines 863–869)
- `player`: position, dimensions, weapon, ammo, invulnerability timer
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: entity arrays
- `score`, `lives`, `stage`, `multiplier`: gameplay progression
- All state resets via `resetGame()` at line 875

### Input Handling (lines 768–861)
- **Keyboard**: Arrow keys / WASD for movement, Space to fire, X/Shift for bombs, P/Esc to pause
- **Mouse**: Move cursor to steer player; click to fire (auto-centers on cursor)
- **Touch**: Left half of screen for movement (visual joystick), right buttons for fire/bomb
- `mouseActive` takes priority when keyboard is idle (line 838)
- Touch event listeners prevent defaults to avoid browser gestures

### Rendering (lines 1520–1638)
- Sky gradient, clouds, and parallax background via `drawClouds()` (lines 1507–1518)
- Plane graphics drawn with custom paths in `drawPlane()` (lines 913–968)
- Boss graphics in `drawBossShip()` (lines 970–1020)
- Screen shake offset applied at line 1523–1527
- Quality adapts shadow/glow effects based on performance (line 1579)
- HUD updates via DOM (hearts, bombs, score, multiplier, boss health bar)

### Update & Collision (lines 1148–1302)
- Player movement with boundary clamping (lines 1162–1172)
- Enemy spawning with difficulty scaling (lines 1197–1208)
- Bullet and enemy projectile movement and cleanup
- Collision detection via `rectsOverlap()` (line 1069)
- Enemy-player, enemy-bullet, boss-bullet, projectile-player checks
- Power-up collection and application (lines 1289–1322)

### Audio (lines 542–676)
- WebAudio synth-based SFX with no external files
- `beep()` creates tones with sweep, filter, and panning
- `noiseBurst()` creates noise-based sounds (explosions, bombs)
- `sfx` object contains all sound effects keyed by event (shoot, laser, explode, hit, bomb, etc.)
- Master gain and compressor applied globally for dynamic range control
- Respects mute setting and audio context suspended state

### Difficulty & Progression (lines 485–515)
- Three difficulties: Easy, Normal (default), Hard
- **DIFFICULTIES**: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier
- **KILLS_PER_STAGE**: 18 kills to trigger boss spawn
- **THEMES**: 6 background color schemes, one per stage (repeats)
- **ENEMY_TYPES**: grunt, interceptor, tank with HP, speed, spawn stage
- **BOSSES**: 6 unique bosses with designs and HP multipliers, each with 2–3 attack patterns

### Boss AI (lines 1324–1434)
- Boss enters from top, bounces horizontally (line 1419)
- **Telegraph** system: boss telegraphs attack for 22 frames (line 1433)
- **BOSS_PATTERNS**: 6 arrays of attack functions (radial, aimed, spread, homing, etc.)
- Boss takes 3 HP per bullet hit, ~24 from bombs
- Boss sprites scale visually; patterns reference `boss.patternIndex` for variety

### Weapon System (lines 1124–1146)
- **Vulcan** (default): rapid-fire bullets, spreads at levels 2–3
- **Laser**: fewer, faster projectiles with pierce (ignores enemies), alternates with Vulcan on weapon pickup
- Weapon level cap: 3, then toggles weapon type
- Fire rate: Laser 0.1s, Vulcan 0.15s

### Scoring & Chain Multiplier (lines 1087–1101)
- `addScore(base)` increments chain counter
- Multiplier = 1 + floor(chain / 5), max 10×
- Chain resets if no kill for 2.2 seconds (chainTimer)
- Boss kill awards 500 points
- Breaks on damage (loseLife)

### Accessibility (lines 468–471, 520–540, 1000–1358)
- Screen reader announcements via `srLive` element with aria-live
- Reduced motion preference respected: disables animations on `prefers-reduced-motion: reduce`
- Toggle via motion button; stored in localStorage
- Pause state announced clearly

### LocalStorage Keys (lines 463–466)
- `skyfighter.bestScore`: high score
- `skyfighter.muted`: audio mute state (bool as '0' or '1')
- `skyfighter.difficulty`: last selected difficulty
- `skyfighter.reducedMotion`: user motion preference

## Key Implementation Details

### Collision Detection
Uses axis-aligned bounding-box (AABB) overlap via `rectsOverlap()`: compares distances between centers to half-width/height sums.

### Performance Tuning
- `quality` variable (0 full, 1 medium, 2 low) adjusts particle count and glow effects
- Tracked via `trackPerf()` every 45 frames; maintains 40–56 FPS window
- Radial boss bullets bypass normal speed; accelerate via `dx`/`dy` (line 1437)

### Visual Effects
- Particle system for explosions: spawns 3–22 particles depending on entity type and quality
- Flash overlay for hit/bomb feedback: triggers CSS animations
- Combo pop and stage banner: fade-in animations with scale
- Vignette (red pulse) on critical HP (< 2 hearts)
- Heart/bomb icons in HUD update dynamically

### Mobile Responsiveness
- Canvas scales via CSS `clamp()` to fit viewport, maintaining 480×720 logical size
- Touch controls show on touch-capable devices (line 779)
- Safe area inset used for HUD offsets (notch support)
- Buttons sized for touch: 78×78 px for fire/bomb buttons

## File Structure

- `index.html`: Single file containing all HTML, CSS (embedded `<style>`), and JavaScript (in `<script>` tag)
- `README.md`: Basic project info and timestamp
- `.git/`: Version control

## Modifying the Game

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502) with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Update `spawnEnemy()` (line 1022) to include new type in probability check
3. Add explosion sound in `EXPLODE_SFX` if custom sound needed (line 678)

### Adding a New Boss
1. Add design to `BOSSES` array (line 508) with colors and HP multiplier
2. Add attack pattern array to `BOSS_PATTERNS` (line 1372)
3. Boss index auto-matches stage: `BOSSES[(stage - 1) % BOSSES.length]`

### Adding a Power-Up Type
1. Add type check in `maybeSpawnPowerUp()` (line 1058)
2. Add spawn condition and visual in `draw()` power-up loop (line 1617)
3. Add application logic in `applyPowerUp()` (line 1304)

### Tweaking Balance
- Difficulty multipliers in `DIFFICULTIES` (line 485)
- Enemy spawn rate: `spawnBase`, `spawnMin` (frames between spawns)
- Boss HP: `baseHp` calculation at line 1046 + difficulty/design multipliers
- Weapon fire rates at line 1177
- Chain timer at line 1089
- Particle count multipliers at line 1074

### Adjusting Game Canvas Size
- Change `BASE_W` and `BASE_H` (line 753)
- Logical canvas resolution is 480×720; CSS scaling handles display size
- Collision bounds, spawn positions, and HUD layout scale with base dimensions

## Common Tasks

- **Test specific difficulty**: Click difficulty button on start screen before pressing Start Game
- **Test boss fight**: Use Easy + console cheat to set `kills = 18`, then wait for spawn
- **Clear high score**: Open DevTools, Application > LocalStorage, delete `skyfighter.bestScore`
- **Benchmark quality**: Disable quality throttling by deleting `trackPerf()` calls or setting `quality = 0`
- **Debug collision**: Add `console.log()` in `rectsOverlap()` or visual rect draw around entities
