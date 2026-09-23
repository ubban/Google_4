# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project: Sky Fighter

Sky Fighter is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. No build step or external dependencies are required—just open `index.html` in a browser to play.

## Architecture Overview

The game is monolithic: all HTML, CSS, and JavaScript are embedded in `index.html` (~1670 lines). The structure follows this pattern:

1. **HTML markup** (lines 361–420): Game canvas, HUD elements (score, lives, bombs, weapon), UI buttons (mute, pause, motion toggle), overlay screen (menu/game-over), and touch control zones.

2. **CSS styling** (lines 10–359): Dark space-themed color scheme with responsive scaling. Uses `clamp()` for adaptive sizing, CSS animations for visual feedback (flashes, pulses, screen shake via canvas transforms), and accessibility support (reduced-motion media query at line 349).

3. **JavaScript game loop** (lines 422–1667): Single IIFE that manages all game state and rendering. Entry point is `requestAnimationFrame(loop)` at line 1665.

## Game Systems & Data

### Core Game Loop
- **Loop function** (1640–1653): Tracks delta-time, updates game state if running/unpaused, renders via canvas 2D context.
- **Update function** (1148–1302): Handles player movement (keyboard, joystick, mouse-following), weapon fire, collision detection (bullets/enemies, bombs, powerups), enemy spawning, and boss behavior.
- **Draw function** (1520–1638): Renders sky gradient, clouds, player plane, bullets, enemies, boss, particles, and powerups. Applies screen shake if active.

### Player Object
```javascript
player = { x, y, w, h, speed: 260, cooldown: 0, weapon: 'vulcan'|'laser', level: 1–3, invuln: 0 }
```
- Constrained to canvas bounds.
- Invulnerability frames prevent rapid re-damage.
- Weapon cooldown controls fire rate (0.15s vulcan, 0.1s laser).
- Movement priority: keyboard/joystick > mouse following.

### Enemy Types
Defined in `ENEMY_TYPES` (502–506):
- **Grunt**: Red, 1 HP, basic enemy. Appears from stage 1.
- **Interceptor**: Orange, 1 HP, fast, weaves side-to-side. Appears from stage 1.
- **Tank**: Purple, 4 HP, slow, heavily armored. Appears from stage 2+. Health bar rendered above.

Each enemy has spawn constraints (minStage), color scheme (dark outline + fill), score value, and firing pattern (spread or aimed).

### Bosses
Six unique bosses rotate through stages (502–515 defines designs; 1372–1409 defines attack patterns):
- **VANGUARD**: Radial bursts, aimed fan, V-fan spreads.
- **BEHEMOTH**: Straight columns, homing spread, dense radial walls.
- **SPECTER**: Teleport dashes, tight spirals, double aimed shots.
- **DREADNOUGHT**: 8-way crosses, homing missiles, radial walls.
- **WRAITH**: Teleport barrages, expanding double rings, cross-spreads.
- **TITAN**: Heavy columns, full rings, relentless homing volleys.

Boss HP scales with stage and difficulty multiplier. Boss enters with animation, then patrols screen edges while attacking. Telegraph effect (white flash of core) precedes each attack pattern.

### Weapons
- **Vulcan** (1126–1134): Spread increases with level (1=center, 2=±center, 3=wide±center). Straight trajectory, fast, instant damage.
- **Laser** (1135–1145): Pierce property allows shots to pass through multiple enemies. Wider projectiles, slightly slower. Weapon swap resets level to 1.

### Difficulty Modes
`DIFFICULTIES` object (485–489) controls:
- Enemy spawn rates (faster on higher difficulty).
- Enemy speed and shoot chance.
- Lives and bomb counts.
- Boss HP multiplier.

### Power-ups
Randomly spawned on enemy death (1058–1067):
- **Life**: Restore 1 HP (up to max).
- **Bomb**: Restore 1 bomb (up to max). Bombs clear all bullets and deal heavy damage.
- **Weapon**: Increase weapon level or swap weapon/reset level.

### Visual Themes
Six themes rotate per stage (493–500), each with unique sky gradients, cloud colors, and sun glow. Theme changes provide visual variety without affecting gameplay.

### Audio
WebAudio API synthesized sounds (542–676) with no external assets:
- Oscillator-based tones (square, sawtooth, sine, triangle).
- Noise bursts via random buffer.
- Filter chains (lowpass, highpass) and gain envelopes for realistic gunshots, explosions, and UI feedback.
- Master compressor prevents clipping.
- Spatial audio: panning based on world position.
- Mute toggle persists to localStorage.

### Screen Effects
- **Screen shake**: Random camera offset during explosions/damage (727–730).
- **Flash**: White overlay for impacts (704–709).
- **Vignette**: Red pulse around screen edges when critical health (75–87).
- **Combo popup**: Large text display when reaching kill milestones (711–717).
- **Reduced motion**: Disabled animations when system prefers it (349–358).

## Customization Points

### Enemy Spawning
- Adjust `DIFFICULTIES` to change spawn frequency and enemy toughness.
- Add new enemy types in `ENEMY_TYPES` with name, dimensions, HP, speed multiplier, color, score, minStage, and optional weave behavior.
- Enemy spawn logic at 1022–1041 chooses type based on stage and random weights.

### Boss Patterns
- Each boss in `BOSSES` maps to a pattern set in `BOSS_PATTERNS`.
- Add new firing patterns by creating functions like `fireRadial()`, `fireAimedSpread()`, etc. (1324–1370).
- Assign patterns to a new boss design by adding to `BOSS_PATTERNS`.

### Game Balance
- `KILLS_PER_STAGE` (491): Enemy kills needed to trigger boss fight.
- `BASE_W`, `BASE_H` (753): Canvas resolution (480×720). Affects all positioning.
- `THEMES`: Change sky colors and cloud/sun appearance.
- Weapon cooldown in `fireWeapon()` calls or `fireDelay` constant (1177).
- Enemy bullet speeds and counts in attack pattern functions.

### UI & Accessibility
- HUD stats styled at line 49–70.
- Buttons and controls at 188–342.
- Screen reader announcements via `announce()` (468–471).
- Reduced-motion preference detected via media query (520), toggleable by player (535–540).

## Performance Optimization

Adaptive quality system (473–483) monitors frame times and reduces particle count if FPS drops:
- `quality = 0`: Full (normal performance).
- `quality = 1`: Medium (60% particle count).
- `quality = 2`: Low (35% particle count).

Glow effects (shadow blur) are disabled at lower quality tiers to save rendering cost.

## Development Workflow

1. **Open in browser**: Direct file open or local HTTP server.
2. **Test game**: Play through multiple stages and difficulties.
3. **Edit & reload**: Browser auto-refreshes on file change (or manual refresh).
4. **Inspect console**: No build errors; watch for runtime issues in DevTools.
5. **Customize**: Modify constants, add enemy types, adjust patterns, tweak balance.

## Key Entry Points for Edits

- **Main loop**: Modify `update()` to change game behavior, `draw()` to change rendering.
- **Enemy behavior**: `spawnEnemy()` to add types; enemy movement at 1210–1227; collision at 1232–1253.
- **Boss behavior**: `updateBoss()` (1411–1439); add patterns to `BOSS_PATTERNS` array.
- **Player controls**: Input handling at 768–861 (keyboard, mouse, touch).
- **Audio**: `sfx` object defines all sound effects; toggle mute via `muted` flag.
- **UI state**: HUD updates at 899–906, 900, 1091–1093, 1463–1467.
- **Local storage**: Best score, muted state, difficulty, reduced-motion preference persist across sessions (517–523, 537, 743, 1477).

## Testing Tips

- Use browser DevTools to pause execution or set breakpoints in `update()` or `draw()`.
- Toggle reduced motion in the UI to verify animations still feel responsive.
- Test touch controls on mobile or use DevTools device emulation.
- Adjust `DIFFICULTIES[difficulty]` constants to quickly verify balance changes.
- Check localStorage in DevTools to verify persistence (best score, etc.).
