# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game (HTML, CSS, and JavaScript) is contained in `index.html`. There is no build process, bundler, or external dependencies—the file runs directly in any modern browser.

## Game Architecture

### Core Systems

The game is organized around several key systems that work together in the main game loop:

1. **Entity Management**: The game tracks four main entity arrays:
   - `player`: The player's aircraft (single entity)
   - `enemies`: Enemy ships that spawn and descend
   - `bullets`: Player projectiles (either vulcan rounds or laser)
   - `enemyBullets`: Projectiles fired by enemies and the boss
   - `powerUps`: Collectible upgrades (weapons, health, bombs)
   - `particles`: Visual explosion/debris particles
   - `boss`: The stage boss (single entity, exists only during boss fights)

2. **Game State**: Core state is tracked in global variables:
   - `stage`: Current stage number (increases after each boss)
   - `kills` / `stageKills`: Enemy kills for stage progression
   - `score` / `multiplier`: Score and combo multiplier
   - `lives` / `bombCount`: Player health and bomb count
   - `running` / `paused` / `gameOver`: Game phase flags

3. **Difficulty System**: Three difficulty levels (easy/normal/hard) with configuration tables (`DIFFICULTIES`) that affect:
   - Enemy spawn rates and minimum spawn intervals
   - Enemy movement speed and shooting frequency
   - Player lives and bomb count
   - Boss health scaling

4. **Rendering**: Canvas-based 2D graphics with:
   - Procedural plane/boss drawings (using Canvas shapes, no sprites)
   - Parallax scrolling clouds with theme-based colors
   - 6 visual themes that cycle with stages
   - Screen shake, flash, and vignette effects for impact feedback
   - Adaptive quality system that scales particle count and shadow effects based on frame time

### Data Structures

**Configuration Objects** (defined at the top of the script):
- `DIFFICULTIES`: Spawn rates, speeds, health multipliers per difficulty
- `THEMES`: 6 sky themes with gradient colors, cloud appearance, and names
- `ENEMY_TYPES`: Grunt, Interceptor, Tank with properties (size, HP, speed, score)
- `BOSSES`: 6 unique boss designs with visual colors and HP multipliers

**Entity Objects** (stored in arrays):
- `player`: `{x, y, w, h, speed, cooldown, rapid, weapon, level, invuln}`
- `enemies`: `{x, y, w, h, speed, hp, type, color, dark, score, pattern, canShoot, weave, ...}`
- `bullets`: `{x, y, w, h, vx, vy, laser}`
- `enemyBullets`: `{x, y, w, h, speed, dx, dy, radial, homing, ...}`
- `powerUps`: `{x, y, w, h, speed, type}` (types: 'life', 'bomb', 'weapon')
- `particles`: `{x, y, vx, vy, life, maxLife, color}`
- `boss`: `{x, y, w, h, hp, maxHp, design, vx, telegraph, attackTimer, patternIndex, ...}`

### Main Game Loop

The update cycle (`update(dt)`) processes in this order:
1. Update player position (from keyboard/mouse/joystick input)
2. Update player weapon cooldown and fire when appropriate
3. Spawn enemies on a timer (interval depends on difficulty)
4. Update all enemy positions and spawn their bullets
5. Update all player and enemy bullets
6. Update particles (explosion debris)
7. Update power-ups
8. Update boss position and attack patterns
9. Check collision between player and bullets/enemies/power-ups
10. Apply effects (screen shake decay, multiplier timeout, invulnerability)
11. Render everything via `draw()`

### Input Handling

Three input methods are supported simultaneously (priority: keyboard > joystick > mouse):
- **Keyboard**: Arrow keys or WASD to move, Space/click to fire, X/Shift to bomb, P/Esc to pause
- **Mouse**: Move mouse to steer, click to fire
- **Touch**: Left 55% of screen is joystick (drag to move), right 45% has FIRE/BOMB buttons

Touch is auto-detected; on touch devices, `#touchControls` is shown with visual buttons and joystick base.

### Audio System

Uses **Web Audio API** for procedural sound synthesis (no external audio files). Core functions:
- `beep(freq, dur, type, vol, opts)`: Generates tones with optional frequency sweep, filter, pan
- `noiseBurst(dur, vol, opts)`: Generates white noise with envelope and filter
- `sfx` object: Named sound effects (shoot, explode, hit, bomb, powerup, stageClear, etc.)

Audio is muted on preference stored in localStorage. User can toggle mute button in top-right corner.

### Visual Effects

- **Screen Shake**: `screenShake(mag, dur)` adds random camera jitter; magnitude/duration reduced in reduced-motion mode
- **Flash**: White screen flash on hit or bomb detonation
- **Vignette**: Red pulsing vignette appears when player has 1 life remaining
- **Combo Pop**: Large text animation showing combo multiplier when chain breaks
- **Stage Banner**: Large text showing stage number at stage transitions

All animations are disabled when reduced-motion preference is active.

### Boss System

Bosses enter after 18 enemy kills per stage. Each boss has:
- Unique visual design with colored hull, wings, and glowing core
- Pattern-based attack system (defined in `BOSS_PATTERNS`)
- Telegraph phase (2-3 seconds) before each attack
- Multiple attack patterns per boss (cycled through)

Boss patterns use helper functions to fire bullets:
- `fireRadial(b, count, speed)`: Radial burst
- `fireColumns(b, speed)`: Vertical columns
- `fireAimedSpread(b, count, speed, angle)`: Aimed spread at player
- `fireDoubleRing(b, innerSpeed, outerSpeed)`: Dual concentric rings
- `fireHomingSeeded(b, count, speed)`: Homing missiles

### Performance Optimization

The adaptive quality system (`trackPerf()`) monitors frame time and adjusts:
- `quality = 0`: Full effects (shadows, glow, 14 particles per explosion)
- `quality = 1`: Medium (6 particles, reduced shadows)
- `quality = 2`: Low (3 particles, no shadows)

This runs automatically based on average frame time over 45 frames.

## Common Development Tasks

### Tweaking Game Balance

- **Enemy spawn rate**: Edit `DIFFICULTIES[difficulty].spawnBase` and `.spawnMin`
- **Enemy speed/difficulty**: Edit `DIFFICULTIES[difficulty].enemySpeed` and `.enemyShootChance`
- **Boss health**: Edit `bossHpMul` in `DIFFICULTIES`; boss designs also have individual `hpMul` in `BOSSES`
- **Weapon damage**: Search for `addScore(base)` calls; "base" is damage per bullet class

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` object with properties: `w, h, hp, speedMul, color, dark, score, minStage`
2. Add optional `weave` property for side-to-side movement
3. Update `spawnEnemy()` random roll if needed (currently 15% tank, 25% interceptor, rest grunt)
4. Add explosion sound to `EXPLODE_SFX` object
5. Add corresponding sound function to `sfx` object if unique sound needed

### Adding a New Boss Pattern

1. Add new pattern function (e.g., `fireSpecialAttack(b)`) that spawns `enemyBullets`
2. Add pattern function to the boss's entry in `BOSS_PATTERNS`
3. Pattern is auto-called in rotation during boss telegraph phase

### Changing Visual Themes

- Edit `THEMES` array to add/modify sky gradients and cloud colors
- Each theme has `{sky: [top, mid, bottom], cloud: 'r,g,b', sun: 'r,g,b', name}`
- Themes cycle automatically per stage: `THEMES[(stage - 1) % THEMES.length]`

### Adjusting Difficulty Progression

Currently, enemies scale by stage (new enemy types unlock via `minStage` in `ENEMY_TYPES`). To make progression steeper:
- Increase enemy spawn rate over stages: modify spawn timer calculation in `update()`
- Increase enemy speed: multiply `enemy.speed` by `1 + (stage - 1) * 0.15`
- Increase boss health: multiply `baseHp * (1 + (stage - 1) * 0.2)`

### Running the Game

No build or server required—simply open `index.html` in a browser. Game persists best score, mute preference, difficulty choice, and reduced-motion setting in localStorage.

### Testing on Mobile

Open the HTML on a mobile device or use Chrome DevTools device emulation. Touch input automatically enables when detected. Test:
- Joystick responsiveness in move zone (left 55%)
- Fire/Bomb button taps in action zone (right 45%)
- Screen shake and animations in reduced-motion mode
- Performance on low-end devices (quality will adapt automatically)

## Key Files & Sections

- **Lines 493–515**: Configuration tables (THEMES, ENEMY_TYPES, BOSSES, BOSS_PATTERNS)
- **Lines 752–765**: Canvas sizing and DPI scaling
- **Lines 767–862**: Input handling (keyboard, mouse, touch, joystick)
- **Lines 863–911**: Game state and `resetGame()`
- **Lines 1022–1041**: `spawnEnemy()` and enemy type selection
- **Lines 1043–1056**: `spawnBoss()`
- **Lines 1087–1094**: Score/multiplier/chain system (`addScore()`)
- **Lines 1411–1439**: `updateBoss()` with attack pattern cycling
- **Lines 1520–1660**: Main `draw()` function with all rendering
- **Lines 1661–1669**: Game loop (`gameLoop()`) and `requestAnimationFrame`

## LocalStorage Keys

The game persists player preferences:
- `skyfighter.bestScore`: Best score ever achieved
- `skyfighter.muted`: '0' or '1' for audio preference
- `skyfighter.difficulty`: 'easy', 'normal', or 'hard'
- `skyfighter.reducedMotion`: '0' or '1' for accessibility setting
