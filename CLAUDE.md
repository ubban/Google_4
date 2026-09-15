# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game (HTML, CSS, JavaScript) is contained in `index.html`. No build tools, transpilers, or external dependencies are required—the game runs directly in any modern browser.

## Game Structure

### Architecture Pattern: Self-Contained Game Loop

The entire game runs inside a self-executing function (`(function () { ... })()`) that:
1. Manages DOM elements and canvas setup
2. Maintains global game state (player, enemies, bullets, score, stage)
3. Runs a `requestAnimationFrame` loop that calls `update(dt)` and `draw()` each frame
4. Handles all input, collision detection, and entity updates in the update phase

**Key Constants:**
- `BASE_W = 480, BASE_H = 720` — logical canvas size (scaled to fit screen)
- `KILLS_PER_STAGE = 18` — enemies to kill before boss spawns
- `DIFFICULTIES` — three difficulty presets (easy/normal/hard) that control spawn rates, enemy speed, lives, bombs, boss HP

### Core Subsystems

#### 1. Game Loop & State Management
- **`resetGame()`** — Initializes/resets all game state when a new game starts
- **`update(dt)`** — Primary update function called once per frame (dt = delta time in seconds)
  - Updates player position based on keyboard/joystick/mouse input
  - Spawns enemies on a spawn timer (decreases difficulty/stage)
  - Updates enemy positions, rotations (weave), and shooting
  - Handles all collision detection (bullets → enemies, bullets → boss, enemies → player)
  - Updates particles, power-ups, and boss state
  - Updates HUD (score, multiplier, hearts, bombs)
- **`draw()`** — Renders all game objects with canvas 2D API
  - Applies screen shake via canvas translate
  - Draws background (sky gradient with sun, clouds)
  - Renders player, enemies, bullets, power-ups, particles, boss
  - Handles invulnerability blinking and laser glow effects
- **`loop(now)`** — RAF callback that measures frame timing, calls update/draw, tracks performance

**Game State Variables:**
- `player` — {x, y, w, h, speed, weapon, level, invuln, cooldown}
- `bullets`, `enemies`, `enemyBullets`, `powerUps`, `particles` — arrays of entities
- `score`, `lives`, `stage`, `kills`, `chain`, `multiplier` — score tracking
- `running`, `paused`, `gameOver` — game phase flags

#### 2. Input Handling

Three input methods (all prioritized intelligently):
- **Keyboard:** Arrow keys or WASD for movement, Space to fire, X/Shift for bomb, P/Esc to pause
- **Mouse:** Move to steer (plane glides toward pointer), click to fire
- **Touch:** Left side drag for joystick movement, FIRE/BOMB buttons on right side

Input priority: keyboard/joystick always overrides mouse steering to avoid fighting controls.

#### 3. Audio System (WebAudio Synthesis)

All sounds are procedurally generated using Web Audio API (no audio files):
- **`beep(freq, dur, type, vol, opts)`** — Oscillator-based tone with optional sweep, filter, panning
- **`noiseBurst(dur, vol, opts)`** — White noise with lowpass filter envelope
- **`sfx` object** — Collection of predefined sound effects (shoot, laser, explode, bomb, powerup, bossHit, etc.)

Sounds support spatial panning (left→right based on x coordinate) and are muted via `muted` flag in localStorage.

#### 4. Collision Detection

Simple **AABB (axis-aligned bounding box)** overlap test:
```javascript
function rectsOverlap(a, b) {
  return Math.abs(a.x - b.x) < (a.w + b.w) / 2 && Math.abs(a.y - b.y) < (a.h + b.h) / 2;
}
```

Collisions are checked in `update()` in this order:
1. Player bullets vs. enemies
2. Player bullets vs. boss
3. Enemies vs. player
4. Enemy bullets vs. player
5. Boss vs. player
6. Power-ups vs. player

#### 5. Scoring & Chain System

- Base score for kills: enemy type specific (grunt=10, interceptor=15, tank=35, boss=500)
- **Multiplier:** Increments chain counter on each kill; multiplier = 1 + floor(chain / 5), capped at 10x
- **Chain Timer:** Resets to 2.2 seconds on each kill; chain breaks if timer expires
- Kill milestones show pop-ups (every 10 kills: "10 KILLS!")
- Best score persisted to localStorage

#### 6. Difficulty & Progression

**Difficulty Settings:**
- **Easy:** slower spawns, slower enemies, lower shoot chance, 4 lives, 3 bombs, 0.8× boss HP
- **Normal:** balanced, 3 lives, 2 bombs, 1.0× boss HP
- **Hard:** fast spawns, fast enemies, high shoot chance, 3 lives, 2 bombs, 1.3× boss HP

**Stage Progression:**
1. Kill 18 enemies → spawn boss
2. Defeat boss → advance to next stage (loops through 6 boss designs)
3. Enemy types unlock per stage (tanks at stage 2+)
4. Visual theme cycles through 6 skybox designs
5. Spawn rate accelerates every 5 seconds (300 frames) of continuous play

#### 7. Weapons & Power-ups

**Weapons (cycle via power-ups):**
- **Vulcan (projectile):** Fast-firing bullets, spread pattern at level 3
  - Level 1: single center lane
  - Level 2: two lanes
  - Level 3: three lanes
- **Laser (piercing):** Slower fire rate, projectiles pierce through enemies
  - Level 1: single lane
  - Level 2: two lanes
  - Level 3: three lanes

**Power-up Types:**
- **Life (6% drop rate):** +1 HP (capped at max)
- **Bomb (8% drop rate):** +1 bomb (capped at max)
- **Weapon (20% drop rate):** Level up weapon or switch weapon/reset to level 1

#### 8. Boss AI

**Boss System:**
- One active boss at a time (entered when 18 enemies killed)
- Boss enters from top with animation (y += 60px/frame until reaching y=120)
- **Attack Patterns:** Each boss has 3 distinct attack patterns that cycle
  - Pattern includes telegraph phase (22 frames of visual warning + sound)
  - Then fires barrage of bullets in predefined formation
  - Attack timer resets to 85 frames after pattern fires
- Boss bounces left-right within canvas bounds

**Boss Types (6 designs, cycle by stage mod 6):**
1. VANGUARD — radial burst + aimed spread + V-fan patterns
2. BEHEMOTH — heavy columns + homing spread + dense radial wall
3. SPECTER — teleport + spiral burst + double aimed shots
4. DREADNOUGHT — 8-way cross + homing missiles + radial wall
5. WRAITH — teleport + expanding double ring + cross-spread
6. TITAN — heavy columns + full ring + relentless homing

Each boss has distinct colors and HP multiplier.

#### 9. Enemy Types

- **Grunt:** 34×34, 1 HP, standard speed, red color, spawn from stage 1
- **Interceptor:** 26×26, 1 HP, 1.7× speed, orange, weaves side-to-side, spawn from stage 1
- **Tank:** 46×46, 4 HP, 0.55× speed, purple, heavily armored, spawn from stage 2+

Enemies spawn at top with random x position, move downward, and shoot at player (either aimed or spread pattern).

#### 10. Adaptive Quality Settings

Performance tracking automatically reduces rendering quality on slower devices:
- **quality = 0:** Full quality (glow shadows, max particles)
- **quality = 1:** Medium (reduced particles ~60%)
- **quality = 2:** Low (minimal particles ~35%, no shadow blur)

Adjusted based on 45-frame rolling average of frame times.

## Running the Game

### Desktop/Browser
1. Open `index.html` directly in any modern browser (Chrome, Firefox, Safari, Edge)
2. Game starts with menu overlay; select difficulty and press "Start Game"
3. No server required; runs entirely client-side

### Testing/Development
- **No build step needed** — just edit `index.html` and reload browser
- Browser DevTools console logs frame timing info if needed
- localStorage stores: best score, muted state, difficulty, reduced motion preference

## Code Organization (within index.html)

1. **HEAD:** Meta tags, CSS (styles), favicon
2. **BODY:** HTML structure (canvas, HUD, overlay, touch controls)
3. **SCRIPT:** Entire game logic (1667 lines)

### Key Function Clusters:

**Setup & Init (lines 423-527):**
- DOM element caching
- Storage key constants
- Best score, muted, difficulty, reduced motion initialization

**Audio (lines 542-676):**
- `ensureAudio()`, `panFor()`, `beep()`, `noiseBurst()`, sfx object

**UI/Display (lines 680-764):**
- `renderHearts()`, `renderBombs()`, `updateMuteBtn()`, screen shake, flash effects

**Input (lines 767-862):**
- Keyboard/mouse/touch event listeners
- Joystick emulation for touch
- Mouse steering logic

**Game State & Spawning (lines 863-1068):**
- State variables, weapon label, `resetGame()`, enemy/boss spawning, power-up logic

**Drawing (lines 1073-1638):**
- `drawPlane()`, `drawBossShip()`, `spawnExplosion()`, `draw()` main render

**Core Loop (lines 1148-1454):**
- `update()` — main update logic (player, enemies, collision, scoring)
- `applyPowerUp()`, boss attack patterns, `updateBoss()`, `killBoss()`, `loseLife()`, `endGame()`

**Game Flow (lines 1491-1665):**
- `togglePause()`, start button handler, RAF loop entry point

## Modifying the Game

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` constant (define w, h, hp, color, score, minStage, speedMul)
2. Add explosion sound to `EXPLODE_SFX` mapping
3. Add sound effect to `sfx` object if unique SFX needed
4. Update `spawnEnemy()` selection logic if needed

### Adding a New Boss
1. Add entry to `BOSSES` array (name, hull colors, core color, hpMul)
2. Add attack pattern array to `BOSS_PATTERNS` (3 functions that call fire* helpers)
3. Boss automatically cycles by `(stage - 1) % BOSSES.length`

### Tweaking Difficulty
Edit `DIFFICULTIES` object (spawn rates, speeds, lives, bomb counts, boss HP multipliers).

### Changing Game Balance
- Weapon fire rates: edit `fireDelay` constant in update() or weapon pattern spreads in `fireWeapon()`
- Projectile speeds: edit `speed` values in bullet/enemy-bullet creation
- Collision sizes: edit `w, h` in entity definitions
- Player speed: edit `player.speed` in resetGame()

### Visual/UI Changes
- Sky themes: edit `THEMES` array (gradient colors, cloud colors, sun colors)
- HUD styling: edit CSS rules in `<style>`
- Particle effects: edit `spawnExplosion()` particle count/color
- Draw functions: edit `draw()`, `drawPlane()`, `drawBossShip()`, `drawClouds()`

## Performance Considerations

1. **Canvas sizing:** Logical size 480×720, scaled to screen via CSS (preserves aspect ratio)
2. **Particle culling:** Particles removed when life <= 0; counts adapted to quality level
3. **Entity arrays:** Enemies/bullets filtered out when off-screen
4. **Reduced motion:** CSS animations and screen effects can be disabled for accessibility
5. **Audio:** Synthesis on-demand (no pre-rendering); mute disables all audio synthesis

Expect 60 FPS on modern devices; adaptive quality handles lower-end hardware.

## Debugging Tips

- **localStorage inspection:** Open DevTools → Application → Cookies → localStorage to see saved preferences
- **Frame timing:** First 45 frames calibrate quality; watch console if jittery
- **Screen shake magnitude:** Controlled by `screenShake(mag, dur)` calls — increase `mag` for more shake
- **Touch controls:** Visible on mobile/tablet; joystick base appears when dragging left side
- **Boss telegraph:** Look for white core flash before attack pattern fires

## Accessibility

- **Reduced Motion:** Toggle button in top-right disables animations, flashes, screen shake
- **Sound Toggle:** Mute button in top-right; sound preferences saved to localStorage
- **Screen Reader:** Uses `#srLive` element with `aria-live="polite"` for status updates
- **Color Contrast:** HUD text uses light colors on dark background; text shadows for legibility
