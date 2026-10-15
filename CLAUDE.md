# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file HTML5 vertical shoot-em-up game (Raiden-style). The entire game is contained in `index.html` with no external dependencies—all CSS, JavaScript, and game assets are embedded inline.

## Quick Start

- **Run the game**: Open `index.html` directly in a web browser
- **No build process**: This is a vanilla HTML5 project with no build tools, transpilers, or package managers
- **No development server required**: Works entirely client-side

## Code Architecture

The game code is organized as a single monolithic IIFE (Immediately Invoked Function Expression) with the following major systems:

### 1. **Game State & Constants** (Top of JS block)
- `STORAGE_*`: localStorage keys for persistence (best score, mute, difficulty, reduced motion)
- `DIFFICULTIES`: Three difficulty presets (easy, normal, hard) affecting enemy spawn rates and damage
- `ENEMY_TYPES`: Configuration for grunt, interceptor, and tank enemies with HP and behaviors
- `BOSSES`: Six boss encounter definitions with unique attack patterns (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- `THEMES`: Three visual themes with different color schemes
- Global mutable state: `running`, `paused`, `plane`, `enemies`, `boss`, `bullets`, `enemyBullets`, `powerUps`, `particles`, `score`, `kills`, `chain`, `quality`

### 2. **Rendering System**
- `draw()`: Main render loop that clears canvas and draws all game entities
- `drawPlane()`: Generic plane sprite renderer (used for player, enemies, bosses)
- `drawBossShip()`: Boss-specific renderer with special effects
- Resolution-independent rendering using viewport scaling (`resize()` function handles canvas sizing)
- Visual effects: screen shake, flash overlays, vignette during damage, combo popups, stage banners

### 3. **Game Loop & Update**
- `loop(now)`: requestAnimationFrame callback, manages delta time and calls `update(dt)`
- `update(dt)`: Main game logic—handles movement, collision, weapon fire, enemy AI, boss patterns
- Adaptive performance tracking (`trackPerf(dt)`) adjusts visual quality if frame time exceeds threshold

### 4. **Input Handling**
- **Keyboard**: WASD or arrow keys for movement, Space for bomb, P to pause
- **Mouse**: Plane glides toward pointer (secondary input, keyboard takes priority)
- **Touch**: Left zone for movement joystick, right zone for fire button, separate bomb button
- `joystickStart()`, `joystickMove()`, `joystickEnd()`: Touch input management
- Mouse steering only active when pointer hovers canvas

### 5. **Weapon & Combat System**
- Four weapon types: VULCAN, LASER, SPREAD, HOMING (with levels 1–3)
- `fireWeapon()`: Dispatches to appropriate firing function based on current weapon
- `fireRadial()`, `fireAimedSpread()`, `fireColumns()`: Boss attack patterns
- Each weapon upgrades through power-ups; max level caps at 3
- `rectsOverlap()`: Collision detection for bullets, powerups, and enemies

### 6. **Boss AI & Attack Patterns**
- Each boss defined in `BOSSES` array with `maxHp`, `x`, `y`, `weapons` (array of attack functions)
- Attack functions called in sequence during boss encounter
- Boss-specific patterns: radial bursts, homing missiles, teleport dashes, expanding rings
- Boss health bar displayed at top during fight (`#bossBarWrap`)

### 7. **Audio System** (WebAudio Synthesis, No External Files)
- `beep()`: Synth sound generator with frequency sweep, filter envelope, panning
- `noiseBurst()`: White noise effects for explosions
- Sound effects synthesized in real-time (no MP3/WAV files)
- `ensureAudio()`: Initializes audio context on first interaction (browser autoplay policy)
- Mute toggle persists to localStorage; button in HUD top-right

### 8. **UI & Overlay System**
- Start/game-over overlay controlled by `#overlay` element
- Game stats in HUD: score, best score, combo multiplier, weapon name/level, hearts (lives), bombs
- Difficulty selection buttons (easy, normal, hard) with `setDifficulty()`
- Icons in top-right: mute, reduced motion toggle, pause
- Screen-reader live region (`#srLive`) for accessibility announcements

### 9. **Persistence & Settings**
- Best score, mute state, chosen difficulty, and reduced motion preference saved to localStorage
- `STORAGE_*` constants define keys
- Accessed via `localStorage.getItem()` / `setItem()`

## Key Functions for Modification

### Game Mechanics
- `resetGame()`: Initialize new game, clear all entities, reset score/lives/bombs
- `applyPowerUp(type)`: Handle life, bomb, weapon powerup acquisition
- `addScore(base)`: Award points with combo multiplier, update chain counter
- `breakChain()`: Reset combo multiplier when player loses momentum

### Enemy Spawning & AI
- `spawnEnemy()`: Randomly pick enemy type based on difficulty and kill count
- `spawnBoss()`: Transition to boss fight, setup boss health bar
- `maybeSpawnPowerUp(x, y)`: Drop powerup after enemy kill (weighted probability)

### Visual Effects
- `screenShake(mag, dur)`: Trigger screen wobble effect
- `flashHit()`, `flashBomb()`: Brief white or full-screen flashes
- `spawnExplosion(x, y, color, count)`: Particle burst effect
- `showComboPop()`, `showStageBanner()`: Animated text overlays

### Difficulty & Balance
- Edit `DIFFICULTIES` object to adjust spawn rates and damage per difficulty
- Edit `ENEMY_TYPES` and `BOSSES` for HP, behavior, and loot tables
- Weapon damage configured in attack functions; adjust projectile counts and speeds

## Testing & Debugging Tips

- **Browser DevTools**: Open Console to inspect `window` globals (all game state is exposed in IIFE scope)
- **Visual Quality**: Press the motion toggle button (🎆) to disable screen shake and flash for testing on lower-end devices
- **Frame Rate**: Quality setting auto-downgrades if frame time exceeds ~32ms (trackPerf function)
- **Difficulty Testing**: Start game, press P to pause, reload page with different `?difficulty=hard` param (not implemented but storable)
- **High Score Reset**: Open DevTools Console: `localStorage.clear()` (clears all saved data)

## Responsive Design

- Canvas scales to fit viewport while maintaining 480×720 base aspect ratio
- Touch controls optimized for mobile with large hit zones
- Safe area insets respect notches and system UI on edge devices
- Viewport units (clamp, vw, vh) ensure text scales appropriately across screen sizes

## Future Expansion Points

- **New Enemy Types**: Add to `ENEMY_TYPES` with movement/firing logic in `update()`
- **New Boss Patterns**: Add to `BOSSES` array with attack functions, call new patterns in boss update code
- **New Weapons**: Add weapon type to switch statement in `fireWeapon()` and rendering in `draw()`
- **Scoring Tiers**: Adjust multiplier formula in `addScore()` and chain break threshold in `breakChain()`
- **Themes**: Add new theme object to `THEMES` array and update color references in CSS and draw functions
