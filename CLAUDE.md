# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game built entirely as a single HTML5 file. Players control a fighter plane, destroy waves of enemies, collect power-ups, and face stage bosses across multiple difficulty levels.

### Technology Stack
- **HTML5 Canvas**: All rendering via 2D Canvas API
- **Vanilla JavaScript**: No frameworks or build tools
- **Web Audio API**: Procedural sound generation (no external audio assets)
- **LocalStorage**: Save best score and player preferences

## Running the Game

Simply open `index.html` in any modern web browser. No build step, no dependencies, no setup required.

**Testing**: Open in multiple browsers and devices to verify rendering, input handling (keyboard, mouse, touch), and audio. Check mobile responsiveness and touch control zones.

## Code Architecture

The entire game logic lives in a single `<script>` tag (~1,200 lines) organized into logical sections:

### Game State (`update()` function, line ~1148)
- **Player**: Position, weapon, health, cooldown, invulnerability frames
- **Enemies**: Array of active enemies with position, type, health, AI patterns
- **Bullets**: Player and enemy projectiles
- **Boss**: Single boss entity with patterns and health bar
- **Particles**: Explosion effects and visual feedback
- **Score/Lives**: Track progress; combo system with multiplier

### Rendering (`draw()` function, line ~1520)
- **Sky**: Dynamic gradients based on stage theme
- **Canvas scaling**: Responsive design; base 480×720 upscaled to fit window
- **Entities**: Player plane, enemies, boss, bullets, power-ups, particles
- **UI**: HUD (score, lives, weapons, bomb count), boss health bar, overlays
- **Effects**: Screen shake, flash on hit, reduced-motion preferences

### Input Handling (lines ~768–861)
- **Keyboard**: Arrow keys/WASD for movement, Space/X/Shift for actions
- **Mouse**: Aim by moving cursor; click to fire
- **Touch**: Left zone for joystick movement (drag to control), right zone for fire/bomb buttons
- **Priority**: Keyboard/joystick override mouse steering to prevent fighting inputs

### Enemy & Boss AI
- **Enemy spawning**: Waves controlled by difficulty-based spawn timers
- **Enemy types** (line ~502): Grunt, Interceptor, Tank—each with speed, health, shooting pattern
- **Boss patterns** (lines ~1372–1409): 6 boss designs with 3 unique attack patterns each (radial bursts, homing shots, walls, etc.)
- **Boss attack telegraph**: Visual telegraph before attack (white glow on core)

### Audio (lines ~542–676)
- **No external files**: All sound effects synthesized via Web Audio
- **Synth functions**: `beep()` (tonal), `noiseBurst()` (percussion), layered for depth
- **SFX map**: Each game event (shoot, explode, bomb, boss hit, etc.) maps to a synth call
- **Mute toggle**: Respects user muting; audio context suspended until first interaction

### Game Difficulty (line ~485)
```
easy   | spawnBase: 75   | enemySpeed: 1.0x  | lives: 4 | bossHpMul: 0.8
normal | spawnBase: 58   | enemySpeed: 1.4x  | lives: 3 | bossHpMul: 1.0
hard   | spawnBase: 40   | enemySpeed: 1.9x  | lives: 3 | bossHpMul: 1.3
```
Settings save to localStorage; player selects before game start.

### Weapon System
- **Vulcan**: Straight shots with upgrade spread (1→2→3 projectiles)
- **Laser**: Single lane shots with pierce (up to level 3 for multi-lane spread)
- **Powerup swap**: Reaching level 3 then collecting weapon powerup switches weapon type to level 1

### Visual Themes (line ~493)
Six sky gradients cycle per stage: Daylight, Sunset, Night, Storm, Arctic, Volcanic. Each has distinct colors for sky, clouds, sun, and ambient lighting.

### Adaptive Performance (lines ~473–483)
Tracks frame time and scales rendering quality:
- `quality = 0`: Full effects (glow shadows, all particles)
- `quality = 1`: Medium (60% particles, reduced shadow blur)
- `quality = 2`: Low (35% particles, no shadows)

## Key Constants & Tuning

| Constant | Purpose |
|----------|---------|
| `BASE_W`, `BASE_H` (753) | Canvas resolution (480×720) |
| `KILLS_PER_STAGE` (491) | Enemies to kill before boss appears |
| `ENEMY_TYPES` (502–506) | Defines grunt/interceptor/tank stats |
| `BOSSES` (508–515) | 6 boss designs with color schemes |
| `BOSS_PATTERNS` (1372–1409) | Attack patterns per boss |
| `THEMES` (493–500) | Sky gradients per stage |

## Extending the Game

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line ~502) with width, health, speed multiplier, score
2. Update spawn logic in `spawnEnemy()` (line ~1022) to select new type
3. Add explosion sound in `EXPLODE_SFX` map if needed (line ~678)
4. Render it in `draw()` loop (line ~1603) if using non-standard shape

### Add a New Boss
1. Add boss design to `BOSSES` array (line ~508) with name and colors
2. Create attack pattern function (or reuse existing from lines ~1324–1370)
3. Add pattern set to `BOSS_PATTERNS` array (line ~1372)
4. Boss will spawn on next stage automatically; index cycles through BOSSES

### Add a Power-Up Type
1. Add to `maybeSpawnPowerUp()` probability check (line ~1058)
2. Handle in `applyPowerUp()` (line ~1304) to apply effect
3. Render in `draw()` (line ~1617) with new color/symbol

### Adjust Difficulty
Modify `DIFFICULTIES` object (line ~485): spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul.

## Accessibility & Preferences

- **Reduced motion**: Toggle via button; disables shake, flash, and animations on user preference
- **Mute**: Saves to localStorage; respects audio context suspension on first interaction
- **Screen reader**: Live region (`#srLive`) announces major events
- **Touch UI**: Responsive button zones; joystick shows on touch device detection

## Common Development Tasks

### Debug a Boss Pattern
- Boss attack pattern functions fire at enemy bullet updates (line ~1436)
- Set breakpoint in pattern function to step through projectile creation
- Modify spawn count or speed for testing new attack shapes

### Tweak Enemy Difficulty
- Adjust `DIFFICULTIES[difficulty].enemySpeed` or `spawnBase` to feel faster/slower
- Test across all three difficulty settings after tuning
- Verify boss health scaling with `bossHpMul`

### Profile Performance
- Open DevTools Performance tab while game runs
- Watch `loop()` frame callback (~1640) for timing
- Check `quality` variable (lines ~473–483) to see if scaling triggered
- Reduce particle count in `spawnExplosion()` if quality=2 feels sparse

### Add Screen Effects
- Flash overlay: Use `flashHit()` (line ~698) and `flashBomb()` (line ~704)
- Screen shake: Call `screenShake(magnitude, duration)` (line ~727)
- Combo popup: Call `showComboPop(text)` (line ~712)
- Both respect reduced-motion preference automatically

## Important Notes

- **No dependencies**: Do not add npm packages or external libraries. Keep the single-file model.
- **Canvas-based**: All rendering is direct Canvas API. No SVG or DOM overlays for game elements.
- **Frame-locked**: Game updates at requestAnimationFrame (60 FPS target); dt clamped to 50ms max to prevent spiral on lag.
- **LocalStorage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Do not change key names without migration.
- **Touch detection**: `isTouchDevice` (line ~778) enables touch controls automatically; keyboard/mouse work on touch devices too.
