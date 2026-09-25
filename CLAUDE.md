# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML file with embedded JavaScript, CSS, and WebAudio synthesis. The game runs entirely in the browser with no build step or external dependencies.

## Quick Start

- **Open the game**: Open `index.html` in any modern browser
- **No build required**: The game is self-contained and runs immediately
- **Development**: Edit `index.html` directly and refresh the browser to see changes

## Architecture & Code Organization

The entire game lives in a single `index.html` file (~1670 lines) organized into logical sections:

### Key Sections (by approximate line ranges)

1. **HTML Structure** (lines 362–420): Game canvas, HUD, overlay UI, touch controls
2. **CSS Styling** (lines 10–358): Responsive design with safe-area support, animations, dark theme
3. **Game Script** (lines 422–1666): All game logic, organized as follows:
   - **Constants & Configuration** (lines 463–523): Difficulty levels, enemy/boss definitions, themes, audio constants
   - **Audio System** (lines 542–676): WebAudio synth, beep/noiseBurst generators, sound effects (sfx.*) for all game events
   - **Input Handling** (lines 768–862): Keyboard, mouse steering (glides plane toward cursor), touch joystick with FIRE/BOMB buttons
   - **Game State** (lines 863–911): Player, bullets, enemies, power-ups, boss; score/lives/stage tracking
   - **Rendering** (lines 1507–1638): Canvas 2D for game objects, CSS for HUD/overlays, adaptive quality levels
   - **Game Logic** (lines 1148–1505): Main update loop, collision detection, power-up application, boss behavior
   - **Boss Patterns** (lines 1372–1409): Array of attack patterns (radial bursts, aimed spreads, columns, spirals, homing) for 6 boss types
   - **Game Loop** (lines 1640–1666): requestAnimationFrame, delta time, performance tracking

### Data Flow

1. **Input** → Keys, mouse, or touch → `update()` moves player, fires weapons
2. **Update** → `update(dt)` spawns enemies, updates positions, checks collisions, applies power-ups
3. **Render** → `draw()` paints sky gradient, clouds, player, enemies, bullets, particles, HUD
4. **Frame** → `loop()` calls update + draw, tracks performance, adapts quality

### Game Rules (Hardcoded)

- **Stages**: Endless progression; 18 kills per stage triggers boss; 6 stages cycle through themes and bosses
- **Weapons**: Vulcan (spread shot, 3 levels) and Laser (piercing beams, 3 levels); unlock by collecting power-ups
- **Combo System**: Kill streaks boost multiplier (1 + floor(chain/5), max 10x); 2.2s window to land next kill
- **Difficulty Modifiers**: Easy/Normal/Hard change spawn rates, enemy speed, boss HP, lives, bombs (see `DIFFICULTIES`)
- **Power-ups**: Life (6%), Bomb (8%), Weapon (20%) spawn on enemy kills; rarity scales with difficulty
- **Boss HP**: `70 + stage*35` × difficulty multiplier × boss design multiplier

## Making Changes

### Adding Features

- **New Enemy Type**: Add to `ENEMY_TYPES` constant, update spawn logic in `spawnEnemy()`
- **New Boss Attack**: Add function to `BOSS_PATTERNS` array; use helpers like `fireRadial()`, `fireAimedSpread()`, `fireColumns()`
- **New Sound Effect**: Add function to `sfx` object; compose using `beep()` (oscillator) and `noiseBurst()` (white noise)
- **New Theme/Sky**: Add to `THEMES` array (requires sky gradient colors, cloud/sun colors)
- **UI Changes**: Edit HTML (lines 362–420) and CSS (lines 10–358); overlay styling uses `#overlay`, `#overlayCard`, `.diffBtn`

### Tuning Gameplay

- **Difficulty balance**: Adjust `DIFFICULTIES[difficulty]` object (spawnBase, enemySpeed, shootChance, bossHpMul)
- **Boss patterns**: Modify functions in `BOSS_PATTERNS` or change timing in `updateBoss()` (telegraph=22, attackTimer)
- **Player speed/hitbox**: Edit `player` object in `resetGame()` (x, y, w, h, speed)
- **Weapon balance**: Adjust fireWeapon() spread, pierce, speed; modify cooldown timing
- **Collision detection**: Uses AABB (axis-aligned bounding box) in `rectsOverlap()`; hitboxes are w/h fields

### Performance Optimization

The game includes adaptive quality scaling (`quality` variable, 0=full/1=medium/2=low):
- Reduces particle count, shadow blur, and glow effects at lower quality
- Tracks frame times and adjusts automatically if avg > 1/40s
- To force quality: Set `quality = 0` (full), `1` (medium), or `2` (low) in console

### Accessibility Features

- **Reduced Motion**: Toggle reduces/disables screen shake, flash, and animations; respects browser `prefers-reduced-motion`
- **Screen Reader**: `#srLive` (aria-live) announces game events (boss defeated, lives remaining, etc.)
- **Pause**: P or Escape key pauses game; shows overlay with option to resume

### Browser Storage

Game uses LocalStorage for persistence:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio muted state (0/1)
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Motion preference (0/1)

## Common Tasks

### Test a Difficulty Tweak

1. Modify `DIFFICULTIES` constant
2. Reload `index.html` in browser
3. Select the difficulty and play to verify balance

### Debug Collision Issues

- Collision uses simple AABB: `Math.abs(a.x - b.x) < (a.w + b.w) / 2`
- If hitbox feels wrong, check object's `w` and `h` fields
- Draw hitboxes for debugging: Add rect drawing code to `draw()` function

### Add a Boss Attack Pattern

1. Write attack function: `(b) => { /* add to enemyBullets, move boss, etc */ }`
2. Use helpers: `fireRadial(b, count, speed)`, `fireAimedSpread(b, count, speed, spreadStep)`, etc.
3. Add to corresponding boss's pattern array in `BOSS_PATTERNS`
4. Boss patterns cycle through their array; change `b.patternIndex` timing in `updateBoss()` to control attack frequency

### Adjust Audio Volume

- Master gain is set in `ensureAudio()`: `masterGain.gain.value = 0.9` (0–1 range)
- Per-effect volume: Second parameter to `beep()` and `noiseBurst()`; typically 0.04–0.2 for balance

## Canvas Layout

- **Canvas size**: 480×720 logical (scaled to fit screen, max 1.6×)
- **Coordinate system**: (0,0) is top-left; Y increases downward
- **Player spawn**: Center X, 100px from bottom
- **Enemy spawn**: Random X, -30 (off-screen top)
- **Safe zones**: Viewport-aware UI uses `safe-area-inset-*` for notched devices

## No External Dependencies

This project has **zero dependencies**:
- No Node.js, npm, or build tools
- No external libraries (no HTML5 canvas frameworks, no WebAudio libraries)
- No asset files (sprites, sounds, fonts) — all generated procedurally

This simplicity makes the codebase easy to understand and modify, but also means all behavior (rendering, collision, audio) is hand-written.

## Testing

There are no unit tests or test runners. To verify changes:
1. Reload the browser
2. Play through a stage on each difficulty
3. Test edge cases: max combo, bomb usage, weapon switching, low FPS detection
4. Use browser DevTools console to inspect variables: `player`, `score`, `stage`, `quality`

## Browser Compatibility

- **Modern browsers**: Chrome, Firefox, Safari, Edge (2020+)
- **Mobile**: Full touch support (joystick + buttons)
- **Audio**: Requires Web Audio API (not supported in older IE)
- **LocalStorage**: Required for persistence; gracefully degrades without it

## Deployment

To host the game:
1. Upload `index.html` to any static web server
2. No configuration, no server-side code required
3. Works over HTTP or HTTPS
