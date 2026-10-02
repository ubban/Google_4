# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 game. It features procedurally generated audio (no external assets), touch and keyboard controls, multiple weapon types, boss battles, and difficulty settings. The entire game is contained in `index.html` (~1670 lines).

## Getting Started

- **Run locally**: Open `index.html` in a web browser (no build step required)
- **Development**: Edit `index.html` directly, then refresh the browser to test changes
- **No dependencies**: The game uses only vanilla JavaScript and Web APIs (Canvas 2D, WebAudio API, localStorage)

## Code Architecture

### Overall Structure

The `index.html` file is organized in this order:

1. **HTML Markup** (lines 1-420): DOM elements for game canvas, HUD, buttons, touch controls
2. **CSS Styling** (lines 10-419): Canvas styling, HUD layout, animations, responsive design
3. **JavaScript** (lines 422-1669): Game logic, all classes and functions

### Game Systems

#### 1. Core Game Loop
- **`loop(now)`** - Main animation frame loop that calls `update()` then `draw()`
- **`update(dt)`** - Updates all game state each frame (player position, bullets, enemies, collision detection)
- **`draw()`** - Renders everything to canvas (background, enemies, player, bullets, particles, UI)
- **`resize()`** - Handles canvas scaling for different screen sizes (responsive design)

#### 2. Player & Combat System
- **Player state variables**: `playerX`, `playerY`, `playerVel`, `playerHp`, `playerMaxHp`, `bombs`, `maxBombs`, `weaponType`, `weaponLevel`
- **`fireWeapon()`** - Handles weapon firing based on current weapon type
- **`useBomb()`** - Activates smart bomb clearing enemies and triggering boss knockback
- **Weapon types** (in `WEAPON_CONFIGS`): 
  - NORMAL (single spread shots)
  - RAPID (fast narrow shots)
  - LASER (wide fan pattern)
  - TRINITY (triple parallel beams)
  - Each has unique firing pattern function (e.g., `fireRadial()`, `fireAimedSpread()`)

#### 3. Enemy System
- **Enemy array** tracks active enemies with position, type, health, velocity, target
- **`spawnEnemy()`** - Creates random enemies based on stage and difficulty
- **Enemy types** (in `ENEMY_TYPES`): grunt, interceptor, tank with different colors, speeds, HP, and scores
- **Collision detection**: `rectsOverlap()` checks player bullets vs. enemies, player vs. enemies
- **Scoring**: Different enemies award different points; chain multiplier increases with consecutive hits

#### 4. Boss System
- **Boss phases**: Each boss spawns at specific stages, has multiple attack patterns
- **6 Boss types** (in `BOSSES`): VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- **Attack patterns** (in `updateBoss()`): Each pattern uses different firing functions
  - `fireRadial()` - Circular burst
  - `fireColumns()` - Vertical columns
  - `fireAimedSpread()` - Spread toward player
  - `fireHomingSeeded()` - Homing projectiles
  - Boss attacks have 0.3-0.4s warning time before firing (player telegraph sound)
- **Boss rendering**: `drawBossShip()` draws multi-part boss sprites with hull, wings, core
- **`killBoss()`** - Boss defeated; plays stage clear sound and spawns power-up

#### 5. Audio System (WebAudio Procedural)
- **`ensureAudio()`** - Initializes AudioContext with dynamic compression
- **`beep(freq, dur, type, vol, opts)`** - Creates square/sawtooth/triangle wave sounds with optional frequency sweep and filters
- **`noiseBurst(dur, vol, opts)`** - Creates explosion-like noise using white noise buffer
- **SFX object** contains all sound effects:
  - Weapon sounds: `shoot`, `laser`
  - Impact sounds: `explode`, `hit`, `bossHit`
  - Power-up & feedback: `powerup`, `uiClick`
  - Boss events: `bossTelegraph`, `bossDown`, `stageClear`, `gameover`
- **Panning**: `panFor(x)` calculates stereo pan based on screen X position
- **Muting**: Audio respects `muted` flag; `ensureAudio()` checks this before playing

#### 6. Input System
- **Keyboard**: Arrow keys for movement, Space to fire, P/Esc to pause, X/Shift to bomb
- **Touch/Mobile**: On-screen virtual joystick (`joystickStart/Move/End`) for movement, tap-to-fire, bomb button
- **Joystick state**: `joyActive`, `joyOrigin`, `joyVec` track the virtual joystick
- **`canvasPointFromEvent(e)`** - Converts browser coordinates to canvas coordinates

#### 7. Visual Effects
- **Particles**: `particles` array stores explosion sprites with velocity and fade-out
- **`spawnExplosion(x, y, color, count)`** - Creates particle bursts
- **Screen shake**: `screenShake(mag, dur)` applies canvas translation offset in draw loop
- **Flash overlays**: `flashHit()` and `flashBomb()` trigger CSS animation classes
- **Vignette effect**: Shows critical health state with red vignette
- **Combo pop-ups**: `showComboPop()` displays multiplier text
- **Stage banners**: `showStageBanner()` shows stage clear text
- **Cloud parallax**: `drawClouds()` renders scrolling background layers with different speeds

#### 8. Game State & Persistence
- **Current game state**: `running`, `paused`, `gameOver`, `stage`, `stageTime`, `stageEnemySpawned`
- **Player progression**: `currentScore`, `bestScore` (stored in localStorage)
- **Difficulty**: `DIFF` object with multipliers; `setDifficulty()` adjusts enemy spawn rates and damage
- **Preferences stored in localStorage**:
  - Best score, mute state, difficulty level, reduced motion preference
- **`resetGame()`** - Initializes all state for new game

#### 9. Power-ups
- **`maybeSpawnPowerUp(x, y)`** - Random chance to spawn when enemy dies
- **Power-up types**: weapon upgrades (each type adds damage level), health restore, bombs
- **`applyPowerUp(type)`** - Applies effect to player (upgrades weapon or restores health/bombs)

#### 10. Game Flow
- **`endGame()`** - Triggered on game over (0 lives); shows final screen
- **`loseLife()`** - Decrements health, plays hit sound, checks game over
- **`togglePause()`** - Pauses/unpauses game, handles audio context
- **`addScore(base)`** - Updates score with chain multiplier; plays score SFX
- **`breakChain()`** - Resets multiplier when time between shots is too long

### Key Constants & Configuration

- **`BASE_W`, `BASE_H`** (480x720) - Game canvas base resolution
- **`WEAPON_CONFIGS`** - Array of weapon definitions (firing functions, upgrade bonuses)
- **`ENEMY_TYPES`** - Enemy stat object (width, height, HP, speed, color, spawn stage)
- **`BOSSES`** - Boss definitions (colors, names, HP multiplier)
- **`DIFF`** - Difficulty multipliers for spawn rate and damage scaling

### Performance Considerations

- **Particle pooling**: Particles are created/destroyed frequently; consider pre-allocating if performance becomes an issue
- **Canvas transforms**: Screen shake uses canvas translation; no performance impact at scale
- **WebAudio**: Procedural sounds use minimal CPU; muting disables audio generation entirely
- **Render batching**: All canvas drawing happens in single `draw()` pass

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with stats (width, height, HP, color, score, minStage)
2. Optionally add explosion SFX to `EXPLODE_SFX` mapping
3. Update `spawnEnemy()` if you need special spawn conditions
4. Add custom drawing code in `update()` if needed (most use `fillRect` with color)

### Adding a New Boss Attack Pattern
1. Create a new firing function (e.g., `fireNewPattern(b, speed)`)
2. Add the pattern to the boss's pattern list in `updateBoss()`
3. Test timing and difficulty by adjusting speeds and counts

### Adding a New Weapon Type
1. Add entry to `WEAPON_CONFIGS` with a unique name
2. Create firing function (e.g., `fireNewWeapon()`) that spawns bullets
3. Update `weaponLabel()` to show weapon name in HUD
4. Bullets should be added to `playerBullets` array with `{x, y, vx, vy, type}`

### Adjusting Difficulty
- Edit the `DIFF` object to change spawn rates and damage multipliers
- Boss spawn timing is in `updateBoss()` under stage checks
- Enemy speeds are in `ENEMY_TYPES` under `speedMul`

## Testing & Debugging

- **Browser DevTools Console**: Check for audio errors or missing AudioContext
- **Reduced Motion**: The game respects `prefers-reduced-motion` media query
- **Mobile Testing**: Test touch controls with browser DevTools device emulation
- **Sound Testing**: Toggle mute button to verify all SFX play correctly
- **Difficulty Testing**: Use difficulty buttons to verify spawn rate and damage changes
- **Profiling**: Monitor frame rate with Chrome DevTools Performance tab

## Known Limitations & Future Improvements

- Single HTML file limits code organization (but keeps deployment trivial)
- Particle count could impact performance on older devices
- Boss patterns are hardcoded; parameterized pattern generation would add flexibility
- No mobile app (web-only, but works full-screen on mobile browsers)

## Git Workflow

- Commit changes directly to the feature branch `claude/inspiring-planck-3rrse5`
- Test game functionality in browser before committing
- Keep commits focused on specific features or bug fixes
