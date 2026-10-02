# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in vanilla HTML/CSS/JavaScript. The entire implementation is contained in a single `index.html` file (1,669 lines).

### Key Characteristics
- **Single-file architecture**: All game code, styling, and audio synthesis in one HTML file
- **No external dependencies**: Pure HTML5 Canvas, Web Audio API, vanilla JavaScript
- **Responsive design**: Scales to fit window, works on desktop and mobile
- **Accessibility-first**: Screen reader support, reduced motion preferences, color-independent UI
- **Performance-adaptive**: Automatically adjusts particle counts and effects based on frame rate

## Architecture at a Glance

### Game Structure (High-Level Flow)

The game runs in a single **IIFE** (Immediately Invoked Function Expression) from line 423 onwards. Key systems:

1. **Initialization** (lines 426-527)
   - DOM element references
   - Storage keys for persistence
   - Difficulty settings loaded from localStorage
   - Audio context setup

2. **Game State Management** (lines 863-869)
   - Core state: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
   - Game flow: `score`, `lives`, `stage`, `running`, `paused`, `gameOver`
   - Scoring chain system: `chain`, `chainTimer`, `multiplier`

3. **Main Game Loop** (lines 1640-1653)
   - Runs every frame via `requestAnimationFrame`
   - Calls `update(dt)` for game logic, then `draw()` for rendering
   - Tracks delta time (capped at 50ms) to handle frame rate variance

4. **Core Subsystems**
   - **Input Handling** (lines 768-862): Keyboard, mouse, touch/joystick support
   - **Game Logic** (lines 1148-1302): Movement, collision detection, entity updates
   - **Rendering** (lines 1520-1638): Canvas drawing with layering and effects
   - **Audio** (lines 542-676): WebAudio synth for all sound effects
   - **Boss AI** (lines 1372-1439): Attack patterns unique per boss design

### Key Game Systems

#### Entity System
All game objects (player, enemies, bullets) are simple objects with physics properties:
- `x, y`: Position
- `w, h`: Collision bounds (half-width/half-height for collision math)
- `speed`: Movement speed or bullet velocity
- **No class inheritance**—objects are plain JavaScript objects with data and behavior driven by type checks

#### Difficulty System (lines 485-489)
Four difficulty objects define spawn rates, enemy behavior, starting lives, bomb count, and boss HP multipliers. Currently three difficulties: Easy, Normal, Hard.

#### Weapon System (lines 871-1146)
- **Vulcan gun**: Multi-shot spread pattern, faster fire rate (0.15s cooldown)
- **Laser**: Piercing shots with wider spread at higher levels (0.1s cooldown)
- Weapons upgrade in three levels, then toggle between them
- Fire patterns are stateless—defined inline in `fireWeapon()`

#### Boss System (lines 1372-1454)
- **6 bosses**: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Each boss has a unique color scheme and three attack patterns
- Patterns use helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.
- Bosses telegraph attacks (visual/audio cue) before firing
- Boss selection cycles through the array based on stage number

#### Performance Adaptation (lines 473-483)
Tracks average frame time over 45 frames. If FPS drops below 40, increase `quality` level (fewer particles). If FPS exceeds 56, decrease quality level. Used in particle spawning and glow effects.

#### Scoring & Chain System (lines 1087-1101)
- Killing enemies increments `chain` and resets `chainTimer` to 2.2 seconds
- Multiplier increases: `1 + floor(chain / 5)`, max 10x
- Chain breaks if no kills for 2.2 seconds
- Each milestone (10 kills, 20, 30…) shows a combo pop-up

### Visual & Audio Design

#### Canvas Sizing (lines 753-765)
- Fixed logical size: **480×720** (portrait orientation)
- Scaled up to fit viewport (max scale 1.6x) maintaining aspect ratio
- Responsive on window resize and orientation change

#### Theming System (lines 493-500)
Six themes cycle by stage, each with gradient sky colors, cloud render color, and sun glow:
- Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge
- Used to vary visual appearance without changing gameplay

#### Audio Synthesis (lines 542-676)
- All sounds synthesized with oscillators and noise buffers
- No audio files—all created dynamically
- Uses frequency sweeps, filters, panning for spatial effects
- Compressor normalizes output
- `muted` flag and mute button control audio

## Common Development Tasks

### Running the Game
Simply open `index.html` in a web browser. The game starts with an overlay menu. No build, no server needed.

### Testing the Game
- **Desktop**: Use keyboard (Arrows/WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause)
- **Desktop**: Or use mouse (move to steer, click to fire, X to bomb)
- **Mobile**: Use on-screen joystick (left side) and FIRE/BOMB buttons (right side)
- Check browser DevTools Console for any JavaScript errors
- localStorage persistence testing: Open DevTools, Application tab, inspect `skyfighter.*` keys

### Common Code Edits

#### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502-506), e.g.:
   ```javascript
   heavy: { w: 50, h: 50, hp: 6, speedMul: 0.4, color: '#...',  dark: '#...', score: 50, minStage: 3 }
   ```
2. Add to choices in `spawnEnemy()` (around line 1026) with a spawn rate check
3. Add explosion SFX mapping in `EXPLODE_SFX` (line 678) if different sound needed
4. Add enemy-specific behavior in `update()` if needed (e.g., weave pattern, shoot logic)

#### Add a New Boss
1. Add to `BOSSES` array (line 508-515) with color scheme and HP multiplier
2. Add attack patterns to `BOSS_PATTERNS` array (line 1372+) at corresponding index
3. Boss selection in `spawnBoss()` uses `(stage - 1) % BOSSES.length` so new bosses automatically cycle

#### Tweak Game Balance
- Difficulty multipliers: Edit `DIFFICULTIES` object (lines 485-489)
- Spawn rates: Tweak `spawnBase` and `spawnMin` in difficulty definitions
- Enemy speeds, shoot chances: Modify `ENEMY_TYPES` or `DIFFICULTIES`
- Boss HP: Adjust `bossHpMul` in boss definitions or base HP in `spawnBoss()` (line 1046)
- Score values: Change numeric arguments to `addScore()` calls throughout

#### Adjust Visual Effects
- Screen shake magnitude/duration: `screenShake(mag, dur)` calls throughout
- Particle count: Line 1074 calculates based on `quality` level
- Glow effects: Toggled by `glow = quality === 0` check (line 1579)
- Flash intensity: Animations in CSS (lines 95-104)

#### Modify Controls
- Keyboard key mapping: Lines 769-776 (add/remove key codes)
- Fire rate (cooldown): Line 1177 (`fireDelay` values)
- Bomb behavior: `useBomb()` function (lines 1103-1122)
- Mouse vs. keyboard priority: Check around line 1161 in `update()`

#### Add a New Power-Up Type
1. Add to `maybeSpawnPowerUp()` spawn chances (lines 1058-1066)
2. Add case in `applyPowerUp()` (lines 1304-1322) to handle effect
3. Add to power-up rendering logic (lines 1617-1626)

### Audio Development
- Synth function is `beep(freq, dur, type, vol, opts)` with frequency sweep and filter options
- Noise is `noiseBurst(dur, vol, opts)` with filter
- All sounds in `sfx` object (lines 624-676) are composed of beep/noise calls
- To preview: Open DevTools Console, type `sfx.shoot()` or similar
- Pan value ranges -1 (left) to 1 (right), calculated by `panFor(x)` for spatial audio

## Architecture Notes for Editors

### Why Single File?
A single HTML file is intentional:
- **No build step required** — pure browser compatibility
- **Minimal deployment footprint** — one file to distribute
- **Clear dependencies** — all code visible in one place
- **Fast loading** — gzip compression is very efficient for homogeneous code

### Performance Considerations
- **Particle system**: Capped by quality level to maintain 60 FPS
- **Collision detection**: Uses cheap AABB (axis-aligned bounding box) in `rectsOverlap()`
- **Canvas shadow/glow**: Disabled at lower quality levels (line 1002)
- **Animation math**: Sine/cos for wave patterns recomputed every frame (acceptable because math is cheap)

### State Mutation Pattern
Objects are mutated directly; no immutability framework. Works because:
- Single-threaded execution
- All state in one scope (IIFE)
- No undo/redo system needed
- Game resets completely on `resetGame()`

### No External Libraries
- No DOM framework (vanilla Element APIs)
- No physics engine (hand-rolled AABB collision)
- No audio library (Web Audio API only)
- No bundler or transpiler needed

## Key Numbers (Constants)

- **Canvas logical size**: 480×720
- **Player speed**: 260 px/sec
- **Base enemy spawn interval**: 58 frames (normal), modulates with difficulty
- **Kills to trigger boss**: 18 per stage
- **Chain timeout**: 2.2 seconds
- **Boss telegraph time**: 22 frames (~367ms at 60 FPS)
- **Player invulnerability on hit**: 1.5 seconds
- **Stage cycle**: THEMES and BOSSES both loop every 6 stages

## Accessibility Features

- **Screen reader support**: Lives, score, stage updates announced via `announce()` function
- **Reduced motion preference**: Respects OS `prefers-reduced-motion` media query, also toggleable in-game
- **Color independence**: UI uses text labels + icons (❤️ for health, 💣 for bombs) not color alone
- **Keyboard-only playable**: All controls work without mouse/touch

## Performance Profiling Tips

- Use Chrome DevTools Performance tab: record a game session, filter by `draw()` and `update()`
- Watch the `quality` variable in Console to see adaptive performance in action
- Disable reduced motion in-game to see full glow/shake effect cost
- Test on low-end devices by throttling CPU in DevTools

## Common Pitfalls

1. **Collision bounds**: Objects use center + half-width for AABB math. Ensure `w` and `h` reflect actual collision size, not just visual size.
2. **Audio context**: May be suspended on first user interaction; `ensureAudio()` is called on input to resume.
3. **Delta time**: Capped at 50ms to prevent large jumps on frame drops. Update logic should use `dt` parameter, not frame count.
4. **Canvas transform state**: `ctx.save()/restore()` pairs are used; ensure they're balanced.
5. **Touch event listeners**: Use `{ passive: false }` on moveZone to allow `preventDefault()` for smooth mobile control.
