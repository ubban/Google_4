# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in a single self-contained HTML file (`index.html`). The game requires no external dependencies, build tools, or assets beyond the HTML file itself.

### Key Features
- **Canvas-based 2D gameplay** at 480x720 resolution (responsive scaling)
- **Three difficulty levels** (Easy/Normal/Hard) with configurable parameters
- **Six themed stages** with procedurally spawned enemies and distinct visual palettes
- **Six unique bosses** with complex attack patterns and telegraphic cues
- **Weapon system**: dual weapons (Vulcan/Laser) with 3 upgrade levels each
- **Power-up system**: health, bombs, weapon upgrades
- **Dynamic score multiplier** based on kill chains (up to 10x)
- **Web Audio synthesis** for all sound effects (no audio files)
- **Accessibility**: reduced motion support, screen reader announcements, high contrast
- **Touch, keyboard, and mouse controls** with joystick for mobile
- **Performance optimization**: adaptive quality settings (3 levels) based on frame time
- **Persistent state**: local storage for best scores, difficulty, mute, and motion preferences

## Running the Game

1. Open `index.html` directly in a browser
2. Click "Start Game" to begin
3. No HTTP server required; works with `file://` URLs
4. Playable on desktop (mouse/keyboard) and mobile (touch)

### Testing Game Behavior
The best way to verify changes is to:
1. Open DevTools (F12) and check the Console for any errors
2. Play through one or two stages to test gameplay mechanics
3. For boss testing, use Hard mode to reach them quickly
4. Check localStorage after playing: `localStorage.getItem('skyfighter.bestScore')`

## Code Architecture

The game code is organized into logical sections within a single `<script>` IIFE:

### 1. **Initialization & Configuration** (lines 463-527)
- Storage keys and difficulty settings
- Theme definitions (6 themes with color palettes and names)
- Enemy type specifications (grunt, interceptor, tank)
- Boss definitions (6 bosses with design colors and HP multipliers)

**Key constants:**
- `DIFFICULTIES`: spawn rates, enemy behavior, lives, bombs by difficulty
- `THEMES`: stage color schemes (sky gradients, cloud/sun colors)
- `ENEMY_TYPES`: enemy stats (HP, speed, score, appearance)
- `BOSSES`: boss designs and attack pattern indices
- `KILLS_PER_STAGE`: kills required before boss spawns (18)

### 2. **Audio System** (lines 542-676)
Procedurally generated WebAudio synthesis (no external files). Uses oscillators and filters to create all sound effects.

**Key functions:**
- `beep(freq, dur, type, vol, opts)`: sine/square/sawtooth tone with optional pitch sweep and filter
- `noiseBurst(dur, vol, opts)`: filtered white noise for explosive sounds
- `sfx` object: 15+ named effects (shoot, laser, explode, bomb, powerup, etc.)

**Pattern:** All audio respects the `muted` flag and `reducedMotion` preference.

### 3. **Rendering** (lines 679-1638)
Canvas 2D drawing functions for all game objects.

**Key functions:**
- `drawPlane(x, y, w, h, color, darkColor, flame)`: player & enemy aircraft
- `drawBossShip(b)`: multi-part boss with animated core, wings, and rotors
- `drawClouds(speed, fillColor, rx, ry)`: parallax cloud layers
- `draw()`: main frame renderer (sky gradient, objects, particles, HUD)

### 4. **Input Handling** (lines 767-861)
- Keyboard: arrow keys, WASD for movement; Space for fire; X/Shift/P for bomb/pause
- Mouse: move to steer, click to fire (glides toward cursor)
- Touch: left zone (joystick) for movement, right buttons for fire/bomb
- Joystick clamping ensures smooth mobile controls

### 5. **Game State & Core Loop** (lines 863-1640)
Main game objects: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`.

**Update flow (line 1148):**
1. Player movement (keyboard/joystick/mouse)
2. Weapon firing (cooldown-based)
3. Bullet position updates
4. Enemy spawning & movement (with weave patterns for interceptors)
5. Collision detection (bullets→enemies, bullets→boss, enemies→player, powerups→player)
6. Boss updates (telegraphing, attack patterns)
7. Particle lifespan decay
8. Chain/multiplier management

**Boss system (lines 1372-1439):**
- Each boss has 3 distinct attack patterns
- Patterns fire radial, aimed spread, or homing bullets
- Telegraph phase (22 frames) telegraphs the upcoming attack

### 6. **Key Subsystems**

#### Weapon System (lines 871, 1124-1146)
- **Vulcan** (3 levels): spread increases at level 2+ (1→2→3 projectiles)
- **Laser** (3 levels): dual lanes at level 2, triple lanes at level 3, pierce enemies (1→2→3)
- Cooldown: 0.15s (Vulcan), 0.1s (Laser)

#### Power-Up System (lines 1058-1067, 1304-1322)
- 6% chance: health (max life)
- 8% chance: bomb
- 20% chance: weapon upgrade (level up or switch weapon)
- Spawn at enemy death location, fall downward

#### Chain/Multiplier System (lines 1087-1101)
- Each kill increments `chain`
- Chain window: 2.2 seconds (resets if gap > 2.2s)
- Multiplier: 1 + floor(chain/5), capped at 10x
- Visual feedback: combo popup at every 10 kills

#### Difficulty Ramping (line 1201)
- Spawn timer decreases over time: `max(d.spawnMin, d.spawnBase - frame/300*4)`
- This progressively increases difficulty throughout a stage

### 7. **Important Game State Variables**
```
player:        { x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }
boss:          { x, y, w, h, hp, maxHp, entering, design, telegraph, attackTimer, ... }
stage:         current stage (1-6+, repeats themes)
stageKills:    kills toward current stage's boss
chain:         consecutive kills without 2.2s gap
multiplier:    1-10x based on chain
lives:         current health
bombCount:     bombs available (0-2 or 0-3 depending on difficulty)
paused:        pause state
```

## Common Modifications

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502-506) with `w, h, hp, speedMul, color, dark, score, minStage, weave`
2. Add explosion sound case in `sfx` if custom audio needed (line 624-676)
3. Add branch in `spawnEnemy()` (line 1022) to control spawn frequency

### Modify Boss Difficulty
- Adjust `DIFFICULTIES[].bossHpMul` (line 485-489)
- Or modify `design.hpMul` in `BOSSES` array (line 508-515)
- Boss base HP is `70 + stage*35`, modified by both multipliers

### Add a New Attack Pattern
1. Write function following the signature: `b => { /* populate enemyBullets */ }`
2. Add to appropriate boss's pattern array in `BOSS_PATTERNS` (line 1372)
3. Example: `fireRadial(b, count, speed)` shoots projectiles in N directions

### Tune Difficulty/Spawn Rates
- `DIFFICULTIES[difficulty]` controls: spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul
- Spawn timer calculation (line 1201): lower spawnBase → more enemies, higher spawnMin → baseline density

### Change Visuals (Colors, Sizes)
- Player plane: `drawPlane()` call at line 1558 (change `#2196f3`, `#1256a3` colors)
- Enemies: `ENEMY_TYPES[].color` and `.dark` fields
- Bullets: `ctx.fillStyle` in draw() loop (lines 1581-1601)
- Themes: add/edit entries in `THEMES` array with new sky gradient stops and cloud/sun colors

### Add Accessibility Features
- Animations: already respect `reducedMotion` flag (line 349-358)
- Text: use `announce()` function (line 468-471) for screen reader announcements
- High contrast: consider adding a `highContrast` flag similar to `reducedMotion`

## Performance Optimization

The game includes adaptive quality (line 474-483):
- **Quality 0** (full): glow effects, shadow blur, particle count
- **Quality 1** (medium): reduced blur, fewer particles (60%)
- **Quality 2** (low): no blur, 35% particles

Quality auto-adjusts based on 45-frame average:
- Upgrades if avg frame time > 1/40 (25ms)
- Downgrades if avg frame time < 1/56 (17.9ms)

**Optimization tips:**
- Reduce particle count: edit multiplier in `spawnExplosion()` (line 1074)
- Disable glow: set `glow = false` (line 1579)
- Limit shadow blur values (line 1002, 1583, 1594, 1622)

## Local Storage Keys

Used for persistence across sessions:
- `skyfighter.bestScore`: numeric best score
- `skyfighter.muted`: '0' or '1'
- `skyfighter.difficulty`: 'easy', 'normal', or 'hard'
- `skyfighter.reducedMotion`: '0' or '1' (respects OS preference if not set)

## Testing Checklist

When making changes, verify:
1. **Gameplay**: Start game, reach stage 2, kill boss, verify score multiplier
2. **Input**: Test keyboard (arrows, space, P), mouse (move + click), touch (joystick + buttons)
3. **Audio**: Mute button works, no audio errors in console
4. **Mobile**: Resize window to test responsive scaling, touch controls
5. **Persistence**: Play, quit, reload page, check best score/difficulty preserved
6. **Animations**: If modifying, test with reduced motion OFF and ON
7. **Difficulty**: Test Easy/Normal/Hard, verify enemy spawn rates and boss HP differ

## Performance Profile

- Base canvas: 480x720
- Target: 60 FPS (16.67ms/frame)
- Typical load on modern hardware: minimal (single HTML file)
- Main bottlenecks: particle rendering, shadow blur (disable on low-end devices)
