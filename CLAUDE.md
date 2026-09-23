# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single-file HTML5 application. It runs entirely in the browser with no build process or external dependencies.

- **File**: `index.html` (1,669 lines)
- **Structure**: Single-page application with embedded HTML, CSS, and JavaScript
- **Runtime**: Modern browser with Canvas 2D API, Web Audio API, and Touch Events
- **Storage**: LocalStorage for high scores, mute state, difficulty, and motion preferences

## Getting Started

### Running the Game

The game runs directly in a browser:

```bash
# Open in default browser (or manually open index.html in any modern browser)
open index.html  # macOS
xdg-open index.html  # Linux
```

The game is fully playable immediately—no build, test, or dev server required.

### Quick Testing

To test the game:

1. Open `index.html` in a browser
2. Click "START" to begin
3. Use mouse or keyboard to play:
   - **Mouse**: Move cursor, click to fire
   - **Keyboard**: Arrow keys or WASD to move, Space to fire, B for bomb
   - **Touch**: Drag on left side to move, tap right side to fire, bomb button for bomb
4. Test all three difficulty levels via the UI buttons
5. Clear LocalStorage to test save/load behavior:
   ```javascript
   // In browser console:
   localStorage.clear();
   location.reload();
   ```

## Architecture

### Game Loop
The main game loop runs at 60 FPS via `requestAnimationFrame`:

```
loop(timestamp)
  ├─ trackPerf(dt)              // Adaptive quality system
  ├─ update(dt)                 // Update all game entities
  ├─ draw()                     // Render canvas
  └─ requestAnimationFrame()    // Schedule next frame
```

The loop caps frame delta at 50ms to prevent death spirals.

### Core Game State (line ~864-869)

```javascript
let player, bullets, enemies, enemyBullets, particles, powerUps, boss;
let score, lives, running, paused, gameOver;
let stage, stageKills, kills, chain, multiplier;
```

All game state is module-scoped to an IIFE, preventing global pollution.

### Update Loop (`update(dt)` at line ~1148)

The update phase processes:

1. **Player input**: Keyboard, mouse, joystick (touch)
2. **Player state**: Position, weapon cooldown, invulnerability timer
3. **Bullet updates**: Movement, collision with enemies
4. **Enemy spawning**: Difficulty-based wave spawning via `spawnTimer`
5. **Enemy updates**: Position, AI (weaving for interceptors), shooting
6. **Enemy bullet collision**: With player and bomb area
7. **Power-up collection**: Weapon upgrades, rapid-fire, bomb refills
8. **Boss logic**: Separate `updateBoss(dt)` handles boss AI
9. **Particle effects**: Explosions, bullet trails
10. **Score multiplier decay**: `chainTimer` drives combo decay

### Render Pass (`draw()` at line ~1520)

1. **Background**: Sky gradient (theme-dependent), scrolling clouds
2. **All entities**: Player, enemies, bullets, particles, power-ups, boss
3. **HUD**: Score, health hearts, bombs, weapon level, multiplier, boss health bar
4. **Effects**: Screen shake, vignette pulse, damage flash

### Difficulty System (line ~485)

Three difficulty tiers tuned via config objects:

```javascript
{
  spawnBase,       // Initial spawn delay (frames)
  spawnMin,        // Minimum spawn delay
  enemySpeed,      // Multiplier on enemy velocity
  enemyShootChance,// Probability enemy shoots each frame
  lives, bombs,    // Starting counts
  bossHpMul        // Boss health multiplier
}
```

Difficulty persists in LocalStorage and affects wave pacing, enemy density, and boss toughness.

### Audio System (line ~545–677)

Uses Web Audio API's built-in oscillator for *all* sounds:

- `beep(freq, dur, type, vol, opts)`: Sine/square waves for UI and weapon
- `noiseBurst(dur, vol, opts)`: Brown noise for explosions
- No external audio files; all synthesis is real-time

Audio context is lazy-initialized on first input (`ensureAudio()`) to comply with browser autoplay policies.

### Entity Types

**Player** (line ~877):
- Single instance at all times
- Properties: `x, y, w, h, speed, cooldown, rapid, weapon, level, invuln`
- Clamped to canvas bounds; moves via input or mouse target
- Weapon states: `'vulcan'` (spread shot) or `'laser'` (narrow laser)

**Enemies** (line ~502):
- Three types: `grunt`, `interceptor` (fast, weaving), `tank` (slow, durable)
- Each has `x, y, w, h, hp, color, speed, shootChance`
- Spawned based on `stageKills` counter; reset per stage at 18 kills
- Destroyed when `hp <= 0`

**Bosses** (line ~508):
- Six boss types: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Each stage after kill 18 spawns the stage-number mod 6 boss
- Complex draw function: `drawBossShip(b)` renders layered hull, wings, core
- Boss AI: `updateBoss(dt)` handles attack patterns, movement, health tracking

**Bullets** (player and enemy):
- Simple objects: `{ x, y, vx, vy, w, h, damage, isBoss }`
- Player bullets destroy on enemy hit; enemy bullets destroy on player hit

**Power-ups** (line ~1320):
- Spawn randomly from defeated enemies: weapon upgrade, rapid-fire, bomb
- Move downward; disappear if off-screen
- Collected on player collision

**Particles** (explosions, trails):
- Temporary visual effects: `{ x, y, vx, vy, life, maxLife, color, size }`
- Decay over time; removed when `life <= 0`

### Input Handling (line ~761–861)

**Keyboard** (line ~761):
- Arrow keys or WASD: Set `playerVel` to move player
- Space: `firing = true` (continuous fire)
- B: `useBomb()`

**Mouse** (line ~836):
- Move cursor over canvas: `mouseTarget` follows; player glides toward it
- Click: Fire (while over canvas)
- Leave canvas: `mouseActive` disabled; input falls back to keyboard

**Touch** (line ~774):
- Left zone (40vw): Virtual joystick for movement
- Right zone (fire button): Continuous fire
- Bomb button: Bomb

Joystick is a visual guide only—the `joyVec` input drives movement the same as keyboard.

### Collision Detection

- **Bullets vs. Enemies**: Distance check with tolerance (line ~1245)
- **Enemy Bullets vs. Player**: AABB check with player collision radius (line ~1267)
- **Bomb area**: Circular blast; all enemies within radius take damage (line ~1288)
- **Power-up collection**: Distance check (line ~1319)

### Scoring & Combos

- Base score = `enemy.score * difficulty` (line ~1234)
- Multiplier starts at 1; each kill extends chain timer
- Multiplier caps at 8x; decays when `chainTimer` expires (line ~869)
- Combo pop shows on kills to signal multiplier state

### Rendering Details

**Planes** (`drawPlane(x, y, w, h, color, darkColor, flame)` at line ~913):
- Drawn as isometric hexagons with flame animation
- Flame height controlled by `rapid` counter or weapon firing state

**Boss ship** (`drawBossShip(b)` at line ~970):
- Layered: hull top/bottom, wings, core
- Core pulses and glows; damaged core dims
- Rendered as complex path with gradients

**Clouds** (`drawClouds(speed, fillColor, rx, ry)` at line ~1507):
- Procedurally drawn circles in scrolling bands
- Parallax effect simulates depth

**Themes** (line ~493):
- Six visual themes (sky gradient, cloud color, sun color)
- Cycle per stage
- Persists across restarts until new game

### Adaptive Quality

Performance tracking (line ~476–483) auto-downgrades rendering quality if frame time average exceeds 1/40 FPS, or upgrades if it drops below 1/56 FPS. Quality levels:
- `0`: Full (all effects)
- `1`: Medium (reduced particle density)
- `2`: Low (minimal particles, no cloud detail)

### Accessibility

- **Reduced motion** toggle: Disables screen shake, vignette pulse, damage flash
- **Screen reader support**: `#srLive` element announces game events (pause, combo, stage)
- **High contrast UI**: Light text on dark semi-transparent backgrounds
- **Keyboard full support**: All controls available via keyboard

### Storage

LocalStorage keys:
- `skyfighter.bestScore`: High score (number)
- `skyfighter.muted`: Audio mute state ('0' or '1')
- `skyfighter.difficulty`: Last selected difficulty ('easy', 'normal', 'hard')
- `skyfighter.reducedMotion`: Motion preference ('0' or '1')

## Editing the Game

### Making a Simple Change (e.g., adjust enemy speed)

1. Open `index.html` in a text editor
2. Find the relevant constant (e.g., `DIFFICULTIES` at line 485)
3. Modify the value
4. Save and reload the browser tab (F5)

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` (line ~502)
2. Add spawn logic in `update()` where enemies are spawned (line ~1180)
3. Ensure `drawPlane()` or custom draw function handles rendering
4. Test in all difficulty levels

### Adding a Boss

1. Add entry to `BOSSES` array (line ~508)
2. Update `updateBoss()` to handle new boss AI/attacks
3. Update `drawBossShip()` to render new boss visually
4. Adjust boss health via `hpMul`

### Tweaking Weapon Balance

- **Weapon stats**: Find where bullets are created (line ~1135 for Vulcan, line ~1142 for Laser)
- **Fire rate**: Adjust `player.cooldown` reset values
- **Damage**: Change `damage` property on bullets

### Modifying Audio

Web Audio API synthesis is in the `sfx` object (line ~624). To adjust:
- Frequency: Change `freq` parameter
- Duration: Change `dur` parameter (seconds)
- Volume: Change `vol` parameter (0–1)
- Waveform: Change `type` ('sine', 'square', etc.)

## Known Patterns & Conventions

- **Entity arrays**: `bullets`, `enemies`, etc. are iterated and filtered in-place for simplicity
- **Lifecycle**: Entities are created, updated, and destroyed within a single update cycle
- **Colors**: Stored as CSS hex strings; used with `ctx.fillStyle` and `ctx.strokeStyle`
- **Timing**: All durations in seconds; frame-aware via `dt` parameter
- **Coordinates**: Canvas coordinates; (0,0) is top-left, Y increases downward
- **Drawing order**: Background → clouds → entities → particles → HUD

## Testing Notes

- **High score persistence**: Close and reopen the browser tab; high score should remain
- **Difficulty selection**: Change difficulty, start game, verify enemy density and boss health match expected tier
- **Reduced motion**: Enable toggle, verify no screen shake/vignette/flash effects
- **Touch controls**: Test on actual touch device or use browser DevTools device emulation
- **Audio**: Verify all sound effects play (weapon, bomb, explosion, UI beeps)
- **Boss battles**: Play through stage 1 to witness first boss encounter; verify boss health bar appears
- **Multiplier decay**: Kill enemies rapidly to build multiplier; wait and verify it decays

## Performance Considerations

- Canvas rendering is the bottleneck at high entity counts
- Adaptive quality system mitigates slowdowns by culling particles
- Touch joystick uses transform for smooth rendering (GPU-accelerated)
- Particle effects are pooled implicitly (no reallocation)
- Audio synthesis is lightweight (oscillator per sound, released immediately)
