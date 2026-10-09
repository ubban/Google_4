# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game. It's a single-file web application (index.html) implementing a complete arcade-style shmup with multiple difficulty levels, boss encounters, weapon systems, and procedurally-generated audio.

**Key Stats:**
- Single 1670-line HTML file with embedded CSS and JavaScript
- Canvas-based 2D graphics (480x720 viewport)
- Web Audio API synthesized sound effects
- Multi-modal input: keyboard, mouse, and touch (with joystick on mobile)

## Architecture Overview

The entire game is contained in `/index.html` with three sections:

### 1. **HTML Structure** (lines 2-420)
- Single `<canvas id="gameCanvas">` for game rendering
- HUD elements: score, health hearts, bomb indicators, weapon status
- Overlay system for menus, pause, and game-over screens
- Touch control zones (left side for movement, right side for fire/bomb)
- Accessibility: screen-reader live region (`#srLive`)

### 2. **CSS Styling** (lines 10-359)
- Responsive design using `clamp()` for fluid scaling
- Dark space theme with gradient backgrounds
- Animation system for UI feedback (combo pop, stage banner, screen flash)
- HUD status indicators with backdrop blur effects
- Safe-area support for notched/curved displays
- Reduced motion preference support (`:prefers-reduced-motion`)

### 3. **JavaScript Game Engine** (lines 422-1666, inside IIFE)

#### Core State Management
- **Game State Variables** (line 864-869): `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, etc.
- **Input State** (line 768-806): keyboard keys tracking, mouse/touch input, joystick state
- **Persistent State** (localStorage): best score, mute, difficulty, reduced motion preference

#### Game Loop (line 1640-1653)
Uses `requestAnimationFrame()` with delta-time (`dt`) for frame-rate independent physics:
- **update(dt)**: Game logic (movement, collision, spawning)
- **draw()**: Canvas rendering with screen shake applied

#### Key Systems

**1. Difficulty Settings** (line 485-489)
Three presets (`DIFFICULTIES` object): Easy/Normal/Hard with variance in:
- Enemy spawn rates and minimum delays
- Enemy speed and shoot probability
- Lives and bombs per run
- Boss HP multiplier

**2. Enemy System** (line 502-506, 1022-1041)
Three enemy types in `ENEMY_TYPES`:
- `grunt`: Basic red square, 1 HP
- `interceptor`: Fast orange, 1 HP, weaving behavior
- `tank`: Slow purple, 4 HP, spawns at stage 2+
- Each enemy has: position, speed, shooting behavior (spread or aimed), weave pattern

**3. Boss System** (line 508-515, 1043-1056, 1411-1454)
Six boss designs in `BOSSES` array, each with:
- Unique visual design (hull colors, core glow)
- HP scaling per difficulty and inherent multiplier
- Animated entry sequence, warning animation
- Boss bar HUD display

**Boss Attack Patterns** (line 1372-1409):
Each boss has 3 distinct attack patterns that cycle:
- VANGUARD: Radial bursts, aimed fans, wide V-formations
- BEHEMOTH: Straight columns, homing spreads, dense walls
- SPECTER: Teleport-dash, spirals, double aimed shots
- DREADNOUGHT: 8-way crosses, homing missiles, radial walls
- WRAITH: Teleport barrage, expanding rings, cross-spreads
- TITAN: Heavy columns, full rings, relentless homing

Pattern attacks use helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`.

**4. Weapon System** (line 871-872, 1124-1146, 1304-1322)
- **Vulcan** (line 1127-1134): Spread shot, 3 levels of spread width
- **Laser** (line 1135-1145): High-speed pierce shot, 3 levels increasing lane count
- Weapon upgrades found as power-ups; reach level 3 to swap weapon
- Upgrade power-ups are ~20% drop chance

**5. Collision System** (line 1069-1071)
Simple axis-aligned bounding box collision: `rectsOverlap(a, b)` checks center distance vs. combined half-widths

**6. Particle Effects** (line 1073-1085)
Procedural explosions spawn particles with:
- Random velocity in all directions
- Life decay and size shrinking
- Color matches explosion type (orange for enemies, pink for boss, red for impact)
- Performance scaling: quality 0 → full particles, 1 → 60%, 2 → 35%

**7. Score & Combo Multiplier** (line 1087-1101)
- Each enemy kill extends the combo chain (2.2-second window)
- Multiplier: 1x base → 1.8x at 5 kills → 2.8x at 15 kills (capped at 9x)
- Failing to kill for 2.2s breaks the chain and resets multiplier
- Visual feedback: combo counter pops up at 10, 20, 30... kills

**8. Visual Themes** (line 493-500)
Six rotating stage themes with sky gradients, cloud colors, and sun tints:
- Daylight Front (blue sky)
- Sunset Corridor (pink/orange)
- Night Approach (dark blue)
- Storm Front (green)
- Arctic Wastes (pale blue)
- Volcanic Ridge (red/orange)

**9. Audio System** (line 542-676)
Web Audio synthesized SFX using oscillators and noise bursts:
- `beep(freq, dur, type, vol, opts)`: Sine/square/sawtooth with optional sweep, filter, pan
- `noiseBurst(dur, vol, opts)`: Procedural noise with lowpass filter
- Sounds: shoot, laser, explosions (per enemy type), hit, bomb, power-up, boss telegraph, stage clear, UI click, game over
- Stereo panning based on event position (`panFor()`)
- Mute toggle persisted in localStorage

**10. Adaptive Performance** (line 474-483)
Tracks frame times over 45 frames:
- Average > 1/40s → quality++
- Average < 1/56s → quality--
- Quality affects: particle count, shadow blur, glow effects
- Mostly used in drawing to reduce expensive operations

**11. Input Handling**
- **Keyboard** (line 769-776): Arrow/WASD movement, Space fire, X/Shift bomb, P/Esc pause
- **Mouse** (line 847-861): Position aims player (smooth glide), click fires, move over canvas to enable steering
- **Touch** (line 814-835): Left side drag for movement joystick, right side buttons for fire/bomb
- Input priority: Keyboard > Joystick > Mouse (prevents conflicts)

## Common Development Tasks

### Running the Game
Open `index.html` in any modern browser (requires no build step or server):
```bash
# On Linux/Mac
open index.html

# Or via a simple server
python3 -m http.server 8000
# Then visit http://localhost:8000/index.html
```

### Testing Specific Scenarios
The game has debug-friendly design:
- **Difficulty toggle**: In-game overlay shows Easy/Normal/Hard buttons (persists in localStorage)
- **Reduced motion**: Top-right button toggles screen shake/flash/animations for testing accessibility
- **Pause**: P key or Esc, or click pause button for debugging specific moments
- **Mute**: Click speaker icon to silence SFX for testing gameplay without audio

### Browser DevTools Usage
- **Canvas inspection**: Open DevTools → Elements tab, find `#gameCanvas`
- **Game state**: Console access to all module-scope variables (enter IIFE context)
- **Performance**: DevTools → Performance tab, record game loop to profile update/draw
- **Network**: Game makes zero network requests; entirely self-contained

### Extending Gameplay
**Adding a New Enemy Type:**
1. Add entry to `ENEMY_TYPES` object (line 502-506) with stats: w, h, hp, speedMul, color, score, minStage
2. Add spawn probability in `spawnEnemy()` (line 1028-1030)
3. Add explosion SFX mapping in `EXPLODE_SFX` (line 678)
4. Add explosion SFX function (e.g., `sfx.explodeMyType()`)

**Adding a New Boss:**
1. Add design to `BOSSES` array (line 508-515): name, hull colors, core color, hpMul
2. Add attack pattern functions (new entries in `BOSS_PATTERNS` array, line 1372-1409)
3. Boss index auto-selects design by `(stage - 1) % BOSSES.length`

**Adding a Stage Theme:**
1. Add to `THEMES` array (line 493-500): sky gradient stops (3-color array), cloud RGB, sun RGB, name
2. Theme auto-rotates by `(stage - 1) % THEMES.length`

### Code Organization Notes
- **All state at module scope**: Variables defined line 864-869 are the single source of truth; no global objects
- **Utility functions grouped by purpose**: Audio (sfx functions), drawing (drawPlane, drawBossShip, drawClouds), input handling (joystickStart, joystickMove), collision (rectsOverlap)
- **Closures for encapsulation**: Event listeners and functions defined inside IIFE; nothing pollutes global scope
- **Canvas coordinate system**: (0, 0) = top-left, x increases right, y increases down; player y-up motion is subtraction

## Performance Considerations

1. **Particle system**: Adaptive quality scaling to maintain 60fps
2. **Enemy spawn rate**: Difficulty-based, increases with frame counter ramp (line 1201-1202)
3. **Boss pattern complexity**: Dynamically scales projectile counts based on quality setting
4. **Shadow/glow effects**: Conditionally applied only in full-quality mode
5. **Canvas clear + redraw**: Every frame; no dirty rect optimization needed for single-screen game

## Accessibility Features

1. **Screen reader support**: Announcements via live region (#srLive, line 468-471)
2. **Reduced motion**: Toggleable via UI (icon button, top-right); CSS `prefers-reduced-motion` respected on load
3. **Keyboard navigation**: All actions playable via keyboard; mouse not required
4. **Touch support**: Full mobile joypad emulation
5. **Color contrast**: All UI text uses high-contrast colors on dark backgrounds
6. **Safe area support**: HUD and buttons respect notched displays via `env(safe-area-inset-*)`

## localStorage Keys

Game persists state via these keys:
- `skyfighter.bestScore`: High score (number)
- `skyfighter.muted`: Audio mute flag (0 or 1)
- `skyfighter.difficulty`: Selected difficulty ("easy", "normal", "hard")
- `skyfighter.reducedMotion`: Reduced motion preference (0 or 1)

## Debugging Tips

**To inspect a boss pattern live:**
```javascript
// In DevTools console, add this before spawning a boss:
BOSS_PATTERNS[0][0](boss); // Fire pattern 0 of VANGUARD immediately
```

**To freeze and inspect state:**
```javascript
// Pause and inspect:
paused = true;
console.log(enemies, player, boss);
```

**To test performance on slow device:**
Manually set `quality = 2` in console to simulate low-end device rendering.

## Future Enhancement Areas

- **Save/load mid-game**: Game restarts on each play; could add checkpoint saves
- **Leaderboard**: No server integration; localStorage only tracks one best score
- **Custom difficulty**: Fine-tune spawn rates, damage, enemy balance per setting
- **New player onboarding**: Tutorial level or hint system for newcomers
- **Replay system**: Record inputs and playback; useful for streaming
- **Screen reader enhancements**: More detailed boss telegraph announcements for accessibility
