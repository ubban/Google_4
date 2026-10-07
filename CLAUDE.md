# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented entirely in a single HTML file. It features:
- Procedurally spawned enemy waves with multiple types
- Dynamic difficulty system (Easy/Normal/Hard)
- 6 unique boss encounters with distinct attack patterns
- Weapon system with power-ups (Vulcan cannon and Laser)
- Touch and keyboard controls
- WebAudio synthesized sound effects (no external assets)
- Adaptive rendering quality based on frame time
- Responsive canvas scaling for all screen sizes

## Running the Game

Simply open `index.html` in any modern web browser. No build process, dependencies, or server required.

**For development:**
- Edit `index.html` directly
- Use your browser's DevTools for debugging (F12)
- Open the file locally or serve via any HTTP server for testing (e.g., `python -m http.server`)

## Architecture Overview

The entire game is a single self-contained IIFE (Immediately Invoked Function Expression) that manages:

### Game Entities
All game objects share a common shape with `x, y, w (width), h (height)`:
- **Player**: Single instance; tracks weapon level, cooldown, invulnerability timer
- **Enemies**: Array of spawned ships (grunt, interceptor, tank); each has HP, shoot pattern, and optional weaving behavior
- **Bullets**: Player projectiles with optional pierce/laser properties
- **Enemy Bullets**: Radial or aimed patterns; support both linear and radial (dx/dy) movement
- **Power-ups**: Life, bomb, or weapon drops; rotate on screen
- **Particles**: Short-lived explosion effects with velocity decay
- **Boss**: Single entity; tracks entering phase, attack timer, telegraph state, health

### Core Systems

#### Game Loop (lines 1640–1666)
- `requestAnimationFrame` driven at ~60 FPS
- Delta-time based physics (clamped to 50ms max per frame)
- **Execution order**: performance tracking → update (if running & not paused) → screen shake decay → draw
- Performance tracker (`trackPerf()`) adjusts `quality` level (0=full, 1=medium, 2=low) based on 45-frame moving average

#### Update Pipeline (lines 1148–1302)
1. **Input handling**: Keyboard, joystick, and mouse targeting (keyboard/joystick takes priority)
2. **Player movement**: Bounded within canvas; 260 px/s base speed
3. **Weapon firing**: Cooldown-based; spreads and pierce vary by weapon and level
4. **Entity updates**: Move bullets, enemies, and enemy bullets; apply weave patterns
5. **Enemy spawning**: Continuous until KILLS_PER_STAGE (18) reached; spawn rate decreases over time via difficulty ramp
6. **Collision detection**: Player-bullet vs enemy, bullet vs boss, bullets vs player, enemies vs player, enemy bullets vs player
7. **Chain combo**: Kills reset on damage taken; multiplier increases with consecutive kills (max 10x)
8. **Boss updates**: Separate handling for entry animation, movement, and attack telegraph

#### Rendering (lines 1520–1638)
- **Layered**: Background gradient → clouds → player → bullets → enemies → boss → power-ups → particles
- **Quality-aware**: Glow/shadow effects disabled at low quality; particle count scales with quality
- **Visual effects**: Screen shake (camera translation), flash overlays (hit/bomb), vignette pulse on critical health
- **Canvas**: Logical size 480×720; scaled to fit screen up to 1.6x

#### Input System (lines 768–861)
- **Keyboard**: WASD/Arrows for movement, Space/Space for fire, X/Shift for bomb, P/Esc for pause
- **Touch**: Two-zone interface (left: movement via joystick overlay, right: FIRE and BOMB buttons)
- **Mouse**: Cursor position steers player (smooth glide to target); click to fire

#### Audio Synthesis (lines 542–676)
All sound is generated via WebAudio (no asset files):
- `beep(freq, dur, type, vol, opts)`: Oscillator-based synth; supports frequency sweep, filters, panning
- `noiseBurst(dur, vol, opts)`: Filtered white noise for explosions
- 14 SFX defined: shoot, laser, enemy explosions (3 types), hits, bombs, power-ups, boss attacks, stage clear, UI, game over
- Each effect is procedurally timed for audio satisfaction (e.g., multi-part boss-down is 5 overlapping beeps)

### Key Game Concepts

#### Difficulty Tiers
Stored in `DIFFICULTIES` (line 485):
- **Easy**: 4 lives, 3 bombs, slower enemies, lower spawn rate, 0.8x boss HP
- **Normal**: 3 lives, 2 bombs, standard tuning
- **Hard**: 3 lives, 2 bombs, faster enemies, higher spawn rate, 1.3x boss HP

Spawn rate ramps down over time: `spawnBase - (frame / 300) * 4` (line 1201).

#### Boss System
6 bosses cycle with stages (line 1044); each has:
- **Design**: Color scheme (hull, wing, core), HP multiplier
- **Attack patterns**: 3 patterns per boss (Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan)
- **Telegraph**: 22-frame pre-attack wind-up; visual (core brightens, SFX plays)
- **Movement**: Bounces left-right at 55 px/s

Patterns use helper functions:
- `fireRadial()`: Full-circle burst
- `fireAimedSpread()`: Fan aimed at player
- `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireDoubleRing()`, `fireHomingSeeded()`

#### Enemy Types
Defined in `ENEMY_TYPES` (line 502):
- **Grunt**: 34×34, 1 HP, red, baseline
- **Interceptor**: 26×26, 1 HP, fast (1.7× speed), weaves side-to-side
- **Tank**: 46×46, 4 HP, slow, purple, spawns from stage 2+
- **Weaving**: Interceptors use sine wave at frame-based position

#### Weapon System
Player has 2 weapons with 3 levels each:
- **Vulcan**: Spray cannon; 150ms cooldown; spreads widen at levels 2 & 3 (up to 3-shot spread)
- **Laser**: Piercing; 100ms cooldown; split lanes at level 2, center lane at level 3 (5-shot total)
- Power-ups cycle through levels; at level 3, toggle between weapons and reset level

#### Visuals & Themes
6 sky gradients (`THEMES`, line 493) cycle per stage:
- Daylight, Sunset, Night, Storm, Arctic, Volcanic
- Cloud layers scroll at different speeds (0.5 and 1.2)
- Sky is gradient (3 color stops); sun is radial glow overlay

#### State Persistence
LocalStorage keys (lines 463–466):
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Reduced-motion preference (respects OS setting)

### Collision Detection
Simple AABB (axis-aligned bounding box):
```javascript
rectsOverlap(a, b) {
  return Math.abs(a.x - b.x) < (a.w + b.w) / 2 && 
         Math.abs(a.y - b.y) < (a.h + b.h) / 2;
}
```
Applied to: player-enemy, player-bullet, player-powerup, boss-bullet, boss-player.

### Performance Tuning
Adaptive quality system responds to frame time:
- Tracks last 45 frames; if avg > 1/40s (25ms), raise quality level → fewer particles, no shadows
- If avg < 1/56s (~17.8ms) and not at quality 0, lower level → enable glow effects
- Particle count scales: quality 0 = 100%, 1 = 60%, 2 = 35%
- Shadow blur at quality 0 only

## Common Editing Tasks

### Adding a New Enemy Type
1. Add to `ENEMY_TYPES` (line 502) with `w, h, hp, speedMul, color, dark, score, minStage`
2. Add to spawn pool in `spawnEnemy()` (line 1026) with `minStage` check
3. (Optional) Add explosion SFX to `EXPLODE_SFX` (line 678)

### Adding a New Boss
1. Add to `BOSSES` array (line 508) with colors and hpMul
2. Create 3 attack pattern functions (e.g., fireRadial, fireColumns, etc.)
3. Add 3-pattern array to `BOSS_PATTERNS` (line 1372) in corresponding index
4. Boss number = stage modulo BOSSES.length

### Tweaking Difficulty
- Modify `DIFFICULTIES` object (line 485)
- Spawn rate: `spawnBase` and `spawnMin`
- Enemy behavior: `enemySpeed`, `enemyShootChance`
- Player resources: `lives`, `bombs`, `bossHpMul`

### Adjusting Sound Design
All SFX live in `sfx` object (lines 624–676):
- Frequency range: 20–4000+ Hz typical
- Duration: 0.05–0.55 seconds
- Envelope: Linear ramp-up to peak, exponential ramp-down
- Filters: Lowpass (freq 1300–5000), Highpass (freq 700)
- Multi-part effects (e.g., bomb) use delayed beep/noise layers

### Tweaking Visual Effects
- **Screen shake**: `screenShake(mag, dur)` line 727; respects reducedMotion
- **Flash overlay**: `flashHit()`, `flashBomb()` (lines 698–709); CSS animations at lines 95–103
- **Vignette**: Opacity animates when health = 1 (line 65)
- **Combo pop**: Shows at every 10 kills (line 1248)

### Changing Canvas Size
Base size is 480×720 (line 753). Scales up to 1.6× to fit screen.
- All in-game coordinates are canvas-local
- HUD and overlay scale with viewport via clamp() functions (e.g., `clamp(12px, 2.8vw, 16px)`)

## Notes for Future Development

- **No external dependencies**: Game runs standalone; keeps deployment simple
- **Frame-based animation**: All temporal effects key off `frame` counter; test responsively
- **Mobile-first**: Touch controls are first-class; mouse fallback for desktop
- **Accessibility**: Reduced-motion mode, screen-reader announcements (`srLive` div), keyboard controls
- **LocalStorage**: Currently light usage; safe for casual state persistence
- **Browser support**: Requires ES6, Canvas 2D, WebAudio (gracefully degrades if unavailable)
