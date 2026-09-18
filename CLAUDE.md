# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single HTML file with embedded CSS and JavaScript. No build process or dependencies required—just open `index.html` in a browser to play.

## Quick Start

To run the game locally:
- Open `index.html` directly in a modern web browser
- Or serve via any HTTP server: `python -m http.server`, `npx http-server`, etc.

No installation, build, or compilation needed.

## Architecture

The entire game lives in `index.html` as a self-contained web application:

- **CSS (lines 10–359)**: Styling, layout, animations (canvas scaling, HUD, UI overlays, reduced-motion support)
- **HTML (lines 361–420)**: Minimal DOM structure (canvas, overlays, buttons, touch controls)
- **JavaScript (lines 422–1667)**: Core game logic in a single IIFE

### Game State

Key variables track the active game:
- `player`: Position, dimensions, weapon, level, invulnerability timer
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: Entity arrays
- `boss`: Active boss state (position, HP, attack pattern)
- `score`, `lives`, `stage`, `chain`, `multiplier`: Game metrics

### Core Systems

**Rendering Pipeline** (`draw()`, `drawPlane()`, `drawBossShip()`)
- Linear gradient sky with animated clouds (parallax effect via frame count)
- Canvas rendering with shadow/glow when quality is high
- Dynamic quality levels (0 = full, 1 = medium, 2 = low) based on frame time

**Game Loop** (`loop()`)
- RequestAnimationFrame-driven at browser refresh rate
- Caps delta-time to 50ms to prevent spiral on frame drops
- Tracks performance and degrades quality automatically if FPS drops below ~40

**Collision Detection** (`rectsOverlap()`)
- Simple AABB (axis-aligned bounding box) collision for all entities
- Bullets vs enemies/boss, enemy bullets vs player, enemies vs player, power-ups vs player

**Input Handling**
- Keyboard: Arrow keys or WASD for movement, Space/X/Shift to fire/bomb, P/Esc to pause
- Mouse: Move to steer, click to fire (priority over keyboard when mouse is active)
- Touch: Virtual joystick on left half, Fire/Bomb buttons on right half

**Audio** (WebAudio synthesis, no external assets)
- Initializes on first user interaction (user gesture required)
- `beep()` for tonal sounds with frequency sweeps and filters
- `noiseBurst()` for noise-based explosions
- Stereo panning based on x-position; dynamic compressor for loudness control
- Each sound effect defined in `sfx` object and called by name

**Difficulty System** (`DIFFICULTIES` object)
- Three presets: easy, normal, hard
- Affects spawn rates, enemy speed, shoot frequency, lives, bombs, boss HP
- Persisted to localStorage

### Entity Management

**Enemy Spawning** (`spawnEnemy()`)
- Random position on x-axis, spawns above screen
- Type determined probabilistically: grunt (common), interceptor (faster, weaving), tank (tanky, 4 HP)
- Each enemy has a shoot timer and pattern (spread or aimed)
- Tank enemies have visible HP bar when damaged

**Weapon System** (`fireWeapon()`)
- **Vulcan**: Fast bullets with spread pattern (widens at higher levels)
- **Laser**: Slower, piercing bullets in lane configuration; can hit multiple enemies
- Switched by collecting weapon power-ups; level resets when switching

**Boss Encounters** (`spawnBoss()`, `updateBoss()`, `BOSS_PATTERNS`)
- Six boss designs with unique visual styles
- Each boss has 3 distinct attack patterns that cycle
- Boss telegraphs before each attack (visual core pulse + sound)
- Attack patterns: radial bursts, aimed spreads, homing missiles, teleport barrage, etc.
- Player must collect 18 enemy kills per stage to trigger boss

**Power-Ups** (`maybeSpawnPowerUp()`, `applyPowerUp()`)
- **Life** (6% chance): Restore a health point
- **Bomb** (8% chance): Add a smart bomb (clears bullets + damages all enemies/boss)
- **Weapon** (20% chance): Upgrade weapon level or switch weapon type
- All spawned by defeated enemies and fall slowly toward bottom

**Combo/Chain System** (`addScore()`, `breakChain()`)
- Multiplier increases: 1x baseline → 2x at 5 kills → 3x at 10 kills (up to 10x)
- Chain resets if player takes damage or doesn't kill for 2.2 seconds
- Displayed in HUD; affects score calculation

### Visual & Audio Polish

- **Screen shake** (`screenShake()`): Magnitude and duration depend on event severity; reduced by 85% if reduced-motion enabled
- **Flash effects** (`flashHit()`, `flashBomb()`): White flash on-screen for impact feedback; suppressed in reduced-motion
- **Particle system**: Colored particles spawned on explosions; fade out over 0.5s
- **Combo pop** (`showComboPop()`): Animated text showing kill milestone (every 10 kills)
- **Stage banner** (`showStageBanner()`): Slides in when stage changes or boss approaches
- **Boss health bar**: Shows real-time boss HP as a gradient bar
- **Reduced motion** support: Respects `prefers-reduced-motion` media query; can be toggled in-game

### Persistence

LocalStorage keys (prefixed `skyfighter.`):
- `bestScore`: High score
- `muted`: Audio mute toggle (0/1)
- `difficulty`: Last selected difficulty (easy/normal/hard)
- `reducedMotion`: Reduced motion preference (0/1)

## Common Development Tasks

**Add a new enemy type:**
1. Add entry to `ENEMY_TYPES` object with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Optionally add explosion SFX to `EXPLODE_SFX` mapping
3. Optionally add spawn chance logic in `spawnEnemy()`

**Add a new boss:**
1. Add entry to `BOSSES` array with hull/wing/core colors and name
2. Add attack pattern function array to `BOSS_PATTERNS` at matching index
3. Boss stats (HP, position) scale automatically by stage and difficulty

**Add a new weapon or modify existing:**
- Edit `fireWeapon()` function: bullet count, speed, pierce count, spread angles
- Adjust `fireDelay` constant to tune rate of fire
- Add SFX call with new sound effect

**Modify audio:**
- Edit existing SFX functions in `sfx` object or add new ones
- Use `beep(freq, duration, type, volume, opts)` for tones
- Use `noiseBurst(duration, volume, opts)` for noise effects
- Adjust filter frequencies, panning, and delay parameters for character

**Adjust visual polish:**
- Screen shake: `screenShake(magnitude, duration)` parameters
- Quality/glow threshold: `const glow = quality === 0` can be tuned
- Theme colors: Edit `THEMES` array for gradient colors and cloud appearance

## Technical Notes

- **Canvas size**: 480×720px (12:18 aspect ratio) with dynamic CSS scaling to fit viewport while preserving aspect ratio
- **Performance**: Adaptive quality degrades particle count and shadow/glow effects on low-end devices
- **Collision box offset**: Hitboxes use half-width and half-height centered on entity position (AABB around center)
- **Radial bullets** (boss and special patterns): Stored with `dx`/`dy` velocity instead of direction angle for efficiency
- **Boss state machine**: Uses `entering` flag, `telegraph` timer, and `attackTimer` to coordinate visual and attack phases
- **Audio resumption**: WebAudio context must be resumed on user interaction (security policy in modern browsers)

## Design Constraints

- Single file, no external assets: Game must be self-contained (all audio is synthesized, no images, no fonts beyond system)
- Mobile-first responsive: Canvas scales, touch controls are primary on mobile, fallback to keyboard/mouse on desktop
- Accessibility: Screen reader live region for announcements; reduced-motion support; keyboard controls for all gameplay
