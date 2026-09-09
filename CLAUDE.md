# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. The entire game is implemented as a single HTML file (`index.html`) with embedded JavaScript and CSS. The game runs in a browser with no external dependencies or build process.

**Key Features:**
- Multiple difficulty levels (Easy, Normal, Hard)
- Dynamic weapon system (Vulcan with spread, Laser with pierce)
- Enemy variety (Grunts, Interceptors, Tanks) with progressively harder enemies per stage
- Boss encounters with unique bullet patterns per boss
- Smart bomb mechanic with limited uses
- WebAudio-synthesized sound effects (no external audio files)
- Full touch + keyboard + mouse input support
- Responsive canvas scaling for different screen sizes
- Performance adaptation (quality setting adjusts particle/shadow rendering)
- Persistence: best score and user preferences (difficulty, mute, reduced motion) stored in localStorage

## Development

### Running the Game

Open `index.html` in a modern web browser. No build process, server, or installation needed.

### Code Structure

All game logic lives within a single IIFE (Immediately Invoked Function Expression) starting at line 423. The structure is:

1. **Constants & Configuration** (lines 463-515)
   - `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`
   - Storage keys for localStorage
   - Base canvas dimensions (480x720)

2. **Audio System** (lines 542-676)
   - WebAudio context initialization
   - Beep synthesis (`beep()`) and noise generation (`noiseBurst()`)
   - Sound effect catalog (`sfx` object) with parametric audio design
   - Example: `sfx.bomb()` layers 3 beeps + 2 noise bursts with frequency sweeps

3. **UI & Input** (lines 679-861)
   - DOM element references and state management
   - Input handlers: keyboard, touch (joystick), mouse steering
   - Canvas sizing and responsive scaling
   - Difficulty/mute/motion-reduction toggles
   - Pause mechanic

4. **Game State & Update Loop** (lines 863-1302)
   - Core game state: `player`, `bullets`, `enemies`, `boss`, `particles`, `powerUps`
   - `update(dt)` function runs physics each frame (movement, collision, spawning)
   - Frame counter-based spawning with difficulty ramping
   - Collision detection using `rectsOverlap()`

5. **Enemy & Boss Systems** (lines 1022-1439)
   - Enemy spawning with type selection based on stage
   - Boss spawning and state machine (entering → combat)
   - Boss-specific attack patterns: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.
   - Each boss has 3 attack patterns chosen per engagement

6. **Rendering** (lines 1507-1638)
   - Canvas drawing: background gradient, clouds (parallax), player, enemies, bullets, particles
   - Boss health bar updates
   - Screen shake effect (reduced if reduced-motion is enabled)
   - Glow effects (shadows) only rendered at high quality setting

7. **Game Loop** (lines 1640-1666)
   - `requestAnimationFrame` callback
   - Delta-time calculation and clamping (max 50ms per frame)
   - Performance tracking (`trackPerf()`) adjusts quality dynamically

### Key Game Constants

- **Stage progression**: 18 enemy kills (`KILLS_PER_STAGE`) trigger boss
- **Weapon progression**: 3 levels per weapon type; at level 3, switching weapons resets to level 1
- **Bomb mechanics**: 2-3 bombs per game (difficulty-dependent); bombs clear all enemy bullets + deal 24 damage to boss
- **Difficulty scaling**: `spawnBase`/`spawnMin` control enemy spawn rate; `enemySpeed`, `enemyShootChance` control enemy behavior; `bossHpMul` scales boss health
- **Themes**: 6 rotating sky gradients with unique cloud/sun colors

### Common Development Tasks

#### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (around line 502):
   ```javascript
   newEnemy: { w: 36, h: 36, hp: 2, speedMul: 1.3, color: '#abc123', dark: '#789def', score: 25, minStage: 3 }
   ```
2. Set `minStage` to control when it first appears
3. If spawning visuals differ from `drawPlane()`, add a case in `enemies.forEach()` rendering section (around line 1603) or create a new `drawNewEnemy()` function
4. If sound differs, add to `EXPLODE_SFX` map and create corresponding `sfx.explodeNewEnemy()` function

#### Adding a Boss Attack Pattern
1. Create firing function using existing helpers: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, etc. (lines 1324-1370)
2. Add 3-element array to `BOSS_PATTERNS` (line 1372) at the index matching the boss's position in `BOSSES` array
3. Each function receives the boss object `b` and fires bullets into `enemyBullets` array
4. Patterns are cycled during the encounter; telegraph time before each shot (22 frames)

#### Creating New Sound Effects
1. Add function to `sfx` object (line 624):
   ```javascript
   newSound: () => {
     beep(freq, duration, type, volume, { sweepTo, filterFreq, pan });
     noiseBurst(duration, volume, { filterFreq, pan });
   }
   ```
2. Call `ensureAudio()` before audio context access (handles initial browser autoplay policy)
3. Frequencies (Hz): gunshot ~200–800, laser ~1000–2500, explosion ~50–200
4. Common filter types: `'lowpass'`, `'highpass'`, `'bandpass'`

#### Adjusting Difficulty Curves
- Modify `DIFFICULTIES` object (line 485)
- `spawnBase` controls initial spawn interval (frames); lower = more enemies
- `spawnMin` is the minimum spawn interval as difficulty ramps
- `enemySpeed` multiplies enemy velocity
- `enemyShootChance` (0–1) probability that an enemy shoots
- `bossHpMul` multiplies base boss HP (70 + stage*35)

#### Tweaking Weapon Balance
- **Vulcan** spread pattern at level 3 (line 1127): adjust `[-0.24, 0, 0.24]` spread angles
- **Laser** pierce count at level 3 (line 1141): increase/decrease bullet `pierce` value
- Cooldown (fire rate): line 1177, `fireDelay` constant per weapon
- Damage: enemy bullets reduce enemy `hp` by 1 (line 1237), boss bullets reduce by 3 (line 1259)

### Performance Notes

- **Quality levels**: 0 (full), 1 (medium, ~60% particle count), 2 (low, ~35% particle count)
- Automatically adapted by `trackPerf()` based on frame time rolling average
- At quality 0, shadows/glows are rendered; at quality > 0, shadows are reduced/removed
- Particle count scaled in `spawnExplosion()` (line 1074)
- Screen shake magnitude also reduced at low quality for reduced-motion preference

### Browser APIs Used

- **Canvas 2D**: drawing, transforms, gradients, shadows
- **WebAudio API**: oscillators, noise generation, filters, stereo panning, gain envelopes
- **localStorage**: best score, settings persistence
- **requestAnimationFrame**: main loop
- **Touch Events**: joystick input (pointer-events based)
- **Mouse/Keyboard Events**: firing, movement, pause

### Testing Strategy

- Open in browser at different screen sizes/orientations (mobile, tablet, desktop)
- Test all 3 difficulty modes
- Test touch, keyboard, and mouse control paths independently
- Verify audio context resumes on first user interaction
- Check localStorage persistence across page reloads
- Monitor performance at 60 FPS on low-end devices (Quality setting should adapt)

## Architecture Notes

**Single-file design rationale:**
- No build process, no dependencies → zero friction to run or modify
- Canvas-based rendering → tight performance control
- WebAudio synthesis → no audio asset loading or licensing concerns

**Why not a framework?**
- Vanilla JS keeps the codebase small (~1670 lines)
- No module bundler needed
- Game loop is simple enough to understand at a glance
- Direct control over performance profiling and optimization

**State management:**
- Global scope within IIFE (not leaked to `window`)
- Game state variables: `player`, `bullets`, `enemies`, `boss`, `score`, `lives`, etc.
- Mutation-based updates (no immutability layer)
- localStorage for cross-session persistence

**Collision & Physics:**
- Axis-aligned bounding box (AABB) collision via `rectsOverlap()`
- Delta-time based movement (`dt` in seconds, clamped to 50ms)
- No physics engine; simple lerp for mouse steering, hardcoded AI patterns for bosses

## Future Improvements (if needed)

- **Particle pooling**: Pre-allocate particle objects to reduce GC pressure
- **Enemy bullet pooling**: Similar to particles
- **Mobile-specific tuning**: Touch target sizes, control responsiveness
- **Accessibility**: More detailed aria-live announcements, keyboard-only mode
- **Level editor**: JSON-based level definitions instead of hardcoded patterns
