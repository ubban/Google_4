# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file web application. It features a fully self-contained HTML file with inline CSS, JavaScript, and WebAudio synthesis—no build tools, dependencies, or external assets required.

The game runs in any modern browser and is fully mobile-responsive with touch controls, keyboard support, and mouse steering.

## How to Run

Simply open `index.html` in a web browser. No build step, server, or dependencies needed.

The game includes:
- **Desktop controls**: Mouse to steer, click/Space to fire, X/Shift to bomb, P/Esc to pause
- **Mobile controls**: Touch joystick on the left, FIRE and BOMB buttons on the right
- **Accessibility**: Screen reader announcements, reduced motion mode, high contrast support

## Code Architecture

All game logic is contained in a single IIFE (Immediately Invoked Function Expression) within a `<script>` tag, organized into logical sections:

### Core Systems

1. **Rendering & DOM** (lines 426–461)
   - Canvas setup and HUD elements
   - UI state management (score, lives, weapons, etc.)

2. **Adaptive Performance** (lines 473–483)
   - Frame-time tracking and automatic quality scaling (0=full, 1=medium, 2=low)
   - Adjusts particle counts, shadows, and effects based on FPS

3. **Audio System** (lines 542–676)
   - WebAudio synth with custom oscillators, filters, and envelopes
   - Sound effects: weapon fire, explosions, hits, power-ups, boss telegraph
   - Uses frequency sweeps and noise bursts for dynamic sound generation
   - All SFX are real-time synthesized; no external audio files

4. **Game State** (lines 863–911)
   - Player object with position, weapon, level, cooldown, invulnerability
   - Arrays for bullets, enemies, enemy bullets, particles, power-ups, boss
   - Scoring, lives, stage, chain/combo multiplier system

5. **Input Handling** (lines 767–861)
   - Keyboard: arrow keys, WASD, Space, X, Shift, P, Esc
   - Touch: joystick in move zone (left 55% of screen), buttons in action zone (right 45%)
   - Mouse: hover-based steering with click-to-fire
   - All input sources can be active simultaneously; keyboard/joystick takes priority over mouse

6. **Game Loop** (lines 1640–1653)
   - `loop()` function runs on `requestAnimationFrame`
   - Calls `update(dt)` for physics and state, then `draw()` for rendering
   - Frame-rate capped at 50ms delta time; adaptive quality tracking

7. **Update Logic** (lines 1148–1302)
   - Player movement and weapon firing
   - Enemy and bullet physics
   - Collision detection
   - Power-up application
   - Screen shake and visual effects

8. **Drawing** (lines 1520–1638)
   - Canvas background with theme sky gradient
   - Parallax cloud animation
   - Player plane rendering (colored based on weapon)
   - Enemy planes with health bars (tanks)
   - Boss rendering with animated wings and telegraph state
   - Bullets, particles, power-ups
   - Screen effects (shake, flash, vignette)

### Data Structures

**Difficulties** (lines 485–489)
- `easy`, `normal`, `hard` with spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier

**Enemy Types** (lines 502–506)
- `grunt`, `interceptor`, `tank` with width, height, HP, speed, score, minimum stage, weave pattern

**Bosses** (lines 508–515)
- 6 designs: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Each with hull colors, wing color, core color, and HP multiplier

**Boss Attack Patterns** (lines 1372–1409)
- 6 pattern sets (one per boss) with 3 attacks each
- Pattern types: radial bursts, aimed spreads, columns, spirals, crosses, homing, double rings
- Patterns cycle and telegraph before firing

**Themes** (lines 493–500)
- 6 visual themes with sky gradient, cloud color, sun color, and name
- Changes per stage (wraps at stage 6)

### Key Constants

- `BASE_W = 480, BASE_H = 720` – Canvas internal resolution (scaled to fit window)
- `KILLS_PER_STAGE = 18` – Enemies killed before boss appears
- Storage keys for persistence: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`

### Important Functions

- `update(dt)` – Physics, spawning, collision, state changes
- `draw()` – All rendering
- `fireWeapon()` – Player shooting logic (Vulcan spread, Laser pierce)
- `useBomb()` – Clear enemies and bullets, damage boss
- `addScore(base)` – Add points with combo multiplier
- `spawnBoss()` – Initialize boss at stage start
- `updateBoss(dt)` – Boss movement, telegraph, attack patterns
- `killBoss()` – Boss defeat, stage increment
- `applyPowerUp(type)` – life, bomb, or weapon upgrade
- `loseLife()` / `endGame()` – Player hit, game over
- `toggle Pause()` – Pause/resume
- `drawPlane(x, y, w, h, color, darkColor, flame)` – Player and enemy rendering
- `drawBossShip(b)` – Boss rendering with animated parts

## Development Notes

### When Adding Features

1. **New enemy type**: Add to `ENEMY_TYPES` (line 502), update `spawnEnemy()` logic (line 1022), add explosion SFX to `EXPLODE_SFX` (line 678)
2. **New boss**: Add to `BOSSES` (line 508), create pattern array in `BOSS_PATTERNS` (line 1372)
3. **New weapon**: Extend `fireWeapon()` (line 1124) and `weaponLabel()` (line 871)
4. **New sound effect**: Add function to `sfx` object (line 624), use `beep()` and `noiseBurst()` helpers
5. **New visual theme**: Add to `THEMES` (line 493)
6. **UI change**: Update HUD structure in HTML (lines 367–419), then add/modify JS element references

### Performance Considerations

- Quality scaling is automatic; the game targets 40+ FPS on lower-end devices
- Particle counts scale with quality level
- Shadow blur and glow effects only at quality 0 (full)
- Use `quality` variable to gate expensive rendering operations
- The `trackPerf()` function runs every 45 frames; avoid modifying the tracking logic

### Accessibility

- Screen reader announcements via `announce()` using `#srLive` element
- Reduced motion support: animations and effects disable/reduce when `reducedMotion` is true
- Always check `if (reducedMotion)` before visual effects like screen shake, flash, animations
- User preference is detected via `prefers-reduced-motion` media query and stored

### Canvas Sizing

- Internal canvas is always 480×720 (16:9 with buffer)
- Scaled to fit the window while maintaining aspect ratio (max 1.6× size)
- `resize()` handles window and orientation changes

### Collision

All collision uses axis-aligned bounding box (AABB): `rectsOverlap(a, b)` checks if two rectangles overlap by comparing centers and half-widths.

## Testing

There are no automated tests. Test manually in a browser:

1. **Gameplay**: Play through 2–3 stages on each difficulty; verify enemy patterns, boss attacks, weapon switching
2. **Touch controls**: Test on mobile device; joystick and buttons should be responsive
3. **Audio**: Check mute toggle; verify different SFX play (shoot, explode, hit, bomb, boss telegraph, boss down, stage clear)
4. **Persistence**: Change difficulty/mute, reload page—settings should persist
5. **Reduced motion**: Enable in browser settings or toggle in-game; screen shake, flash, and animations should stop
6. **Responsiveness**: Resize browser window; canvas should scale smoothly; HUD should remain readable

## Known Limitations

- Single file constrains refactoring; any major restructuring would require splitting into separate files
- No sprite sheet; all rendering is procedural (geometry and gradients)
- No external assets; loading new enemies or bosses requires code changes, not asset files
- Audio synthesis may not work on very old browsers (requires modern WebAudio API)
- Performance on extremely low-end devices may require further quality scaling
