# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented entirely in a single HTML file (~1700 lines). It features:
- Canvas-based graphics rendering
- Synthesized sound effects via Web Audio API (no external audio files)
- Responsive design for desktop and mobile
- Three difficulty levels with progressive challenges
- Boss battles with distinct attack patterns
- Multiple weapon systems and power-ups

**Technology**: Pure HTML5/Canvas, JavaScript (ES6+), Web Audio API. No build system or external dependencies.

## Architecture Overview

The entire game is contained in `index.html` with three main layers:

### 1. Presentation Layer (HTML + CSS)
- **Markup**: Game canvas, HUD elements, overlays, buttons, touch controls
- **Styling**: CSS animations, responsive scaling, accessibility features
- **Key elements**:
  - `#gameCanvas`: 480×720 px canvas (main game view)
  - `#hud`: Real-time stats display (score, weapon, lives, bombs)
  - `#overlay`: Start/pause/game-over screens
  - `#touchControls`: Mobile input zones

### 2. Game Engine (JavaScript)
The core game loop runs via `requestAnimationFrame`, calling:

1. **`update(dt)`** — Physics & logic (~150 lines):
   - Player movement (keyboard, mouse glide, joystick)
   - Collision detection (enemies, bullets, power-ups, boss)
   - Enemy spawning and behavior (weaving, shooting patterns)
   - Boss state machine (entering, attacking, telegraphing patterns)
   - Score chain/multiplier tracking
   - Performance adaptation (quality downgrading on low FPS)

2. **`draw()`** — Canvas rendering (~120 lines):
   - Background gradient with parallax clouds
   - Player and enemy planes (procedurally drawn)
   - Bullets, particles, power-ups
   - Boss ship with core glow effects
   - Mouse target reticle (when using mouse control)

3. **`loop(now)`** — Main game loop:
   - Throttles `dt` to max 50ms (prevents tunneling)
   - Manages pause/resume state
   - Ties update and draw together

### 3. Game State

**Player Object**:
```javascript
{ x, y, w: 40, h: 40, speed: 260, weapon: 'vulcan'|'laser', level: 1-3, invuln: 0, cooldown: 0 }
```

**Enemies Array** — Three types (ENEMY_TYPES):
- `grunt`: 1 HP, 34×34, basic red plane
- `interceptor`: 1 HP, 26×26, fast with weaving behavior
- `tank`: 4 HP, 46×46, slow but durable

**Boss Object** — Six designs (BOSSES):
- Each has unique colors, health multiplier, and 3-pattern attack sequence
- VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Pattern pools in `BOSS_PATTERNS[6]` (e.g., radial bursts, aimed spreads, homing missiles)

**Difficulty System**:
```javascript
DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, lives: 3, bombs: 2, bossHpMul: 1.3 }
}
```

### 4. Audio (Web Audio Synthesis)

All sounds are procedurally generated using oscillators and noise buffers:
- **`beep(freq, dur, type, vol, opts)`** — Tonal sounds with sweep/filter/pan
- **`noiseBurst(dur, vol, opts)`** — Percussive noise envelopes
- **SFX functions**: `shoot`, `laser`, `explodeGrunt`, `explodeTank`, `bomb`, `bossHit`, `gameover`, etc.
- Master gain with compressor for consistent output levels

## Key Game Constants

- **Canvas**: 480×720 px (logical), scales responsively on-screen
- **KILLS_PER_STAGE**: 18 enemies before boss appears
- **THEMES**: 6 sky gradients with color palettes and theme names
- **Performance tiers**: quality ∈ {0: full, 1: medium (60% particles), 2: low (35% particles)}
- **Spawn difficulty ramp**: Enemy spawn rate increases every 5 seconds

## Common Development Tasks

### Understanding a Bug
1. Check if it's **state-related** → inspect `resetGame()` or difficulty-specific constants
2. Check if it's **collision-related** → review `rectsOverlap()` and collision loops (lines 1232-1287)
3. Check if it's **audio** → ensure `ensureAudio()` is called and `muted` flag is checked
4. Check if it's **rendering** → look at `draw()` order (backgrounds first, then game objects)
5. Check if it's **mobile-specific** → test with `isTouchDevice` flag and touch event handlers

### Adding a New Enemy Type
1. Add to `ENEMY_TYPES` with `{ w, h, hp, speedMul, color, dark, score, minStage }`
2. Add spawning logic to `spawnEnemy()` (random selection weighted by difficulty)
3. Add explode SFX to `EXPLODE_SFX` mapping
4. Create a SFX function if needed in the `sfx` object

### Adding a Boss Attack Pattern
1. Create a fire function (e.g., `fireRadial`, `fireAimedSpread`) that pushes to `enemyBullets`
2. Add the pattern function to `BOSS_PATTERNS[boss_index]` array
3. Boss executes patterns on telegraph countdown (22 frames of warning)
4. Use `b.patternIndex` to cycle through the 3 patterns; design determines pool

### Performance Optimization
- **Particle count** scales with `quality` (see `spawnExplosion`)
- **Shadow blur** disabled when `quality > 0` (lines 1002, 1579-1583)
- Check `trackPerf(dt)` — automatically downgrades if avg frame time > 1/40 = 25ms

## Input Handling

### Keyboard
- **Arrow keys / WASD**: Move
- **Space / Click**: Fire
- **X / Shift**: Bomb
- **P / Esc**: Pause

### Mouse
- **Move**: Player glides toward pointer (only when no keyboard/joystick input)
- **Click**: Fire
- Reticle drawn when active (lines 1561-1577)

### Touch
- **Left 55% of screen**: Joystick (drag to move)
- **Right 45% bottom**: Fire and Bomb buttons
- Touch priority over mouse (set `mouseActive = false` on `mouseleave`)

## Storage Keys (localStorage)

- `skyfighter.bestScore` — High score
- `skyfighter.muted` — Audio mute state
- `skyfighter.difficulty` — Last selected difficulty
- `skyfighter.reducedMotion` — Reduced motion override (defaults to OS preference)

## Accessibility

- **Screen reader support**: `#srLive` (aria-live region) for announcements
- **Reduced motion**: Disables all animations if `prefers-reduced-motion` is set or toggled
- **Contrast**: Light text on dark backgrounds; emoji icons have fallback descriptions
- **Keyboard-only playable**: No mouse required (arrow keys + space + X)

## Testing & Validation

Since this is a single-file game with no build process:
1. **Open in browser**: Simply open `index.html` directly
2. **Test all difficulties**: Verify spawn rates, enemy speeds, boss HP scale
3. **Test input methods**: Keyboard, mouse, touch (use device emulation)
4. **Test accessibility**: Mute audio, enable reduced motion, use screen reader
5. **Test mobile**: Check responsive scaling, touch zones, safe-area-inset handling
6. **Performance**: Use DevTools Perf tab to check frame times under `trackPerf` logic

## Code Organization by Line Ranges

| Range | Section |
|-------|---------|
| 1-359 | HTML structure + CSS (styles, animations, responsive layout) |
| 422-478 | DOM element references |
| 485-527 | Constants (DIFFICULTIES, KILLS_PER_STAGE, THEMES, ENEMY_TYPES, BOSSES) |
| 542-676 | Audio system (WebAudio context, beep/noiseBurst, SFX definitions) |
| 752-862 | Input handling (keyboard, mouse, touch, joystick) |
| 863-1123 | Game state & core mechanics (reset, powerups, bomb, scoring) |
| 1124-1302 | Update loop (physics, collision, spawning) |
| 1372-1439 | Boss attack patterns & boss update logic |
| 1441-1505 | Game over / pause / win conditions |
| 1507-1638 | Rendering functions (clouds, planes, boss, particles) |
| 1640-1666 | Main loop & startup |

## When Modifying the Code

- **Timing-sensitive**: Frame-based logic (lines 974, 1002, 1212) uses `frame` counter; `dt`-based logic uses physics (`speed * dt`)
- **Collision**: All use AABB (axis-aligned bounding box) via `rectsOverlap()`
- **Rendering order matters**: Background → players/enemies → bullets → particles → UI overlays
- **Canvas coordinates**: (0,0) is top-left; Y increases downward
- **Audio**: Wrap calls in `ensureAudio()` to handle initial user interaction requirement
