# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. The entire game is a single-file HTML5 Canvas implementation with no external dependencies. The game features:

- Vertical scrolling gameplay with player movement and continuous shooting
- Three difficulty levels (Easy, Normal, Hard) with configurable enemy spawning and behavior
- 6 unique boss encounters with distinct attack patterns
- Two weapon types: Vulcan (rapid fire) and Laser (piercing shots)
- Power-ups: weapon upgrades, extra lives, smart bombs
- Dynamic stage themes with parallax cloud effects
- Score multiplier system based on kill streaks
- WebAudio synth-based sound effects (no external audio files)
- Touch and keyboard controls with mouse steering support
- Adaptive performance quality setting
- LocalStorage persistence for high score, difficulty, audio/motion preferences

## Architecture & Code Organization

The game is implemented as a single-file HTML/CSS/JavaScript application. All code lives in `index.html` within a `<script>` tag starting at line 422. The structure follows a typical game loop pattern:

### High-Level Flow

1. **Initialization** (lines 423-527): DOM references, constants, difficulty/theme definitions, local storage loading
2. **System Setup** (lines 542-876): Audio (WebAudio synth), input handling (keyboard/touch/mouse), game state initialization
3. **Core Game Loop** (lines 1640-1652): `requestAnimationFrame(loop)` → `update(dt)` → `draw()` every frame
4. **Game Subsystems**: Organized by function, not by file

### Key Game Systems

**Player & Movement** (lines 753-861, 1161-1181)
- Player object: `{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }`
- Input handling supports three control schemes: keyboard (arrows/WASD), mouse (position tracking), touch (virtual joystick)
- Player is constrained within canvas bounds; mouse input applies "glide" physics toward cursor

**Weapon System** (lines 1100-1147)
- Two weapons: "vulcan" (rapid spread fire) and "laser" (piercing narrow beam)
- Weapon upgrades progress through 3 levels, then cycle to the other weapon at level 1
- Fire cooldown: 0.15s for vulcan, 0.1s for laser
- Vulcan fires 3 bullets per shot with slight spread; Laser fires 1 bullet with pierce property

**Enemy System** (lines 502-506, 1022-1041, 1210-1227)
- 3 enemy types defined in `ENEMY_TYPES` constant: grunt (1 HP), interceptor (1 HP, fast), tank (4 HP, slow)
- Enemies spawn at top with random horizontal positions; some types have weave AI (sinusoidal movement)
- Shooting patterns: "spread" (3-bullet fan) or "aimed" (single bullet tracking player)
- Enemy spawn rate increases with time and stage; difficulty setting controls spawn frequency and enemy behavior

**Stage & Boss System** (lines 491, 508-515, 1043-1056, 1198-1207, 1411-1454)
- Stages progress after 18 enemy kills (`KILLS_PER_STAGE`); boss spawns after warning banner
- 6 boss designs in `BOSSES` array, cycled per stage; each with unique color scheme and HP multiplier
- Boss attack patterns stored in `BOSS_PATTERNS` array; each pattern function fires bullets with specific geometry
- Boss telegraphs attacks with visual core glow before executing; telegraph duration is 22 frames

**Collision & Damage** (lines 1069-1071, 1232-1287)
- AABB collision via `rectsOverlap(a, b)`: centers and half-widths
- Player bullets damage enemies (1 HP per hit) and bosses (3 HP per hit); laser bullets pierce
- Enemy bullets and bosses damage player (1 life per hit) with 1.5s invulnerability after
- Collision detection runs backward through arrays to safely splice during iteration

**Scoring & Multiplier** (lines 1087-1102)
- Base score per enemy type: grunt=10, interceptor=15, tank=35, boss=500
- Kill chain: +1 per enemy, resets if 2.2s pass without a kill
- Multiplier: 1 + floor(chain / 5), capped at 10x
- Score = base × multiplier

**Particle & Visual Effects** (lines 1073-1085, 1507-1545)
- Explosion particles: random velocity, 0.5s lifetime, alpha fade
- Particle count scales with performance quality setting (full → half → third)
- Screen shake via canvas translation; intensity and duration configurable
- Flash (white screen) overlay for hits and bombs
- Parallax clouds rendered every frame at two depths

**Difficulty Settings** (lines 485-489)
- Easy: spawn base 75, min 40, enemy speed 1.0, shoot chance 45%, 4 lives, 3 bombs, 0.8× boss HP
- Normal: spawn base 58, min 26, enemy speed 1.4, shoot chance 70%, 3 lives, 2 bombs, 1.0× boss HP
- Hard: spawn base 40, min 18, enemy speed 1.9, shoot chance 95%, 3 lives, 2 bombs, 1.3× boss HP

**Theme System** (lines 493-500)
- 6 themes cycle per stage with distinct sky gradients, cloud colors, sun colors
- Themes are purely visual; no gameplay difference

**Audio** (lines 542-676)
- WebAudio synth-based; no external files needed
- Oscillators and filters create tones; noise bursts create percussion
- Spatial audio: bullets and hits pan based on X position
- Mute state stored in localStorage

**Adaptive Quality** (lines 473-483)
- Tracks frame times over 45 frames
- Reduces particle count and shadow effects if avg frame time > 1/40s
- Re-increases quality if frame time < 1/56s
- Accessed via `quality` variable: 0=full, 1=medium, 2=low

### State Management

Game state variables live in global scope (lines 863-869):
- `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- `score`, `lives`, `maxLives`, `running`, `paused`, `gameOver`
- `bombCount`, `maxBombs`, `spawnTimer`, `frame`, `kills`, `stage`, `stageKills`, `bossWarned`
- `chain`, `chainTimer`, `multiplier`

### Canvas & Rendering

- Base canvas size: 480×720 (hardcoded at line 753)
- Scales with CSS to fit window, respecting aspect ratio (max 1.6× upscale)
- Context saved/restored for transformations and global state changes
- Rendering order: sky gradient + sun + clouds → player → bullets → enemies → boss → power-ups → particles

### Input Handling

Three input sources with priority system:
1. **Keyboard** (lines 769-776): Arrow keys or WASD for movement, Space to fire, X/Shift for bomb, P/Esc to pause
2. **Touch** (lines 814-834): Virtual joystick (left 55% of screen) and fire/bomb buttons (right 45%)
3. **Mouse** (lines 838-861): Position drives glide target; click to fire; no priority, coexists with keyboard

Priority: keyboard/joystick > mouse. If keys or joy are active, mouse input is ignored.

## Development & Building

### Running the Game

No build step required. Open `index.html` directly in a browser. The game is fully self-contained:
- All CSS inline in `<style>` tag
- All JavaScript inline in `<script>` tag
- No external assets or dependencies

### Testing Changes

After editing `index.html`:
1. Save the file
2. Refresh the browser (Ctrl+R or Cmd+R)
3. Reload if changes don't appear (hard refresh: Ctrl+Shift+R or Cmd+Shift+R)

### Common Modifications

**Adjust Difficulty Settings** (line 485-489)
- Edit `DIFFICULTIES` object to tweak spawn rates, enemy speed, lives, etc.
- Changes apply next game (no restart needed)

**Add/Modify Enemy Types** (line 502-506)
- Edit `ENEMY_TYPES` object; add `minStage` property to gate appearance
- Update spawn logic in `spawnEnemy()` if needed

**Add Boss Patterns** (line 1324-1409)
- Create new pattern functions (e.g., `fireRadial()`, `fireColumns()`)
- Add them to `BOSS_PATTERNS[index]` array

**Adjust Boss HP/Behavior** (lines 508-515)
- Modify `BOSSES` array: color properties, hpMul
- Damage dealt per hit is hardcoded at line 1259 (3 HP)

**Tweak Audio** (line 624-676)
- Modify synth parameters in `sfx` object: frequency, duration, waveform, filter type
- Add new sound effects by creating functions with `beep()` and `noiseBurst()`

**Change Theme Colors** (line 493-500)
- Edit `THEMES` array: sky gradient stops, cloud colors, sun colors

## Performance Considerations

- Particle counts scale automatically with `quality` variable
- Shadow effects (glows) disabled at lower quality
- Collision detection runs in nested loops but counts are typically < 50 total objects
- Canvas is cleared every frame; no dirty-rect optimization
- LocalStorage access only on settings changes and game over

## LocalStorage Keys

- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state (1 or 0)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Accessibility setting (1 or 0)

## Browser Compatibility

- Requires Canvas 2D context
- Requires WebAudio API (gracefully degrades to no sound if unavailable)
- Touch API for mobile support
- LocalStorage for persistence
- `matchMedia` for reduced-motion preference detection
