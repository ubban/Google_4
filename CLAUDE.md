# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game, fully implemented in a single HTML file. The entire codebase (HTML, CSS, JavaScript) is contained in `index.html` (1669 lines).

## Architecture Overview

The game is structured around an IIFE (Immediately Invoked Function Expression) to encapsulate all game logic in a single scope. Key architectural components:

### Core Game Loop
- **Rendering**: Canvas 2D context, responsive sizing (BASE_W: 480, BASE_H: 720)
- **Update cycle**: `update(dt)` → `draw()` → `requestAnimationFrame` for 60 FPS
- **Delta time**: Capped at 50ms to prevent large jumps on lag spikes

### Game State Management
- **Player object**: position, velocity, weapon/level, invulnerability timer, cooldown
- **Entity arrays**: `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- **Game flags**: `running`, `paused`, `gameOver`, `stage`, `lives`, `score`, `chain`/`multiplier`

### Input System
Three input layers (keyboard/gamepad > mouse > touch):
1. **Keyboard**: Arrow keys + WASD for movement, Space/click for fire, X/Shift/mobile buttons for bomb
2. **Mouse**: Player glides toward pointer when hovering canvas (low priority, overridden by keys/joystick)
3. **Touch**: Virtual joystick (left 55% of screen) + FIRE/BOMB buttons (right 45%, bottom)

### Collision Detection
- Simple AABB (axis-aligned bounding box): `rectsOverlap(a, b)` checks `a.x`, `a.y`, `a.w`, `a.h`
- Checked every frame: bullets vs enemies, bullets vs boss, enemies vs player, bullets vs player, powerups vs player

### Audio System
- Procedural WebAudio synthesis (no external files)
- `beep()`: oscillator with frequency sweep, filter, panning, envelope
- `noiseBurst()`: filtered white noise for explosions
- SFX prefixed by category: `sfx.shoot`, `sfx.explodeGrunt`, `sfx.bomb`, `sfx.bossTelegraph`, etc.
- Muted state persisted to localStorage

### Visual Effects
- **Screen shake**: `screenShake(magnitude, duration)` → applies random canvas translation
- **Flash overlay**: `flashHit()` / `flashBomb()` trigger CSS animations
- **Vignette pulse**: Red vignette flashes when health = 1 (reduced motion respects prefers-reduced-motion)
- **Particle system**: Explosions spawn expanding/fading circles with velocity

### Enemy/Boss AI
- **Enemy types**: grunt (fast, weak), interceptor (fastest, weak), tank (slow, durable)
- **Spawn mechanics**: Decreasing spawn timer based on difficulty; 18 kills triggers boss
- **Shooting patterns**: "spread" (fan) or "aimed" (toward player)
- **Weave behavior**: Interceptors oscillate left/right (baseX + sine wave)
- **Boss patterns**: Each boss (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) has 3+ unique attack patterns:
  - `fireRadial()`: omni-directional burst
  - `fireAimedSpread()`: fan aimed at player with spread angle
  - `fireColumns()`: straight vertical streams
  - `fireSpiralBurst()`: rotating 8-way pattern
  - `fireHomingSeeded()`: scattered aimed shots with noise
  - Boss telegraph: 22 frames of warning before pattern fires

### Difficulty Progression
- **Three presets**: easy (4 lives, 3 bombs, 0.8× boss HP), normal (3 lives, 2 bombs, 1× boss HP), hard (3 lives, 2 bombs, 1.3× boss HP)
- **Stage scaling**: Base HP = `70 + stage * 35`, multiplied by difficulty's `bossHpMul`
- **Spawn acceleration**: Over 300 frames, spawn timer decreases by 4 per tier
- **Six stage themes**: Cycling sky gradients, clouds, sun effects

### Performance Optimization
- **Adaptive quality**: Tracks frame times, reduces particle count / shadow effects if avg > 25ms
- Quality levels: 0 = full (glow, shadows), 1 = medium (reduced particles), 2 = low (minimal effects)
- Bullet/particle arrays pruned off-screen every frame

### Weapon System
- **Vulcan**: Rapid-fire bullets; spread widens at levels 2–3 (spread angles [-0.24, 0, 0.24])
- **Laser**: Piercing shots; extra lanes unlocked at level 2, and level 3 adds center lane; pierce count determines how many enemies it passes through
- Fire rate: Vulcan 0.15s, Laser 0.1s
- Cycle weapon on level-up past 3

### Combo/Scoring
- **Chain**: Increments on each kill; resets after 2.2s of no kills
- **Multiplier**: 1 + floor(chain / 5), capped at 10×
- **Base scores**: grunt 10, interceptor 15, tank 35, boss 500; powerup kills via bomb worth 5 each

### UI/HUD
- **Top bar**: Score, Best, Multiplier, Weapon, Lives (❤️), Bombs (💣)
- **Boss health bar**: Appears when boss spawns, hidden when defeated
- **Stage banner**: Slides in on stage start / boss approach
- **Combo indicator**: Pops up at 10, 20, 30... kills with scale animation
- **Overlay screens**: Start (difficulty buttons), pause, game over (new best indicator)

### Accessibility
- **Screen reader live region** (`#srLive`): Announces kill milestones, difficulty, pause, life loss
- **Reduced motion toggle**: Respects OS preference but user-overridable; disables animations when enabled
- **Focus management**: Icon buttons (mute, motion, pause) in top-right

### Persistence (localStorage)
- `skyfighter.bestScore`: Best score ever
- `skyfighter.muted`: Audio mute state (0 or 1)
- `skyfighter.difficulty`: Last selected difficulty (easy, normal, hard)
- `skyfighter.reducedMotion`: Motion override (0 or 1)

## Development Notes

### Modifying Gameplay
- **Enemy spawn rates**: Edit `DIFFICULTIES` object (`spawnBase`, `spawnMin`)
- **Enemy types**: Add new entries to `ENEMY_TYPES` with `w`, `h`, `hp`, `color`, `score`, `minStage`
- **Boss patterns**: Add new function references to `BOSS_PATTERNS` array (each boss gets 3+ patterns)
- **Themes**: Add entries to `THEMES` with sky gradient stops, cloud/sun colors, name
- **Stage progression**: Change `KILLS_PER_STAGE` (default 18) to alter when boss appears

### Canvas Sizing
- Base canvas: 480×720 (portrait)
- Scales up to max 1.6× or window bounds, maintaining aspect ratio
- Safe areas handled via `env(safe-area-inset-*)` for notch-aware mobile

### Audio Quirks
- WebAudio requires user interaction before sounds play (`ensureAudio()` on first input)
- Panning is mono (single pan value per sound, no binaural spatializing)
- Filter frequencies in Hz; higher = brighter
- Gain values are 0–1; typical sfx are 0.04–0.2

### Rendering Pipeline
1. Clear canvas
2. Apply screen shake (random translation)
3. Draw sky gradient + sun glow + parallax clouds (two layers, different speeds)
4. Draw player (with flame when not paused, blink when invulnerable)
5. Draw player bullets (with glow if quality = 0)
6. Draw enemy bullets (red circles with glow)
7. Draw enemies + tank health bars
8. Draw boss
9. Draw power-ups (spinning colored squares)
10. Draw particles (fading circles)

### Testing Tips
- **Easy testing**: Open in browser, difficulty buttons on start screen, use K to toggle reduced motion
- **Boss patterns**: Spawn boss immediately by setting `stageKills = KILLS_PER_STAGE` before defeat
- **Stress test**: Lower `quality` to 2 to see low-end performance path
- **Touch simulation**: DevTools Device Toolbar (Ctrl+Shift+M), tap fire/bomb buttons

## Commands

### Running/Playing
```bash
# Open in browser (macOS)
open index.html

# Or use a local server
python3 -m http.server 8000
# Then visit http://localhost:8000
```

### Linting & Formatting
This is a vanilla-JS single-file project with no build system. ESLint/Prettier not configured. Code style:
- Semicolons required
- 2-space indentation
- Single quotes for strings
- No trailing commas in arrays/objects

### No Tests
This project has no test suite. Quality assurance is manual play-testing. To add tests, you'd need to refactor into modules (currently an IIFE) or use a tool that can instrument the bundled code.

## Common Modifications Checklist

- **Adjust difficulty**: Edit `DIFFICULTIES` in the game state section (~line 485)
- **Add a new enemy type**: Add to `ENEMY_TYPES`, pick spawn rate in `spawnEnemy()` logic
- **Create new boss**: Add to `BOSSES` array, assign 3+ patterns in `BOSS_PATTERNS[newIndex]`
- **Tweak collision hitboxes**: Modify `a.w` / `a.h` in entity initialization (player, enemies, bullets, boss)
- **Change score values**: Edit numeric values in `addScore()` calls or entity definitions
- **New SFX**: Write a function in the `sfx` object, call `beep()` or `noiseBurst()`
- **Adjust visual effects**: Edit CSS animations (@keyframes) or Canvas drawing code in `draw()`
