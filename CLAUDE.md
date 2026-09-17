# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in vanilla JavaScript, HTML, and CSS. The entire game is contained in a single `index.html` file (~1670 lines) with no external dependencies or assets.

### Key Characteristics
- **Single-file architecture**: All game logic, rendering, audio, and UI in one HTML file
- **Asset-free**: No external images, fonts, or sounds — rendering is canvas-based, audio uses Web Audio API synthesis
- **Full-featured game**: Multiple enemy types, 6 boss encounters, difficulty levels, progression system
- **Mobile-friendly**: Touch controls, responsive scaling, accessibility features (reduced motion, screen reader support)
- **Storage-based persistence**: High scores and settings stored in localStorage

---

## Development & Testing

### Running the Game
- Open `index.html` directly in a web browser (no server required)
- Test on desktop, tablet, and mobile devices
- Recommended browsers: Chrome, Firefox, Safari, Edge (all modern versions support required APIs)

### No Build Process
There is no build step, bundler, or compilation. Changes to `index.html` take effect immediately — refresh the browser to see changes.

### Key Testing Scenarios
- **Gameplay**: Start a game, reach different stages, test boss encounters
- **Difficulty modes**: Easy/Normal/Hard should spawn enemies at different rates with different behaviors
- **Input methods**: Test keyboard (arrows/WASD, space/shift for bomb, P to pause), mouse steering, and touch controls
- **Mobile responsiveness**: Canvas scales correctly on different screen sizes
- **Audio**: Both WebAudio synthesis (sfx) and Web Audio Context state (resume on user interaction)
- **Storage**: Check localStorage for persisted best score, difficulty, mute state, and reduced-motion preference

---

## Architecture & Key Systems

### Game Loop (lines 1640-1665)
```
requestAnimationFrame(loop) → update(dt) → draw() → next frame
```
Uses delta-time (`dt`) for frame-rate-independent movement. Performance is tracked (`trackPerf`) with adaptive quality scaling.

### Core Game Objects
All game state lives in the global scope within the IIFE (Immediately Invoked Function Expression). Key variables:

- **`player`**: Position, size, weapon, level, cooldown, invulnerability timer
- **`enemies`**: Array of active enemy objects (type, position, HP, AI state)
- **`boss`**: Current boss object (null when no boss active) with patterns and attack state
- **`bullets`**, **`enemyBullets`**: Player and enemy projectiles
- **`powerUps`**: Dropped items (life, bomb, weapon)
- **`particles`**: Visual effects (explosions)
- **Score/progression**: `score`, `lives`, `stage`, `chain`/`multiplier`, `kills`/`stageKills`

### Rendering Pipeline
1. **Sky background**: Linear gradient that changes per stage (6 themes in `THEMES`)
2. **Clouds**: Scrolling parallax layers with opacity
3. **Entities**: Enemies and bosses drawn with custom shape code (no sprite sheets)
   - `drawPlane()`: Used for player and enemies
   - `drawBossShip()`: Boss-specific rendering with rotating gun ports
4. **Bullets & particles**: Simple circles and rectangles with glow effects when `quality === 0`
5. **UI overlay**: HUD (score, weapon, health, bombs) positioned absolutely above canvas

### Input Handling
- **Keyboard**: `keys` object tracks held keys; parsed in `update()` for smooth movement
- **Mouse**: Sets `mouseTarget` for steering; click to fire
- **Touch**: Joystick in left 55% of screen, fire/bomb buttons in right 45%
  - `joystickStart/Move/End` manage the virtual stick visualization
  - `moveZone`, `fireBtn`, `bombBtn` are touch-sensitive zones
- **Pause**: P or Esc toggles pause state; displays overlay

### Enemy Spawning & AI
- **Spawn timing**: Controlled by `spawnTimer` (decreases each frame)
- **Enemy types** (in `ENEMY_TYPES`): grunt, interceptor (weaving), tank (multi-HP)
- **Patterns**: Each enemy picks "spread" (fixed angles) or "aimed" (toward player) shooting
- **Weaving**: Interceptors oscillate side-to-side using sine wave
- **Stage progression**: Killing 18 enemies (`KILLS_PER_STAGE`) triggers boss spawn after 1.2s

### Boss System
- **6 bosses** (in `BOSSES`): VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- **Lifecycle**:
  1. Spawn off-screen, enter arena (1.2s delay, `entering` flag)
  2. Move side-to-side; telegraph attack (22 frames warning glow)
  3. Fire pattern (one of 3 per boss in `BOSS_PATTERNS`)
  4. Repeat until HP ≤ 0
- **Patterns**: Radial, aimed spreads, columns, spirals, homing shots — each boss has unique set
- **HP scaling**: Difficulty multiplier × boss design multiplier × stage-based formula

### Weapon & Power-Up System
- **Vulcan**: 1–3 spread projectiles (depending on level)
- **Laser**: 1–2 pierce projectiles that pass through enemies
- **Weapon upgrade path**: Level 1→2→3, then toggle between weapons (reset to level 1)
- **Power-ups** (6% life, 8% bomb, 20% weapon chance per enemy kill):
  - Life: +1 health (max `maxLives`)
  - Bomb: +1 bomb (max `maxBombs`)
  - Weapon: Upgrade level or switch weapon

### Collision & Scoring
- **Rect overlap**: `rectsOverlap()` checks `Math.abs(a.x - b.x) < (a.w + b.w)/2`
- **Enemy-bullet**: Damages enemy, consumes bullet (or reduces pierce count for lasers)
- **Boss-bullet**: 3 damage per hit
- **Player damage**: Colliding with enemy/boss/enemy bullet → lose life
- **Scoring**: Base score × multiplier; killing breaks in combo reset multiplier to 1
- **Chain multiplier**: Increases 1× → 2× → 3× (max 10×) every 5 consecutive kills in 2.2s window

### Audio (Web Audio API, No External Files)
- **Synth functions**: `beep()` (oscillator with optional sweep and filter), `noiseBurst()` (filtered noise)
- **Sound effects** (in `sfx` object): shoot, laser, explosions (per enemy type), hit, bomb, powerup, boss sounds
- **Stereo panning**: Shots pan left/right based on screen X position (`panFor()`)
- **Mute state**: Stored in localStorage; clicking mute button toggles `muted` flag
- **Audio context resume**: On first user interaction, `ensureAudio()` creates and resumes AudioContext (required by browsers)

### Difficulty System
Three presets in `DIFFICULTIES` (easy/normal/hard):
- Enemy spawn rate and minimum spawn interval
- Enemy bullet speed and accuracy
- Starting lives, bombs, and boss HP multiplier

Stored in localStorage and persists across sessions.

### Accessibility & Quality of Life
- **Reduced motion**: Respects `prefers-reduced-motion` OS setting; can be toggled in-game
  - Disables animations: heart beat, vignette pulse, combo pop, stage banner, flash effects
  - Reduces screen shake magnitude
- **Screen reader support**: `#srLive` element with `aria-live="polite"` announces key events (game over, stage cleared, bomb count)
- **Responsive canvas**: Scales to fit window while maintaining 480×720 aspect ratio
- **Safe-area insets**: HUD and buttons respect notches on mobile devices using `env(safe-area-inset-*)`

---

## Common Modifications

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Add spawning logic in `spawnEnemy()` (adjust probability roll values)
3. If new attack pattern, add entry to `EXPLODE_SFX` mapping and create `sfx.explode*` function
4. Enemies are drawn by generic `drawPlane()` call in rendering loop

### Adding a New Boss
1. Add entry to `BOSSES` array with `name`, hull/wing/core colors, `hpMul` multiplier
2. Create 3 attack pattern functions and add to `BOSS_PATTERNS` array (indexed to match boss index)
3. Patterns use helpers: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, etc.
4. Boss rendering uses `drawBossShip()` — customize by accessing `b.design` properties

### Tweaking Game Balance
- **Spawn rate**: `DIFFICULTIES[mode].spawnBase` and `spawnMin` control timing
- **Enemy speed**: `enemySpeed` multiplier in difficulty presets
- **HP values**: Edit boss base HP formula at line 1046
- **Score values**: `score` property in enemy types, or `addScore()` calls for boss/combo rewards
- **Weapon balance**: Spread angles, bullet speeds, cooldown in `fireWeapon()` function
- **Bomb damage**: Edit damage values in `useBomb()` (currently 24 to boss, score multiplier for enemies)

### Changing Visuals
- **Sky themes**: Edit `THEMES` array colors (RGB arrays for sky gradient, cloud/sun colors)
- **Enemy/boss colors**: Modify `color`/`dark` properties in `ENEMY_TYPES` and `BOSSES`
- **Particle colors**: Change color strings in explosion spawn calls and `spawnExplosion()` rendering
- **HUD styling**: CSS in `<style>` block (lines 10–359) controls all UI appearance

### Adding Game Features
- **New difficulty mode**: Add entry to `DIFFICULTIES` object; difficulty selection already wired in UI
- **New power-up type**: Modify `maybeSpawnPowerUp()` probability check; add type case in `applyPowerUp()`
- **Stage-dependent behavior**: Check `stage` variable in enemy spawn, boss selection, or theme rotation logic
- **New game mode**: Add mode state alongside `paused`/`gameOver`/`running`; update pause overlay and flow

---

## Code Organization

The entire game is in one IIFE to avoid polluting global scope. Key sections (by line range):

- **Declarations & setup** (424–530): Canvas, DOM elements, storage keys, game mode constants
- **Audio system** (542–676): Web Audio context, synth functions, effect definitions
- **UI rendering** (680–739): HUD updates (hearts, bombs, scores)
- **Canvas & input** (752–862): Sizing, keyboard, mouse, touch handling
- **Game state & initialization** (864–911): Variable declarations, `resetGame()`
- **Drawing functions** (913–1020): Plane & boss rendering, explosion particles
- **Spawning** (1022–1067): Enemy, boss, power-up creation
- **Physics & collision** (1069–1295): Movement, bullet updates, rect overlap, hit detection
- **Boss patterns & mechanics** (1324–1454): Attack patterns, boss update logic
- **Game flow** (1456–1505): End state, pause, life loss handling
- **Main loop** (1520–1665): Draw, update, requestAnimationFrame

---

## Performance Considerations

- **Adaptive quality**: Frame timing tracked over 45 frames; `quality` level 0–2 reduces particle count and glow effects
- **Particle counts**: Scale with quality (full → 60% → 35% of intended count)
- **Shadow blur**: Only applied when `quality === 0`; reduced or disabled at lower quality levels
- **Object pooling**: Enemies, bullets, particles are added/removed arrays, not pooled (adequate for this game scale)
- **Canvas clearing**: Full `clearRect()` every frame (no dirty-rect optimization needed for current complexity)

---

## Debug Tips

- Check browser console for errors
- Inspect localStorage: `localStorage.getItem('skyfighter.*')` to view persisted values
- Monitor frame time: `trackPerf()` populates `frameTimes` array visible in DevTools
- Pause and inspect: Pause game (P key), use DevTools to inspect `player`, `boss`, `enemies` variables
- Audio issues: Verify AudioContext state and check if `muted` flag is true
