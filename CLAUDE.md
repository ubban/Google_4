# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a browser-based vertical shoot-'em-up game (Raiden-style). The entire game is contained in a single `index.html` file (~1670 lines) with embedded CSS styling and vanilla JavaScript game engine.

**No build process or external dependencies are required.** Open `index.html` in a browser to play.

## Architecture

The game is organized logically within the single HTML file as follows:

### 1. HTML Structure (lines 361-420)
- Canvas element for rendering
- HUD elements (score, health, bombs, weapon display)
- Boss health bar
- Game overlay (title screen, game over screen)
- Touch control zones (mobile joystick and buttons)
- Accessibility elements (screen reader live region)

### 2. CSS Styling (lines 10-359)
- Responsive canvas scaling with `clamp()` for mobile/desktop
- Dark theme with glowing effects and animations
- Grid-based UI layout using flexbox
- Safe area insets for notched mobile devices
- Reduced motion support via `prefers-reduced-motion` media query
- Animations: combo popups, stage banners, health pulses, button presses

### 3. JavaScript Game Engine (lines 422-1666)

#### Core Systems

**Game State & Configuration** (lines 461-530)
- Difficulty levels (easy/normal/hard) with parameters
- 6 visual themes with distinct sky gradients and palettes
- 3 enemy types: grunt, interceptor, tank (with different HP/speed/scoring)
- 6 boss types: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Persistent settings via localStorage: best score, muted state, difficulty, reduced motion preference

**Input Handling** (lines 767-861)
- Keyboard: arrows/WASD move, Space/click fire, X/Shift bomb, P/Esc pause
- Mouse: moves are smooth-following (not direct), click fires
- Touch: left half = joystick movement, right half = fire/bomb buttons
- Input priority: keyboard/joystick > mouse movement (hybrid input supported)

**Canvas & Rendering** (lines 752-1638)
- Fixed 480×720 logical canvas, scaled responsively to fit screen
- Two main draw functions: `drawPlane()` and `drawBossShip()` (vector art, not sprites)
- Layered rendering: sky gradient → clouds → entities → particles
- Screen shake and flash effects (respects reduced motion setting)
- Mouse targeting crosshair overlay

**Audio System** (lines 542-676)
- WebAudio API synthesis, no external audio files
- Oscillators + noise bursts with dynamic filters and panning
- 11 SFX functions: shoot, laser, various explosions, bomb, powerup, boss effects, UI click, gameover
- Master gain control with compressor for consistent levels
- Mute toggle with persistence

**Entity Systems**

*Player* (lines 863-910)
- Position, velocity, dimensions
- Weapon system: vulcan (spread shots) or laser (piercing shots), 3 upgrade levels
- Cooldown management for fire rate
- Invulnerability timer after hit
- Health tracked as "lives"

*Enemies* (lines 1022-1041, 1210-1228)
- Spawned at intervals that ramp with difficulty and frame count
- Three types with distinct visual designs and behaviors
- Weaving interceptors have sine-wave movement patterns
- Each enemy has shoot timer and pattern (spread or aimed at player)
- Despawn when off-screen

*Bullets* (lines 1188-1189)
- Player bullets: fast upward movement, can have vx (sideways) for spread
- Enemy bullets: slower, various patterns (radial, aimed, homing)
- Collision detection with enemies and boss (line 1232-1266)
- Laser bullets have pierce count (can pass through multiple enemies)

*Boss System* (lines 508-515, 1043-1056, 1372-1439)
- 6 boss designs with unique color schemes
- Spawned after 18 kills per stage
- Phase-based attack patterns stored in BOSS_PATTERNS (6 patterns per boss)
- Telegraph (visual charging) before each attack
- Health bar UI that appears when boss enters
- Boss uses specialized firing functions: fireRadial, fireAimedSpread, fireColumns, fireSpiralBurst, fireCross, fireHomingSeeded, fireDoubleRing

*Particles & Effects* (lines 1073-1085, 1296-1297)
- Explosion particles spawn on enemy/boss destruction
- Fade out over time with velocity
- Quality setting affects particle count (full vs medium vs low detail)

*Power-ups* (lines 1058-1066, 1289-1294)
- Drop from defeated enemies (6% life, 8% bomb, 20% weapon)
- Three types: heal (red), bomb (gold), weapon (green)
- Apply effects: restore health, add bomb, upgrade/switch weapon
- Powerup() function handles all type logic (lines 1304-1322)

**Game Loop & Timing** (lines 1148-1302, 1640-1653)
- `update(dt)`: process input, move entities, handle collisions, spawn enemies/bosses
- `draw()`: clear canvas, render background, entities, particles
- `loop(now)`: requestAnimationFrame callback, manages delta time, calls update/draw
- Performance tracking: adaptive quality (full/medium/low) based on frame time

**Score & Progression** (lines 1087-1101)
- Score multiplied by combo chain (1 + min(9, floor(chain/5)))
- Chain decays if no kills for 2.2 seconds
- Killing enemies increments kill counters; stage advances at 18 kills per stage
- Each stage increments to next boss design
- 6 themes cycle per stage

## Key Gameplay Mechanics

1. **Weapon System**: Vulcan (spread bullets, fast) ↔ Laser (piercing, slower)
   - Upgrade path: level 1 → 2 → 3, then switches weapon
   
2. **Bomb Mechanic**: Clears all enemy bullets, damages/destroys enemies and boss
   - Limited uses per game (2 or 3 by difficulty)
   - Cooldown: none; limited only by count

3. **Scoring**: Base score × combo multiplier
   - Combos break if player takes damage
   - Max multiplier is 10× (at 50+ chain)

4. **Boss Patterns**: Six unique attack scripts per boss design
   - Telegraph visual (white glow on core) signals incoming attack
   - ~1.2 second window between attacks
   - Boss moves left-right bounded within screen
   - Enters from top, exits on defeat

5. **Difficulty**: Affects enemy spawn rate, speed, shoot chance, player lives, bomb count, boss HP
   - Easy: forgiving, common powerups
   - Normal: balanced
   - Hard: relentless, sparse drops

## Common Development Tasks

**Adjusting Difficulty Parameters** (line 485-489)
- `spawnBase`: initial spawn timer (larger = slower spawning)
- `spawnMin`: minimum spawn timer (difficulty ramps over time)
- `enemySpeed` & `enemyShootChance`: enemy behavior multipliers
- `lives`, `bombs`, `bossHpMul`: player starting health and boss scaling

**Adding New Enemy Types** (lines 502-506)
- Add object to `ENEMY_TYPES` with w, h, hp, speedMul, color, dark, score, minStage, optional weave
- Update `spawnEnemy()` logic to control spawn chance
- Add explosion SFX mapping if unique (line 678)

**Adding New Boss Patterns** (lines 1372-1409)
- Create a function using helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`
- Add 3-5 attack patterns to BOSS_PATTERNS array
- Boss pattern functions receive boss object `b`, can modify `b.x` for teleport attacks

**Tweaking Boss Speed/Difficulty** (lines 1043-1051)
- `baseHp`: base health before multipliers (line 1046)
- `hpMul` in boss designs: relative difficulty scaling (line 514)
- Boss movement speed: `b.vx * 55` in updateBoss (line 1418)

**Creating New Visual Themes** (lines 493-500)
- Add theme object with `sky` (3-stop gradient), `cloud`, `sun` RGB values, and `name`
- Themes cycle per stage: stage 1 = themes[0], stage 2 = themes[1], etc.

**Sound Adjustments** (lines 624-676)
- Frequency values in beep() calls (Hz; lower = deeper)
- `sweepTo`: frequency glide endpoint
- `vol`: master volume (0.0-1.0); adjust masterGain.gain.value (line 556) for global level
- `dur`: duration in seconds

**Responsive Design & Mobile** (lines 752-779)
- Canvas scales via `Math.min(maxW / BASE_W, maxH / BASE_H, 1.6)` (line 757)
- Touch controls only shown on touch devices (line 779)
- Safe area insets handled via CSS `max()` and `env(safe-area-inset-*)`

## Code Structure Notes

- **No external libraries**: pure Canvas 2D API, WebAudio API, DOM APIs
- **State is global** within the IIFE wrapper (lines 423-1666) for encapsulation
- **Collision detection**: `rectsOverlap()` (line 1069-1071) uses axis-aligned bounding boxes
- **Performance adaptation**: quality variable (0/1/2) reduces particle count and shadow blur on low-end devices
- **Storage keys** (lines 463-466): persist game settings and best score to localStorage
- **requestAnimationFrame**: 60fps target; delta-time clamped to 50ms max (line 1644)

## Testing the Game

1. Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge)
2. Desktop: use mouse (move + click), keyboard (arrows + space), or click Start button to begin
3. Mobile: tap Start, then use left-side joystick to move and right-side buttons to fire/bomb
4. Difficulty defaults to Normal; change via buttons on title screen
5. Audio requires user gesture (click/tap) to unlock AudioContext
6. Best scores persist across sessions via localStorage
