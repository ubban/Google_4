# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The entire game engine, rendering, audio, and UI are contained in `index.html` with no external dependencies or build process required. The game runs directly in modern web browsers using Canvas 2D for rendering and Web Audio API for procedurally-generated sound effects.

## Architecture

### Single-File Structure

Everything is in `index.html`:
- **HTML markup**: Basic DOM elements (canvas, overlay UI, touch controls)
- **CSS**: Embedded styling for UI, animations, responsive design with `clamp()` for scaling
- **JavaScript**: ~1,600 lines of game logic in a single `<script>` tag

### Core Game Loop

The game uses a standard update-render pattern driven by `requestAnimationFrame`:

```
loop(timestamp)
  → update(dt)          // Game logic: movement, spawning, collision, score
  → draw()              // Canvas rendering: player, enemies, bullets, UI
  → requestAnimationFrame(loop)
```

**Key game state variables** (`const` and `let` at the top of the script):
- `player` — position, speed, weapon, level, invulnerability
- `bullets` — projectiles fired by player (laser or vulcan)
- `enemies` — active enemy ships with HP, patterns, and behavior
- `boss` — current stage boss (null when not present)
- `score`, `lives`, `chain`, `multiplier` — scoring and combo system
- `stage`, `stageKills` — progression tracking

### Difficulty System

Three difficulty levels stored in `DIFFICULTIES` object control:
- Enemy spawn rate and speed
- Enemy HP and shoot chance
- Boss HP multiplier
- Starting lives and bomb count

Selected difficulty persists in localStorage.

### Constants and Configuration

Key configuration objects near the start of the script:
- `DIFFICULTIES` — Game balance parameters per difficulty
- `ENEMY_TYPES` — grunt, interceptor, tank with stats and visual designs
- `BOSSES` — 6 unique boss designs with color schemes and HP multipliers

Tweak these objects to modify game balance, enemy behavior, or boss difficulty.

### Canvas Rendering

All graphics are drawn procedurally using Canvas 2D `fillRect`, `arc`, `beginPath`/`fill`, and gradient fills. No external images are used.

**Key rendering functions:**
- `drawPlane(x, y, w, h, color, darkColor, flame)` — Draws player and enemy ships
- `drawBossShip(boss)` — Boss-specific rendering with animated core and moving parts
- `draw()` — Main render function; updates HUD, game world, and overlays each frame

**Particle effects** use simple circles with opacity for explosions and visual feedback.

### Input System

**Keyboard:** Arrow keys or WASD for movement, Space to fire, P/Escape to pause, X/Shift for bomb

**Mouse:** Pointer position steers the player (smooth tracking), left-click to fire

**Touch:** Virtual joystick on left side (move zone), fire button on right (fire zone), bomb button below

**Priority:** Keyboard/joystick input always overrides mouse steering to avoid conflicts.

The input handlers set flags (`keys`, `firing`, `joyVec`) which the update loop reads each frame.

### Audio System

All sound effects are synthesized using Web Audio API with no external audio files. Synth sounds are generated on-demand using:
- Oscillators (sine, triangle, square, sawtooth)
- Filter nodes (lowpass, highpass)
- Stereo panning (sounds originate from bullet positions)
- Envelope envelopes for fade-in/out

The `sfx` object contains named sound effects (shoot, laser, explode, bomb, etc.). Muting toggles with the mute button; muted state persists in localStorage.

### UI and Overlays

The overlay system shows the start screen, pause screen, and game-over screen. HUD displays (hearts, bombs, score, weapon, multiplier) update each frame from game state. Stage banners and boss warnings animate in using CSS keyframes.

## Development Workflow

### Running the Game

Since there's no build step, simply:
1. Open `index.html` directly in a browser (file:// protocol works)
2. Or serve it over HTTP if needed for CORS or audio policies
3. Refresh the page after any edits

### Testing Changes

- Use **browser DevTools** (F12) for debugging:
  - Inspect canvas rendering and DOM
  - Console logs for variable inspection
  - Performance profiler to check frame rate
- **LocalStorage is persistent**, so game progress and settings survive page reloads
  - Clear with `localStorage.clear()` in console to reset
- Test on both **desktop** (keyboard, mouse) and **mobile** (touch) for controls

### Performance Considerations

**Graphics quality** is adjustable via `quality` variable (0=high, 1=medium, 2=low):
- Controls particle count in explosions
- Disables shadow/glow effects at lower quality to improve performance on slower devices

**Frame rate:** Capped implicitly by the screen refresh rate (typically 60fps). Large numbers of enemies or particles can cause dropped frames.

## Common Modifications

### Adjust Game Balance

Edit values in `DIFFICULTIES` to change:
- Spawn rate: `spawnInterval` controls how often enemies appear
- Enemy stats: `enemySpeed`, `enemyShootChance`
- Boss toughness: `bossHpMul`

Edit `ENEMY_TYPES` and `BOSSES` to adjust individual enemy/boss properties (HP, speed, color, score value).

### Add or Modify Enemies

Add a new enemy type to `ENEMY_TYPES` with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`. Then update the spawn logic in `spawnEnemy()` to include it in the random selection.

### Change Weapon Behavior

Edit `fireWeapon()` to change bullet patterns, speed, or count per shot. Adjust `fireDelay` in the update loop to change firing rate.

### Add New Power-up Types

Add a new power-up type in `maybeSpawnPowerUp()` (currently: life, bomb, weapon). Handle collection logic in the collision detection section of `update()`.

### Tweak Audio

Modify the `sfx` object sound definitions. Each sound uses `beep()` or `noiseBurst()` with parameters:
- Frequency (Hz) and sweep direction
- Duration (seconds)
- Volume (0.0–1.0)
- Waveform type (sine, square, sawtooth, triangle)
- Filter settings (type, frequency)

## Key Functions

- `update(dt)` — Core game logic; called once per frame with delta time
- `draw()` — Render all visuals
- `resetGame()` — Initialize a new game
- `fireWeapon()` — Spawn player bullets
- `spawnEnemy()` — Create a new enemy
- `spawnBoss()` — Transition to boss fight
- `useBomb()` — Smart bomb (clears bullets, damages all enemies)
- `addScore(base)` — Add points with chain multiplier
- `rectsOverlap(a, b)` — Axis-aligned bounding box collision check
- `ensureAudio()` — Lazy-init Web Audio context (required for browser audio policies)

## Browser Compatibility

The game uses standard HTML5 APIs and works on all modern browsers:
- Canvas 2D rendering (all browsers)
- Web Audio API (all except IE 11 and older)
- LocalStorage (all)
- Touch events (mobile)
- CSS clamp() (all modern browsers; falls back on older browsers)

Test on target devices/browsers if compatibility matters.

## Debug Tips

Check the browser console for:
- `frame` — Current animation frame count
- `score`, `lives`, `multiplier` — Current game state
- `enemies.length`, `bullets.length` — Active entity counts

Add `console.log()` statements directly in `update()` or `draw()` to trace game state each frame.

Use browser DevTools Performance tab to profile rendering and identify bottlenecks.
