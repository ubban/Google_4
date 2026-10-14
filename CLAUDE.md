# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in vanilla JavaScript and Canvas. It's a single-file, self-contained web application with no build process, dependencies, or external assets.

The game features:
- Multiple difficulty levels (easy, normal, hard)
- Six unique boss encounters per playthrough
- Three enemy types with distinct behaviors
- Adaptive performance quality tuning
- Touch and keyboard controls
- Six visual themes (different sky and color schemes)
- WebAudio synthesized sound effects
- Score tracking with local storage persistence

## Architecture

### Single-File Design

All code is contained in `index.html` (1669 lines):
- Lines 1–421: HTML markup + embedded CSS styling
- Line 422+: Self-contained IIFE with all JavaScript game logic

This monolithic design simplifies deployment and avoids dependency management, at the cost of no modular separation. Major changes may require careful line-by-line editing.

### Game Loop Architecture

The core game loop runs via `requestAnimationFrame` and follows a standard pattern:
1. **Input handling** – keyboard/touch events update player state
2. **Update** – entity movement, collision detection, state transitions
3. **Render** – canvas drawing with adaptive quality scaling
4. **Performance tracking** – frame time averaging to adjust quality

Key functions:
- `gameLoop(now)` – main frame handler, calls update and render
- `update(dt)` – physics, movement, collisions, state changes
- `render()` – canvas drawing (quality-aware particle culling)
- `trackPerf(dt)` – adaptive quality management (3 quality levels)

### Entity System

Entities (player, enemies, bullets, explosions) are stored in simple arrays:
- `player` – single object with position, velocity, health, weapon state
- `enemies` – array of active enemies (type, position, health, behavior)
- `bullets` – array of projectiles (origin, velocity, type, owner)
- `particles` – array of visual effects (position, velocity, life remaining)

No class inheritance; entities are plain objects with direct property mutation.

### Game State Progression

- **Title screen** – overlay showing best score, difficulty selection
- **Active play** – HUD with score, health, weapon, boss bar; entities update and collide
- **Stage transitions** – visual banner, stage count increment, enemy spawn rate increase
- **Boss encounters** – triggered after 18 enemy kills per stage; special spawn behavior
- **Game over** – overlay with final stats, best-score highlight if beaten
- **Pause** – freezes update, renders semi-transparent overlay

### Difficulty System

```javascript
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
};
```

Each difficulty adjusts spawn frequency, enemy behavior, starting lives, and boss health scaling.

## Key Systems & Configuration

### Enemy Types (ENEMY_TYPES)

- **grunt** – slow, weak baseline enemy
- **interceptor** – fast, weaving pattern, introduced early
- **tank** – heavy armor, slow movement, appears from stage 2+

### Bosses (BOSSES)

Six unique boss types cycle per playthrough. Each has custom colors, health scaling, and movement patterns. `hpMul` adjusts boss HP relative to difficulty multiplier.

### Themes (THEMES)

Six visual themes cycle per stage. Each sets a color gradient for the sky background and sky/sun RGB values for particle effects.

### Player Weapons

- **Gun** – basic projectile, upgrades increase fire rate and damage scaling
- **Laser** – continuous beam, damage per frame, uses energy system
- **Spread** – multi-directional shots with slight divergence
- **Burst** – rapid-fire with damage falloff

Weapon switching is automatic via cycle or can be forced. Upgrades are applied via pickup system.

### Audio

WebAudio API synthesizes all sound:
- `beep(freq, dur, type, vol, opts)` – tone generator with sweep/filter/pan
- `noiseBurst(dur, vol, opts)` – noise generator for explosions

No external audio files. Mute state persists in localStorage.

## Development Workflow

### Running the Game

1. Open `index.html` directly in a browser (Chrome, Firefox, Safari, Edge all work)
2. No server required; file:// protocol works fine
3. Game state is persisted in localStorage (best score, mute/motion preferences)

### Testing Changes

- Edit `index.html` directly
- Reload the browser tab to see changes
- Test all three difficulty levels (click difficulty buttons before starting)
- Verify touch and keyboard controls work
- Check browser console for any JavaScript errors

### Key Edit Points

**Game constants** (lines ~485–515):
- Modify `DIFFICULTIES` to change game balance
- Edit `THEMES` for visual variety
- Adjust `ENEMY_TYPES` spawn behavior, or `BOSSES` colors/scaling

**Game loop behavior** (search for `gameLoop`, `update`, `render`):
- Core frame handler and entity updates live here
- Collision detection and physics calculations
- Canvas drawing and animation frames

**Audio tuning**:
- Adjust oscillator types, frequencies, durations in `beep()` calls
- Modify volume via `vol` parameter or `masterGain.gain.value`

**UI/HUD**:
- Overlay and stats display are DOM elements, styled via CSS (lines ~10–420)
- Update HUD during gameplay via `score`, `health`, `weapon` state mutations

### Performance Notes

- Adaptive quality (variable particle/effect count) activates if frame time exceeds 1/40 ms
- Canvas resolution is fixed at 800×600 (CSS scales to viewport)
- Particle culling and batch rendering keep high enemy counts smooth
- Touch controls use a virtual joystick rendered via canvas

## Local Storage Keys

```javascript
const STORAGE_BEST = 'skyfighter.bestScore';
const STORAGE_MUTE = 'skyfighter.muted';
const STORAGE_DIFF = 'skyfighter.difficulty';
const STORAGE_REDUCED = 'skyfighter.reducedMotion';
```

All stored as strings or '0'/'1' for booleans.

## Browser Compatibility

Works on all modern browsers (Chrome, Firefox, Safari, Edge) and mobile. Touch controls are available on mobile devices. Uses standard Web APIs:
- Canvas 2D
- WebAudio
- localStorage
- requestAnimationFrame
- PointerEvents / TouchEvents

## Future Development Considerations

1. **Modularization** – split HTML/CSS/JS into separate files if adding complexity
2. **Level editor** – external JSON or in-game tool to define boss patterns and stages
3. **Leaderboard** – remote storage of scores
4. **Mobile optimization** – further refinement of touch-zone sizing and haptic feedback
5. **Accessibility** – more comprehensive screen-reader support (already uses aria-live for announcements)
