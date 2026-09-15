# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely as a single HTML5 file with embedded CSS and JavaScript. Players pilot a fighter aircraft, switching between two weapon systems, collecting power-ups, and defeating stage bosses across progressively harder stages.

## Architecture

### Single-File Structure
- **index.html** (~1670 lines): Contains all HTML markup, CSS styling, and JavaScript game logic
  - Embedded favicon as SVG data URI
  - All styling uses flexbox and CSS animations
  - All game logic in a self-executing function with `'use strict'` mode

### High-Level Code Sections (in order)

1. **DOM References (lines 426-461)**: Caches all HTML element references for performance
2. **Constants & Settings (lines 463-527)**:
   - `STORAGE_*`: localStorage keys for persistence (best score, mute state, difficulty, reduced motion)
   - `DIFFICULTIES`: Three difficulty levels with spawn/speed/enemy stats
   - `THEMES`: Six sky backgrounds (Daylight, Sunset, Night, Storm, Arctic, Volcanic)
   - `ENEMY_TYPES`: Three enemy variations (grunt, interceptor, tank) with stats
   - `BOSSES`: Six boss designs with unique coloring and HP multipliers
   - `BOSS_PATTERNS`: Attack pattern arrays for each boss (6 patterns per boss)

3. **Audio System (lines 542-677)**: WebAudio API synthesis (no external assets)
   - `ensureAudio()`: Lazy-initializes AudioContext with compressor
   - `beep()`: Tone synthesis with optional frequency sweep and filtering
   - `noiseBurst()`: White noise with filters for explosions
   - `sfx` object: Pre-defined sound effects (shoot, laser, various explosions, hit, bomb, etc.)

4. **UI Helpers (lines 680-750)**:
   - `renderHearts()`: Updates hearts display with critical warning
   - `renderBombs()`: Updates bomb count indicator
   - `flashHit()` / `flashBomb()`: Screen flash animations
   - `showComboPop()` / `showStageBanner()`: Floating text animations
   - `screenShake()`: Applies camera shake effect

5. **Canvas & Input Setup (lines 752-861)**:
   - Canvas sized to 480×720 with responsive scaling
   - Keyboard input: Arrow/WASD/Space for movement & firing, X/Shift for bomb, P/Esc to pause
   - Mouse input: Steer to pointer (passive), click to fire
   - Touch input: Joystick-style movement zone (left 55%), action buttons (right 45%)
   - Joystick base/stick visual feedback appears only on touch

6. **Game State (lines 863-911)**:
   - `player`: Position, speed, weapon, level, cooldown, invulnerability
   - `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: Entity arrays
   - `score`, `lives`, `stage`, `kills`, `chain`, `multiplier`: Game metrics
   - `resetGame()`: Full state initialization per difficulty

7. **Drawing Functions (lines 913-1020)**:
   - `drawPlane()`: Renders player/enemy aircraft with 2D vector shapes, optional flame
   - `drawBossShip()`: Boss-specific design with hull gradient, core glow, rotating engines
   - `drawClouds()`: Parallax scrolling background clouds at two speeds

8. **Enemy Spawning & Behavior (lines 1022-1227)**:
   - `spawnEnemy()`: Randomly selects type, position, and fire pattern
   - Weaving interceptors use sine wave movement
   - Enemy shoot patterns: "spread" (3-way) or "aimed" (toward player)
   - Enemy cleanup removes off-screen entities

9. **Boss System (lines 1043-1439)**:
   - `spawnBoss()`: Creates boss after 18 regular kills per stage
   - Boss designs cycle through BOSSES array based on stage
   - Boss attack patterns combine multiple firing functions (radial, aimed spread, homing, columns, spirals, crosses)
   - Telegraph phase (22 frames) signals attack then fires bullets
   - Boss bullet movement: radial bullets track `dx`/`dy`, others track `vx`/`vy`
   - updateBoss: Handles entering animation, horizontal patrol with boundary reversal, telegraph/attack timing

10. **Collision & Scoring (lines 1232-1322)**:
   - Rect-based AABB collision detection
   - Chain combo multiplier (1-10x) resets if player doesn't kill for 2.2 seconds
   - Power-ups: life (rare), bomb (less rare), weapon (common) with level up/swap mechanics
   - Boss damage: 3 HP per player bullet, 24 HP per bomb, bullets with pierce property can hit multiple

11. **Game Loop (lines 1640-1665)**:
   - `update(dt)`: Physics, input polling, collision checks, enemy/boss updates
   - `draw()`: Canvas clear + sky gradient + clouds + entities + particles
   - Adaptive quality: Tracks frame time, reduces quality if avg > 1/40s or increases if < 1/56s
   - Screen shake applied via canvas context translate during draw

12. **Input Handling Summary**:
   - Keyboard + Joystick: Movement input normalized and prioritized over mouse
   - Mouse: Passive steering target only if no keyboard/joy input
   - Touch: Split screen—left 55% for movement, right 45% for buttons
   - All firing (keyboard Space, mouse click, touch FIRE button) sets `firing = true`
   - Bomb input: X/Shift/touch BOMB button calls `useBomb()` directly

## Development & Testing

### Running Locally
1. Serve the directory with any HTTP server (e.g., `python -m http.server 8000` or `npx serve`)
2. Open `index.html` in browser—no build step required
3. Game state (best score, mute, difficulty) persists in localStorage

### Key Testing Areas
- **Difficulty Modes**: Easy/Normal/Hard spawn rates and enemy speed scale
- **Touch vs. Keyboard**: Ensure joystick appears on touch devices, doesn't interfere with keyboard
- **Audio**: Verify synth sounds play on first user interaction (audioCtx.resume required)
- **Performance**: Watch adaptive quality toggle when frame time fluctuates
- **Collisions**: AABB overlap detection in update loop—test boss collision radius
- **Boss Patterns**: Each boss type uses different attack arrays via `BOSS_PATTERNS`

### No External Dependencies
- Pure HTML5: Canvas 2D, WebAudio API, localStorage
- CSS: Flexbox, grid, animations, backdrop-filter (no build tools)
- Accessibility: Screen reader live region for announcements, reduced-motion support

## Code Style & Patterns

- **Self-executing function** wraps all code to avoid global scope pollution
- **Lazy audio init**: AudioContext created on first interaction (browser autoplay policy)
- **Entity arrays**: Bullets/enemies/particles mutated in-place during frame, filtered at end
- **Constants before state**: All game data (DIFFICULTIES, BOSSES, THEMES) at top
- **Frame-based logic**: Some animations keyed to `frame` counter for consistency
- **Screen shake**: Applied as translate transform only when `shakeTime > 0`
- **Reduced motion**: CSS animations disabled, physics shake reduced to 15% if flag set

## Common Modifications

- **Add enemy type**: Define in `ENEMY_TYPES`, add spawn condition in `spawnEnemy()`
- **Add/modify boss**: Extend `BOSSES` array, add pattern set to `BOSS_PATTERNS`, update boss HP scaling
- **New weapon type**: Add to weapon switch logic in `applyPowerUp()`, define fire pattern in `fireWeapon()`
- **New audio effect**: Add to `sfx` object using `beep()` and `noiseBurst()` helpers
- **Adjust difficulty**: Modify `DIFFICULTIES` values (spawnBase, enemySpeed, bossHpMul, etc.)
- **UI cosmetics**: Edit CSS variables in `:root`, theme colors in `THEMES`, boss bar styling

## Performance Notes

- Adaptive quality (`quality` variable 0-2) scales particle count and shadow rendering
- Boss shadow rendering disabled at quality > 0
- Particles use simple circle rendering without complex paths
- No image assets—all graphics drawn procedurally via Canvas 2D
- Audio context compression prevents clipping on heavy SFX moments
