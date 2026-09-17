# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The entire game (~1670 lines) is contained in `index.html` with inline CSS and JavaScript (no external build system, no dependencies).

The game features multiple weapon types, bosses, difficulty modes, and is fully playable on desktop (keyboard/mouse) and mobile (touch controls). Audio is generated via Web Audio API synthesis (no external audio assets).

## How to Run

1. **Open in browser**: Simply open `index.html` directly in any modern browser (desktop or mobile)
2. **Local testing**: Use a simple HTTP server if needed:
   ```bash
   python -m http.server 8000
   # then visit http://localhost:8000/index.html
   ```

No build step, no dependencies, no installation required. The game is fully self-contained.

## Code Architecture

The single `index.html` file is organized into logical sections (no separate JS files). All code runs within a single IIFE (Immediately Invoked Function Expression) starting at line 423.

### Core Game Systems

**Game Loop & State Management**
- Main game loop via `requestAnimationFrame` with `update(dt)` and `draw()` functions
- Game states: menu overlay, active gameplay, paused, game over
- Frame tracking for adaptive performance (see "Performance Adaptation" below)

**Player System** (around line 850-900)
- Player object: position, movement speed, weapon type/level, health, invulnerability timer, cooldown
- Two weapon modes: `vulcan` (spread shots) and `laser` (piercing shots), each with 3 levels
- Input handling: keyboard (arrows/WASD), mouse aiming, touch joystick controls
- Movement boundaries to keep player on canvas

**Enemy System** (around line 1022-1227)
- Three enemy types defined in `ENEMY_TYPES`: grunt, interceptor, tank
- Spawning controlled by `spawnTimer` based on difficulty and stage progression
- Enemy types have: hp, speed, sprite dimensions, shooting patterns, score value, visual colors
- Two firing patterns: "aimed" (tracks player) and "spread" (radial shots)
- Interceptors "weave" side-to-side; tanks are slow but durable
- Enemies tracked in `enemies` array; cleanup filters screen-off entities

**Boss System** (around line 1043-1120, and `updateBoss()`)
- 6 boss types in `BOSSES` array (rotate based on stage: `(stage - 1) % BOSSES.length`)
- Each boss has unique hull colors, core color, and `hpMul` difficulty multiplier
- Boss enters with animation (`entering` flag), then attacks via telegraph patterns
- Boss deals damage on collision; can be hit by bullets (3 damage per bullet vs. 1 for enemies)
- Stage progression: defeat 18 enemies per stage, then boss fight

**Scoring & Progression**
- Combo multiplier: increases by 1 for every 5 kills in rapid succession (2.2s window)
- Breaking chain resets multiplier to 1×
- Each enemy type has base score; final score = base × multiplier
- Kills tracked to progress stages; boss victory triggers next stage

**Weapon & Power-up System** (around line 1058-1066)
- Power-ups spawn randomly on enemy defeat: life (6%), bomb (8%), weapon (20%)
- Weapon power-up cycles: `vulcan` ↔ `laser` and increases level 1→2→3
- At level 3, weapons gain extra shot lanes/spread
- Bomb clear: destroys all on-screen enemies, deals 24 damage to boss, gives 5 points per enemy

**Particle & Visual Effects**
- Particles (explosions) spawned at hit locations; count scales with quality setting
- Hit flash, bomb flash, vignette pulse (on critical health)
- Combo pop-up, stage banner announcements
- Boss health bar with color gradient

### Input Handling

- **Keyboard**: Arrow keys / WASD for movement, Space for fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Move to cursor position, click to fire
- **Touch**: Left 55% of screen is joystick zone (draggable), right buttons for FIRE/BOMB
- **Mobile Joystick**: Rendered as visual circle overlay when touch detected

### Audio System (around line 542-676)

Web Audio API synthesis with no external files:
- **Beep engine**: Oscillator with frequency sweep, optional filter, stereo pan, envelope
- **Noise burst**: Brown-noise-like effect with optional filter
- **Positional audio**: Panning varies based on X position (gunshots/explosions sound left/right)
- Lazy initialization: audio context created on first user interaction (required by browser)
- Mute toggle in settings; persisted to localStorage

Sound effects include shoot, laser, explosions (type-specific: grunt/interceptor/tank), bomb, power-up, boss hit, telegraph, stage clear, game over.

### Performance Adaptation (around line 474-482)

Three quality tiers (0=full, 1=medium, 2=low):
- Tracks last 45 frame times
- If average > 1/40s (25fps): downgrade quality
- If average < 1/56s (~18ms): upgrade quality
- Lower quality reduces particle counts (60%, 35% of full)
- Particle shadow blur also scales with quality

### Data Persistence

All state stored in `localStorage`:
- `skyfighter.bestScore`: Best score achieved
- `skyfighter.muted`: Audio mute state (0 or 1)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Reduced motion toggle (0 or 1)

### Difficulty Scaling (around line 485-489)

Three preset difficulty modes control:
- Spawn rate base/min (enemy spawn frequency)
- Enemy speed multiplier
- Enemy shoot chance (probability per enemy shoots)
- Player lives (3-4)
- Bombs available (2-3)
- Boss HP multiplier (0.8x–1.3x)

## Game Balance Constants

- `KILLS_PER_STAGE`: 18 enemies must be defeated to summon boss
- `THEMES`: 6 sky gradient + cloud color themes (cycle per boss)
- Combo window: 2.2 seconds between kills to maintain chain
- Multiplier cap: 1× to 10× (one extra multiplier level per 5 kills)

## Collision Detection

Simple AABB (axis-aligned bounding box) overlap via `rectsOverlap()`:
- Player vs. enemies, bullets, bosses, power-ups
- Bullets vs. enemies, boss
- Enemy bullets vs. player
- All tracked objects have `.x`, `.y`, `.w`, `.h` properties

## Rendering

Canvas 2D context drawing in `draw()` function:
- Background: Gradient sky, scrolling parallax clouds
- Player: Sprite-like polygon art (no image files)
- Enemies: Colored shapes (rects, circles), telegraph flashes before boss attacks
- Bullets: Small rectangles (player bullets white/cyan, enemy bullets red)
- HUD: Rendered as HTML div overlay (separate from canvas)
- Touch controls: Canvas overlay with transparent zones and visual feedback

## Accessibility

- Screen reader live region (`#srLive`) for announcements (mute toggle, reduce motion, etc.)
- Respects `prefers-reduced-motion` media query (disables shake/flash by default on preference)
- Reduced motion toggle in UI to override preference
- Clear color contrast, readable font sizes that scale with viewport

## Making Changes

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` object (around line 502-506):
   ```javascript
   newType: { w: 40, h: 40, hp: 2, speedMul: 1.3, color: '#xyz', dark: '#abc', score: 25, minStage: 3, weave: false }
   ```
2. Add rendering code in the enemy draw loop (around line 1317)
3. If custom shoot pattern, update enemy firing logic (around line 1216-1225)
4. Add optional explosion sound to `EXPLODE_SFX` map (line 678)

### Adding a New Boss

1. Add to `BOSSES` array (around line 508-515) with color scheme and name
2. Implement attack pattern in `updateBoss()` function (around line 900+)
3. Adjust `hpMul` to tune difficulty relative to other bosses
4. Rendering happens in `drawBoss()` (line 995+) using stored `design` colors

### Tuning Difficulty

Modify `DIFFICULTIES` object (line 485-489) to adjust spawn rates, enemy behavior, or boss HP scaling. Playtest across all three difficulty modes.

### Adding Sound Effects

1. Write new oscillator + gain sequence in `sfx` object (line 624+)
2. Use `beep(freq, duration, waveform, volume, opts)` or `noiseBurst(duration, volume, opts)`
3. Call from relevant game event (e.g., `sfx.newEffect()`)
4. Mute state checked automatically inside beep/noiseBurst

### Adjusting Visual Effects

- Particle explosion colors and counts: modify `spawnExplosion()` calls
- Screen shake intensity: adjust `screenShake(intensity, duration)` parameters
- Vignette pulse: CSS animation in `<style>` section (search `#vignette.on`)
- Flash effects: controlled by `.hit` and `.bomb` classes on `#flash` element

## Known Patterns & Constraints

- **No external dependencies**: Pure vanilla JS, HTML, CSS. Avoid adding npm packages.
- **Mobile-first rendering**: Canvas size adapts to viewport; touch controls overlay on mobile.
- **Audio initialization**: Web Audio context is lazy-created on first user interaction (browser policy).
- **Delta-time updates**: Game uses `dt` (seconds since last frame) for smooth movement regardless of frame rate.
- **Object cleanup**: Bullets, enemies, particles use filter-based culling (`array.filter(...)`) rather than explicit removal.
- **localStorage limits**: Very generous on modern browsers, but keeps usage minimal (4 small keys).
- **Canvas coordinate system**: (0,0) is top-left; Y increases downward.

## Testing the Game

1. **Manual gameplay**: Open in browser, play through all difficulty modes, verify audio works, test mobile controls
2. **Edge cases**: 
   - Boss defeat with exact HP (== 0)
   - Combo chain break at boundary (2.2s timer)
   - Power-up spawn rate on different difficulties
   - Reduced motion mode disables shake/flash correctly
3. **Performance**: Load on older devices; check adaptive quality downgrade on frame lag
4. **Browser compatibility**: Test in Chrome, Firefox, Safari (especially mobile Safari for audio context)

---

**Last updated**: 2026-09-17
