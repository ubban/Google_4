# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML5 canvas application. The entire game logic, rendering, and UI are contained in `index.html` as an IIFE (Immediately Invoked Function Expression) with embedded CSS and JavaScript.

**Key Features:**
- Canvas-based 2D rendering with parallax scrolling clouds
- Multiple weapon types with upgrade system
- Boss battles at stage completion
- Touch and keyboard input support
- Web Audio API for procedural sound effects
- Local storage for best score and settings (mute, difficulty, reduced motion)
- Responsive design with safe-area support for mobile notches

## Architecture

The game uses a traditional game loop pattern: **Update → Draw → RequestAnimationFrame**

```
loop(now)
├─ update(dt)          // Game logic: movement, collisions, state changes
│  ├─ Player movement and firing
│  ├─ Enemy spawning and movement
│  ├─ Collision detection with scoring
│  ├─ updateBoss()     // Boss-specific logic
│  └─ Power-up handling
├─ draw()              // Render everything
│  ├─ drawClouds()     // Parallax background
│  ├─ Projectiles
│  ├─ Particles/explosions
│  ├─ Player ship
│  ├─ Enemies/bosses
│  └─ HUD updates
└─ requestAnimationFrame(loop)
```

**Game State:**
- `gameState`: one of `'title'`, `'running'`, `'paused'`, `'over'`
- `gameRunning`: boolean tracking if actively playing
- Enemy/boss/particle arrays updated each frame
- Player position, health, score, weapon, etc. as module-scoped variables

## Key Systems

### 1. **Rendering & Canvas**
- **Functions:** `draw()`, `drawPlane()`, `drawBossShip()`, `drawClouds()`
- **Location:** Lines ~1507–1639
- Canvas context stored in `ctx`, accessed via `canvas.getContext('2d')`
- Parallax clouds move at different speeds for depth
- Particles spawn with `spawnExplosion()` for visual feedback

### 2. **Audio System**
- **Procedural sounds** via Web Audio API in `beep()` and `noiseBurst()`
- **Location:** Lines ~545–678
- Audio managed by `ensureAudio()` which creates an AudioContext on first interaction (mobile requirement)
- Mute state stored in `localStorage` under key `skyfighter.muted`
- `panFor(x)` sets stereo pan based on bullet position

### 3. **Input Handling**
- **Keyboard:** Arrow keys / WASD for movement, Space / X / Shift for fire / bomb
- **Mouse:** Move to position, click to fire
- **Touch:** Joystick on left, buttons (FIRE / BOMB) on right
- **Location:** Lines ~785–868
- `joystickStart/Move/End()` handle touch input via `pointer*` events
- `canvasPointFromEvent()` converts client coordinates to canvas space

### 4. **Game Mechanics**
- **Enemies:** Defined in `ENEMY_TYPES` constant (line ~502); spawned via `spawnEnemy()` at configured intervals
- **Bosses:** Defined in `BOSSES` array (line ~508); one per stage; defeated after `killBoss()` score threshold
- **Weapons:** Five types (VULCAN, SCATTER, LASER, HOMING, RAPID); level 1–5; selected via `fireWeapon()` (line ~1124)
- **Power-ups:** Spawned by `maybeSpawnPowerUp()` on enemy kill; applied via `applyPowerUp()`
- **Score Multiplier:** Increases per hit in current chain; resets on miss via `breakChain()`

### 5. **Difficulty Settings**
- **Location:** `DIFFICULTIES` constant (line ~485)
- Three presets: `easy`, `normal` (default), `hard`
- Affects enemy spawn rate, movement speed, bullet density
- Stored in `localStorage` under key `skyfighter.difficulty`
- Set via `setDifficulty(name)` (line ~741)

### 6. **Collision Detection**
- Simple axis-aligned bounding box overlap via `rectsOverlap(a, b)` (line ~1069)
- Each entity has `x`, `y`, `w` (width), `h` (height) properties
- Player collision: check vs all enemy bullets; damage with `loseLife()`
- Enemy/boss collision: check vs all player bullets; kill with score via `addScore()`

## Common Modifications

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line ~502) with pattern: `{ x, y, vx, vy, w, h, type, hp, spawn, maxSpawn, ... }`
2. Implement firing in `updateBoss()` if it's a boss, or handle in `update()` for regular enemies
3. Handle rendering in `draw()` loop (search for enemy rendering)

### Adding a New Weapon
1. Add entry to `WEAPON_TYPES` object (if it exists; currently weapons are hardcoded in `fireWeapon()`)
2. Implement firing logic as a new case in `fireWeapon()` (line ~1124)
3. Add bonus multiplier adjustment in `applyPowerUp()` if upgrading weapon affects DPS

### Adding a New Boss
1. Add object to `BOSSES` array (line ~508) with `name`, `hp`, `firePattern`, `pos`, etc.
2. Implement the boss firing pattern as a function called from `updateBoss()` (line ~1411)
3. Add rendering in the `draw()` loop via `drawBossShip()`

### Modifying Difficulty
1. Edit `DIFFICULTIES` object to change spawn rates, enemy speeds, or bullet patterns
2. Test across all three difficulty levels with the difficulty buttons on the title screen

## Performance & Quality Settings

- **Automatic quality downgrade:** Tracks frame time via `trackPerf(dt)` (line ~476)
- `quality` variable: 0 = full, 1 = medium, 2 = low
- Reduces particle count, effect complexity, and cloud draw distance at lower quality
- Check `quality` before expensive operations (e.g., `if (quality < 2) spawnExplosion(...)`)

## Testing & Debugging

**Pause/Resume:** Press P or Esc during gameplay to pause; click the ⏸ button
**Mute:** Click 🔊 button to toggle sound
**Motion Effects:** Click 🎆 button to toggle reduced motion (disables screen shake, flashes, animations)
**Difficulty Selection:** Choose Easy/Normal/Hard on title screen

## File Structure

- **Single file:** `index.html` (1669 lines)
  - `<style>` block (~420 lines): CSS for HUD, buttons, animations, responsive layout
  - `<script>` block (~1250 lines): Game logic, rendering, and input handling

No build step required—open `index.html` in a browser to play.

## Key Constants

- `KILLS_PER_STAGE`: 18 enemy kills to advance to next stage
- `THEMES`: Difficulty-dependent spawn rates and enemy mix
- `STORAGE_*`: localStorage keys for persistence
- Frame rate targeting: ~60 FPS via `requestAnimationFrame`

## Accessibility

- **Keyboard full support:** All actions playable without mouse
- **Touch support:** Full joystick + button controls
- **Screen reader:** Live region (`#srLive`) for announcements via `announce(text)`
- **Reduced motion:** Toggle via button; disables animations when `prefers-reduced-motion` is set
- **Safe area support:** Uses CSS `env(safe-area-inset-*)` for notched devices
