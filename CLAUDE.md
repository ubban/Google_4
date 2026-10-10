# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML file with embedded CSS and JavaScript. It features procedurally generated sound effects via Web Audio API, responsive touch and keyboard controls, and progressive difficulty with dynamic enemy spawning and multi-stage boss encounters.

## Running the Game

- **Local development**: Open `index.html` directly in any modern browser (no build step required)
- **Live version**: Deploy `index.html` to any web server
- No external dependencies; runs entirely in the browser with localStorage for persistence

## Architecture Overview

The game is a single-file vanilla JavaScript application (~1670 lines) structured as follows:

### Core Game Loop
- **`loop(now)`** (line 1640): RequestAnimationFrame callback; calls `update()` and `draw()` at ~60fps
- **`update(dt)`** (line 1148): Physics, collision detection, spawning; runs every frame
- **`draw()`** (line 1520): Canvas rendering; runs every frame
- Performance tracking via `trackPerf()` (line 476) scales quality based on frame time

### Game State
- **Player**: Plane object with position, weapon system (vulcan/laser), health, invulnerability frames
- **Enemies**: Arrays of active enemies, bullets (player & enemy), power-ups, particles
- **Boss**: Special enemy with unique attack patterns; spawns after 18 kills per stage (KILLS_PER_STAGE)
- **Scoring**: Chain/combo multiplier; increments with rapid kills, resets on damage

### Input Handling
- **Keyboard**: Arrow keys / WASD for movement, Space/click to fire, X/Shift/P to bomb/pause
- **Mouse**: Plane glides toward cursor (keyboard/joystick takes priority)
- **Touch**: Dual-zone controls — left half for movement joystick, right half for fire/bomb buttons
- Separate `mouseActive` flag prevents mouse and keyboard input from fighting

### Canvas Rendering
- **Base resolution**: 480×720 px (BASE_W, BASE_H); scales up to 1.6× via CSS for responsive display
- **Themes**: 6 rotating sky gradients + parallax cloud layers (lines 493–500)
- **Player ship**: Procedurally drawn plane with flame effect during thrust
- **Enemies**: Grunt, Interceptor (weaving), Tank (multi-hp); each rendered as procedural plane variant
- **Bosses**: 6 unique designs with gradient hulls, animated cores, moving turrets
- **Particles**: Explosion effects scale based on `quality` setting
- **Glow effects**: Shadowed bullets and power-ups at full quality; omitted at lower quality

### Audio Synthesis
- **Web Audio API**: Procedurally generated SFX via oscillators and noise bursts; no external audio files
- **Master chain**: Compressor → Gain → destination; allows muting and volume control
- **Sound effects** (lines 624–676):
  - Single-note beeps: `beep(freq, dur, type, vol, opts)` with sweep, filter, pan
  - Noise bursts: `noiseBurst(dur, vol, opts)` for explosions and impacts
  - Complex sounds: Built from multiple overlapping beeps/noise layers (e.g., `sfx.bomb()`)
- Panning: `panFor(x)` converts x-coordinate to stereo pan value (-1 to 1)
- Muting stored in localStorage

### Game Systems

**Difficulty**
- 3 levels: easy, normal, hard (DIFFICULTIES object, line 485)
- Affects spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier
- Stored in localStorage; selectable from start screen

**Weapon System**
- **Vulcan** (default): Rapid-fire straight bullets; spread pattern increases with level (1–3)
- **Laser**: Piercing beam; fires from multiple lanes at higher levels; slower fire rate
- Power-ups cycle through weapon levels, then toggle to opposite weapon at max level
- Player starts at weapon level 1 on each game

**Boss Attack Patterns**
- 6 boss designs, each with 3 unique attack patterns (BOSS_PATTERNS, line 1372)
- Pattern functions fire radial spreads, aimed bursts, homing shots, etc.
- Boss telegraphs before each attack (telegraph timer, visual glow)
- Boss HP scales with stage and difficulty multiplier

**Collision Detection**
- Axis-aligned bounding-box overlap via `rectsOverlap(a, b)` (line 1069)
- Checked for: player ↔ enemies, player ↔ enemy bullets, player ↔ boss, bullets ↔ enemies/boss, player ↔ power-ups
- Player invulnerability frames (1.5s) after taking damage

**Staging**
- Stage increments after boss defeat
- Enemy spawning increases in difficulty as stage progresses (line 1201: difficultyRamp)
- Theme and boss design cycle via modulo (e.g., `THEMES[(stage - 1) % THEMES.length]`)
- Boss spawns after 18 enemy kills per stage

**Chain/Combo Multiplier**
- Increments on each kill within 2.2s window
- Multiplier = 1 + floor(chain / 5), max 10× at 50 kills
- Resets on player damage or time expiry
- Visual feedback: "X KILLS!" popup at multiples of 10 kills

**Reduced Motion**
- Respects `prefers-reduced-motion` system preference
- Toggleable via UI button; stored in localStorage
- Disables: screen shake, camera flash, animations on HUD elements
- Uses 90% shorter durations and reduced intensity for remaining effects

**UI & HUD**
- **Overlay**: Modal for start screen, pause, game-over; slide-up card animation
- **HUD**: Top-center bar showing score, best, multiplier, weapon level, hearts, bombs
- **Boss bar**: Health bar appears when boss enters (top-center, below HUD)
- **Combo pop**: Large gold text animation at screen center on kill milestones
- **Stage banner**: Large text intro on stage transitions (e.g., "STAGE 2 WARNING")
- **Accessibility**: Screen-reader live region (srLive) announces key events

## Common Development Tasks

### Adding a New Enemy Type
1. Define in `ENEMY_TYPES` object (line 502): `{ w, h, hp, speedMul, color, dark, score, minStage, weave? }`
2. Update spawn logic in `spawnEnemy()` to include in random selection
3. Add explosion SFX to `EXPLODE_SFX` map if custom sound desired
4. Enemy is automatically rendered via existing plane-drawing code

### Adding a New Boss
1. Add entry to `BOSSES` array (line 508): `{ name, hullTop, hullBottom, wing, core, hpMul }`
2. Create attack pattern array in `BOSS_PATTERNS` (line 1372); reference attack helper functions
3. Boss automatically cycles through designs based on stage modulo
4. Adjust `baseHp * hpMul` in `spawnBoss()` to scale difficulty

### Modifying Difficulty
1. Adjust values in `DIFFICULTIES` object (line 485)
2. Parameters: `spawnBase`, `spawnMin` (spawn timer range), `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`
3. New parameters require updates to difficulty scale logic in `update()` (spawn timer calculation) and boss spawn

### Tuning Weapon Balance
1. **Vulcan**: Spread angles and projectile speed in `fireWeapon()` (line 1126)
2. **Laser**: Pierce count and lane offsets (line 1136)
3. Fire rate: `fireDelay` constants (line 1177) — lower = faster
4. Boss damage per hit: hardcoded in boss collision check (line 1259: `boss.hp -= 3`)

### Adjusting Visual Polish
- **Particle scale**: `spawnExplosion()` particle count scales with quality (line 1074)
- **Glow effects**: Controlled by `if (glow)` blocks throughout `draw()`; glow = quality 0
- **Screen shake**: Magnitude and duration in `screenShake(mag, dur)` calls
- **Color schemes**: Theme sky gradients, cloud colors, UI accent colors (search `theme.` in code)

### Adding Sound Effects
1. Add a function to the `sfx` object (line 624); use `beep()`, `noiseBurst()`, or combinations
2. Configure frequency sweep, filter type/freq, panning as needed
3. Call from game logic (e.g., `sfx.shoot()` on player fire)
4. Respects mute state and system audio context

## Key Constants & Configuration

| Constant | Purpose | Location |
|----------|---------|----------|
| `BASE_W`, `BASE_H` | Canvas resolution (480×720) | Line 753 |
| `KILLS_PER_STAGE` | Enemy kills before boss | Line 491 |
| `DIFFICULTIES` | Easy/normal/hard parameters | Line 485 |
| `THEMES` | 6 sky/cloud color schemes | Line 493 |
| `ENEMY_TYPES` | Grunt, Interceptor, Tank stats | Line 502 |
| `BOSSES` | 6 unique boss designs | Line 508 |
| `BOSS_PATTERNS` | Attack patterns per boss | Line 1372 |

## LocalStorage Keys

| Key | Purpose |
|-----|---------|
| `skyfighter.bestScore` | High score |
| `skyfighter.muted` | Audio muted flag (0/1) |
| `skyfighter.difficulty` | Last selected difficulty (easy/normal/hard) |
| `skyfighter.reducedMotion` | Reduced motion preference (0/1) |

## Performance Optimization

- **Adaptive quality**: Tracked frame times; quality level (0/1/2) reduces particle counts and shadow blur at lower quality
- **Object pooling**: Particle arrays pre-allocated; enemies/bullets filtered (not spliced) for efficiency
- **Canvas batching**: Grouped draw calls (bullets, enemies, particles) with save/restore contexts
- **Touch optimization**: Passive event listeners where possible; touch-action: none on canvas to prevent defaults

## Accessibility Features

- **Screen reader support**: srLive region announces score, events, game state
- **Reduced motion**: Respects `prefers-reduced-motion`; toggleable via UI with persistent preference
- **Keyboard controls**: Full game playable without mouse
- **Semantic HTML**: Proper ARIA labels, button semantics, color contrast (light text on dark bg)
- **Focus management**: UI buttons accessible via keyboard Tab navigation

## Testing Checklist

When modifying gameplay, verify:
- [ ] All three difficulty levels are playable to stage 3+
- [ ] Touch controls work on mobile (move zone on left, fire/bomb on right)
- [ ] Keyboard and mouse controls don't interfere
- [ ] Boss patterns execute without hanging (check browser console for errors)
- [ ] Collision detection is accurate (especially player vs. walls, bullets vs. enemies)
- [ ] Score multiplier resets on damage
- [ ] Power-ups cycle correctly (level 1→2→3 then weapon toggle)
- [ ] Pause/resume state is clean; no sound playing after pause
- [ ] Reduced motion mode removes animations but game remains playable
- [ ] localStorage persists best score and settings across page reload
