# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game—including graphics, sound, game logic, and UI—is contained in `index.html` (1600+ lines).

### Running the Game

- **Local Development**: Open `index.html` directly in any modern web browser (Chrome, Firefox, Safari, Edge)
- **No Build Process**: No compilation, bundling, or dev server required
- **No Dependencies**: Entirely self-contained; uses only standard HTML5 Canvas and Web Audio APIs

## Game Architecture

### Core Systems

The game is organized around a classic game loop pattern with three main phases:

1. **Initialization** (lines 423-800): Game state setup, event listeners, audio system initialization, and UI elements
2. **Game Loop** (`loop()` at line 1640): Runs via `requestAnimationFrame`, calls `update(dt)` then `draw()` each frame
3. **Update Phase** (`update(dt)` at line 1148): Physics, collision detection, state transitions
4. **Render Phase** (`draw()` at line 1520): Canvas drawing of backgrounds, entities, and UI overlays

### Key Objects and State

**Player** (`player` object):
- Properties: `x`, `y`, `w` (width), `h` (height), `vx`, `vy` (velocity), `weapon`, `health`, `bombs`, `invuln` (invulnerability frames)
- Weapons: `'vulcan'`, `'laser'`, `'missile'`, `'plasma'` (each with different fire rates, speeds, and damage)
- Updated via keyboard input (WASD/Arrow keys) or touch joystick

**Enemies** (`enemies` array):
- Spawned based on stage patterns; move downward and fire projectiles
- Simple collision: bounding box checks with player and bullets
- Destroyed when health reaches 0; drop pickups (bomb refills, weapon upgrades)

**Boss** (`boss` object, set during specific stages):
- Larger, multi-phase enemy with attack patterns defined in `BOSSES` array (line 508)
- Designed to spawn at stage 5, 10, 15, etc. (cycles through `BOSSES` array)
- Uses same collision system as regular enemies but with larger health pools

**Game State Variables** (lines 423-466):
- `running`, `paused`, `gameOver`: Game state flags
- `score`, `bestScore`: Score tracking (best score persists in `localStorage`)
- `stage`, `stageProgress`: Current stage and progress counter
- `combo`, `comboBonus`: Combo multiplier for consecutive hits
- `difficulty`: Selected difficulty mode ('easy', 'normal', 'hard')
- `frame`: Frame counter for animations and timing

### Configuration Data

**THEMES** (line 493): Array of 4 visual themes (colors for sky gradient, clouds, sun) that cycle by stage

**BOSSES** (line 508): Array of boss ship designs with custom attack patterns; includes visual properties and behavior

**Weapon System**: Each weapon type has fixed fire rate, projectile speed, and damage baked into the update logic (search for `if (player.weapon ===` patterns)

### Visual Effects

- **Screen Shake**: `screenShake(magnitude, duration)` triggered by hits or bombs (line 727)
- **Flash Overlays**: Hit flash (`flashHit()`) and bomb flash (`flashBomb()`) for visual feedback
- **Combo Popup**: Large animated text showing combo count when streak builds
- **Stage Banner**: Animated text shown at stage start
- **Vignette Red Pulse**: Activates when health is critical (1 HP remaining)
- **Cloud Parallax**: Two layers of animated clouds at different speeds

### Audio System

Web Audio API implementation (`sfx` object, lines 545-677):
- Synthesized sounds: `beep()`, `noiseBurst()` for effects
- Spatial audio: `panFor(x)` pans sound left/right based on screen position
- Sound events tied to game actions: player fire, enemy spawn, hits, bombs, UI clicks
- Mute button controls all audio; persists mute state in `localStorage`

### Input Handling

**Keyboard** (lines 861-896):
- WASD or Arrow keys move player
- Space or Z fires weapon
- Q switches weapon
- E drops bomb
- P pauses game

**Touch** (lines 785-828):
- Virtual joystick on left side for movement
- Right side of screen used for aiming/firing (when mouse/touch active)
- Two-finger tap for bomb (on supported devices)

**Mouse** (line 837):
- Aim crosshair at mouse position while holding mouse button down; player faces and fires toward cursor

### Collision and Physics

- **Bounding Box Collision**: All collision checks use rectangular overlap (x/y position + width/height)
- **Projectile Management**: `bullets`, `enemyShots` arrays track in-flight projectiles
- **Pickup System**: `pickups` array for dropped items (bomb refills, weapon upgrades)
- **Damage Application**: Direct integer subtraction on health properties; entity removed when health ≤ 0

### Scoring and Progression

- **Score Multiplier**: Increases with consecutive hits (combo system); resets on miss
- **Stage Progression**: Enemy count per stage increases; every 5th stage spawns a boss
- **Difficulty Scaling**: 'easy', 'normal', 'hard' adjust enemy health and spawn frequency
- **Best Score**: Persisted to `localStorage` (key: `'sf_best'`); updated when current score exceeds it

## Common Development Tasks

### Testing a Game Feature

1. **Run the game**: Open `index.html` in browser
2. **Test a weapon**: Press Q to cycle weapons; notice different fire rates and visual effects
3. **Test enemy spawns**: Watch enemy waves increase in size across stages
4. **Test difficulty**: Use difficulty buttons (shown on start screen) to verify enemy behavior changes

### Adding a New Weapon

1. Add weapon definition to the weapon properties section (around line 1000)
2. Add fire logic in `update()` under the existing weapon fire conditions
3. Update `weaponNameEl` to display the new weapon name when selected

### Adding a New Boss

1. Create a new entry in the `BOSSES` array (line 508) with properties: `health`, `color`, `dark`, `attackPattern` function
2. The boss will spawn automatically every 5 stages (cycling through the array)
3. Implement the attack pattern function to define projectile spawning behavior (see existing bosses for patterns)

### Adjusting Game Balance

- **Enemy Difficulty**: Change `enemyCountByStage` or enemy health in the difficulty object (around line 1100)
- **Player Power**: Modify weapon fire rate or damage in weapon fire logic sections
- **Spawn Patterns**: Edit the stage progression and enemy wave patterns in the update loop
- **Boss Difficulty**: Adjust boss health or attack rate in the `BOSSES` array entries

### Debugging

- Open browser DevTools (F12 or Cmd+Option+I)
- Inspect `frame` counter for timing
- Check `console.log()` output by adding to `update()` or specific event handlers
- Monitor performance with DevTools Performance tab (watch canvas draw time)

## File Structure

```
Google_4/
├── index.html          (Main game file: 1600+ lines of HTML/CSS/JS)
├── README.md           (Project metadata)
└── CLAUDE.md           (This file)
```

## Performance Considerations

- **Canvas Rendering**: All graphics drawn to a single `<canvas>` element each frame
- **Frame Rate**: Targets 60 FPS via `requestAnimationFrame`; actual frame time logged if performance tracking is enabled
- **Asset Loading**: No external assets loaded; all visuals drawn programmatically or embedded as data URIs
- **Memory**: Enemy and bullet arrays grow/shrink dynamically; pickups cleaned up after collection or timeout

## Key Code Locations

| Feature | Location |
|---------|----------|
| Game loop | `loop()` at line 1640 |
| Update logic | `update(dt)` at line 1148 |
| Drawing logic | `draw()` at line 1520 |
| Input handling | Lines 785–896 (touch and keyboard) |
| Audio system | Lines 545–677 (`sfx` object) |
| Boss definitions | `BOSSES` at line 508 |
| Visual themes | `THEMES` at line 493 |
| Player draw function | `drawPlane()` at line 913 |
| Difficulty setup | Lines 741–752 |

## Notes for Future Work

- The entire game is self-contained for simplicity; breaking it into separate modules would require a build step
- Weapon and enemy balance has been tuned; large changes to spawn rates or projectile speeds may require rebalancing
- Touch controls use a virtual joystick; ensure new input schemes maintain mobile playability
- All visual feedback (flashes, shakes, popups) can be toggled via the reduced-motion button for accessibility
