# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game, implemented as a single-file HTML5 canvas application. The game features progressive stage-based gameplay, multiple weapons, boss battles with distinct attack patterns, combo scoring, and three difficulty modes.

## Development and Testing

### Running the Game
- **Local Testing**: Open `index.html` directly in a browser. The game runs completely client-side with no build step.
- **Responsive Design**: The game scales to fill the viewport and adapts to mobile/touch devices automatically.

### Making Changes
Since this is a single-file game, all edits go to `index.html`. The file is organized as:
1. **Head section**: Metadata and complete CSS styles
2. **Body section**: HTML markup for canvas, UI elements, and overlays
3. **Script section**: Complete game logic in a single IIFE (Immediately Invoked Function Expression)

### Testing After Changes
- **Visual Changes**: Reload the browser to see CSS/rendering updates immediately
- **Game Logic Changes**: Reload to test gameplay changes; verify that:
  - Enemy spawning and behavior work correctly
  - Collision detection responds as expected
  - Boss patterns fire in the right sequence
  - Scoring and multipliers update properly
  - Power-ups and weapon switching function
- **Audio Changes**: Test procedural sound effects by checking both muted and unmuted states
- **Mobile/Touch**: Test touch controls on device or use browser DevTools device emulation

## Architecture and Key Systems

### Canvas and Rendering
The game renders at a fixed 480×720 canvas size (portrait orientation) that scales to fit the viewport. The main loop calls:
1. `update(dt)` — update all game state
2. `draw()` — render everything to canvas

Rendering includes theme-based sky gradients, animated clouds, entities (player, enemies, boss, bullets, particles), and visual feedback (screen shake, flashes, combo popups).

**Theme System**: Six themes cycle by stage, each with distinct sky gradients, cloud colors, and sun glows (lines 493–500).

### Game State and Flow
- **Game States**: Title screen (overlay shown) → Running → Paused → Game Over
- **Stage Progression**: Player kills enemies to progress through stages; after KILLS_PER_STAGE (18) kills, a boss spawns
- **Boss System**: Each of 6 bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) has unique attack patterns and color schemes. Boss patterns are stored in BOSS_PATTERNS (lines 1372–1409) and index into different firing functions.

**Key State Variables** (lines 864–869):
- `player`: Position, speed, weapon, level, invulnerability timer
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: Entity arrays
- `score`, `lives`, `stage`, `kills`: Game progress
- `chain`, `multiplier`: Combo system

### Player Controls
**Keyboard**: Arrows/WASD to move, Space to fire, X/Shift/P/Esc for bombs and pause.
**Mouse**: Move pointer over canvas to steer; click to fire. Keyboard/joystick input takes priority when active (lines 837–861).
**Touch**: Left half of screen is a virtual joystick; right half has FIRE and BOMB buttons (lines 391–398, 814–834).

### Weapon System
- **Vulcan** (default): Spread shot with increasing width at levels 2–3
- **Laser**: Piercing beam; can chain through multiple enemies; upgraded with more lanes at levels 2–3
- **Power-ups**: Weapons upgrade, or when maxed at level 3, switch to the other weapon at level 1 (lines 1304–1322)

### Enemy System
Three enemy types with different HP, speed, and scoring:
- **Grunt**: 1 HP, normal speed, 10 points
- **Interceptor**: 1 HP, 1.7× speed, weaving pattern, 15 points
- **Tank**: 4 HP, slow, 35 points

Enemies spawn based on difficulty; spawn rate increases over time. Each enemy can fire either aimed or spread patterns at the player (lines 1022–1041, 1210–1227).

### Boss System
Bosses enter from top, patrol side-to-side, telegraph attacks, then fire. Each boss has multiple attack patterns (lines 1372–1409):
- **Pattern Functions**: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`
- **Boss State**: Position, HP, telegraph timer, attack pattern index, design (colors/name)
- **Difficulty**: Boss HP scales by difficulty (0.8× easy, 1.0× normal, 1.3× hard)

Defeating a boss increments the stage, plays victory SFX, and resets enemy spawning (lines 1441–1454).

### Combo and Scoring
- **Chain Mechanic**: Each enemy kill increments a chain counter; chain decays if no kill for 2.2 seconds
- **Multiplier**: `1 + floor(chain / 5)`, max 10× multiplier
- **Milestones**: Every 10 kills shows "X KILLS!" popup (line 1248)
- **Boss Bonus**: 500 points on boss defeat

### Collision Detection
Simple rect-rect overlap using center positions and half-widths (line 1069). Checks:
- Bullets vs enemies (pierce and level matter)
- Bullets vs boss
- Enemies vs player (invulnerability window after hit)
- Enemy bullets vs player
- Boss vs player
- Power-ups vs player

### Input Handling
- **Keyboard**: Stored in a `keys` object; checked each update (lines 768–776)
- **Mouse**: `mouseActive` flag and `mouseTarget` position for steering glide (lines 838–861)
- **Touch/Joystick**: `joyActive`, `joyVec`, and `joyOrigin` track multi-touch input (lines 781–827)
- **Firing**: Triggered by Space key or UI buttons; cooldown prevents spray (line 1178)

### Audio (WebAudio Synthesis)
No external sound files—all SFX are synthesized in real-time using the Web Audio API:
- **Synth Functions**: `beep()` (frequency sweep with filters) and `noiseBurst()` (filtered white noise)
- **Effects Library**: Stored in `sfx` object (lines 624–676)
- **Master Gain**: Compressor for consistent volume (lines 549–554)
- **Panning**: Bullets and explosions pan left/right based on screen position

Mute state is persisted in localStorage.

### Accessibility
- **Reduced Motion**: Respects `prefers-reduced-motion` media query; toggleable in UI. Disables animations and screen shake when active (lines 520–540, 349–358, 727–729)
- **Screen Reader Support**: Live region (`#srLive`) announces key events (boss defeats, stage changes, lives) via `announce()` function (lines 468–470)
- **Keyboard Navigation**: All controls accessible via keyboard; buttons focusable

### Performance Optimization
- **Adaptive Quality**: Tracks frame time over last 45 frames; adjusts `quality` level (0=full, 1=medium, 2=low) to maintain 40+ FPS (lines 473–483)
- **Particle Limit**: Particle count scales with quality (line 1074)
- **Shadow/Glow Effects**: Only rendered at full quality (lines 1002, 1579–1591, 1622)

### Difficulty Modes
Three presets defined at lines 485–489:
- **Easy**: Slower spawns, slower enemies, fewer boss HP
- **Normal**: Baseline
- **Hard**: Fast spawns, fast enemies, 1.3× boss HP

Difficulty is selectable before game start and persisted in localStorage.

### Local Storage
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Mute preference
- `skyfighter.difficulty`: Selected difficulty
- `skyfighter.reducedMotion`: Accessibility preference

## Extending the Game

### Adding a New Boss
1. Add boss design to `BOSSES` array (line 508), specifying colors and HP multiplier
2. Add attack pattern function set to `BOSS_PATTERNS` array at the same index
3. Define firing functions if new patterns needed (see examples lines 1324–1370)
4. Boss will cycle through the six bosses as stages progress

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502) with dimensions, HP, speed multiplier, color, score, and minimum stage
2. Update spawn logic in `spawnEnemy()` (lines 1022–1041) if conditional spawning is desired
3. Add explosion SFX to `EXPLODE_SFX` map (line 678) if custom sound needed

### Adding Weapon Variety
1. Modify weapon labels and firing logic in `fireWeapon()` (lines 1124–1146)
2. Add new weapon case to power-up logic (lines 1304–1322)
3. Update HUD display if needed (weaponNameEl, weaponLvlEl)

### Modifying Visuals
- **Canvas Size**: Change `BASE_W` and `BASE_H` (line 753)
- **Colors**: Edit theme definitions (lines 493–500) or entity color properties
- **Drawing Functions**: `drawPlane()` (lines 913–968) and `drawBossShip()` (lines 970–1019) can be extended

### Adding UI Features
All UI elements are in the HTML section. The overlay system (lines 200–233, 401–419) is reused for title, pause, and game-over screens by toggling visibility and updating text. The HUD is a flex container; add new stat divs as needed.

## Common Edits and Patterns

### Tweaking Game Balance
- **Enemy Difficulty**: Adjust `DIFFICULTIES` object or individual enemy `speedMul` values
- **Weapon Strength**: Change bullet speed (line 1130, 1140) or pierce values (line 1141)
- **Spawn Rate**: Modify `spawnBase` and `spawnMin` in difficulties (lines 485–489)
- **Combo Timing**: Change `chainTimer` duration (line 1089)

### Debugging
Add console logs in `update()` or collision checks to trace state. The game runs in a single frame loop, so breakpoints in DevTools will pause the entire game.

### Performance Bottlenecks
The main loop updates all entities every frame. If performance degrades:
1. Check particle count (quality adaptation should handle most cases)
2. Verify collision checks aren't duplicating work
3. Profile rendering: shadow/glow effects are expensive at full quality
4. Consider pooling bullets/particles instead of splice operations

## File Organization Notes

The entire game is in one file to keep it self-contained and distributable. If extracted, logical sections are:
- **Constants & Config** (lines 422–515): Difficulty, enemy types, bosses, themes, audio
- **State & UI Setup** (lines 517–860): Initialization, input handling, localStorage
- **Game Loop** (lines 863–1302): Main update and collision logic
- **Boss/Enemy Logic** (lines 1324–1454): Specialized patterns and updates
- **Rendering** (lines 1507–1638): Canvas drawing
- **Main Loop Driver** (lines 1640–1666): RequestAnimationFrame and game start
