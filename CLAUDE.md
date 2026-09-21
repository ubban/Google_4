# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game—including UI, game logic, rendering, and audio—is contained in `index.html` with embedded CSS and JavaScript.

The game features:
- Difficulty levels (Easy, Normal, Hard) with different spawn rates and enemy behaviors
- Six bosses with distinct attack patterns
- Multiple weapon types (Vulcan, Laser) that level up
- Power-ups (lives, bombs, weapons)
- Procedural audio synthesis (no external audio files)
- Touch and keyboard controls
- Adaptive performance scaling
- Accessibility features (screen reader support, reduced motion preference)

## Architecture & Key Systems

### High-Level Structure

The JavaScript is organized conceptually (though not in separate files) into these major systems:

1. **DOM & Canvas Setup** – Canvas element sizing and responsive scaling
2. **Audio System** – WebAudio API synthesizer with procedural sound effects
3. **Input Handling** – Keyboard, mouse, and touch controls with adaptive steering
4. **Game State** – Player, enemies, bullets, particles, power-ups, boss
5. **Game Loop** – Frame update, collision detection, rendering
6. **Rendering Engine** – Canvas 2D drawing with themes, visual effects
7. **Difficulty & Configuration** – Constants for enemy types, boss patterns, themes

### Game Loop

The main loop (`loop` function, line 1640) runs at ~60 FPS via `requestAnimationFrame`:
- Tracks delta time (capped at 0.05s) to ensure smooth gameplay across frame rates
- Calls `update(dt)` when game is running and not paused
- Calls `draw()` to render the current frame
- Performs adaptive performance tracking to scale quality down on slower devices

### Entity Systems

**Player** (line 877): Single player aircraft with position, velocity, weapon state, invunerability timer, and cooldown.

**Enemies** (lines 1022–1040): Array of enemy ships with:
- Type-based stats (grunt, interceptor, tank)
- Weaving movement patterns
- Shoot timers and AI patterns (spread vs. aimed fire)
- Health system (tanks have 4 HP, others 1)

**Bullets**: Two separate arrays:
- `bullets`: Player projectiles with pierce levels (lasers pierce enemies)
- `enemyBullets`: Radial and aimed enemy fire

**Boss** (lines 1043–1056, 1411–1439): Single boss entity with:
- Six designs with distinct visual styles
- Attack pattern queues (3 patterns per boss)
- Telegraph phase before firing
- Entry animation and health bar

**Power-ups** (lines 1058–1067): Spawn from enemy kills with types: life, bomb, weapon.

**Particles**: Temporary visual feedback from explosions.

### Difficulty & Configuration

**DIFFICULTIES** (line 485): Object with profiles (easy/normal/hard) defining:
- `spawnBase`, `spawnMin`: Enemy spawn rate limits (frame-based)
- `enemySpeed`: Base multiplier for all enemy velocity
- `enemyShootChance`: Probability enemy fires when eligible
- `lives`, `bombs`: Starting counts
- `bossHpMul`: Health multiplier for difficulty scaling

**ENEMY_TYPES** (line 502): Object defining three enemy classes—grunt, interceptor, tank—with width, height, HP, score values, and availability by stage.

**BOSSES** (line 508): Array of six boss designs with color schemes and HP multipliers.

**BOSS_PATTERNS** (line 1372): Array of 6 pattern sets, one per boss. Each set has 3 attack functions:
- `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`
- Pattern functions mutate `enemyBullets` array directly

**THEMES** (line 493): Six visual themes with sky gradients, cloud/sun colors, and stage names. Applied cyclically as stages progress.

### Audio System

**Procedural Audio** (lines 542–676):
- Uses WebAudio API synthesizer (no external files)
- `beep()` and `noiseBurst()` primitives generate tones and noise
- `sfx` object maps named sounds to synthesis calls
- Sounds are panned by entity X position for spatial audio
- Respects mute toggle and user playback authorization

**Key SFX:**
- Weapon fire: `shoot` (square wave sweep), `laser` (sawtooth sweep)
- Enemy explosions: Type-specific synthesis (grunt/interceptor/tank)
- Boss telegraph: Rising sine tones
- Power-up collection: Chord progression

### Input System

**Keyboard** (lines 768–776): WASD/Arrow keys for movement, Space for fire, X/Shift for bomb, P/Esc to pause.

**Mouse** (lines 836–862): Cursor follows mouse when hovering canvas. Keyboard input takes priority so controls don't fight.

**Touch** (lines 778–834):
- Left half of screen: Virtual joystick (drag to move)
- Right half: Fire and bomb buttons
- Automatically enabled on touch devices

**Joystick State** (lines 781–812): Tracks active touch ID, vector, and base position to support multi-touch without interference.

### Rendering Pipeline

**Themes** (line 1529): Sky gradient, sun glow, parallax clouds drawn first (lines 1530–1544).

**Layers** (highest to lowest Z-order):
1. Overlay/UI (pause screen, game over, start menu)
2. Flash effect (hit/bomb animation)
3. Boss health bar
4. HUD (score, lives, weapon level, multipliers)
5. Game objects: Planes (player, enemies), bullets, boss, particles, power-ups
6. Background: Sky and clouds

**Visual Effects:**
- Screen shake on damage/bomb (lines 727–730)
- Flash white on hit/bomb (lines 698–709)
- Combo pop text animation (lines 711–717)
- Stage banner animation (lines 719–724)
- Vignette pulse when critical health (line 687)

### Collision Detection

**`rectsOverlap`** (line 1069): AABB collision using axis-aligned rectangles. All collision checks iterate arrays and use this function:
- Enemy-to-bullet (line 1232–1253)
- Boss-to-bullet (line 1255–1266)
- Enemy-to-player (line 1268–1274)
- Enemy bullet-to-player (line 1276–1282)
- Boss-to-player (line 1284–1287)
- Power-up-to-player (line 1289–1294)

### Performance Scaling

**Adaptive Quality** (lines 474–483): Tracks frame times over 45-frame window:
- Quality 0 (full): All visual effects, glow shadows, particle count
- Quality 1 (medium): Reduced particle count (60%), no glow on projectiles
- Quality 2 (low): Minimal particles (35%), no shadows
- Adjusts based on average frame duration to maintain target 40 FPS

Particle count scaling is applied in `spawnExplosion` (line 1074).

### Game State & Flow

**States:**
- `running`: Game in progress (not game over)
- `paused`: Game paused (overlay shown, no update/draw)
- `gameOver`: Final state after lives reached 0

**Stage Progression:**
- Kill 18 enemies (`KILLS_PER_STAGE`, line 491) to trigger boss spawn
- Boss defeat advances stage and increments difficulty (spawn rate increases)
- Theme cycles through 6 themes based on stage number

**Lives & Bombs:**
- Lives regenerate via power-ups to `maxLives`
- Bombs use and regenerate similarly
- Invulnerability window (1.5s) after taking damage

**Scoring:**
- Each enemy kill adds base score × multiplier
- Multiplier increases by 1 for every 5 consecutive kills within 2.2s
- Bomb kills add 5 points each per bomb use
- Boss kills add 500 points
- Best score persists in localStorage

### Storage Keys

- `skyfighter.bestScore` – High score (number)
- `skyfighter.muted` – Audio mute state ('0' or '1')
- `skyfighter.difficulty` – Last selected difficulty ('easy'/'normal'/'hard')
- `skyfighter.reducedMotion` – Accessibility preference ('0' or '1')

## Common Modifications

### Adjusting Difficulty

Modify the `DIFFICULTIES` object (line 485):
- `spawnBase` controls initial spawn interval (frames between enemies)
- `spawnMin` sets minimum spawn rate as difficulty ramps
- `enemySpeed` multiplies all enemy velocities
- `bossHpMul` scales boss health on that difficulty

### Adding a New Boss

1. Add an entry to `BOSSES` (line 508) with name, colors, and hpMul
2. Create a new attack pattern array in `BOSS_PATTERNS` (line 1372)
3. Update `updateBoss` pattern selection (line 1424) if needed—it auto-indexes by boss order

### Adjusting Weapon Balance

Modify `fireWeapon` (lines 1124–1146) for damage/spread, or `DIFFICULTIES` `enemyShootChance` for enemy pressure.

Laser pierce values are hardcoded per level in the spread-angle array (lines 1136–1143).

### Tuning Visual Effects

- Screen shake magnitude and duration: `screenShake(mag, dur)` calls (lines 727–730)
- Flash animations: CSS keyframes `flashHit` and `flashBomb` (lines 97–104)
- Particle behavior: Spawn count in `spawnExplosion` (line 1074), velocity spread (lines 1076–1084)
- Enemy bullet colors: Hardcoded `#ff5252` (line 1595)

### Adding New Enemy Types

1. Add to `ENEMY_TYPES` (line 502)
2. Add spawn logic to `spawnEnemy` (line 1022)
3. Add explosions/audio handling to `EXPLODE_SFX` (line 678)

### Customizing Audio

Sound synthesis parameters are in the `sfx` object (line 624). Modify frequency, duration, filter type, and envelope to change tone. Add new sounds by following the pattern of existing functions (using `beep` and `noiseBurst` primitives).

## Canvas & Responsive Scaling

Base canvas size is **480×720** (portrait, line 753). The `resize` function (lines 754–765) scales to fit window while maintaining aspect ratio, with a maximum 1.6× upscale. Triggered on window resize and orientation change.

All game logic uses logical canvas coordinates (480×720); scaling is purely visual.

## Accessibility

- **Screen reader support**: `#srLive` announcements for game state changes (lines 468–471)
- **Reduced motion**: User preference respected; animations disabled when flag is set (lines 349–358)
- **Semantic HTML**: Buttons have titles; icon buttons labeled with emoji
- **Color contrast**: UI text uses high-contrast colors with text-shadow for readability

## Testing & Debugging

No build step or test framework; the game runs directly in a browser.

To test locally:
1. Open `index.html` in a modern browser
2. Game auto-starts audio context on first user interaction
3. Difficulty selection persists across sessions
4. Chrome DevTools can throttle CPU/GPU to test performance scaling

**Debugging Tips:**
- Open console (F12) to see any audio initialization errors
- Check `localStorage` values to inspect persisted state
- Reduce `KILLS_PER_STAGE` (line 491) for faster boss encounters during testing
- Modify `spawnTimer` initialization in `resetGame` (line 888) to spawn more frequently
