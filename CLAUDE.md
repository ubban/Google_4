# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a browser-based Raiden-style vertical shoot-'em-up game, entirely self-contained in a single HTML file (`index.html`). No build process, no external dependencies, no package manager needed. Just open it in a browser and play.

## Running the Game

Since there's no server or build step:
- Open `index.html` directly in any modern web browser
- Or use a local HTTP server: `python3 -m http.server 8000` then visit `http://localhost:8000/index.html`
- Game state (best score, difficulty, mute preference, reduced-motion setting) persists in browser `localStorage`

## Architecture

The entire game is a single ~1,670 line HTML file with three sections:

### 1. **CSS Styling** (lines 10–359)
- Dark theme optimized for gameplay visibility
- Responsive canvas scaling to fit screen
- HUD elements: score, multiplier, hearts, bombs, boss bar
- UI overlays for title screen, pause, and game-over
- Animations: screen shake, flash effects, combo popups, stage banners
- Reduced-motion support for accessibility

### 2. **Game State & Constants** (lines 422–528)
Key game constants are defined at the top:
- `DIFFICULTIES`: tunable parameters for Easy/Normal/Hard (spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier)
- `ENEMY_TYPES`: grunt, interceptor, tank with color, speed, HP, score
- `BOSSES`: six boss designs with hull/wing/core colors and HP multipliers
- `THEMES`: six stage themes with sky gradient colors, cloud color, sun color
- `BOSS_PATTERNS`: attack pattern functions for each boss

Game state variables:
- `player`: position, weapon, level, invuln timer
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`: game objects
- `score`, `lives`, `stage`, `kills`, `chain`, `multiplier`: gameplay counters
- `running`, `paused`, `gameOver`: state flags

### 3. **JavaScript Logic** (lines 542–1666)

#### Core Loop
- `loop(now)` (line 1640): `requestAnimationFrame` callback that calls `update()` then `draw()` each frame
- `update(dt)` (line 1148): physics, collision, spawning, state changes (60+ lines of logic)
- `draw()` (line 1520): rendering with optional screen shake and quality-based glow effects

#### Player Control
- Keyboard: arrow keys or WASD for movement, Space to fire, X/Shift for bombs, P/Esc to pause
- Mouse: move to steer, click to fire
- Touch: drag left side to move (joystick), tap FIRE and BOMB buttons
- Input state in `keys` object; mouse and joystick have dedicated handlers

#### Combat System
- `fireWeapon()` (line 1124): spreads based on level; Vulcan is hitscan, Laser pierces
- Enemy shooting: aimed or spread patterns at random intervals
- Boss patterns: 6 different attack sequences (`BOSS_PATTERNS`, line 1372) including radial bursts, aimed spreads, teleport dashes, spirals
- `useBomb()` (line 1103): destroys all enemies and bullets, damages boss, costs a bomb

#### Game Flow
- `resetGame()` (line 875): initializes/resets all state
- `spawnEnemy()` (line 1022): random enemy with difficulty-scaled properties
- `spawnBoss()` (line 1043): after 18 kills per stage
- `killBoss()` (line 1441): advances stage, +500 score, plays fanfare
- `endGame()` (line 1469): saves best score to `localStorage`, shows overlay
- `loseLife()` (line 1456): handles hit, invuln timer, game-over check

#### Power-ups
- `maybeSpawnPowerUp()` (line 1058): 6% life, 8% bomb, 20% weapon
- `applyPowerUp()` (line 1304): upgrades weapon/level or restores life/bombs

#### Audio (Web Audio Synthesis)
- No audio files; all sound generated procedurally (lines 542–676)
- `beep()`: tones with optional frequency sweep, filter, pan, delay
- `noiseBurst()`: filtered noise (explosions, booms)
- SFX for: shoot, laser, explosions (grunt/interceptor/tank), hit, bomb, powerup, boss-hit, boss-telegraph, stage-clear, UI-click, game-over
- Master gain and compressor to prevent clipping

#### Rendering
- Canvas: 480×720 base, scaled to fit window (max 1.6×)
- Draws in order: sky gradient + sun glow + clouds, player, bullets, enemies (with health bars for tanks), bosses, power-ups, particles
- Glow effects (shadow blur) enabled when quality is high
- Player blinks when invulnerable; laser weapon has glow aura

#### Accessibility
- Reduced-motion preference: toggles animations, screen shake, flashing
- Screen-reader announcements via `announce()` to `#srLive` element
- High-contrast text shadows for visibility

#### Performance Tuning
- Adaptive quality scaling: tracks frame time, reduces particles/shadow blur if FPS drops
- `quality` variable: 0=full, 1=medium, 2=low
- Adjusts particle count and glow effects based on performance

## Key Game Mechanics

### Difficulty Levels
Modify `DIFFICULTIES` object (line 485) to tune:
- `spawnBase`, `spawnMin`: frames between enemy spawns (smaller = more frequent)
- `enemySpeed`: multiplier on enemy speed (1.0–1.9)
- `enemyShootChance`: probability enemy will shoot (0.45–0.95)
- `lives`, `bombs`: starting resources
- `bossHpMul`: multiplier on base boss HP

### Weapon System
- **Vulcan** (default): instant hitscan spread fire (1–3 bullets depending on level)
  - Level 1: single bullet
  - Level 2: two bullets
  - Level 3: three bullets
- **Laser**: pierce through enemies, fires 1–3 beams depending on level
  - Level 1: one beam
  - Level 2: two beams
  - Level 3: three beams + reduced cooldown
- Cycle weapons via power-up; upgrades level or swaps weapon + resets level to 1

### Scoring & Combo Chain
- Kill enemies to build a combo chain (up to 10× multiplier at 50+ kills)
- Chain timer resets to 2.2s per kill; breaks if no kills for 2.2s
- Enemies worth 10–35 base points × multiplier
- Boss kill: +500 points (always, before multiplier)
- Bomb kills: +5 points each

### Stage Progression
- 18 kills per stage before boss warning
- Boss spawns ~1.2s after 18th kill
- Boss defeat advances stage, resets enemy spawn timer, cycles theme and boss design

## Extending the Game

### Add a New Enemy Type
1. Add to `ENEMY_TYPES` (line 502):
   ```javascript
   newType: { w: 40, h: 40, hp: 2, speedMul: 1.3, color: '#00ff00', dark: '#008000', score: 25, minStage: 3 }
   ```
2. Add an explosion sound to `sfx` (line 624):
   ```javascript
   explodeNewType: (x) => { /* beep/noiseBurst calls */ }
   ```
3. Map it in `EXPLODE_SFX` (line 678):
   ```javascript
   newType: 'explodeNewType'
   ```

### Add a New Boss Pattern
1. Define pattern function in `BOSS_PATTERNS` (line 1372):
   ```javascript
   b => { /* populate enemyBullets array */ }
   ```
2. Add or edit a boss in `BOSSES` (line 508) with colors and name

### Add a New Theme
Insert into `THEMES` (line 493):
```javascript
{ sky: ['#color1', '#color2', '#color3'], cloud: 'r,g,b', sun: 'r,g,b', name: 'Theme Name' }
```

### Adjust Difficulty / Balance
- Tweak `DIFFICULTIES` spawn and speed values
- Adjust `BOSS_PATTERNS` firing speeds or counts
- Change base boss HP calculation in `spawnBoss()` (line 1046)
- Modify weapon fire rates in `update()` (line 1177)

## Storage Keys

Game data stored in browser `localStorage`:
- `skyfighter.bestScore`: best score (number)
- `skyfighter.muted`: mute state (0 or 1)
- `skyfighter.difficulty`: last selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: reduced-motion toggle (0 or 1)

## Browser Compatibility

Requires:
- Canvas 2D context
- Web Audio API (graceful fallback to muted)
- `localStorage` for persistence
- ES6 support (arrow functions, const/let, template literals, for…of)

Tested on modern Chrome, Firefox, Safari, Edge.

## Performance Notes

- Adaptive quality system reduces visual effects under load
- No external assets (images, audio files)—everything is code-generated
- ~60 FPS target; performance degrades gracefully on older devices
- Particle count scales with frame budget
- Shadow effects (glow) disabled at lower quality settings
