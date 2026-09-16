# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game built as a single HTML5 file with vanilla JavaScript (no frameworks). The entire game runs in the browser with Canvas rendering and Web Audio API for sound.

## Quick Start

- **Run the game**: Open `index.html` in a web browser. No build step, no dependencies.
- **Development**: Edit `index.html` directly. Changes are live on refresh.

## Architecture

The game is a self-contained IIFE (Immediately Invoked Function Expression) in `index.html:422-1666`. The architecture follows a classic game loop pattern:

### Core Game Loop Flow
1. **Input handling** (keyboard, mouse, touch) → updates `keys` object and player position/firing state
2. **Update phase** (`update(dt)`) → moves entities, checks collisions, updates game state
3. **Draw phase** (`draw()`) → renders all entities to canvas
4. **Loop** (`requestAnimationFrame`) → runs at ~60fps

### Key Data Structures

**Player** (`player` object):
```
{ x, y, w, h, speed, cooldown, weapon, level, invuln, rapid }
```
- `weapon`: 'vulcan' (spread) or 'laser' (piercing)
- `level`: 1–3, affects spread/pierce count
- `invuln`: invincibility timer after taking damage

**Enemies** (array of objects):
```
{ x, y, w, h, type, hp, speed, canShoot, pattern, weave, weaveSeed, ... }
```
- `type`: 'grunt', 'interceptor' (fast), or 'tank' (tanky, appears stage 2+)
- `pattern`: 'spread' (wide fan) or 'aimed' (tracks player)
- `weave`: interceptors weave side-to-side

**Bosses** (one object at a time):
```
{ x, y, w, h, hp, maxHp, design, vx, entering, attackTimer, telegraph, patternIndex, ... }
```
- `design`: selected from `BOSSES` array (colors and hpMul)
- `telegraph`: countdown before firing; visual telegraph effect
- `patternIndex`: cycles through attack patterns in `BOSS_PATTERNS`

**Bullets** (player's):
```
{ x, y, w, h, speed, vx, pierce, laser }
```
- `pierce`: (laser only) how many enemies it can penetrate; decrements on hit

**Enemy Bullets**:
```
{ x, y, w, h, speed, vx, radial, dx, dy }
```
- `radial`: if true, uses `dx`/`dy` velocity instead of `vx` for 360° patterns

**Power-ups**:
```
{ x, y, w, h, speed, type }
```
- `type`: 'life', 'bomb', or 'weapon'

### Game Flow Stages

1. **Startup**: `resetGame()` → initializes player, resets enemies/bullets, sets lives/bombs
2. **Normal waves**: `spawnEnemy()` spawns enemies at intervals; player shoots; chain multiplier builds
3. **Stage progression**: After `KILLS_PER_STAGE` (18) enemies killed, boss is spawned
4. **Boss phase**: Boss appears with telegraph attacks; uses pattern rotation
5. **Stage clear**: Boss dies → `killBoss()` → advance stage, reset enemies, repeat
6. **Game over**: Lives reach 0 → `endGame()` → show score/best, return to menu overlay

### Collision System

`rectsOverlap(a, b)` → axis-aligned bounding box check (AABB), simple magnitude-based center distance:
```js
Math.abs(a.x - b.x) < (a.w + b.w) / 2 && Math.abs(a.y - b.y) < (a.h + b.h) / 2
```

Checked in `update()`:
- Player bullets ↔ enemies → damage enemy, consume bullet (or pierce)
- Player bullets ↔ boss → damage boss
- Enemies/bullets ↔ player → lose life, trigger invuln
- Power-ups ↔ player → apply effect

### Combat & Progression

**Weapons**:
- **Vulcan** (default): rapid fire, spread increases with level (1x, 2x, 3x spread)
- **Laser**: slower fire rate but pierces enemies; multi-lane (level 2+)
- Toggle via weapon power-up cycle: upgrade level 1→3, then switch

**Scoring**:
- Base score per enemy type: grunt 10, interceptor 15, tank 35
- Multiplied by combo: `multiplier = 1 + floor(chain / 5)` (max 10x)
- `chainTimer` resets if 2.2s passes without a kill
- Boss kill: +500 points

**Difficulties** (`DIFFICULTIES` object):
- **Easy**: slower spawns, slower enemies, fewer bombs, 4 lives, boss 0.8× HP
- **Normal**: balanced
- **Hard**: faster spawn/enemies, fewer bombs, 1.3× boss HP

### Visual Systems

**Theme rotation** (`THEMES[stage % 6]`):
- Each stage cycles through 6 themes (Daylight, Sunset, Night, Storm, Arctic, Volcanic)
- Controls sky gradient, cloud color, sun color

**Particles** (explosions):
- Spawned by `spawnExplosion(x, y, color, count)`
- Radiate outward; fade and shrink; deleted when `life <= 0`

**Screen effects**:
- **Screen shake**: `screenShake(mag, dur)` → random jitter; reduced if `reducedMotion` enabled
- **Flash**: brief white flash on hit; animations disabled if `reducedMotion` on
- **Vignette**: red pulse when health = 1 (critical)
- **Combo pop**: large text "X KILLS!" animates in center

**Drawing pipeline** (`draw()`):
1. Clear canvas
2. Apply screen shake (random translate)
3. Draw sky gradient + sun glow
4. Draw cloud layers (2× passes at different speeds/alpha)
5. Draw player (with flame if moving)
6. Draw bullets + enemy bullets
7. Draw enemies + tank health bars
8. Draw boss (if present)
9. Draw power-ups (rotating, glowing)
10. Draw particles (explosions)

### Boss Attack Patterns

`BOSS_PATTERNS[designIndex]` is an array of 3 attack functions per boss (rotated via `patternIndex`):
- **Radial**: bullets fire in a circle
- **Aimed spread**: fan of bullets tracking the player with spread
- **Columns**: vertical lines of bullets
- **Spiral burst**: expanding spiral
- **Cross**: 8-way cardinal + diagonal
- **Homing**: seeded spread toward player with random offset
- **Double ring**: two concentric circles at different speeds
- **Teleport**: boss jumps to new X, fires
- **Spread combo**: dual aimed spreads offset

Telegraph system: `telegraph` timer counts down; when it reaches 0, fire attack and reset `attackTimer`.

### Input & Control

**Keyboard**:
- Arrow keys or WASD → move
- Space or mouse click → fire
- P or Esc → pause
- X or Shift → use bomb

**Mouse**: Move cursor to steer (smooth glide), click to fire

**Touch**:
- Left half: drag to move (shows joystick overlay)
- Right half: FIRE and BOMB buttons (large circular buttons)

Priority: keyboard/joystick > mouse (mouse only used if neither keyboard nor joystick active)

### UI & State Persistence

**HUD** (top center):
- Score, best score, multiplier, weapon name/level, hearts, bombs

**Top right buttons**:
- Mute (🔊/🔇), toggle reduced motion (🎆/🚫), pause (⏸/▶)

**Overlay** (modal):
- Game start screen (difficulty selector, play button, controls hint)
- Pause screen (same as start, shows "Paused", resume button)
- Game over screen (final score, stage, best score, "new best" indicator)

**Persistence** (localStorage):
- `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`

### Audio System

**WebAudio synth** (no external files):
- `beep(freq, dur, type, vol, opts)` → oscillator with optional sweep, filter, pan
- `noiseBurst(dur, vol, opts)` → noise for explosions/impacts
- `sfx` object: named sound effects (shoot, laser, explode, bomb, hit, powerup, bossHit, etc.)

Each sound is stateless; no sound pool or channel limit. If muted or `audioCtx` unavailable, all sfx no-op.

### Accessibility

- **Screen reader support**: `announce(text)` updates `#srLive` with status updates
- **Reduced motion**: `reducedMotion` toggle disables animations; animations use `@media (prefers-reduced-motion: reduce)` in CSS
- **Semantic HTML**: `role="status"` on SR live region
- **Keyboard controls**: full game playable without mouse/touch
- **Color contrast**: high contrast text/UI for legibility

### Performance Optimization

**Adaptive quality** (`quality` variable: 0 = full, 1 = medium, 2 = low):
- Tracks frame time over 45 frames
- If avg > 1/40s, bump quality down (fewer particles, no glow, reduced shadows)
- If avg < 1/56s, bump quality up
- Used in: particle count, shadow rendering, glow effects

**Arrays**: enemies, bullets, particles managed with filter + splice for removal (not pooling, but acceptable for small counts)

**Canvas**: Fixed 480×720 internal resolution, scaled with CSS (responsive scaling)

## Modifying the Game

### Adding a new enemy type:
1. Add entry to `ENEMY_TYPES` object (w, h, hp, color, dark, score, minStage)
2. Update `spawnEnemy()` filter to include it when `stage >= minStage`
3. Add to EXPLODE_SFX if custom explosion sound needed

### Adding a new boss:
1. Add entry to `BOSSES` array (name, colors, hpMul)
2. Add corresponding attack pattern array to `BOSS_PATTERNS`
3. Boss auto-selects by `(stage - 1) % BOSSES.length`

### Adding a new weapon:
1. Add `weapon: 'name'` option to weapon cycle in `applyPowerUp()`
2. Add firing logic to `fireWeapon()`
3. Update HUD display in `weaponLabel()`

### Changing difficulty numbers:
Edit `DIFFICULTIES` object (spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul)

### Tweaking visuals:
- **Colors**: Edit CSS variables or hardcoded hex values throughout
- **Themes**: Modify `THEMES` array (sky gradients, cloud/sun colors)
- **Animation timing**: Adjust keyframe percentages or duration literals
- **Canvas size**: Change `BASE_W`/`BASE_H` (affects aspect ratio and coordinate system)

## Testing Considerations

**No automated tests** — verify by playing:
1. Launch with each difficulty
2. Test all weapon progressions (Vulcan 1–3, switch to Laser, back to Vulcan)
3. Verify boss spawn after stage kill threshold
4. Test all three boss attacks per boss
5. Verify bomb clears all enemies + damages boss
6. Check score multiplier resets on death
7. Test pause/resume
8. Test touch controls on mobile

**Performance**: Check framerate under heavy particle load (use DevTools Performance tab)

**Audio**: Mute toggle, check audio context initialization on first input
