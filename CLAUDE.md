# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 Canvas application. The game features progressive difficulty stages, weapon upgrades, power-ups, boss encounters, and both desktop and mobile controls.

## Quick Start

Since this is a single-file HTML game, no build process is required:

- **Run the game**: Open `index.html` directly in a web browser or use `python -m http.server 8000` and navigate to `http://localhost:8000`
- **Develop locally**: Edit `index.html` and refresh the browser to see changes
- **No build tools**: CSS and JavaScript are embedded directly in the HTML file

## Architecture

The codebase is organized within a single `index.html` file (1669 lines) containing:

### Structure Layers

1. **HTML (lines 1-421)**
   - Game canvas container
   - HUD overlays (score, lives, weapons, boss health bar)
   - Main menu/game-over screen (#overlay)
   - Touch control zones (#touchControls)
   - Screen reader announcements (#srLive)

2. **CSS (lines 10-358)**
   - Game styling with flexbox/grid layout
   - Canvas viewport management with responsive sizing
   - Animations: combo popups, stage banners, screen shake effects, vignette pulses
   - Touch button styling for mobile
   - Accessibility: `.sr-only` for screen readers, `reduced-motion` media query support
   - Theme: dark blue/sci-fi aesthetic with gradient skies

3. **JavaScript (lines 422-1669)**
   - Runs in IIFE to avoid global scope pollution
   - Game state and subsystems (see below)

### Core Game Systems

#### Audio System (starts ~line 585)
- **Web Audio API** implementation for sound effects
- `ensureAudio()`: Lazy initializes audio context (waits for user interaction)
- **SFX functions**:
  - `beep(freq, dur, type, vol, opts)`: Tonal sound effects (oscillator + filter + panning)
  - `noiseBurst(dur, vol, opts)`: White noise (explosions, bombs)
- **Effects library** (`sfx` object): `fire`, `powerUp`, `explodeGrunt`, `explodeTank`, `explodeInterceptor`, `bossShot`, `bossExpire`, `stageUp`
- Audio can be muted/unmuted (stored in localStorage)

#### Input System (starts ~line 637)
- **Desktop**: Mouse (move + click to fire), keyboard (arrows/WASD move, space fire, X/Shift bomb, P/Esc pause)
- **Mobile**: 
  - Left zone: drag to move (shows joystick base when dragging)
  - Right zone: FIRE and BOMB buttons
  - Joystick tracking in `joystickStart/Move/End` functions
- `canvasPointFromEvent()`: Converts mouse/touch events to canvas coordinates
- `keys` object: Tracks currently pressed keys

#### Game State Management
- **Difficulty levels** (`DIFFICULTIES` object): 
  - Easy, Normal, Hard with different spawn rates, enemy speeds, shoot chances, lives, bombs
  - Affects game balance multiplicatively (enemySpeed, spawnBase, etc.)
- **Themes** (`THEMES` array): 6 visual themes with sky gradients and cloud colors
- **Quality/Performance** (`quality` variable, `trackPerf()`): Adaptive quality scaling (0-2) based on frame time
- **Pause state**: Toggled via UI button or P/Esc key

#### Rendering System (`draw()` and related functions, starts ~line 1507)
- **Canvas rendering**: 2D canvas context (`ctx`) for all graphics
- **Sprite drawing**:
  - `drawPlane()`: Player ship with dynamic wing angles, colors, flame effects
  - `drawBossShip()`: Boss with circle tracking, shield visualization
  - Particles drawn via loops (bullets, enemies, explosions)
- **HUD updates**: Score, lives (hearts ♥), bombs (💣), weapon level, multiplier, boss health bar
- **Visual effects**:
  - `drawClouds()`: Parallax scrolling clouds for depth
  - Flash overlays (#flash.hit / .bomb) for damage/bomb feedback
  - Vignette pulse (#vignette) during damage
  - Combo popup animations (#comboPop, #stageBanner)
- **Screen shake**: Applied via canvas translation in `screenShake(mag, dur)`

#### Game Loop (`loop(now)`, line 1640)
- RequestAnimationFrame-based at ~60fps
- **Per-frame**: 
  1. Calculate delta time (dt)
  2. Call `update(dt)` for logic
  3. Call `draw()` for rendering
  4. Track performance with `trackPerf(dt)`

#### Update/Logic System (`update(dt)`, line 1148)
- **Player movement**: Smoothly toward mouse/joystick position with max speed
- **Spawning**: Enemy spawning on countdown; increases difficulty as stage progresses
- **Enemy updates**: Position, shooting, collision with player bullets
- **Bullet updates**: Move, check collisions with enemies/boss, despawn off-screen
- **Powerup updates**: Move, collision with player
- **Explosion particles**: Fade and despawn
- **Combo multiplier**: Resets to 1 if no kills within timeout (5 seconds)
- **Stage progression**: After `KILLS_PER_STAGE` (18) enemy kills, spawn boss and transition to next stage

#### Weapons System
- **Weapon types** (`ENEMY_TYPES.player.weapons`): 
  - VULCAN (1-3 levels): Rapid fire bullets
  - LASER (1-3): Laser beams
  - MISSILES (1-3): Homing projectiles
  - MORTAR (1-3): Arcing projectiles
- **Upgrades**: Weapon powerups increase level and ammo count (3 shots per pickup)
- **Firing patterns**: 
  - `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`
  - Boss uses different patterns per phase; player uses current weapon
- **Fire rate**: Controlled by `fireDelay` cooldown

#### Enemy System (starts ~line 1022)
- **Enemy types** (ENEMY_TYPES): grunt, interceptor, tank with different speeds, HP, shoot chances, point values
- **Spawn logic** (`spawnEnemy()`): 
  - Countdown-based spawning
  - Increases spawn frequency as stage progresses (minSpawn decreases)
  - Halts when boss is active
- **AI**: 
  - Fly down in loose formations
  - Shoot when they pass player X coordinate (chance-based)
  - Random horizontal drift
- **Collision**: With player (lose life) and with player bullets

#### Boss System (starts ~line 1043)
- **Boss phases**: Boss switches firing patterns as health depletes (3-4 patterns per boss, roughly every 25% HP)
- **Spawning**: After 18 enemy kills per stage, `spawnBoss()` creates a boss and increases `stage`
- **Health**: Scales by difficulty (`bossHpMul` varies by difficulty)
- **Firing patterns**: Boss has 3-4 firing methods per phase; all patterns spawn bullets in circular/spread formations
- **Death**: `killBoss()` shows stage banner, adds bonus score, spawns powerups, resets enemy spawn counter

#### Combo & Scoring System
- **Base points**: Enemies worth 10-100 points; bosses worth 5000
- **Multiplier**: Increases by 0.1x per 2 enemy kills (caps at ~2x); resets to 1 if no kills in 5 seconds
- **Bonus**: Stage completion adds base * stage * 100 bonus points
- **Display**: Combo popup shows on kill streaks
- **Best score**: Saved to localStorage (`STORAGE_BEST`)

#### Power-up System
- **Types** (`maybeSpawnPowerUp()`): 
  - Weapon powerups (drop on enemy kill; player picks up for weapon upgrade)
  - Bomb powerup (uncommon; increases bomb count)
  - Heart powerup (uncommon; restores 1 life)
- **Drop chance**: ~40% per enemy kill; type determined by rarity weights
- **Physics**: Gravity, max speed downward
- **Collision**: With player (apply effect immediately)

#### Accessibility Features
- **Screen reader support** (`srLive` element, `announce()` function): 
  - Announces score updates, stage changes, game over
  - Uses `aria-live="polite"` region
- **Reduced motion support** (`prefers-reduced-motion` media query): 
  - Disables animations (vignette, screen shake, combo pop, stage banner)
  - Toggle button (#motionBtn) to override preference
  - Stored in localStorage
- **Keyboard controls**: Full keyboard support (no mouse required)
- **Mobile touch**: Full touch controls with visual feedback

### Key Constants & Tuning

- `KILLS_PER_STAGE = 18`: Enemy kills before boss spawns
- `DIFFICULTIES`: Easy (spawnBase 75), Normal (58), Hard (40) — lower = more frequent spawns
- `THEMES`: 6 visual themes with different sky gradients
- Weapon cooldowns, enemy speeds, spawn ranges: Tune these to adjust game difficulty/pacing
- Screen shake magnitude and duration: In `screenShake()` calls throughout the code

## Modifying the Game

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` object (~line 501) with `speed`, `hp`, `shootChance`, `points`, `size`
2. Update `spawnEnemy()` (~line 1022) to randomly include the new type
3. Add explosion sound effect for the new type in `EXPLODE_SFX` (~line 509)
4. Update boss collision logic if needed (~line 1232)

### Adjusting Difficulty

- Modify `DIFFICULTIES` constants (spawn rates, enemy speeds, lives, bombs, boss HP)
- Adjust `KILLS_PER_STAGE` to change how quickly players face bosses
- Tune enemy shoot chances to change incoming fire pressure
- Adjust weapon fire rates in `fireWeapon()` (~line 1124) or individual fire pattern functions

### Changing Game Visuals

- **Theme**: Add to `THEMES` array with new sky colors and cloud tones (line ~493)
- **Colors**: HUD background/border colors in CSS (~line 34-48)
- **Animations**: Modify `@keyframes` in CSS (heartBeat, comboPulse, stagePulse, etc.)
- **Canvas rendering**: Modify `draw()` function (~line 1520) for ship/enemy appearance

### Adding Sound Effects

1. Create new beep/noise in Web Audio functions (beep/noiseBurst)
2. Add to `sfx` object as a function that plays the sound
3. Call `sfx.effectName()` when event occurs (e.g., `sfx.fire()` on player shot)
4. Mute/unmute respects global `muted` flag

## Common Tasks

**Tweak weapon fire rate**: In `fireWeapon()` (~line 1124), adjust `fireDelay` multiplier per weapon or globally.

**Change player speed**: Modify player movement in `update(dt)` (~line 1170) — currently `moveSpeed = 320` pixels/sec.

**Adjust enemy formation**: In `spawnEnemy()` (~line 1022), change `x` offset calculation for spacing.

**Add a new difficulty setting**: Duplicate an entry in `DIFFICULTIES`, adjust parameters, add to difficulty button HTML.

**Adjust stage progression**: Change `KILLS_PER_STAGE` constant or per-difficulty settings in `DIFFICULTIES`.

## Testing & Debugging

- **Browser DevTools Console**: Logs any runtime errors
- **Performance**: Check frame rate in the browser's performance profiler; game adaptively scales quality if FPS drops
- **Responsive testing**: Resize browser window or test on mobile device (touch controls auto-enable)
- **Accessibility**: Test keyboard and screen reader with VoiceOver/NVDA
- **Reduced motion**: Enable `prefers-reduced-motion: reduce` in browser DevTools (Accessibility tab) to verify animations disable
- **Audio debugging**: Check browser console for Web Audio API errors if sound doesn't work

## Browser Compatibility

- **Required**: HTML5 Canvas, ES6 JavaScript, Web Audio API
- **Tested**: Modern browsers (Chrome, Firefox, Safari, Edge)
- **Mobile**: iOS Safari, Android Chrome
- **Fallback**: Game works without sound if Web Audio is unavailable (audio initializes on user interaction)

## localStorage Keys

Game persists these to browser storage:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state (0/1)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Motion preference override (0/1)

## Performance Notes

- Single HTML file simplifies deployment
- Canvas rendering is efficient for the particle/entity count (~50-200 entities typical)
- Adaptive quality (`quality` variable) scales particle density and effect frequency if FPS drops below ~45fps
- Frame time tracking uses sliding window (45-frame buffer)
