# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game implemented as a single-file HTML5 application. The game features:
- Player-controlled ship dodging and destroying enemies
- Multiple weapon types with upgrade system
- Boss encounters at stage transitions
- Difficulty levels (easy/normal/hard) affecting spawn rates, enemy behavior, and lives
- 6 visual themes with distinct color palettes for stages
- 3 enemy types (grunt, interceptor, tank) with different behavior and HP
- 6 unique bosses with individual visual designs and attack patterns
- Combo/scoring multiplier system based on consecutive enemy kills
- Smart bombs that clear the screen and stun bosses
- Web Audio API synthesis for all sound effects (no external audio files)
- Touch controls with virtual joystick for mobile
- Accessibility: reduced motion preference, screen reader support
- Local storage for best score, difficulty, and settings persistence
- Adaptive performance quality (3 levels) that adjusts based on frame timing

## Code Architecture

The entire game logic is contained in a single `<script>` block in `index.html` (lines 422–1669), wrapped in an IIFE for scope isolation.

### Major Subsystems

1. **Game State & Settings**
   - Difficulty presets (DIFFICULTIES object)
   - Theme definitions (THEMES array with sky/cloud/sun colors)
   - Enemy type specs (ENEMY_TYPES) and boss definitions (BOSSES)
   - Settings storage: best score, mute state, difficulty, reduced motion preference

2. **Rendering Pipeline**
   - Canvas context setup and clear
   - Background gradient rendering (sky layers)
   - Entity rendering: player ship, enemies, projectiles, explosions, particle effects
   - UI HUD: score, health, bombs, weapon info, combo notifications
   - Boss health bar and stage banners
   - Vignette and flash effects for hits

3. **Game Logic Loop**
   - `gameLoop()`: Main tick function (requestAnimationFrame)
   - Player input handling (keyboard, mouse, touch)
   - Enemy spawning based on stage progress and difficulty
   - Collision detection (projectiles vs enemies, enemy projectiles vs player, bombs)
   - Entity lifecycle management (creation, updates, destruction)
   - Weapon system: primary fire, laser mode, bomb activation

4. **Audio Synthesis (WebAudio)**
   - `beep()`: Frequency-modulated tone synthesis with optional filter/pan/sweep
   - `noiseBurst()`: Procedural noise with filter
   - `sfx` object: Pre-defined sound effect compositions
   - Spatial audio: panning based on screen position

5. **Weapons & Combat**
   - Primary weapon with ammo, fire rate, and damage scaling
   - Laser mode: alternate fire that consumes ammo faster
   - Weapon leveling system (upgrades fire rate/damage)
   - Enemy projectiles with spawning rules and patterns

6. **Stage & Progression**
   - Stage counter and kill tracking (18 kills per stage)
   - Boss encounters triggered at stage intervals
   - Theme cycling (one per stage)
   - Enemy type introduction gates (tank appears from stage 2)

7. **Touch Controls**
   - Virtual joystick in lower-left (moveZone)
   - Fire button (lower-right)
   - Bomb button (upper-right)
   - Joystick base and stick elements for visual feedback

8. **UI Overlays**
   - Game start screen with difficulty selection
   - Game over / stage clear screens
   - Real-time HUD (top-center): score, health, bombs, weapon name/level
   - Combo popup (center screen)
   - Stage banner (center screen when entering new stage)
   - Boss name and health bar (top-center during boss fights)

### Key Data Structures

- **Player object**: `{ x, y, w, h, vx, vy, hp, maxHp, ammo, maxAmmo, bombCount, maxBombs, weaponLevel, fireRate, lastFire, isBomb }`
- **Enemy array**: Each enemy is `{ x, y, vx, vy, w, h, type, hp, maxHp, shootTimer, color, dark }`
- **Projectile arrays**: Separate `playerBullets`, `enemyBullets`, `lasers` with `{ x, y, vx, vy, w, h }`
- **Particle effects**: Explosion and hit particles with fade and movement
- **Boss object**: `{ x, y, w, h, hp, maxHp, armored, teleport, pattern, fireCount, etc. }`

### Critical Variables

- `gameRunning`, `gamePaused`: Game state flags
- `player`: Main player ship object
- `currentStage`, `stageKills`: Stage progression tracking
- `score`, `mult`, `bestScore`: Scoring system
- `difficulty`: Current difficulty preset name
- `currentTheme`: Active visual theme index
- `quality`: Adaptive quality level (0–2)
- `muted`, `reducedMotion`: User preferences

## Development Workflow

### Running the Game

Simply open `index.html` in a web browser. The game starts with the title screen showing difficulty selection.

**Testing locally:**
- Use `python3 -m http.server 8000` or similar to serve the file (some browser features like localStorage require HTTP)
- Navigate to `http://localhost:8000/index.html`

### Common Development Tasks

**Add a new sound effect:**
- Add a new function to the `sfx` object using `beep()` and/or `noiseBurst()`
- Call it from the relevant game event (e.g., `sfx.myNewSound()` in the collision handler)

**Add a new enemy type:**
- Add an entry to the `ENEMY_TYPES` object with specs: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optionally `weave`
- Add spawn logic in the enemy spawning section (around line 1100) checking `minStage` and difficulty
- Add rendering logic in the render function (around line 750)

**Add a new visual theme:**
- Add an object to the `THEMES` array with `sky` (3-color gradient stops), `cloud`, `sun`, and `name` properties
- The stage progression automatically cycles through themes

**Add a new boss:**
- Add an object to the `BOSSES` array with `name`, `hullTop`, `hullBottom`, `wing`, `core` colors, and `hpMul`
- Add rendering logic in the boss drawing section (around line 850)
- Add attack pattern logic in the boss update section (around line 1200)

**Adjust difficulty balance:**
- Modify `DIFFICULTIES` object values: `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`
- These are multiplied into the spawn rate, enemy speed, and boss HP calculations

**Test weapon upgrades:**
- Weapon level affects `fireRate` and damage per projectile
- Adjust the weapon leveling thresholds or multipliers in the weapon upgrade section (around line 950)

### Testing Tips

- **Rapid iteration:** Edit the file and refresh the browser
- **Test boss fights:** Set `currentStage` to 5 or 6 in the console (`console.log(currentStage)`)
- **Debug collisions:** Add temporary canvas outline drawing or console logs in collision detection loops
- **Test touch controls:** Resize the browser to mobile width or use DevTools device emulation
- **Audio testing:** Mute button (🔇) at top-right; audio synthesis requires a user interaction to initialize (first click/tap)

## File Size & Performance Considerations

- **1669 lines** in a single file (compressed HTML/CSS/JS)
- **Adaptive quality system**: Automatically reduces effect quality if FPS drops below 40 on average
- **Canvas-only rendering**: No external assets (images, fonts, audio files)
- **Web Audio synthesis**: All sounds generated procedurally; no audio file loading
- **Reduced motion support**: Disables animations on user/system preference to improve performance on slower devices

## Accessibility

- **Keyboard controls**: Arrow keys for movement, spacebar for fire, Z for bomb, P for pause
- **Touch controls**: Virtual joystick and buttons for mobile
- **Screen reader support**: Live region (`#srLive`) for status announcements
- **Reduced motion**: Respects `prefers-reduced-motion` and has a toggle button
- **Color contrast**: All UI text uses white on dark/semi-transparent backgrounds

## Git & Deployment

- Single file simplifies deployment: copy `index.html` to any web server
- No build step, no external dependencies
- localStorage is browser-local; clearing storage resets scores and settings
