# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game. Players pilot a fighter jet through stages, collecting weapon power-ups, deploying smart bombs, and engaging boss squadrons. The entire game is implemented in a single HTML file with inline CSS and JavaScript (~1670 lines).

**Key Features:**
- Multiple difficulty levels (Easy, Normal, Hard) with adaptive spawning and enemy behavior
- Three destructible enemy types (Grunt, Tank, Interceptor) that appear as stages progress
- Two weapon systems: Vulcan (spread shots) and Laser (piercing shots) with 3 power levels each
- 6 unique boss designs across stages with varied attack patterns
- Smart bomb mechanic that clears all enemies and damages the current boss
- Dynamic kill-streak multiplier system for score chaining
- Visual feedback: screen shake, vignette flashing, combo notifications
- Full mobile support: touch controls with joystick input and button zones
- Accessibility: reduced-motion preference support, screen-reader announcements
- 6 procedural sky themes that cycle per stage
- Adaptive quality scaling based on frame time performance
- Sound effects using Web Audio API (synthesized audio, no external files)

## Architecture

### Core Game Loop & State

The game runs an `requestAnimationFrame` loop that calls `update(dt)` then `draw()` each frame. Game state is managed through:
- **Player state**: position, health, weapon type/level, invulnerability timer, fire cooldown
- **Actors**: arrays of `enemies[]`, `bullets[]`, `enemyBullets[]`, `powerUps[]`, `particles[]`
- **Boss**: single object when active, manages health, attack patterns, telegraphing
- **Game flags**: `running`, `paused`, `gameOver`, `difficulty`, `stage`, `score`, `chain` multiplier

**Storage:** Best score, mute state, difficulty preference, reduced-motion setting use `localStorage`.

### Content Data (Game Parameters)

These are defined as const objects at the top of the script and are the primary extension points:

- **`DIFFICULTIES`** (easy/normal/hard): Spawn rates, enemy speed, shoot chance, lives/bombs, boss HP multiplier
- **`THEMES`**: 6 sky gradients, cloud/sun colors, theme names cycling by stage
- **`ENEMY_TYPES`** (grunt/tank/interceptor): Speed multiplier, HP, score, fire pattern, appearance, min stage
- **`BOSSES`** (6 unique): Design colors, HP multiplier, attack phases with timing
- **`KILLS_PER_STAGE`** (18): Determines when to spawn the boss

### Rendering

Canvas 2D API handles all rendering:
- **`drawPlane()`**: Draws player and enemy fighters
- **`drawBossShip()`**: Renders boss with animated parts
- Bullets and particles rendered as simple shapes
- HUD overlays (score, health, weapon, multiplier) rendered on top canvas
- Adaptive quality: particle count and shadow effects scale down under performance strain

### Input Handling

**Keyboard:** Arrow keys + WASD for movement, Space for fire, X/Shift for bomb, P/Esc for pause
**Mouse:** Movement steers via target lerp, click to fire
**Touch:** Left zone (55% width) for drag-to-move joystick, right zone (45% width) with FIRE/BOMB buttons

The input system uses:
- `keys[]` object tracking held keys
- `mouseActive` / `mouseTarget` for mouse steering
- `joyActive` / `joyVec` for touch joystick
- `firing` flag for click/tap fire
- Input handling is disabled in overlay (start/game-over screens)

### Physics & Collision

Collision detection uses `rectsOverlap()` (AABB). No physics engine; movement is direct position updates:
- Bullets travel straight or angled (vx, vy); removed when off-screen
- Enemies weave in a sine pattern (controlled by `weaveSeed`) or move straight
- Boss moves left/right while entering, then executes attack phases
- Power-ups fall straight down
- Particles drift with velocity, fade by lifetime

### Sound

Web Audio API synthesizes all sound effects (no audio files):
- `sfx.shoot()` / `sfx.laser()`: High-frequency beeps
- `sfx.bomb()`: Pitched noise burst
- `sfx.hit()` / `sfx.bosshit()`: Lower beep
- `sfx.kill()`: Rising-pitch beep
- `sfx.powerup()`: Arpeggio
- `sfx.bossdefeated()`: Victory sound
Mute state persists in localStorage and bypasses all `sfx.*()` calls.

### UI & Overlays

- **Overlay system**: Shows title screen, pause screen, and game-over screen with final stats and difficulty selector
- **HUD**: Real-time display of score, best, multiplier, weapon, health (hearts), bombs
- **Accessibility**: Screen-reader announcements via `#srLive` role="status" (announces stage/score changes)
- **Visual effects**: Vignette red flash on damage, white screen flash on bomb, combo popups, stage banners

## Running the Game

### Play Locally
Open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge). No build step required.

**URLs:** 
- File protocol: `file:///path/to/index.html`
- HTTP server: `python3 -m http.server 8000` then `http://localhost:8000`

### Playtesting Checklist
- [ ] Start game on each difficulty; verify spawning rate and enemy speed feel correct
- [ ] Test weapon power-up progression: Vulcan → Laser, level 1 → 3
- [ ] Verify bomb clears all enemies and damages boss
- [ ] Check score multiplier: kill streak should chain, breaks after ~2 seconds idle
- [ ] Confirm boss appears at 18 kills, advances to next stage after boss defeat
- [ ] Test each of 6 boss patterns; verify telegraph effect (bright core) before attacks
- [ ] Verify health/life system: taking damage costs 1 life, game over at 0
- [ ] Confirm reduced-motion preference disables animations
- [ ] Test mobile touch controls: joystick movement, fire/bomb buttons
- [ ] Verify best score persists in localStorage across page reloads

## Development Guidelines

### Modifying Difficulty, Spawn Rates, or Boss Parameters
Edit `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or `KILLS_PER_STAGE` constants. These are order-independent and directly parameterize the game:
```js
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, ... },
  ...
}
```
Rebalance `spawnBase` (higher = fewer/slower spawns) or `enemySpeed` to tune difficulty. Boss HP scaling via `bossHpMul`.

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES`:
```js
newtype: { w: 28, h: 24, hp: 45, speedMul: 1.3, shootChance: 0.6, weave: true, color: '#00ff00', dark: '#008800', score: 140, minStage: 4 }
```
2. Add the type key to the spawn filter in `spawnEnemy()` (already generic; no change needed)
3. `drawPlane()` renders all enemy types identically; customize by passing different color values

### Adding a New Boss
1. Add entry to `BOSSES` with design colors and attack phases:
```js
{ name: 'Phoenix', wing: '#ff6600', hullTop: '#ff8833', hullBottom: '#8800ff', core: '#ffdd00', hpMul: 1.1 }
```
2. Attacks are hardcoded in the boss update loop (`attackTimer`, `patternIndex`). Search `if (boss && boss.attacking)` and add new pattern branches
3. `drawBossShip()` uses `design` colors but shape is fixed; customize colors only without changing shape

### Changing Themes or Sky Colors
Edit the `THEMES` array. Each theme is a gradual sky background, cloud color, and sun color. Themes cycle per stage:
```js
{ sky: ['#0d2f5e', '#1e6fd8', '#7fc1ff'], cloud: '255,255,255', sun: '255,255,220', name: 'Daylight Front' }
```

### Performance Tuning
The `quality` variable auto-scales based on average frame time:
- Quality 0 (full): All particles, shadows, vignette effects
- Quality 1 (medium): 60% particle count, reduced shadows
- Quality 2 (low): 35% particles, no shadows

Adjust particle count in `spawnExplosion()` or shadow blur in `drawBossShip()` if needed on slow devices.

### Adding Features
**General rule**: Keep additions in `index.html`; no new files. For larger features, isolate in a section comment block and use clear naming.

Common extensions:
- **New weapon type**: Add to weapon selection in `fireWeapon()`, define spread/pierce/speed
- **New visual effect**: Add animation to CSS or draw call; use `quality` flag for performance fallback
- **New scoring rule**: Modify `addScore()` logic or create multiplier tiers
- **Pause/resume state**: Already supported; extend `paused` flag logic
- **Touch mobile refinement**: Adjust zone sizes in `#moveZone`, `#actionZone` CSS or joystick responsiveness in touch handlers

### Debugging
- Open browser DevTools (F12) → Console to check for errors
- Use `console.log()` in `update()` or draw functions; avoid logging every frame (spam)
- Check `localStorage` values: `localStorage.getItem('skyfighter.bestScore')`
- Throttle frame rate in DevTools → Performance tab to test low-end devices
- Test with `prefers-reduced-motion: reduce` in DevTools → Rendering to verify accessibility

## Key Code Sections

| Line Range | Purpose |
|-----------|---------|
| 1–180 | CSS styles (HUD, overlay, animations, mobile layout) |
| 181–390 | Game constants (DIFFICULTIES, THEMES, ENEMY_TYPES, BOSSES) |
| 391–700 | Initialization, DOM setup, input handlers, event listeners |
| 701–900 | Drawing functions (drawPlane, drawBossShip) and score/life rendering |
| 901–1100 | Game logic: enemy spawn, boss spawn, collision, bomb, weapon fire |
| 1101–1400 | Main update loop: movement, bullet physics, enemy AI, boss phases |
| 1401–1550 | Boss attack patterns and phase management |
| 1551–1669 | Particle effects, sound synthesis, game loop (requestAnimationFrame) |

## Known Limitations & Future Work

- **Boss variety**: All 6 bosses share the same ship silhouette; only colors differ. Future: unique ship shapes per boss
- **Enemy patterns**: Enemies use simple sine weaving or straight movement; no formation flying or coordinated attacks
- **Procedural generation**: Stages are identical except spawn rate. Future: stage-specific layouts or hazards
- **Replay system**: No recording/playback; scores are append-only
- **Pause mid-game**: Pause works but shows the overlay; no persistent pause UI state
- **Audio customization**: Sound design is baked in; no volume slider (only all-or-nothing mute)

## Mobile & Accessibility Notes

- Game scales to 100% width/height with `max-width: 100%; max-height: 100%;` on canvas
- Safe-area insets respected: `env(safe-area-inset-top)` used for HUD positioning on notched devices
- Touch `touch-action: none` prevents default browser panning on the game canvas
- Screen-reader announcements use `#srLive` with `role="status"` aria-live="polite"
- Reduced-motion preference (`prefers-reduced-motion: reduce`) disables animations and screen shake
