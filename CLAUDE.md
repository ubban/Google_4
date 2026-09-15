# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file Raiden-style vertical shoot-'em-up game built with HTML5 Canvas, Web Audio API, and vanilla JavaScript. The entire game logic, rendering, and audio synthesis are contained in a single `index.html` file (1669 lines). It's a browser-based game with:
- Multiple difficulty levels (Easy, Normal, Hard)
- Boss encounters with pattern-based attacks
- Weapon upgrade system
- Persistent high score tracking
- Full keyboard, mouse, and touch input support
- Procedurally generated sound effects via Web Audio synthesis
- Accessibility features (reduced motion support, screen reader announcements)

## Code Architecture

The game follows a monolithic single-file structure organized into logical sections:

### Core Sections (in reading order)
1. **DOM element caching** (lines 426-461): Caches all frequently-used DOM references
2. **Configuration constants** (lines 463-515): Game constants like difficulties, enemy types, bosses, themes
3. **Audio system** (lines 542-676): Web Audio context and procedural SFX generation
4. **Input handling** (lines 767-861): Keyboard, mouse, touch/joystick input management
5. **Game state** (lines 863-910): State initialization and player object
6. **Game loop** (lines 1640-1652): Main update/draw loop using `requestAnimationFrame`
7. **Rendering** (lines 913-1638): All canvas drawing functions
8. **Game logic** (lines 1200-1454): Enemy spawning, collision detection, boss logic

### Key Game Mechanics

- **Enemy spawning**: Difficulty-based spawn rate that increases over time (line 1201)
- **Damage and collision**: Rectangles overlap checking (line 1236+); enemies have HP, bullets have pierce levels
- **Boss patterns**: 6 boss types with 3 distinct attack patterns each (lines 1372-1409)
- **Weapon system**: Two weapon types (Vulcan/Laser) with 3 levels each; switching at max level (line 1315)
- **Scoring**: Kill streaks trigger combo multiplier messages; boss kills award 500 points (line 1446)
- **Difficulty progression**: Stage advances after 18 kills per stage; difficulty ramps spawn rate (line 1202)

## Code Organization

### State Management
All game state is in module scope (lines 864-869):
- `player`: Player aircraft object with position, speed, weapon, invuln timer
- `enemies`, `enemyBullets`, `bullets`: Dynamic arrays that are filtered each frame
- `score`, `lives`, `stage`, `chain`: Gameplay counters
- `boss`: Single boss object or null when not active

### Performance Adaptations
- Adaptive quality levels (0=full, 1=medium, 2=low) based on frame time averages (lines 474-483)
- Frame time is capped at 50ms (line 1644) to prevent large jumps when tab is backgrounded
- `quality` variable can be used to reduce particle count or effect complexity in future updates

### Audio Design
No external audio files; all sounds are synthesized via Web Audio API:
- `beep()` function (line 566): Creates tonal sounds with frequency sweeps
- `noiseBurst()` function (line 597): Creates noise-based explosion sounds
- Panning uses `panFor()` helper to spatialize sounds left/right (line 562)
- Audio context is lazily initialized on first user interaction

### Rendering Pipeline
1. Clear canvas and optionally apply screen shake translation (line 1523-1527)
2. Draw sky gradient (theme-based) (lines 1529-1535)
3. Draw sun glow effect (lines 1537-1541)
4. Draw parallax clouds at two speeds (lines 1543-1544)
5. Draw player, enemies, boss, particles (lines 1549-1636)
6. DOM HUD elements update separately (not canvas)

### Storage
Uses browser `localStorage` with 4 keys (lines 463-466):
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Mute state (1 = muted)
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Accessibility preference

## Common Development Tasks

### Adding a new enemy type
1. Add entry to `ENEMY_TYPES` constant (line 502-506) with properties: `w, h, hp, speedMul, color, dark, score, minStage`
2. Add spawn logic in `spawnEnemy()` function (around line 1180)
3. Add explosion SFX mapping in `EXPLODE_SFX` (line 678)
4. Optionally add unique SFX generator in `sfx` object (lines 624-676)

### Adding a new boss
1. Add entry to `BOSSES` array (lines 508-515) with hull/wing/core colors and HP multiplier
2. Create attack pattern functions (following existing patterns at lines 1324-1370)
3. Add patterns to `BOSS_PATTERNS` array in correct index order (lines 1372-1409)
4. Note: Boss index in `BOSS_PATTERNS` must match position in `BOSSES` array

### Adding a new stage theme
1. Add theme object to `THEMES` array (lines 493-500) with `sky` gradient stops, `cloud`, `sun` RGB values, and `name`
2. Themes cycle automatically: `THEMES[(stage - 1) % THEMES.length]` (line 1529)
3. Sky gradient uses 3 stops; cloud colors are RGBA format strings

### Modifying difficulty parameters
- Edit `DIFFICULTIES` object (lines 485-489)
- Key parameters: `spawnBase` (initial spawn timer), `spawnMin` (minimum), `enemySpeed` multiplier, `lives`, `bombs`, `bossHpMul`
- Spawn rate formula: `Math.max(spawnMin, spawnBase - Math.floor(frame/300)*4)` (line 1202)

### Input handling patterns
- Keyboard: Cached in `keys` object by `e.code` (line 769)
- Mouse: Steering via `mouseTarget` position; firing via `firing` flag (line 856)
- Touch: Virtual joystick in `moveZone`; buttons in `actionZone` (lines 814-834)
- All input types are checked independently so they can work together

## Technical Notes

### Rectangle Collision Detection
Uses axis-aligned bounding boxes: `rectsOverlap(a, b)` (not defined in provided excerpt, but used throughout). Assumes objects have `x, y, w, h` properties representing center position and dimensions.

### Frame Timing
- `frame` counter increments once per update (used for animation)
- `dt` is delta time in seconds, clamped to 50ms
- Use `frame * 0.15` for smooth rotation speeds (e.g., line 974)

### Screen Shake
- `screenShake(magnitude, duration)` sets shake state; magnitude is pixel offset
- Reduced motion mode dampens shake to 15% (line 728)
- Applied via `ctx.translate()` each frame while `shakeTime > 0` (line 1524-1527)

### Particle System
Simple particles with position, velocity, lifetime, and color. Added via `spawnExplosion()`. Filtered each frame to remove dead particles (line 1297).

### Accessibility
- Screen reader announcements via `announce()` function (line 468)
- Reduced motion support: animations are removed via CSS class toggle (lines 349-358)
- High-contrast HUD with text shadows for readability
- Touch-friendly button sizes (78px for mobile buttons, line 311)

## Workflow Tips

- **Canvas size**: Base 480x720, scales up to 1.6x on large screens (line 757)
- **Weapon firing mechanics**: Player fires when `firing` flag is true; cooldown controlled by `player.cooldown` (not shown in excerpt)
- **Boss telegraphing**: 22-frame telegraph period before attack (line 1433)
- **Stage progression**: After `KILLS_PER_STAGE` (18) kills, boss spawns after 1.2s delay (line 1206)

## Git Development Branch

All work should be committed to: `claude/inspiring-planck-2q0o6b`

Use clear commit messages describing gameplay changes or bug fixes. Keep commits atomic and focused on single features or bug fixes.
