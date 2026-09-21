# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single self-contained HTML file. The entire game engine, UI, physics, audio synthesis, and rendering run in the browser using Canvas 2D and Web Audio API.

## How to Run

Simply open `index.html` in a modern web browser. The game works on desktop (mouse + keyboard) and mobile (touch controls with on-screen joystick and buttons).

- **Desktop**: Mouse to aim, click/space to fire, X/Shift for bomb, P/Esc to pause
- **Mobile**: Drag left zone to move, tap FIRE button, tap BOMB button

## High-Level Architecture

### Entry Point & Game Loop
- Single IIFE (Immediately Invoked Function Expression) wrapping all game logic
- `requestAnimationFrame` driven game loop: `loop()` updates state and renders each frame
- Frame timing with delta-time (`dt`) capped at 50ms to prevent large jumps on lag

### Core Game Systems

#### 1. **Input Handling** (lines 767-862)
- Keyboard: arrows/WASD for movement, Space/mouse click to fire, X/Shift for bombs, P/Esc to pause
- Mouse: hovers over canvas to set a target; plane glides toward cursor (keyboard/joystick takes priority)
- Touch: joystick emulation in left half of screen (moveZone), fire/bomb buttons in right half (actionZone)
- All inputs stored in `keys` object; touch tracking manages active joystick ID/position

#### 2. **Player & Movement** (lines 876-901, update logic 1148-1172)
- Player object: `{x, y, w, h, speed, cooldown, rapid, weapon, level, invuln}`
- Movement: keyboard/joystick move vectors are normalized; mouse cursor steering only activates when no keyboard input
- Boundaries: clamped to canvas edges
- Invulnerability: after taking damage, player blinks and is invulnerable for 1.5s

#### 3. **Weapon System** (lines 1124-1146)
- Two weapons: **Vulcan** (fast, spread shot with level-based fan) and **Laser** (piercing, slow)
- Weapon upgrades via power-ups: level 1→3 upgrades firepower (more bullets), then level 3 toggle to other weapon, reset to level 1
- Fire cooldown: 0.15s for Vulcan, 0.1s for Laser
- Bullets track pierce count (Laser bullets pass through enemies up to pierce limit)

#### 4. **Enemy Spawning & AI** (lines 1022-1041, update 1210-1228)
- Three enemy types defined in `ENEMY_TYPES`: grunt, interceptor (fast), tank (slow, multi-HP)
- Spawn every 40-75 frames (difficulty-scaled); enemies enter from top and move downward
- Weaving: interceptors follow a sine-wave pattern
- Shoot patterns: spread (3-bullet fan) or aimed (tracks player)
- Each enemy has shoot timer and HP; killed enemies spawn power-ups (6% life, 8% bomb, 20% weapon)

#### 5. **Boss System** (lines 508-515, 1043-1056, 1411-1439)
- Six boss designs cycled per stage, each with unique visual colors and HP multiplier
- Boss appears after 18 enemy kills per stage
- Boss patterns defined in `BOSS_PATTERNS` (line 1372): each boss has 3 attack patterns
- Attack phases: telegraph → fire (22 frames telegraph; then 85 frames of incoming fire)
- Pattern types: radial bursts, aimed spreads, columns, spirals, crosses, homing, double rings
- Bullets are radial (x/y velocity per frame), not screen-bound like player bullets

#### 6. **Scoring & Multiplier** (lines 1087-1101)
- Chain system: killing enemies increments chain counter (2.2s window)
- Multiplier: 1 + floor(chain / 5), capped at 10x (when chain ≥ 45)
- Each kill adds `score += base * multiplier`
- Chain breaks on taking damage or timeout

#### 7. **Collision Detection** (lines 1069-1071, 1232-1294)
- `rectsOverlap()`: AABB collision for all dynamic objects
- Enemy bullets vs player: remove bullet, trigger damage
- Player bullets vs enemies: destroy enemy if HP hits 0, spawn particles/power-ups, play SFX
- Player bullets vs boss: 3 damage per hit; laser pierces, others don't
- Player vs enemy/enemy bullets/boss: trigger damage and invulnerability

#### 8. **Rendering** (lines 1520-1638)
- Canvas 480×720 (BASE_W/BASE_H); styled to scale responsively via CSS
- Sky gradient per stage (6 themes in `THEMES` with seasonal/weather color schemes)
- Sun glow radial gradient
- Parallax cloud layers at different speeds
- Entities drawn: player plane, enemies, boss, bullets, power-ups, particles
- Screen shake applied via canvas translate (respects reduced-motion preference)
- Quality setting (0=full, 1=medium, 2=low) reduces particle/shadow counts for perf

#### 9. **Audio** (lines 542-676)
- **No external assets**: all sound generated via Web Audio API oscillators + noise
- Master gain + dynamics compressor for consistency
- Per-event SFX: shoot, laser, explosion (grunt/interceptor/tank variants), hit, bomb, power-up, boss telegraph, boss down, stage clear, game-over
- Panning: stereo based on bullet x-position on screen
- Muted state saved to localStorage

#### 10. **UI & Overlays** (lines 681-709, 1479-1504)
- HUD (top center): Score, Best, multiplier, weapon name/level, heart lives, bomb count
- Boss health bar: appears when boss enters
- Combo pop: animated text when reaching 10, 20, 30... kills
- Stage banner: displayed at start of each stage
- Pause overlay: shows pause menu, resume button
- Game-over overlay: final score, stage reached, new best highlight
- Flash/vignette effects: screen flash on hit, red vignette pulse when 1 life left

#### 11. **Difficulty & Settings** (lines 485-489, 741-750, 529-540)
- Three difficulties: Easy (4 lives, slower enemies), Normal (3 lives), Hard (3 lives, faster enemies, more shots)
- Saved to localStorage; applies at game reset
- Reduced motion toggle: disables animations, screen shake, flash effects (respects OS `prefers-reduced-motion`)
- Mute toggle: silences audio without stopping playback

#### 12. **Performance Adaptation** (lines 473-483)
- Tracks frame times over 45 frames
- If avg > 25ms (40fps), reduce quality (fewer particles, less shadow blur)
- If avg < 18ms (56fps), increase quality back up
- Quality affects particle spawn counts and shadow blur on rendering

### Game Loop Flow
```
loop(now)
  └─ update(dt)
      ├─ Move player (keyboard/mouse/joystick)
      ├─ Fire weapon (if input active)
      ├─ Update bullets (move, remove out-of-bounds)
      ├─ Update particles (move, fade)
      ├─ Spawn enemies (timer-based)
      ├─ Update enemies (move, shoot, weave)
      ├─ Update boss (if active)
      └─ Collision checks (bullets→enemies/boss, enemies→player, bullets→player)
  └─ draw()
      ├─ Clear canvas
      ├─ Apply screen shake
      ├─ Draw sky gradient + sun glow
      ├─ Draw clouds (parallax layers)
      ├─ Draw player, enemies, boss
      ├─ Draw bullets and particles
      └─ Draw UI elements (HUD, overlays)
```

## Key Constants & Configuration

- `DIFFICULTIES`: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier
- `KILLS_PER_STAGE`: 18 (when reached, boss spawns after 1.2s delay)
- `THEMES`: 6 sky gradients (one per stage cycle) with color schemes
- `ENEMY_TYPES`: defines sprite dimensions, HP, speed, colors, score value
- `BOSSES`: 6 boss designs with visual colors and HP multipliers
- `BOSS_PATTERNS`: attack patterns indexed by boss design

## LocalStorage Keys

- `skyfighter.bestScore`: best score ever
- `skyfighter.muted`: audio mute state
- `skyfighter.difficulty`: selected difficulty
- `skyfighter.reducedMotion`: reduced motion toggle state

## Common Development Tasks

### Adjust Difficulty Balance
Edit `DIFFICULTIES` object (line 485) for spawn rates, enemy speed, lives, bombs, boss HP.

### Add/Modify Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502) with dimensions, HP, speed, color, score
2. Update spawn logic in `spawnEnemy()` (line 1022) to include in roll
3. Add explosion SFX variant if needed in `EXPLODE_SFX` (line 678)

### Add/Modify Boss
1. Add design to `BOSSES` (line 508) with visual colors and HP multiplier
2. Add attack pattern array to `BOSS_PATTERNS` (line 1372)
3. Pattern functions use helpers: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`

### Add Visual Theme/Stage
1. Add theme to `THEMES` (line 493) with sky gradient colors, cloud/sun colors, and name
2. Themes cycle: stage 1→theme 0, stage 2→theme 1, etc. (line 1529)

### Add New Particle Type or Effect
1. Spawn via `spawnExplosion(x, y, color, count)` (line 1073)
2. Particles stored in `particles` array; update in `update()` (line 1296)
3. Render in `draw()` (line 1628) with fade-out based on `life / maxLife`

### Add New Sound Effect
1. Create function in `sfx` object (line 624) using `beep()` or `noiseBurst()`
2. Call from appropriate event (kill, damage, power-up, etc.)
3. Sound parameters: frequency, duration, waveform type, volume, optional sweep/filter/pan

### Change Canvas Resolution
Edit `BASE_W` and `BASE_H` (line 753); aspect ratio is 2:3 (portrait). Resize logic scales to fit window while maintaining aspect.

## Testing & QA Notes

- Combo multiplier scales non-linearly (every 5 kills adds 1x)
- Boss telegraph phase is 22 frames; attack phase follows for ~85 frames
- Invulnerability after damage is exactly 1.5 seconds
- Screen shake and flash effects disabled in reduced-motion mode
- Touch controls show visual buttons on mobile (detected via `ontouchstart` or `maxTouchPoints`)
- Performance degrades gracefully: quality auto-reduces on slow frames, raising back when frame rate improves
