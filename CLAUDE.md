# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in a single HTML file with no external dependencies. The game runs in the browser with procedurally generated stages, multiple difficulty levels, and local leaderboard persistence via localStorage.

## Architecture

### Single-File Structure
The entire game logic, rendering, and UI is contained in `index.html` (~1700 lines):
- **CSS (lines 10-358):** Responsive layout, animations, theme colors, HUD styling
- **HTML (lines 361-420):** DOM elements for canvas, overlay, touch controls, HUD
- **JavaScript (lines 422-1666):** Game engine, physics, AI, audio synthesis

### Core Game Loop
```
requestAnimationFrame(loop) → update(dt) → draw() → repeat
```
- `update(dt)`: Handles input, collision detection, entity movement, spawning
- `draw()`: Renders background, entities, particles, UI
- Adaptive performance quality scaling (0-2) based on frame times

### Key Systems

#### 1. **Input & Control**
- **Keyboard:** Arrow keys / WASD for movement, Space for fire, X/Shift for bomb, P/Esc pause
- **Mouse:** Move to steer (when over canvas), click to fire
- **Touch:** Left zone for joystick (normalized movement), right zone for FIRE/BOMB buttons
- Joystick visualization appears only on touch devices
- Priority order: Keyboard/joystick > mouse (prevents fighting)

#### 2. **Entity Management**
All entities are plain objects updated each frame:
- `player`: Single player ship with position, velocity, weapon state
- `enemies[]`: Grunt, Interceptor, Tank types with different HP/speed/score
- `bullets[]`: Player projectiles (Vulcan spread or Laser piercing)
- `enemyBullets[]`: All enemy fire (simple bullets or radial patterns)
- `boss`: Single active boss with multi-phase attack patterns
- `powerUps[]`: Life/Bomb/Weapon upgrades (6%, 8%, 20% spawn rates)
- `particles[]`: Explosion debris (count scaled by quality setting)

#### 3. **Combat & Scoring**
- **Kill Chain Multiplier:** Resets if 2.2s without a kill; multiplier = 1 + floor(chain/5) (max 10x)
- **Weapons:** 
  - Vulcan: 1-3 projectiles depending on level, spread pattern
  - Laser: Pierce multiple enemies, 1-3 lanes depending on level
  - Weapon pickups upgrade level or swap weapon and reset to level 1
- **Boss Patterns:** 6 boss types × 3 attack patterns each; patterns determined by boss design index

#### 4. **Difficulty System**
Three presets stored in `DIFFICULTIES` object, persisted in localStorage:
- **Easy:** Longer spawn intervals, slower enemies, lower boss HP (0.8x)
- **Normal:** Default experience, standard boss HP
- **Hard:** Aggressive spawning, fast enemies, higher boss HP (1.3x), higher bullet count

Difficulty affects enemy spawn rate, enemy speed, enemy accuracy, lives, bombs, boss HP.

#### 5. **Audio (WebAudio API)**
Procedural synth-based SFX with no external files:
- **beep():** Tone with optional sweep, filter, panning, gain envelope
- **noiseBurst():** White noise with decay, used for impact/explosion tails
- `sfx` object maps game events to sound functions
- Master gain with dynamics compressor; muted state persisted

#### 6. **Visual Theme System**
Six rotating sky themes (one per stage, wraps) with:
- Sky gradient (3 colors)
- Cloud fill color
- Sun fill color
- Theme name
Each theme has procedural scrolling clouds at two speeds/scales.

#### 7. **Collision & Physics**
- **rectsOverlap():** AABB collision between entities (center-to-center bounding box)
- Player movement clamped to canvas bounds
- Enemies weave horizontally (sine wave offset from base X)
- Boss bounces between left/right boundaries
- Knockback/screen shake triggered by hits and bombs

#### 8. **Boss AI**
Boss phases:
1. **Entering:** Slides from top to position (y=120) over ~1.3s
2. **Combat:** Bounces horizontally, periodically telegraphs then attacks
   - Telegraph phase: 22 frames of core glow, sfx plays
   - Attack: Calls one of 3 pattern functions (radial, aimed spread, special)
   - Attack cooldown: 85 frames between patterns

Each boss has unique attack patterns (Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan).

#### 9. **Screen Effects**
- **Screen Shake:** Amplitude + duration; scaled down if reduced-motion is on
- **Flash:** White overlay fade; separate animation for hit vs. bomb
- **Vignette:** Red pulsing overlay when at 1 HP
- **Combo Pop:** Center-screen floating text for kill milestones
- **Stage Banner:** Large text announcing new stage or boss warning
- All effects respect user's reduced-motion preference

#### 10. **Persistent State**
Four localStorage keys:
- `skyfighter.bestScore`: Best score achieved (Number)
- `skyfighter.muted`: Audio mute toggle (0/1)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Motion preference (0/1), defaults to OS preference

#### 11. **Stage Progression**
- Each stage requires killing 18 enemies (KILLS_PER_STAGE)
- After 18 kills, boss spawns with 1.2s delay
- Boss defeated → +500 score, stage increments, kill counter resets
- Themes and boss designs rotate via modulo (stage - 1)
- Spawn rate increases over time via frame counter

## Common Development Tasks

### Running the Game
Open `index.html` in any modern browser. No build step, no dependencies.
- **Dev Testing:** Open in Firefox/Chrome/Safari, use DevTools console for debugging
- **Mobile Testing:** Use browser DevTools device emulation or actual device

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line ~502): specify `w`, `h`, `hp`, `speedMul`, `score`, `minStage`, optional `weave`
2. Optionally add SFX to `sfx` object and `EXPLODE_SFX` map if unique death sound needed
3. Update enemy spawn logic in `spawnEnemy()` if new type needs special probability

### Adding a New Boss
1. Add entry to `BOSSES` array (line ~508): specify `name`, hull colors, wing color, core color, `hpMul`
2. Add corresponding pattern array to `BOSS_PATTERNS` (line ~1372): array of functions that call attack functions (fireRadial, fireAimedSpread, etc.)
3. Boss index auto-selects based on stage modulo, patterns auto-select on telegraph

### Tuning Difficulty Curves
- Adjust `DIFFICULTIES` constants (lines 485-489): spawn intervals, enemy speed multiplier, shoot chance, lives, bombs, boss HP multiplier
- Adjust individual enemy type stats in `ENEMY_TYPES` if needed
- Adjust base boss HP formula in `spawnBoss()`: `70 + stage * 35`

### Tweaking Physics/Balance
- Player speed: `player.speed = 260`
- Bullet speeds: Set individually in `fireWeapon()` and boss patterns
- Enemy speeds: Base in `DIFFICULTIES`, multiplied by `speedMul` in `ENEMY_TYPES`
- Knockback magnitude: `screenShake()` call in damage/bomb handlers

### Performance Optimization
Performance is tracked via frame times and adaptive quality (0=full, 2=low):
- At quality 0: Full glow effects (shadowBlur, shadowColor)
- At quality 1-2: Reduced particle counts, no shadow effects
- Modify `trackPerf()` thresholds (lines 476-482) to adjust quality targets
- Particle count scaled in `spawnExplosion()`: `Math.max(3, Math.round((count || 14) * (quality === 0 ? 1 : quality === 1 ? 0.6 : 0.35)))`

## Key Constants & Formulas

| What | Where | Formula/Value |
|------|-------|---------------|
| Canvas size | Line 753 | 480×720 |
| Kills per stage | Line 491 | 18 |
| Multiplier cap | Line 1090 | 1 + min(9, floor(chain/5)) |
| Chain timeout | Line 1089 | 2.2 seconds |
| Boss base HP | Line 1046 | 70 + stage × 35 |
| Boss telegraph time | Line 1050 | 22 frames |
| Boss attack cooldown | Line 1429 | 85 frames |
| Player invuln on hit | Line 1459 | 1.5 seconds |
| Max mouse steering dist | Line 1167 | 1.7× player speed |

## Accessibility Features

- **Reduced Motion:** Toggle via motion button (🎆/🚫); disables all animations and screen shake
- **Screen Reader Support:** `#srLive` element with `role="status"` for announcements
- `announce()` function (line 468) queues text to SR
- Dark theme only; no contrast issues with current palette
- Touch controls properly sized (78px buttons)

## Browser Compatibility

- **Required:** Canvas 2D, WebAudio API, localStorage, touch events
- **Tested:** Chrome, Firefox, Safari (desktop & iOS)
- **Fallbacks:** AudioContext with webkit prefix; localStorage checks before use
- **Polyfills:** None; game gracefully degrades (no audio if WebAudio unavailable)

## Common Bugs & Gotchas

1. **Pause state not checked:** Ensure `!paused` check in input handlers and update loop
2. **Invuln window too short:** Player can take rapid hits if invuln < collision time
3. **Boss entering collision:** Boss can collide during entering animation; either skip collision or lower entry Y
4. **Particle memory leak:** Ensure particles are filtered out when `life <= 0`
5. **Touch firing stuck:** Release events may miss if listener not on window; currently on window
6. **localStorage quota:** Extremely unlikely with only 4 keys, but add try/catch if needed

## Testing Checklist

- [ ] All three difficulty levels spawn correctly
- [ ] Boss patterns fire at telegraph (watch inspector for pattern calls)
- [ ] Weapon upgrade pickups cycle Vulcan→Laser→Vulcan
- [ ] Kill chain resets after 2.2s without kill
- [ ] Bomb clears all enemies and deals damage to boss
- [ ] Screen shake and flash respect reduced-motion toggle
- [ ] Touch joystick and buttons work on mobile
- [ ] Best score persists across page reloads
- [ ] Mute button persists state
- [ ] All six bosses appear in sequence when progressing stages
