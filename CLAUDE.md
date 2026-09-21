# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a browser-based vertical shoot-'em-up game (Raiden-style) implemented as a single HTML file (1669 lines). The game features:
- Player-controlled aircraft with multiple weapon types
- Procedurally spawned enemy waves and stage bosses
- Power-ups (health, bombs, weapons)
- Progressive difficulty levels
- Touch/joystick support for mobile
- Sound effects and visual feedback (screen shake, flash, vignette)

## Running & Development

**To play the game:**
- Open `index.html` directly in a web browser (no build or server required)
- Responsive design supports desktop, tablet, and mobile

**No build tools or dependencies.** The entire game is self-contained in one file. Changes to `index.html` are immediately visible when you refresh the browser.

## Architecture: Game Systems

The game is structured as a monolithic JavaScript IIFE with these core systems:

### Game State & Loop
- `running`, `paused`, `gameOver` flags control game flow
- `loop()` function runs via `requestAnimationFrame`, calls `update(dt)` then `draw()`
- `update(dt)` advances entity positions, checks collisions, spawns enemies/bosses
- `draw()` renders everything to canvas using 2D context

### Entities
- **Player** (`player` object): position, health, bombs, weapon choice; updates via keyboard/touch input
- **Enemies** (`enemies` array): spawned by `spawnEnemy()`; types defined in `ENEMY_TYPES` constant (fighter, tank, zigzag); each has HP, spawn chances increase with difficulty
- **Boss** (`boss` object): single stage boss; spawned at score thresholds via `spawnBoss()`; multiple weapons, complex attack patterns
- **Projectiles** (`shots` array): bullets from player; `playerBullets` and `bossBullets` tracked separately
- **Power-ups** (`powerUps` array): life, bombs, weapons; spawned by `maybeSpawnPowerUp()` when enemies die
- **Particles** (`particles` array): visual effects; created by collisions and explosions

### Weapons System
- Tracked in `player.weaponIdx` (0=cannon, 1=spread, 2=laser, etc.)
- Weapon parameters stored in `WEAPONS` array with fire rate, bullet speed, behavior
- Weapon switching via `W` key; displays current weapon in HUD
- Multiplier (`multiplier`) increases with consecutive hits, resets on miss

### Collision Detection
- `rectsOverlap(a, b)` checks bounding box intersection
- Player collision with enemies deals damage
- Player bullets destroy enemies; boss bullets damage player
- Pick-ups collected when player overlaps them

### Visual Rendering
- All drawing via canvas 2D context in `draw()` function
- Sprite-like shapes drawn procedurally: `drawPlane()` for player/enemies, `drawBossShip()` for bosses
- Boss health bar (`#bossBarFill`), HUD stats, combo pop-ups
- Screen shake effect (`shakeTime`, `screenShake()`) applied to camera translation
- Visual effects: vignette (red pulse on damage), white flash on hit, particle explosions

### Audio System
- Lazy-initialized via `ensureAudio()` (required for Web Audio on user interaction)
- Mono oscillator synth: `beep(freq, dur, type, vol)` for tones
- Noise burst: `noiseBurst(dur, vol)` for explosion sounds
- Pan audio based on screen position: `panFor(x)`
- Mute toggle via button

### Input Handling
- **Keyboard**: Arrow keys or WASD for movement, W to switch weapon, Space to fire, P to pause, M to mute
- **Touch/Mobile**: Visual joystick in bottom-left; drag to move, tap elsewhere to shoot
- `joystickStart/Move/End()` manage touch input state

### Difficulty & Progression
- `difficulty` levels (0=Easy, 1=Medium, 2=Hard, 3=Brutal); affects enemy spawn rates, health, bullet damage
- Set via difficulty buttons or `setDifficulty()`
- Score targets trigger boss spawns at `BOSS_THRESHOLDS`
- Game state saved to `localStorage` (high score, difficulty)

### UI & Overlays
- Start/title screen: `#overlay` with difficulty selection
- Game over screen: shows final stats, new best score highlight
- HUD during play: health hearts, bombs remaining, weapon type, multiplier, boss health bar
- Status elements: `#hud`, `#comboPop`, `#stageBanner`, `#bossBarWrap`

## Key Constants & Tuning

Located at the top of the game loop, easy to adjust:
- `CANVAS_W`, `CANVAS_H`: game resolution
- `ENEMY_TYPES`: object with fighter, tank, zigzag definitions (HP, speed, fire rate, spawn chance)
- `WEAPONS`: array with cannon, spread, laser, etc. (fire rate, bullet speed, spread angle)
- `BOSS_THRESHOLDS`: score values that trigger boss spawns
- `DIFFICULTY_MODS`: multipliers for enemy spawn rate, health, bullet damage per difficulty level

## Making Changes

### Adding a new weapon type
1. Add entry to `WEAPONS` array with desired properties
2. Update weapon cycling logic if needed
3. Particle effects and sounds are automatic based on weapon fire rate

### Adjusting difficulty
- Modify `ENEMY_TYPES`, `WEAPONS`, or `DIFFICULTY_MODS` constants
- Tweak spawn rates: `Math.random() < spawnChance * difficultyMod`

### Visual tweaks
- CSS in `<style>` tags: colors, animations, HUD layout
- Canvas drawing: modify `draw()`, `drawPlane()`, `drawBossShip()`
- Screen shake: adjust `screenShake(mag, dur)` calls

### Adding new enemy or boss behaviors
- Create new type in `ENEMY_TYPES` or modify `spawnBoss()`
- Update in `update()` where enemies move and fire
- Collision & rendering already generic

### Performance notes
- Particle count and collision checks are the main bottleneck
- `trackPerf(dt)` logs FPS to console for debugging
- `dt` capped at 50ms per frame to prevent lag-related skips

## Testing Tips

- Open browser DevTools (F12) → Console to see perf logs
- Adjust `DIFFICULTY_MODS` to quickly test different difficulty curves
- Comment out `ensureAudio()` call if audio causes issues on some browsers
- Test touch controls on mobile or use Chrome DevTools device emulation
