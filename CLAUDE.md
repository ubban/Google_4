# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely as a single HTML5 file with no external dependencies. The entire game—rendering, audio synthesis, game logic, input handling, and UI—is self-contained in `index.html`.

## Architecture

### Single-File Structure
The entire game is in `index.html` (~1670 lines) consisting of:
- **HTML**: Canvas element, HUD overlays, controls, touch UI
- **CSS**: Responsive styling, animations (screen shake, combo popups, transitions)
- **JavaScript**: ~1300 lines of vanilla game engine code

### Core Game Systems

#### 1. **Rendering Pipeline** (Canvas 2D)
- **Main loop**: `loop(now)` → tracks delta time, calls `update()` then `draw()`
- **Draw order**: Sky gradient + sun glow → clouds → player → bullets → enemies → boss → enemy bullets → powerups → particles
- **Performance adaptation**: `quality` variable (0=full, 1=medium, 2=low) scales particle counts and shadow effects based on frame timing
- **Screen shake**: Applied via `ctx.translate()` when taking damage or using bombs
- **Theme system**: 6 alternating sky themes as you progress through stages

#### 2. **Game Loop & State** (Update 60fps)
- **Input priority**: Keyboard/joystick > Mouse aiming (mouse is fallback for steering)
- **Entity arrays**: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- **Collision detection**: Simple AABB via `rectsOverlap()` checks enemy-to-bullet, bullet-to-boss, enemy-to-player, etc.
- **Stage progression**: Kill 18 enemies (`KILLS_PER_STAGE`) to trigger boss encounter; defeat boss to advance stage and reset spawn rate

#### 3. **Player Mechanics**
- **Movement**: Position clamped to canvas bounds; speed is 260 units/sec
- **Weapons**: Vulcan (spread bullets, faster) or Laser (piercing, slower spawn); each has 3 upgrade levels
- **Power-ups**: Life (6% drop), Bomb (8% drop), Weapon (20% drop)
- **Invulnerability**: 1.5 sec after being hit
- **Bomb**: Clears all enemy bullets + all on-screen enemies + deals fixed damage to boss

#### 4. **Enemy System**
- **Types**: Grunt (fast, 1 HP), Interceptor (very fast, 1 HP, weaves), Tank (slow, 4 HP, appears stage 2+)
- **Spawning**: Timer-based; spawn rate accelerates with frame count (difficulty ramp)
- **AI**: 
  - Movement: Linear or weaving sine pattern based on type
  - Shooting: Timer-based; 45% (easy) to 95% (hard) chance to fire; either spread or aimed at player
  - Cleanup: Removed when off-screen
- **Visual**: Drawn as procedural planes with color and shading

#### 5. **Boss Encounters**
- **Design**: 6 unique boss templates (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) cycle by stage
- **Behavior**: Enters from top, bounces horizontally, follows attack patterns with telegraph phase (shows warning)
- **Attack patterns**: Each boss has 3 unique patterns (radial bursts, aimed spreads, teleport attacks, etc.); rotate through during fight
- **HP scaling**: Base 70 + 35×stage, multiplied by difficulty and boss design multiplier
- **Visual**: Procedural design with hull, wings, and glowing core; telegraphing pulses when about to attack
- **Defeat**: 500 point bonus, stage advance, chain reset

#### 6. **Audio System** (Web Audio Synthesis)
- **No external files**: All sounds generated procedurally
- **Synth approach**: Oscillators (sine, square, sawtooth) + noise bursts with filters, envelopes, panning
- **Effect library**: `sfx` object maps actions to generator functions
- **Master gain**: 0.9 with dynamics compressor for consistency
- **Mute state**: Persisted to localStorage; mute check in every beep/noiseBurst

#### 7. **Input Handling**
- **Keyboard**: Arrow keys / WASD for movement, Space to fire, X / Shift for bomb, P / Esc to pause
- **Mouse**: Position tracked passively; pressing fire button or moving over canvas enables steering
- **Touch**: 
  - Left half: Joystick (drag to move)
  - Right half: FIRE and BOMB buttons
  - Joystick visualized with base ring and draggable stick
- **Accessibility**: Screen-reader announcements via `srLive` element

#### 8. **Game State & Persistence**
- **Stored in localStorage** (keys prefixed `skyfighter.`):
  - `bestScore`: High score across sessions
  - `muted`: Audio mute toggle
  - `difficulty`: Last selected difficulty (easy/normal/hard)
  - `reducedMotion`: User preference (respects system `prefers-reduced-motion`)
- **Difficulty multipliers**: Each preset (easy/normal/hard) scales spawn rates, enemy speed, shoot chance, lives, bombs, and boss HP

#### 9. **Combo & Multiplier System**
- **Chain**: Increments on each kill, resets if 2.2 sec elapses without a kill
- **Multiplier**: 1 + floor(chain/5), capped at 10x
- **Visual feedback**: "COMBO" popup and multiplier display in HUD
- **Score formula**: `baseScore × multiplier`

#### 10. **Particle & Effect System**
- **Explosion particles**: Random velocity bursts; fade out over 0.5 sec
- **Count adaptation**: Scaled by quality setting (quality 2 = 35% of full particles)
- **Combo/stage popups**: CSS animations with scale and fade
- **Flash effects**: Screen white flashes on hit or bomb (CSS animation)
- **Vignette pulse**: Red edge vignette when health is critical (1 life)

### Key Algorithms & Constants

#### Difficulty Configuration
```javascript
{
  spawnBase: base spawn delay,
  spawnMin: minimum delay (prevents infinite spawn),
  enemySpeed: multiplier on enemy base speed,
  enemyShootChance: probability enemy fires,
  lives: starting HP,
  bombs: starting bombs,
  bossHpMul: boss HP multiplier
}
```

#### Collision Detection
Simple axis-aligned bounding box overlap: entities are rectangles (x, y, w, h); overlap if `abs(dx) < (w1+w2)/2 && abs(dy) < (h1+h2)/2`.

#### Boss Attack Patterns
Each boss has a `BOSS_PATTERNS` array with 3 functions. During telegraph phase, a pattern fires enemy bullets in configured formation (radial ring, aimed spread, homing, cross, etc.). Patterns rotate on each attack cycle.

## Common Development Tasks

### Running the Game
1. Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge)
2. No build step required; no server needed (runs locally as a file: URL)
3. Responsive to window resize and orientation change (calls `resize()`)

### Testing Changes
- **Desktop**: Browser DevTools (F12) for console, breakpoints, network tab
- **Mobile**: Use browser's remote debugging (Chrome: chrome://inspect) or test in device browser directly
- **Performance**: Monitor `frame` counter in console; check `quality` variable (logs frame timing)

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` constant with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Update `spawnEnemy()` to include in random choice logic
3. Add visual rendering in `draw()` (enemy drawing delegates to `drawPlane()`)
4. Add explosion sound mapping in `EXPLODE_SFX` if needed
5. Update `KILLS_PER_STAGE` or spawn rates if balance changes

### Adding a New Boss
1. Add design template to `BOSSES` array (name, color scheme, HP multiplier)
2. Create attack patterns in `BOSS_PATTERNS` array (3 pattern functions per boss)
3. Boss index maps automatically: `BOSSES[(stage-1) % BOSSES.length]`
4. Adjust `spawnBoss()` base HP formula if needed
5. Test difficulty scaling and player feedback

### Modifying Weapon Behavior
- **Vulcan spread**: Edit spread angles in `fireWeapon()` 
- **Laser pierce**: Change `pierce` value on bullet; bullets with `laser: true` don't despawn until pierce ≤ 0
- **Fire rate**: Adjust `fireDelay` in `update()`
- **Speed**: Modify `speed` property on bullet object

### Adjusting Difficulty
Edit `DIFFICULTIES` object constants; changes apply on game restart.

### Adding Visual Polish
- **Animations**: CSS animations in `<style>` (keyframes for combo pop, stage banner, card entry, etc.)
- **Screen effects**: `screenShake()` for magnitude/duration; `flashHit()` / `flashBomb()` for flash animations
- **Glow/shadows**: Toggle with `quality` setting; `ctx.shadowColor` and `ctx.shadowBlur` in `draw()`

## Code Patterns

### Entity Updates
Entities are objects with position (x, y), size (w, h), and type-specific properties. Update loop iterates arrays and applies physics, then cleans up off-screen entities.

### Collision Response
After detecting overlap, specific logic applies: bullets removed or pierce reduced, enemies spawn explosion and grant score, player loses life, etc. Boss uses same pattern but with different damage rules.

### Audio Synthesis
All sound is procedural: call `beep()` with frequency, duration, waveform; or `noiseBurst()` for percussive sounds. Options object allows sweep, filter, pan, and delay for richer timbres.

### Canvas Transform Stack
Heavy use of `ctx.save()` / `ctx.restore()` to manage translation, rotation, alpha, and shadow settings without affecting other draws.

## Performance Considerations

- **Adaptive quality**: Frame timing tracked in `frameTimes`; quality downscales particle count and disables shadow effects under load
- **Object pooling**: Not used; entities are created/destroyed freely (small counts allow this)
- **Collision optimization**: Nested loops are acceptable given small entity counts (~20 enemies max)
- **Canvas redraw**: Full screen clear every frame (no dirty-rect optimization needed for 60fps on modern hardware)
- **Audio**: Oscillators are short-lived; master gain + compressor prevents clipping

## Testing Checklist

- [ ] Game starts and displays correctly on desktop and mobile
- [ ] Keyboard input (arrows, space, X) works for movement/fire/bomb
- [ ] Mouse steering works when hovering canvas
- [ ] Touch joystick appears on mobile and responds to drag
- [ ] Difficulty selection saves and applies on restart
- [ ] Mute/motion toggle persists across sessions
- [ ] Boss encounter triggers after 18 kills
- [ ] All 6 bosses appear and cycle correctly across stages
- [ ] Weapon power-ups cycle (vulcan → laser, upgrade levels)
- [ ] Screen shake and flash effects work (test with reduced-motion OFF)
- [ ] Combo multiplier increments and displays correctly
- [ ] High score updates and persists
- [ ] Game over screen shows stats and allows restart
