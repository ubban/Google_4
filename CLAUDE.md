# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5, CSS, and JavaScript. The entire game is contained in a single `index.html` file (~1670 lines). There are no external dependencies, build tools, or tests — the game runs directly in any modern web browser.

## Running the Game

**To play the game locally:**
1. Open `index.html` directly in a web browser (double-click the file or drag it to a browser window)
2. The game will start with the main menu overlay showing difficulty selection and controls
3. Click "Start Game" or tap the start button on mobile

**To test on a local server (optional):**
```bash
# Using Python 3
python -m http.server 8000

# Using Node.js with http-server
npx http-server
```
Then navigate to `http://localhost:8000/index.html`

## Code Architecture

The entire game logic lives in a single `<script>` tag (lines 422–1666). The code is organized into logical sections:

### Game State Management (lines 863–911)
- `player`: Position, speed, weapon, level, invulnerability timer
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: Game object arrays
- `score`, `lives`, `stage`, `chain`, `multiplier`: Game metrics
- `running`, `paused`, `gameOver`: Game state flags
- `DIFFICULTIES`: Config object defining spawn rates, enemy speed, lives, bombs per difficulty level
- `THEMES`: 6 visual themes with sky gradients, cloud colors, sun colors
- `ENEMY_TYPES`: Grunt, Interceptor, Tank definitions (size, HP, speed multiplier, score)
- `BOSSES`: 6 boss ship designs (colors, core, HP multiplier)

### Game Loop (lines 1640–1652)
The core loop runs at 60 FPS via `requestAnimationFrame`:
1. Calculate delta time `dt`
2. Call `update(dt)` to advance game logic
3. Call `draw()` to render the scene
4. Loop continues

### Key Game Systems

**Input Handling (lines 767–861)**
- Keyboard: Arrow keys / WASD for movement, Space to fire, X/Shift for bombs, P/Esc to pause
- Mouse: Move to steer (when no keyboard input), click to fire
- Touch: Left zone is movement joystick, right zone has FIRE and BOMB buttons
- `firing` flag tracks whether fire is held
- `joyVec` object stores normalized joystick input

**Update Loop (lines 1148–1302)**
Runs every frame with delta time:
1. Read input and update player position (with boundary clamping)
2. Update weapon cooldown and fire on input
3. Move and update all bullets, enemies, enemy bullets, power-ups
4. Spawn new enemies based on difficulty spawn timer
5. Detect collisions:
   - Player bullets vs enemies (deal damage, spawn explosions, award score)
   - Player bullets vs boss (deal damage)
   - Enemies vs player (lose life)
   - Enemy bullets vs player (lose life)
   - Boss vs player (lose life)
   - Power-ups vs player (apply effect)
6. Update particles (explosion debris)
7. Trigger boss spawn when `stageKills >= KILLS_PER_STAGE`
8. Update boss if alive

**Rendering (lines 1507–1638)**
- Clear canvas and apply screen shake if active
- Draw sky gradient background based on current theme
- Draw sun glow overlay
- Draw cloud layers at different parallax speeds
- Draw player (or blank if invulnerable and blinking)
- Draw player aiming reticle (if using mouse)
- Draw bullets with glow effects
- Draw enemy bullets
- Draw enemies (using `drawPlane()` helper)
- Draw tank HP bars
- Draw boss (if alive) using `drawBossShip()` helper
- Draw rotating power-up icons
- Draw explosion particles

**Collision Detection (lines 1069–1071)**
- `rectsOverlap(a, b)`: AABB (axis-aligned bounding box) collision check
- Checks if centers are within combined half-widths/half-heights

**Audio (lines 542–676)**
- Web Audio API synthesis (no external audio files)
- `beep()`: Generate tonal sounds with optional frequency sweep, filter, panning
- `noiseBurst()`: Generate noise-based explosion/impact sounds
- `sfx` object contains 10+ sound effects (shoot, laser, explosions, power-up, boss telegraph, game over, etc.)
- Respects `muted` flag and reduced-motion preference

**Weapon System (lines 1124–1146)**
- **Vulcan**: Straight shots, spreads at levels 2–3
- **Laser**: Fast piercing shots, multi-lane at levels 2–3
- Weapon upgrades cycle through levels 1→2→3 then switch weapon
- Different fire delay per weapon type

**Boss System (lines 1043–1056, 1372–1439)**
- Boss enters from top, moves left/right bouncing off walls
- 6 boss designs, each with unique attack patterns (see `BOSS_PATTERNS` array)
- Attack patterns include: radial bursts, aimed spreads, columns, spirals, crosses, homing missiles, double rings
- Boss telegraph (white flash) indicates incoming attack
- Boss damage comes from player bullets (3 HP per bullet) or bombs (24 HP per bomb)

**Power-up System (lines 1058–1067, 1304–1322)**
- Spawns with 6% chance on enemy kill (life), 8% (bomb), 20% (weapon)
- Types: `life` (+1 HP), `bomb` (+1 bomb), `weapon` (upgrade weapon or switch)

**Combo/Chain System (lines 1087–1101)**
- Every enemy kill increments `chain`
- Multiplier = 1 + floor(chain / 5), capping at 10x
- Chain resets if player takes damage or 2.2 seconds elapse without a kill
- Multiplier affects all score awards

**Input Priority**
Keyboard/joystick > Mouse movement for player steering (prevents conflict)

### Performance Optimization (lines 473–483)
- Adaptive quality scaling based on frame time:
  - Quality 0 = Full (glows, shadows, particles)
  - Quality 1 = Medium (reduced particle count/glow)
  - Quality 2 = Low (minimal effects)
- Tracks last 45 frame times; adjusts quality if average exceeds 1/40 or drops below 1/56

### Accessibility & Usability (lines 529–540, 349–358)
- Reduced-motion preference toggle respects browser's `prefers-reduced-motion` setting
- Screen reader support via `#srLive` announcements
- Mute button for audio control
- All values persisted to localStorage

### UI Overlays
- **Main menu/end screen** (lines 401–419): Overlay div shown at start and game over, hidden during gameplay
- **HUD** (lines 367–374): Always visible during gameplay (score, best, multiplier, weapon, hearts, bombs)
- **Boss bar** (lines 376–379): Shows when boss is active
- **Combo pop** (lines 381): Animates combo milestones (10 kills, 20 kills, etc.)
- **Stage banner** (line 382): Announces new stages and boss arrivals
- **Touch controls** (lines 392–399): Movement joystick, FIRE/BOMB buttons on mobile
- **Top bar** (lines 384–388): Mute, reduced-motion, pause buttons

## Important Notes

### Single-File Structure
All logic is in `index.html`. When modifying:
- **Game mechanics**: Edit `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, weapon stats, boss patterns
- **Visuals**: Modify CSS (lines 10–358) for colors/layout, `drawPlane()` and `drawBossShip()` for shapes
- **Sounds**: Modify `sfx` object (lines 624–676)
- **Controls**: Modify input event listeners (lines 768–834) and control logic in `update()`

### Canvas Scaling
The canvas is 480×720 (BASE_W/BASE_H) but scales to fit the window up to 1.6x. This is handled by `resize()` (lines 752–764). Canvas coordinates always use logical 480×720 space; the CSS transform handles physical display scaling.

### Game Balance
- Difficulty modifiers affect spawn rate, enemy speed, enemy aggression, lives, bombs, and boss HP
- Enemy spawn rate decreases over time (difficulty ramp at line 1201)
- Stage progression adds new enemy types (`minStage` field) and themed bosses
- Boss HP scales with difficulty and boss type (`hpMul`)

### localStorage Keys
- `skyfighter.bestScore`: Best score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Motion preference override

### Frame Counter
The global `frame` variable increments every update and is used for:
- Animation timing (particles, boss core glow, combo pop, stage banner)
- Spawn timers and attack patterns
- Player invulnerability blinking
- Cloud parallax movement

## Modifying the Game

**Add a new difficulty level:**
Add entry to `DIFFICULTIES` object (line 485–489).

**Add a new enemy type:**
Add to `ENEMY_TYPES` object and update `spawnEnemy()` logic (lines 1022–1041).

**Add a new boss:**
Add to `BOSSES` array and add corresponding pattern array to `BOSS_PATTERNS` (lines 1372–1409).

**Change visual theme:**
Modify `THEMES` array (lines 493–500) or add new entry.

**Adjust weapon balance:**
Modify spread/lane arrays in `fireWeapon()` (lines 1124–1146) or change fire delay (line 1177).

**Tune boss difficulty:**
Modify attack patterns in `BOSS_PATTERNS`, bullet speeds, or HP multipliers in `BOSSES`.
