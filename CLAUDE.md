# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. It's a self-contained HTML5 canvas game delivered as a single `index.html` file with no external dependencies. The game features procedural audio synthesis (WebAudio API), multiple difficulty levels, boss encounters, weapon progression, and mobile/desktop controls.

## Running and Testing the Game

- **Play the game**: Open `index.html` in any modern web browser (Firefox, Chrome, Safari, Edge)
- **Development server** (optional): Any local HTTP server works: `python -m http.server 8000` then visit `http://localhost:8000`
- **Mobile testing**: Open on a phone or tablet; the game auto-detects touch and shows on-screen buttons and joystick

No build step, compilation, or dependencies required.

## Code Architecture

The entire game logic is in `<script>` at the end of `index.html` (lines 422–1666). It uses a single IIFE (Immediately Invoked Function Expression) to keep everything in one scope.

### Core Game Loop

```
requestAnimationFrame(loop)
  → update(dt)        // physics, collisions, entity behavior
  → draw()            // render to canvas
  → trackPerf(dt)     // adaptive quality scaling
```

### Key Systems

#### 1. **Canvas & Rendering** (lines 752–1638)
- Canvas base: 480x720 (desktop), scales responsively
- Drawing order: sky gradient → clouds → player → bullets → enemies → boss → particles
- Adaptive quality: reduces particle count on low-end devices based on frame time tracking
- Screen shake & visual effects (vignette, flash) disabled if user prefers reduced motion

#### 2. **Input Handling** (lines 767–861)
- **Keyboard**: Arrow keys/WASD to move, Space/Click to fire, X/Shift to drop bomb, P/Esc to pause
- **Mouse**: Hover to aim (plane glides to pointer), click to fire
- **Touch**: Left zone = draggable joystick, right zone = FIRE and BOMB buttons
- Keyboard input takes priority over mouse aiming when active

#### 3. **Audio System** (lines 542–676)
- WebAudio API synth (no external files)
- Functions: `beep(freq, dur, type, vol, opts)` for oscillator-based sounds
- `noiseBurst(dur, vol, opts)` for white noise with filters
- Synth effects: frequency sweeps, lowpass/highpass filters, stereo panning
- Sound effects object `sfx` with pre-configured effects for: shoot, laser, explosions, bomb, powerup, hits, boss sounds
- Master gain + compressor for clean audio

#### 4. **Game State** (lines 863–910)
- Player: position, weapon (vulcan/laser), level (1–3), invulnerability, cooldown
- Enemies: velocity, firing patterns (spread/aimed), health (hp system for tanks)
- Boss: health bar, pattern index, telegraph warning, special attack phases
- Scoring: base points × multiplier (chain-based)
- Lives: difficulty-dependent (easy=4, normal/hard=3)
- Bombs: limited stock, bomb uses clear all enemy bullets + damage boss

#### 5. **Collision Detection** (lines 1069–1282)
- Simple AABB (axis-aligned bounding box): `rectsOverlap(a, b)` checks all overlaps
- Bullet–enemy collisions trigger damage and explosions
- Bullet–boss collisions trigger boss damage
- Enemy/boss–player collision triggers invulnerability period and life loss
- Powerup pickup on overlap

#### 6. **Difficulty System** (lines 485–489)
- Three presets: `easy`, `normal`, `hard`
- Parameters: spawn rate, enemy speed, enemy firing chance, starting lives, bombs, boss HP multiplier
- Selected difficulty persists in `localStorage`

#### 7. **Weapon System** (lines 1124–1146, 1304–1322)
- **Vulcan**: spread shot that improves with level (level 1: 1 bullet, level 2: 2 bullets, level 3: 3 bullets)
- **Laser**: piercing shots that improve spread (level 1: center only, level 2: dual-lane, level 3: triple with center)
- Weapon pickups toggle between vulcan/laser and reset level to 1
- Firing cooldown: 0.15s for vulcan, 0.1s for laser

#### 8. **Enemy System** (lines 502–1041)
- **Three types**: grunt (fast, 1 hp), interceptor (weaving, fast, 1 hp), tank (slow, 4 hp)
- Spawn patterns: random-walk + optional weaving (interceptor only)
- Fire patterns: spread (arc) or aimed (toward player)
- Difficulty gates: tanks only appear stage 2+
- Stage progression: every 18 kills triggers stage boss

#### 9. **Boss System** (lines 508–515, 1043–1409)
- Six boss designs with unique visuals and attack patterns
- Boss cycle repeats: stage N uses boss N mod 6
- Each boss telegraphs before attack (22-frame warning flash)
- Boss patterns stored in `BOSS_PATTERNS` array (6 entries, 3 patterns each):
  - **VANGUARD**: radial burst, aimed fan, wide V-fan
  - **BEHEMOTH**: heavy columns, homing spread, dense radial
  - **SPECTER**: teleport + radial, spiral, dual aimed shots
  - **DREADNOUGHT**: 8-way cross, homing missiles, radial wall
  - **WRAITH**: teleport barrage, double ring, cross-spread
  - **TITAN**: heavy columns, full ring, relentless homing
- Boss enters from top, bounces side-to-side during fight
- Stage progression on boss defeat + score reward (500 points)

#### 10. **Visual Themes** (lines 493–500)
- Six rotating themes with unique sky gradients and cloud colors
- Theme cycles per stage: stage N uses theme N mod 6
- Themes are purely visual; no gameplay impact

#### 11. **Particle System** (lines 1073–1085, 1296–1297)
- Simple particles: position, velocity, life timer
- Explosions spawn varying particle counts based on quality setting
- Particles fade over time

#### 12. **HUD & UI** (lines 367–419, HTML section)
- Score, best score, multiplier, weapon name/level, hearts, bomb count (top)
- Boss health bar (appears when boss spawns)
- Stage banner and combo popups (center screen)
- Vignette red flash on 1 life remaining
- Full-screen overlay for menu, pause, game over

#### 13. **Accessibility**
- Reduced motion preference respected (`prefers-reduced-motion` media query)
- Screen reader announcements via `#srLive` (aria-live region)
- Button titles and semantic HTML
- Safe-area insets for notched phones

### State Variables to Know

- `running`, `paused`, `gameOver`: game state flags
- `frame`: global frame counter (increments each update)
- `chain`, `chainTimer`, `multiplier`: combo system (resets on gap > 2.2s)
- `quality`: 0=full, 1=medium, 2=low (adaptive; scales particles and shadow blur)
- `stage`, `stageKills`, `kills`: progression tracking
- `shakeTime`, `shakeMag`: screen shake parameters (reduced if reduced-motion enabled)

## Common Development Tasks

### Add a New Enemy Type

1. Define in `ENEMY_TYPES` object (line 502), with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Add spawn logic to `spawnEnemy()` (line 1022); use `stage` to gate appearance
3. Add explosion sound effect to `EXPLODE_SFX` map (line 678) if unique sound needed
4. Test: run game, trigger stage with new enemy, verify appearance and behavior

### Add a Boss Pattern

1. Create pattern function in `BOSS_PATTERNS[bossIndex]` (line 1372)
2. Pattern function receives boss object `b` and fires enemy bullets via helper functions:
   - `fireRadial(b, count, speed, offset)`: N bullets in circle
   - `fireAimedSpread(b, count, speed, spreadStep)`: N bullets toward player
   - `fireColumns(b, speed)`: 5 vertical columns
   - `fireSpiralBurst(b, speed)`: 8 bullets spiraling
   - `fireCross(b, speed)`: 8-way cross
   - `fireHomingSeeded(b, count, speed)`: seeded homing missiles
   - `fireDoubleRing(b, speedA, speedB)`: two concentric rings
3. Test: progress to that stage, observe telegraphing and fire pattern

### Tweak Game Balance

1. **Difficulty**: modify `DIFFICULTIES` object (line 485)
   - `spawnBase`/`spawnMin`: enemy spawn frequency (lower = faster spawning)
   - `enemySpeed`: multiplier on enemy velocity
   - `enemyShootChance`: probability enemy fires (0–1)
   - `lives`, `bombs`: starting lives and bombs
   - `bossHpMul`: boss health multiplier
2. **Weapon**: modify cooldown (line 1177), spread angles (line 1127), or projectile speeds
3. **Boss health**: `baseHp` formula at line 1046 or design `hpMul` in `BOSSES` array (line 508)
4. **Spawn rate formula**: line 1201 scales difficulty over time; adjust divisor to change ramp speed

### Add a New Sound Effect

1. Write new `sfx.name` function in `sfx` object (line 624)
2. Use `beep(freq, dur, type, vol, opts)` or `noiseBurst(dur, vol, opts)` to compose
3. Call it from game code (e.g., `sfx.mySound()`)
4. Common options:
   - `sweepTo`: frequency glide target
   - `filterFreq`, `filterType`: biquad filter (lowpass, highpass, etc.)
   - `pan`: stereo position (-1 left to 1 right); use `panFor(x)` to pan by screen position
   - `delay`: onset delay in seconds

### Change a Theme

1. Modify entry in `THEMES` array (line 493)
2. Properties: `sky` (gradient stops: top, mid, bottom), `cloud` (RGBA values), `sun` (RGBA), `name`
3. Colors are strings (hex or rgba); gradients are drawn top to bottom

### Performance Tuning

- Quality system at lines 474–483 auto-scales particles based on frame time
- Manually force quality: set `quality` global (0=high, 1=medium, 2=low)
- High-cost operations: shadow rendering, particle spawning, boss screen
- Low-cost: input handling, collision checks (AABB is fast), audio synthesis

## Storage & Persistence

- `localStorage` keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- No server communication; all state is client-side

## Browser Compatibility

- Requires: ES6 (arrow functions, const/let), Canvas 2D, WebAudio API
- Works: Firefox 60+, Chrome 60+, Safari 12+, Edge 80+
- Mobile: iOS 11+, Android 5+

## Accessibility Considerations

- Animations respect `prefers-reduced-motion` media query
- All buttons have `aria-label` or `title` attributes
- Screen reader live region for game state changes
- Touch controls support reduced manual dexterity
- High contrast UI on dark background
