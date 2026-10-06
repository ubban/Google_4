# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game written entirely in vanilla HTML5/Canvas/JavaScript. The entire game (including CSS and WebAudio sound synthesis) is contained in a single `index.html` file (~1,670 lines).

## Running the Game

- **Browser**: Open `index.html` directly in any modern browser
- **Live server**: `python -m http.server 8000` or any static server, then navigate to `http://localhost:8000`
- No build step, dependencies, or setup required

## Code Architecture

The game follows a classic game engine pattern within a single IIFE (Immediately Invoked Function Expression):

### Core Game Loop (line 1640)
- `requestAnimationFrame(loop)` - runs at display refresh rate
- Tracks delta time (`dt`) and clamps to 0.05s to prevent large jumps
- Calls `update(dt)` for game logic, then `draw()` for rendering

### Major Systems

**Game State** (line 863-870)
- `player`: Position, dimensions, weapon, cooldown, invulnerability
- `bullets`, `enemies`, `enemyBullets`, `powerUps`: Arrays of active game entities
- `score`, `lives`, `stage`, `kills`, `chain`: Persistent gameplay state
- `boss`: Current boss object or null

**Update Logic** (line 1148-1302)
- Player movement from keyboard/joystick/mouse with collision bounds
- Bullet and projectile updates
- Enemy spawning with difficulty-based parameters
- Collision detection between player/enemies/bullets
- Stage progression triggers (18 kills per stage spawn boss)
- Power-up application

**Rendering** (line 1520-1637)
- Background: Multi-layer sky gradient with parallax clouds (lines 1529-1544)
- Player: Custom 2D plane sprite with flame when firing (line 1558)
- Bullets: Yellow (Vulcan) or cyan (Laser) with glow effects (lines 1580-1591)
- Enemies: Planes drawn with colors based on type (line 1603)
- Boss ships: Complex multi-part design with animated core (line 1615)
- Particles: Explosion effects with fade-out
- Screen shake applied at translation layer (lines 1523-1527)

### Gameplay Mechanics

**Difficulty Settings** (lines 485-489)
- `DIFFICULTIES`: Configures spawn rates, enemy speed, shoot chance, lives, bombs, boss HP
- Three levels: Easy, Normal, Hard
- Stored in localStorage

**Enemy Types** (lines 502-506)
- `grunt`: Red, slow, 1 HP, shoots at player
- `interceptor`: Orange, fast, weaves, 1 HP
- `tank`: Purple, heavily armored, 4 HP, worth more points
- Spawn rate increases as frame count increases (line 1201)

**Boss System** (lines 508-515, 1372-1409)
- 6 unique boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Each has unique attack patterns defined in `BOSS_PATTERNS` array
- Boss enters after 18 kills per stage
- Telegraph phase (22 frames) before attack execution
- Boss telegraphing effect: core glows and flashes

**Weapons** (lines 1124-1146)
- `vulcan`: Default hitscan weapon, spreads at level 2+
- `laser`: High damage, pierces enemies, faster fire rate
- Level progression: Max level 3, then toggles between weapons
- Power-up drops determine weapon upgrades

**Scoring & Chain** (lines 1087-1101)
- Points scale by multiplier (1.0x to 10.0x based on kill chain)
- Chain timer: 2.2 seconds per kill
- Combo multiplier: `1 + min(9, floor(chain/5))`
- Breaking chain resets to 1x multiplier

**Audio** (lines 542-676)
- WebAudio API with oscillators and noise bursts
- No external sound files
- Frequency sweeps and panning effects
- Mute toggle stored in localStorage
- Function signatures: `beep(freq, duration, type, volume, opts)`, `noiseBurst(duration, volume, opts)`

### Input Handling

**Keyboard** (lines 768-776)
- Arrow keys or WASD for movement
- Space for fire
- X or Shift for bombs
- P or Escape for pause

**Mouse** (lines 838-861)
- Move cursor over canvas to steer player (smooth glide)
- Click to fire
- Crosshair indicator when active

**Touch** (lines 814-834)
- Left 55% of screen: Joystick for movement (visual base shows when active)
- Right 45% of screen: Fire button (red) and Bomb button (yellow)
- Touch starts audio context

**Priority**: Keyboard/joystick input always overrides mouse to prevent fighting between control schemes

### Adaptive Performance (lines 473-483)
- Tracks frame timing over 45 frames
- Adjusts quality levels based on average frame time:
  - Quality 0: Full effects (glow, shadows)
  - Quality 1: Medium (reduced particles/effects)
  - Quality 2: Low (minimal visual effects)

### Canvas & Scaling

- Base resolution: 480×720 (portrait)
- Scales up to 1.6x on larger screens via CSS
- Responsive to window resize and orientation changes
- Touch-action and user-select disabled for game feel

### UI Elements

**HUD** (line 367-374): Score, Best score, Multiplier, Weapon name/level, Hearts (lives), Bombs

**Boss Bar** (line 376-379): Shows boss name and health bar

**Top Bar** (line 384-388):
- Mute button (toggles 🔊/🔇)
- Reduced Motion button (toggles effects like screen shake and flash)
- Pause button

**Overlay** (line 401-419): Start screen with difficulty selector and game over screen

**Accessibility**:
- Reduced motion support (prefers-reduced-motion media query)
- Screen reader announcements via `#srLive` (line 390)
- Font scaling with viewport

### Storage Keys

- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Motion effects preference

## Common Modifications

**Adding a New Enemy Type**
1. Add entry to `ENEMY_TYPES` (line 502) with sprite properties and behavior flags
2. Add spawn condition in `spawnEnemy()` (line 1026)
3. Add explosion sound effect to `EXPLODE_SFX` (line 678) if unique
4. Implement custom drawing if not using `drawPlane()`

**Adding a Boss Attack Pattern**
1. Write pattern function (e.g., `fireRadial()`) that pushes objects to `enemyBullets`
2. Add to `BOSS_PATTERNS` array (line 1372)
3. Functions are called during boss telegraph phase (line 1427)
4. Rotate patterns via `patternIndex` modulo

**Adjusting Game Balance**
- Difficulty parameters: `DIFFICULTIES` (line 485)
- Enemy spawning: `spawnBase`, `spawnMin` control timing
- Scoring: `addScore()` multiplier in `DIFFICULTIES`, point values in `ENEMY_TYPES`
- Boss health: Base HP at line 1046, multiplied by difficulty and boss design

**Changing Visual Theme**
- Sky gradients: `THEMES` array (line 493) — each has 3 gradient stops + cloud/sun colors
- Enemy colors: `color` and `dark` fields in `ENEMY_TYPES`
- Boss colors: `hullTop`, `hullBottom`, `wing`, `core` in `BOSSES`

## Key Constants

- `BASE_W = 480, BASE_H = 720`: Canvas resolution
- `KILLS_PER_STAGE = 18`: Enemies to spawn before boss
- `quality`: 0 (full glow), 1 (medium), 2 (low effects)
- Frame-based timing uses `frame` counter (incremented each update)

## Notes for Development

- The entire game state is functional and non-OOP — entities are plain objects
- Collision detection uses `rectsOverlap()` utility (line 1069)
- Particle effects are simple objects with position, velocity, and lifespan
- Screen shake is applied at canvas translation layer, not individual entities
- All drawing uses canvas 2D context directly; no rendering library
- Animations use frame-based timing (not time-based) for predictability
