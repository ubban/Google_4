# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML application. The game runs entirely in the browser with no external dependencies or build process.

### Key Features
- Canvas-based 2D rendering with dynamic lighting and particle effects
- Vertical scrolling shoot-'em-up gameplay with progressive difficulty
- 6-stage boss encounters with distinct attack patterns
- Weapon upgrade system (Vulcan/Laser with 3 upgrade levels)
- Procedural audio using WebAudio synthesis (no external audio files)
- Touch and mouse/keyboard input support
- Responsive design scaling for mobile and desktop
- Accessibility features: reduced motion, screen reader support, semantic HTML

## Architecture Overview

The entire game is contained in `index.html` (~1670 lines: 360 lines CSS + 1307 lines JavaScript).

### Game Systems

**Rendering & Canvas**
- `BASE_W = 480, BASE_H = 720`: Fixed game resolution, scaled responsively
- Canvas context draws: sky gradient, clouds, player, enemies, projectiles, particles, UI overlays
- Adaptive quality system: reduces particle count when frame time exceeds thresholds (40fps and 56fps targets)

**Game State**
- `player`: Position, weapon, level, lives, invulnerability timer
- `running`, `paused`, `gameOver`: State flags
- `score`, `multiplier`, `chain`: Scoring system with combo chains (breaks after 2.2 seconds of no kills)
- `stage`, `stageKills`, `boss`: Progressive stage system, boss encounter tracking

**Entity Systems**
- `enemies`: Array of enemy objects with type-based stats (grunt, interceptor, tank)
- `bullets`: Player projectiles (Vulcan: spread pattern, Laser: piercing)
- `enemyBullets`: Enemy projectiles with radial, aimed, or pattern-based fire
- `powerUps`: Drop from enemies (life, bomb, weapon)
- `particles`: Explosion debris, collision effects
- `boss`: Single active boss with health, attack patterns, telegraph warning

**Difficulty Settings** (`DIFFICULTIES` object)
- easy/normal/hard: Controls spawn rates, enemy speed, shoot chance, lives, bombs, boss HP scaling
- Adaptive spawning: `spawnBase - Math.floor(frame / 300) * 4` increases difficulty over time

**Boss System**
- 6 unique bosses cycling with stages, each with distinct visual design and 3-pattern attack sequences
- Boss patterns: radial bursts, aimed spreads, columns, spirals, homing shots, teleport dashes
- `telegraph` timer (22 frames): Boss telegraphs attack with visual glow before firing

**Audio (WebAudio Synth)**
- No external files; all SFX generated programmatically via `beep()` and `noiseBurst()`
- Muting controlled by `muted` flag, stored in localStorage
- Positional panning based on entity x-position

**Input Handling**
- Keyboard: Arrow keys/WASD move, Space/click fire, X/Shift bomb, P/Esc pause
- Mouse: Move cursor to steer (smooth glide toward mouse), click to fire
- Touch: Left side joystick for movement, right side buttons for fire/bomb
- Touch joystick: Visual base + stick, constrained to 50px radius

**Persistence (localStorage)**
- `skyfighter.bestScore`: High score
- `skyfighter.difficulty`: Last selected difficulty (easy/normal/hard)
- `skyfighter.muted`: Audio mute state
- `skyfighter.reducedMotion`: User's motion reduction preference

**Visual Feedback**
- `screenShake()`: Magnitude and duration, reduced by 85% when reduced motion enabled
- `flashHit()` / `flashBomb()`: CSS animation on overlay element
- `showComboPop()`: Animated combo text at 30% vertical position
- `showStageBanner()`: Stage transition banner with 2-second hold
- Vignette red pulse when health is critical (1 life remaining)

## Development Workflow

**No build step required** — open `index.html` directly in a browser to play/test.

### Quick Start
```bash
# On macOS/Linux, start a local server:
python3 -m http.server 8000

# Then open http://localhost:8000/index.html
# Or directly: file:///path/to/index.html (works but breaks service workers if ever added)
```

### Manual Testing Checklist
- **Gameplay**: Player movement (arrow/WASD), firing (space/click), bombing (X/Shift)
- **Touch**: Joystick movement, FIRE/BOMB buttons on right side
- **Enemies**: All three types spawn, correct difficulty spawning, attack patterns
- **Bosses**: Stage progression, boss telegraphs, attack variety, defeat triggers stage advance
- **Scoring**: Kills increment score, combo chains multiply, breaks on gap
- **Difficulty**: Easy/Normal/Hard buttons change spawn rate and enemy behavior
- **Accessibility**: Reduced motion toggle disables shake/flash/animations, screen reader announces events
- **Audio**: Mute button, all SFX play correctly, positional panning works
- **Persistence**: Best score saved, difficulty/mute state persists across reloads

## Code Organization

All code is in a single IIFE (Immediately Invoked Function Expression) to avoid global scope pollution.

### Key Sections (in order)
1. **Constants & Configuration** (lines 463–515)
   - Storage keys, difficulty profiles, themes, enemy types, boss definitions
   
2. **State Variables** (lines 517–527, 864–869)
   - Game state, UI element references, input tracking
   
3. **Audio Setup** (lines 543–676)
   - `ensureAudio()` initializes WebAudio context
   - `beep()`, `noiseBurst()` generate synth sounds with filters/panning
   - `sfx` object maps game events to audio functions
   
4. **UI Updates** (lines 680–750)
   - `renderHearts()`, `renderBombs()`: HUD rendering
   - `showComboPop()`, `showStageBanner()`: Overlay animations
   - Difficulty, mute, motion button handlers
   
5. **Canvas Rendering** (lines 752–763)
   - Dynamic canvas sizing to fit viewport while maintaining 480×720 aspect
   
6. **Input Handling** (lines 767–861)
   - Keyboard, mouse, touch event listeners
   - Joystick implementation with visual feedback
   - Input priority: keyboard/joystick > mouse (mouse can't override active input)
   
7. **Game Update Loop** (lines 1148–1302)
   - `update(dt)`: Player movement, weapon firing, entity updates, collision detection
   - Called once per frame when not paused
   
8. **Entity Management**
   - Spawning: `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`
   - Collision: `rectsOverlap()` for all entity types
   - Updates: Player/enemy/bullet position, particle life, boss telegraph
   
9. **Rendering** (lines 1520–1638)
   - `draw()`: Full frame render with screen shake, backgrounds, all entities, overlays
   - Quality-dependent effects: glow shadows, particle count
   
10. **Game Loop** (lines 1640–1665)
    - `loop(now)`: requestAnimationFrame entry point, dt capped at 50ms
    - Calls `update()` then `draw()` if not paused
    
11. **Pause & Game Over** (lines 1469–1505)
    - Overlay state management for pause/game-over screens

## Common Modification Points

### Adding a New Enemy Type
1. Add to `ENEMY_TYPES` object with `{ w, h, hp, speedMul, color, dark, score, minStage, weave? }`
2. Add to `spawnEnemy()` choices filter if stage-gated
3. Create explosion SFX entry in `EXPLODE_SFX` object
4. Add corresponding SFX function in `sfx` object

### Adding a New Boss
1. Add to `BOSSES` array: `{ name, hullTop, hullBottom, wing, core, hpMul }`
2. Create 3-4 attack pattern functions (fireRadial, fireAimedSpread, etc.)
3. Add pattern array to `BOSS_PATTERNS` at matching index
4. Boss auto-cycles by stage: `BOSSES[(stage - 1) % BOSSES.length]`

### Adjusting Difficulty
- Modify `DIFFICULTIES.{easy,normal,hard}` object properties
- `spawnBase/spawnMin`: Frames between enemy spawns (lower = harder)
- `enemySpeed`, `enemyShootChance`: Enemy behavior scaling
- `lives`, `bombs`: Starting resources
- `bossHpMul`: Multiplier on boss health

### Particle Effects & Visuals
- `spawnExplosion(x, y, color, count)`: Creates debris
- `screenShake(mag, dur)`: Adds camera jitter (reduced 85% with reduced-motion)
- `flashHit()`, `flashBomb()`: Flash overlay animations in CSS

### Sound Design
- All sounds use `beep(freq, dur, type, vol, opts)` or `noiseBurst(dur, vol, opts)`
- `opts` can include: `sweepTo`, `filterType`, `filterFreq`, `pan`, `delay`
- Adjust frequency, duration, volume, or filters to tweak tone

## Performance Considerations

- **Adaptive Quality**: Tracks frame time; reduces particles and shadow effects at lower frame rates
- **Collision**: Nested loop over enemies and bullets — O(n²) but acceptable for ~50 entities max
- **Canvas Rendering**: No dirty-rect optimization; full redraw each frame is acceptable at 480×720
- **Audio**: WebAudio context pooling via `ensureAudio()` (lazy init on first user interaction)

## Browser Compatibility

- **Modern browsers**: Chrome, Firefox, Safari, Edge (2020+)
- **Requirements**: Canvas 2D, WebAudio API, LocalStorage, requestAnimationFrame
- **Touch support**: Detected via `ontouchstart` in window or `navigator.maxTouchPoints`
- **Accessibility**: ARIA live region for screen reader announcements, prefers-reduced-motion media query

## Testing Notes

- Game state is deterministic given same input sequence (no randomness in core physics)
- Randomness is intentional: enemy spawns, bullet patterns, power-up drops — used for replayability
- Manual testing is primary; no automated test suite
- Visual regression: Play through multiple stages, check enemy variety, boss patterns, explosion effects
