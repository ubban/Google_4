# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-page HTML5 canvas application. The entire game—gameplay logic, rendering, input handling, and audio synthesis—is contained in a single 1669-line `index.html` file.

## Running and Testing

### Local Development

**Open in browser:**
```bash
# Simply open index.html in any modern browser
# On macOS/Linux:
open index.html
# or use a simple HTTP server to avoid CORS issues:
python3 -m http.server 8000
# Then visit http://localhost:8000
```

**Playwright headless testing** (from previous commits):
The game has been verified to work headlessly with Playwright for automated testing. Key test scenarios:
- Firing and collisions
- Weapon pickups and progression
- Boss spawn and attack patterns
- Boss death and stage advancement
- All three difficulty modes
- Accessibility features (reduced-motion, screen reader announcements)

## Architecture and Game Systems

The game is organized into several interconnected systems within a single event-driven loop. Read the code in this order to understand the architecture:

### Core Game Loop
- **`requestAnimationFrame(loop)`** (line ~1630): Main loop that calls `update(dt)` for game logic and `draw()` for rendering. Maintains a target frame rate with delta-time (dt) clamped to 50ms.
- **`update(dt)`**: Handles all game state: player movement, shooting, collision detection, enemy AI, boss patterns, stage progression.
- **`draw()`**: Canvas rendering of all game objects using a custom `drawPlane()` function.

### Player System
- **Player state**: position, health (lives), weapon type/level, bomb count
- **Movement**: Keyboard (WASD/arrows), touch joystick (on-screen analog stick), or mouse steering (position-tracking at 1.7x speed cap)
- **Shooting**: Held button fires continuously; weapon type (Vulcan/Laser) determines bullet shape and behavior
- **Bombs**: Limited-use screen-clear attacks that damage bosses; regenerate over time

### Weapon System
- **Vulcan**: Yellow rectangular bullets, rapid fire (default)
- **Laser**: Light-blue bullets, fewer but stronger, visually distinct
- **Power levels**: 3 tiers that increase fire rate and damage. Power-ups (+weapon) advance the level; losing a life resets to level 0
- **Weapon pickups** are green spinning squares; laser pickups are rarer

### Enemy System
Three enemy archetypes with distinct behavior:
- **Grunt** (light blue): Simple direct approach, explodes quickly
- **Interceptor** (yellow): Weaves side-to-side, more evasive
- **Tank** (dark red): Slower, multi-hit (health bar visible above), graded rewards

Each enemy type has its own explosion sound and visual.

### Boss System
Six boss designs on a rotation (3 designs per every 6 stages):
- **Vanguard**: Radial burst, aimed fan, wide V patterns
- **Behemoth**: Heavy columns, homing spread, bullet wall
- **Specter**: Teleport dash, spiral, twin aimed shots
- **Dreadnought, Wraith, Titan** (alternates with above)

Each boss has:
- Unique color scheme
- 3–4 distinct attack patterns with a telegraph flash before firing (for fair readability)
- Health bar displayed above
- Distinct death stinger audio and stage-clear fanfare

### Stage and Progression
- **Waves**: Each stage contains multiple waves of enemies. Pattern of enemy types and counts is defined per stage.
- **Stage themes**: Cycle through 6 background visuals (daylight, sunset, night variations) as stages progress
- **Boss spawn**: After the final wave of a stage, a boss appears; defeating it advances to the next stage

### Scoring System
- **Kill multiplier** ("combo"): Increments for each kill; resets on a 3-second timer since the last kill. Displayed in top-center HUD
- **High score**: Persisted to localStorage; displays on game over
- **Points per kill**: Enemy type and current combo multiplier determine the score awarded

### Audio System
A custom synthesis engine (lines ~980–1200) replaces simple beeps:
- **Master gain and dynamics compressor**: Overlapping explosions don't clip
- **Per-sound biquad filtering**: Shapes tone character
- **Pitch sweeps**: Give impact weight to events
- **Stereo panning**: Events pan left/right based on on-screen x position for spatial depth

Events with audio:
- Enemy explosions (3 types, one per enemy class)
- Hit feedback (when player takes damage)
- Bomb/power-up pickup cues
- Boss telegraph (warning before an attack pattern fires)
- Boss death stinger
- Game-over stinger
- UI clicks (mute, pause, difficulty buttons)

### Accessibility
- **Reduced-motion toggle**: Respects OS `prefers-reduced-motion` setting by default; user can toggle. Disables screen shake, flash, and pulse animations for photosensitive players.
- **Screen reader support**: An `aria-live` region announces life loss, boss defeats, and game-over results.
- **Fixed z-index bug**: The top control bar (mute/motion/pause) was previously behind the start overlay and unclickable before starting; now correctly interactive.

### Performance Adaptation
A frame-time monitor (lines ~1500–1530) automatically scales quality in response to device performance:
- **Quality 2** (default): Full particle effects, shadow glow on bullets and power-ups
- **Quality 1**: Reduced particle counts
- **Quality 0**: No glow effects (shadow blur disabled)

The system steps down when frame time exceeds 18ms (≈55 FPS), and steps back up once it recovers to sub-16ms (≈60 FPS).

## Key Implementation Details

### Collision Detection
- Simple axis-aligned bounding box (AABB) checks for all collision types:
  - Player bullets vs enemies
  - Player bullets vs boss
  - Enemy bullets vs player
  - Player vs power-ups

### Canvas Coordinate System
- Origin (0, 0) at top-left
- X-axis rightward, Y-axis downward
- Camera is fixed; no scrolling or parallax beyond the static cloud-layer backgrounds

### State Management
- All game state is global: `enemies`, `bullets`, `enemyBullets`, `powerUps`, `particles`, `boss`, `player`, `stage`, `wave`, etc.
- No class-based architecture; all updates are procedural within `update(dt)`
- Game states: "running" (in-play), "paused", and "game over" (determined by flags)

## Editing and Extending

### To add a new feature:
1. **Identify the relevant system** (player, weapon, enemy, boss, audio, etc.) and locate its code sections
2. **Modify the appropriate update/draw functions** and add new fields to relevant state objects
3. **Test in the browser** using keyboard/mouse/touch controls; use browser DevTools console to watch for errors
4. **For performance-sensitive changes**, measure frame time with the built-in performance monitor (visible in top-right)
5. **For new audio**, add a new synthesis routine following the existing patterns (see synth functions like `playEnemyExplosion`, `playBombPickup`, etc.)

### To add a new boss design:
1. Add a new entry to the `BOSS_DESIGNS` array with unique colors and attack patterns
2. Define attack pattern functions (e.g., `radialBurst`, `aimmedFan`) in the BOSS system section
3. Update the cycle logic in `spawnBoss()` to include the new design
4. Add corresponding death audio if desired

### To add a new enemy type:
1. Add an entry to `ENEMY_TYPES` object with stats (speed, health, behavior)
2. Create a unique shape in `drawPlane()` by adding color/dark flag logic
3. Add behavior logic in the enemy update loop
4. Add a unique explosion sound in the audio system

## Testing Checklist

Before committing changes:
- [ ] Game starts and reaches playable state with no console errors
- [ ] Player can move (keyboard/touch/mouse), shoot, and bomb (if changes touch those)
- [ ] Enemies spawn, move, and die correctly
- [ ] Bosses spawn after the final wave and exhibit correct attack patterns
- [ ] Audio plays without clipping or errors (check browser DevTools Audio tab)
- [ ] High score persists across game restarts (localStorage)
- [ ] Reduced-motion mode dampens animations without breaking gameplay
- [ ] Game runs at stable 60 FPS (or gracefully adapts to lower frame rates)
- [ ] Mobile touch controls work on a real device or Chrome DevTools mobile emulation

## Performance Considerations

- **Canvas rendering**: Avoid expensive operations (repeated `ctx.save()`/`restore()`, complex path draws) in the draw loop
- **Audio context**: Audio nodes are created only once per event; reusing them prevents memory leaks
- **Particle system**: Adjust max particle count or disable glow in low-quality mode if frame time is critical
- **Enemy spawn rate**: Waves can have many enemies; verify frame time doesn't degrade during heavy spawn waves

## File Structure

```
Google_4/
├── index.html       (entire game: HTML structure, CSS, and 1400+ lines of JavaScript)
├── README.md        (minimal placeholder; update with game description if needed)
└── .git/            (version history)
```

The single-file design makes the game easy to share and deploy but requires discipline when adding features—keep related logic grouped and use consistent naming conventions to maintain readability.
