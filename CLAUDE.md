# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file HTML5 game — a Raiden-style vertical shoot-'em-up with player weapon progression, boss battles, and difficulty scaling. The entire game (HTML, CSS, JavaScript) is contained in `index.html` (1669 lines).

## Running the Game

- **Play locally**: Open `index.html` directly in a browser (any modern desktop or mobile browser)
- **Test in Claude Code**: Use the `/run` skill to launch and test the game in a browser
- **No build process**: This is a standalone HTML file with no compilation or build step needed

## Game Architecture

### Core Game Structure

The game uses an **event loop pattern** with frame-based updates:
- `requestAnimationFrame` drives the main game loop
- `update(dt)` handles game logic and state changes each frame
- `draw()` renders the current frame to canvas
- Performance tracking (`trackPerf()`) dynamically adjusts quality (0=full, 1=medium, 2=low)

### Key Game Systems

**1. Game State Management**
- States: `overlay` (menu/end), `running` (active play), `paused`, `gameOver`
- Persistent storage via `localStorage`: best score, mute state, difficulty, motion preferences
- Stage progression: `KILLS_PER_STAGE` enemies trigger boss spawn, then next stage

**2. Player System (`player` object)**
- Properties: `x`, `y`, `w` (width), `h` (height), `health`, `invuln` (invulnerability frames), `weapon`, `weaponLvl`, `bombs`
- Weapon types: `'gun'` (rapid bullets), `'laser'` (narrow damage beam), `'wave'` (spread pattern)
- Weapon progression: kills increase weapon level, rotating through weapon types
- Controls: keyboard (WASD/arrow keys), mouse (crosshair), touch (joystick)
- Bombs: limited smart bombs, damage all on-screen enemies

**3. Rendering & Theme System**
- Canvas at native resolution, centered on screen
- `THEMES` array rotates stage backdrops (sky color, clouds, sun)
- Dynamic sky gradients and cloud parallax
- Screen shake effect on player damage/bomb: `screenShake(magnitude, duration)`
- Vignette flash on hits, white flash on bombs

**4. Enemy & Boss System**
- `ENEMY_TYPES`: scouts (fast, weak), heavies (slow, tanky), zigzags (pattern movers)
- Enemies spawn in waves, count toward stage completion
- `BOSSES` array defines boss patterns for each stage (sprite data, attack pattern, health)
- Enemies drop pickups: weapons, health, score multipliers
- Collision detection: per-entity bounding boxes vs. bullets and player

**5. Audio System**
- Web Audio API with synth-based sound generation (no audio files)
- `sfx` object: methods like `sfx.shoot()`, `sfx.hit()`, `sfx.gameover()`
- Mute toggle persisted to `localStorage`
- Lazy initialization: audio context created on first sound (respects autoplay restrictions)
- Panning based on horizontal position

**6. Input & Controls**
- Keyboard: WASD/arrow keys (move), space (shoot), B key (bomb), P key (pause)
- Mouse: crosshair target, click to pause/menu interaction
- Touch: virtual joystick + fire/bomb buttons (shown only on touch devices)
- Motion sensors: tilt controls (toggle with motion button, stored preference)

### Code Organization

**Initialization block** (post-DOMContentLoaded):
- Get DOM references (canvas, HUD elements, buttons)
- Set up event listeners
- Load saved settings from `localStorage`
- Initialize overlay (start screen, stats display)

**Game state & constants** (top of script):
- `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES` — game balance data
- Global state: `running`, `paused`, `gameOver`, `score`, `stage`, `difficulty`
- Player & enemy collections: `player`, `enemies[]`, `projectiles[]`, `pickups[]`

**Update functions** (frame logic):
- `update(dt)`: player movement, collision checks, enemy spawning/movement, projectile updates
- Difficulty scaling: enemy count, projectile speed, spawn rate change per `DIFFICULTIES[difficulty]`

**Draw functions** (rendering):
- `draw()`: background (sky gradient + clouds), entities (player, enemies, projectiles, pickups), HUD
- Helpers: `drawPlane()` (player sprite), `drawEnemy()` (enemy sprites), `drawClouds()` (parallax)

**Event handlers**:
- `startGame()`: reset state, hide overlay, begin game loop
- `endGame()`: freeze state, show final score and best score
- `togglePause()`: pause/resume, show overlay with pause state
- Canvas mouse/touch listeners for targeting and UI interaction

## Common Development Tasks

### Adjusting Game Difficulty

Edit `DIFFICULTIES` object (line ~485):
```javascript
const DIFFICULTIES = {
  easy: { enemyMult: 0.7, enemySpeed: 1, projectileSpeed: 1 },
  normal: { enemyMult: 1, enemySpeed: 1, projectileSpeed: 1 },
  hard: { enemyMult: 1.4, enemySpeed: 1.3, projectileSpeed: 1.2 }
}
```
- `enemyMult`: how many enemies per wave
- `enemySpeed`: enemy movement speed multiplier
- `projectileSpeed`: projectile movement speed multiplier

### Adding a New Boss

1. Add entry to `BOSSES` array with sprite pattern (lines in `drawPlane()` call) and `health` value
2. Increase `BOSSES.length` expectation or handle gracefully in stage logic
3. Test by advancing to that stage (kill `KILLS_PER_STAGE` enemies)

### Adding a New Weapon Type

1. Add new weapon name to weapon cycle logic (check `player.weapon` switch statements)
2. Define firing pattern in `update()` where `player.weapon` is checked
3. Add sprite rendering in `draw()` for the new weapon effect
4. Add SFX in weapon fire branch

### Tweaking Game Balance

Key values scattered throughout:
- `KILLS_PER_STAGE`: enemy kills required per stage (line ~491)
- `player.health` max (initial player health)
- Pickup spawn rates: adjust random thresholds where `pickups.push(...)` is called
- Weapon level progression: edit the weapon cycling logic in `update()`

### Screen Shake & Visual Effects

- Trigger with `screenShake(magnitude, duration)` (e.g., `screenShake(16, 0.4)` on damage)
- Vignette flash: toggle `vignetteEl.classList.toggle('on')`
- White flash: `flashEl.classList.add('hit')` or `'bomb'`

## Testing & Debugging

### Running the Game

Simply open `index.html` in a browser or use `/run` skill in Claude Code.

### Key Features to Verify When Making Changes

1. **Player Movement**: WASD/arrows move smoothly, stays in bounds
2. **Weapon Firing**: Each weapon type fires correctly, kills enemies
3. **Collision Detection**: Player takes damage from enemies/projectiles, enemies destroyed by bullets
4. **Stage Progression**: After `KILLS_PER_STAGE` kills, boss spawns; after boss defeat, next stage starts
5. **Audio**: Sounds play (if unmuted), mute toggle works
6. **Difficulty Scaling**: Hard mode has more/faster enemies
7. **Touch Controls**: Joystick and buttons functional on mobile/touch devices
8. **Persistence**: Best score saved, mute/difficulty/motion preferences restored after refresh

### Browser DevTools Tips

- Open Console to see `announce()` messages (logs game events)
- Watch `frame` (global counter) in watch list to understand timing
- Breakpoint in `update()` to inspect `player`, `enemies`, `score` state
- Check `localStorage` in DevTools → Application tab to verify saved settings

## Performance Notes

- Game auto-reduces quality on low-end devices (checks FPS, sets `quality` 0/1/2)
- Canvas drawn at native resolution; responsive sizing with CSS `max-width`/`max-height`
- Reduced motion preference respected (no animations if `prefers-reduced-motion`)
- Web Audio API: lazy-init, no audio files to load

## Browser Compatibility

- Requires: Canvas 2D context, Web Audio API, ES6 (arrow functions, template literals)
- Works on: Chrome, Firefox, Safari, Edge, mobile Safari, Android browsers
- Note: Autoplay audio restricted in modern browsers; audio context initialized on user interaction
