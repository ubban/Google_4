# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game, implemented as a single-file HTML5 application. The entire game engine, assets, and UI are contained in `index.html` (1669 lines), with no external build tools or dependencies required.

## Running and Testing

The game runs directly in any modern web browser with no build step:
1. Open `index.html` in a browser
2. Click "Start Game" to begin
3. The game works on desktop (keyboard/mouse) and mobile (touch/motion controls)

**Testing approach:**
- Manual browser testing is the primary testing method (no test framework)
- Test gameplay across difficulty levels: Easy, Normal, Hard
- Verify touch controls and joystick input on mobile
- Check that high scores persist in localStorage across sessions
- Test audio in both muted and unmuted states

## Code Architecture

The game is structured as a single IIFE (Immediately Invoked Function Expression) with distinct functional systems:

### Core Game State
- **Difficulty system:** Three difficulty presets (easy, normal, hard) that affect enemy spawn rates, speeds, and shoot chances
- **Player data:** Lives, current weapon level, bomb count, health
- **Game progression:** 6 stages × ~18 kills per stage = full game completion; boss fights at stage boundaries
- **Persistence:** Best score, difficulty preference, muted state, and motion preference stored in localStorage using `skyfighter.*` keys

### Main Game Loop
- **`update(dt)`** – Updates all entities (player, enemies, projectiles, particles, explosions) and collision detection
- **`draw()`** – Renders the sky gradient, clouds, all entities, HUD, and overlays
- Game loop runs at target 60 FPS with adaptive quality tracking to maintain performance

### Entity Systems

**Player:**
- Centered horizontally, controlled by keyboard (WASD/arrows) or touch/joystick
- Health represented as hearts (1–4 depending on difficulty)
- Weapon system with 7 different firing patterns (accessed via E key or UI)
- Bombs (smart bomb) available per difficulty preset

**Enemies:**
- Three types (grunt, interceptor, tank) with different HP, speeds, and point values
- Spawn rate increases as stages progress; reduced by performance scaling (quality variable)
- Have a basic AI (patrol down, shoot occasionally)
- Drop power-ups when killed (weapon level-up or bomb)

**Bosses:**
- Six unique boss ships, one per stage, with different color schemes and attack patterns
- Boss attacks include radial fire, aimed spreads, spiral bursts, homing shots, and double rings
- Boss defeat triggers stage clear, score multiplier, and progression to next stage
- Boss health scales with difficulty via `bossHpMul` parameter

**Projectiles & Effects:**
- Player projectiles: Spawned by fireWeapon() based on current weapon type
- Enemy projectiles: Simple straight shots
- Explosions: Particle-based with color variation
- Screen shake and flash effects for feedback

### Weapon System
Seven weapon types, each with distinct firing patterns:
1. **Standard** – Two parallel columns of bullets
2. **Spread** – Angled spread pattern
3. **Homing** – Tracking projectiles toward nearest enemy
4. **Radial** – 360° burst
5. **Spiral** – Rotating spiral pattern
6. **Cross** – Four-directional spread
7. **Double Ring** – Two concentric spinning rings

Weapons can be cycled (E key) and level up by collecting power-ups, increasing projectile speed and count.

### Audio System
All sounds are synthesized in real-time (no external files):
- `beep(freq, dur, type, vol)` – Generates sine/square/sawtooth tones for UI feedback, hits, weapon fires
- `noiseBurst(dur, vol)` – White noise bursts for explosions
- Uses Web Audio API with stereo panning for spatial effects
- Mute state persisted across sessions

### Visual Themes
Six visual themes (sky gradients, cloud colors, sun colors) cycle per stage:
1. Daylight Front (bright blue)
2. Sunset Corridor (orange/pink)
3. Night Approach (dark blue)
4. Storm Front (green/dark)
5. Arctic Wastes (cyan)
6. Volcanic Ridge (red/orange)

Defined in `THEMES` array; each stage uses a different theme.

### Input & Control
- **Keyboard:** WASD/arrows to move, Space to fire, E to swap weapon, Q for bomb
- **Touch:** Joystick in lower-left corner for movement, fire button in lower-right
- **Motion:** Gyroscope support on devices with motion sensors (toggle via button)
- Joystick implemented with virtual stick rendering and event tracking

### UI & HUD
- **Overlay screens:** Title screen, pause, game over, stage banners
- **Live HUD:** Score, best score, multiplier, current weapon, weapon level, heart/bomb count
- **Visual feedback:** Screen shake on hit, flash on explosion, combo popups, boss health bar
- **Accessibility:** ARIA live region for screen reader announcements

### Performance Optimization
- **Adaptive quality system:** Tracks frame times and scales quality (full → medium → low) based on FPS
- Reduced projectile/particle counts at lower quality levels
- Prefers-reduced-motion media query support for accessibility

## Making Changes

### Adjusting Game Balance
- **Difficulty:** Edit `DIFFICULTIES` object (spawnBase, enemySpeed, enemyShootChance, lives, bombs)
- **Enemy stats:** Modify `ENEMY_TYPES` object (hp, speed, color, score, minStage)
- **Boss stats:** Edit `BOSSES` array (hpMul relative to difficulty) or boss attack patterns in `updateBoss()`
- **Stage progression:** Change `KILLS_PER_STAGE` constant

### Adding New Weapons
1. Create a new firing pattern function (e.g., `fireNewPattern(b, speed)`) following the pattern of existing ones
2. Add the new weapon to the weapon cycling logic in `fireWeapon()`
3. Add visual/audio feedback in the weapon fire code

### Tweaking Visuals
- **Theme colors:** Edit `THEMES` array (sky gradients, cloud color, sun color)
- **Enemy colors:** Modify `ENEMY_TYPES` colors (color and dark variants)
- **Boss appearance:** Edit `BOSSES` array colors (hullTop, hullBottom, wing, core)
- **Screen shake intensity:** Adjust `screenShake()` magnitude and duration

### Modifying Audio
- Adjust `beep()` frequency, duration, and volume parameters in firing, hit, and UI code
- Change synthesizer type (sine, square, sawtooth) for different timbres
- Increase/decrease master volume in audio initialization

## File Organization Notes

The single-file structure means all code shares one namespace. Key patterns:
- Use descriptive function names to indicate their purpose (e.g., `drawPlane`, `updateBoss`, `spawnExplosion`)
- Game state variables are declared at the top (scores, difficulty, persisted settings)
- Configuration objects (DIFFICULTIES, ENEMY_TYPES, BOSSES, THEMES) are near the top for easy tweaking
- Main game loop (update/draw) and initialization logic are at the bottom
- All DOM references are cached at the top to avoid repeated queries

## Common Tasks

**Test a specific difficulty:** Open browser console and run:
```javascript
setDifficulty('hard');
resetGame();
```

**Skip to a specific stage:** Modify `stage` variable in console or add temporary code to `spawnBoss()` to spawn a specific boss index.

**Adjust weapon balance:** Edit the firing pattern functions or the projectile count/speed parameters in `fireWeapon()`.

**Check performance:** The adaptive quality system logs frame times in `trackPerf()`. Monitor FPS in the game HUD or browser DevTools.

**Reset player progress:** Clear localStorage with:
```javascript
localStorage.removeItem('skyfighter.bestScore');
localStorage.removeItem('skyfighter.difficulty');
```
