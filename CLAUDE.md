# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game (HTML, CSS, and JavaScript) is in `index.html` (~59KB). There are no external dependencies, build tools, or test frameworks—just open the file in a browser to play.

## Running the Game

**Desktop:** Open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge).

**Mobile:** Same—the game is fully responsive with touch controls on phones/tablets.

**Development:** Use Claude Code's `/run` command or launch a local web server:
```bash
python3 -m http.server 8000
# Then open http://localhost:8000/index.html
```

## Code Architecture

All game logic lives in a single `<script>` tag in `index.html`, structured by feature area rather than files. The code uses an IIFE (Immediately Invoked Function Expression) for scope isolation and is organized into these sections:

### 1. **Rendering & UI** (styles + DOM setup)
- `<style>` block: All CSS (layout, animations, responsive design, dark theme support, accessibility)
- DOM elements cached as variables for fast reference (canvas, HUD, overlay, buttons)
- Responsive canvas sizing: base 480×720px, scaled to fit window (max 1.6×)
- HUD (heads-up display): score, multiplier, weapon level, hearts, bombs

### 2. **Audio System** (WebAudio API)
- **No external audio files**—all sounds synthesized in real-time using oscillators, noise, and filters
- `beep(freq, duration, type, volume, opts)`: Pure-tone synthesis with frequency sweeps and filters
- `noiseBurst(duration, volume, opts)`: White noise with envelope for impact/explosion sounds
- `sfx` object: Pre-configured sound effects (shoot, explode, powerup, boss hit, etc.)
- Adaptive to mute state and respects audio context suspend/resume

### 3. **Performance** (adaptive quality)
- `trackPerf()`: Measures average frame times and auto-adjusts quality level (full → medium → low)
- Reduces particle/effect density on slower devices
- No frame-skipping or async operations—pure synchronous game loop

### 4. **Game State** (player, enemies, bullets)
- **Player object**: position, velocity, health, weapon state, cooldown timers
- **Enemy types**: grunt, interceptor, tank (each with HP, speed, color, scoring)
- **Boss system**: 6 boss types (VANGUARD, BEHEMOTH, etc.) with unique visuals and HP multipliers
- **Multiplier chain**: Score multiplier increases on consecutive kills; resets if bullet fired without hitting (encourages burst fire)
- **Difficulty settings**: Easy/Normal/Hard with persistent localStorage for player preference

### 5. **Input Handling** (keyboard, mouse, touch)
- **Keyboard**: Arrow keys/WASD to move, Space to fire, X/Shift to bomb, P/Esc to pause
- **Mouse**: Move pointer to steer, click to fire (dual input: keyboard takes priority)
- **Touch**: Left-side drag zone for joystick, right-side buttons for fire/bomb
- `keys` object: Tracks held keys; input polling in main loop (not event-driven)

### 6. **Game Loop** (main update → render cycle)
```
requestAnimationFrame(gameLoop)
  ├─ Update phase (logic, collisions, spawning)
  │  ├─ Player movement (keyboard/mouse/joystick priority)
  │  ├─ Weapon fire & cooldown
  │  ├─ Enemy spawning & AI (weaving, shooting, movement)
  │  ├─ Collision detection (bullets ↔ enemies, bullets ↔ player, bombs)
  │  ├─ Particle physics & lifetime
  │  ├─ Power-up pickup detection
  │  └─ Stage progression & boss spawn
  └─ Render phase (canvas drawing)
     ├─ Background (gradient sky + parallax clouds)
     ├─ Enemies & boss
     ├─ Bullets & particles
     ├─ Player with rotation/tilt
     └─ Screen effects (shake, flash, vignette on low health)
```

### 7. **Combat Systems**
- **Weapon types**: Vulcan (wide spray) and Laser (piercing, single-target)
- **Weapon levels**: Increase rate of fire and/or damage with weapon pickups
- **Smart bombs**: Area-of-effect explosion triggered by button; clears bullets and damages enemies
- **Chain multiplier**: Increases with each enemy killed; resets on weapon fire (not on bomb)
- **Enemy AI**: Gravity-based downward movement; shooters fire randomly; weaving enemies dodge sideways

### 8. **Visual Effects**
- **Screen shake**: Triggered by player hit or bomb; respects reduced-motion preference
- **Flash overlay**: Brief white flash on hit; more intense for bombs
- **Vignette (red pulse)**: Appears when health is critical (1 heart)
- **Particle effects**: Small bursts on enemy death, weapon pickup, bomb explosion
- **Animations**: Combo pop-up text, stage banner entrance, card slide-in, heart beat, etc.

### 9. **Accessibility**
- **Keyboard-only play**: Fully navigable without mouse
- **Touch support**: On-screen joystick and buttons for mobile
- **Screen reader**: Live region announcements for game events (mute toggle, motion setting)
- **Reduced motion**: Disables animations, screen shake, flash when enabled (respects `prefers-reduced-motion` media query)
- **Color contrast**: Dark theme with light text; game is playable without color (shapes/labels matter)

### 10. **Persistent State**
- Uses `localStorage` for: best score, mute setting, difficulty, reduced-motion preference
- Keys are scoped (`skyfighter.*`) to avoid collisions
- No sync or server interaction

## Key Development Patterns

**Game objects** (player, enemies, bullets) are plain objects with position, velocity, size, and type-specific properties. No classes or prototypes—simpler update logic.

**Collision detection** uses AABB (axis-aligned bounding box): `x1 < x2 + w2 && x1 + w1 > x2 && y1 < y2 + h2 && y1 + h1 > y2`. Called per-bullet per-enemy per-frame; O(n²) but acceptable for arcade game scale.

**Canvas transforms** use `translate()` + `rotate()` for player tilt and screen shake. No matrix math library needed.

**Difficulty tuning** lives in the `DIFFICULTIES` object: enemy spawn rate, speed, shoot chance, starting health, bomb count, boss HP multiplier. Adjust these to rebalance gameplay.

**Stage progression**: Tracks `stageKills` (kills on current stage). When `stageKills >= KILLS_PER_STAGE` (18), spawns boss. Boss defeat advances stage and resets counters. Six bosses cycle; after TITAN, game loops back to VANGUARD on stage 7+.

## Making Changes

- **Bug fix**: Locate the relevant section (audio, input, rendering, etc.) and edit inline.
- **New enemy type**: Add to `ENEMY_TYPES` object, set spawn logic in enemy creation, add SFX variant in `sfx` object.
- **New weapon**: Add to weapon type checks; adjust fire pattern in `update()` firing code.
- **Balance tuning**: Tweak `DIFFICULTIES`, `KILLS_PER_STAGE`, particle lifetime, damage thresholds, screen shake magnitude.
- **Visual tweaks**: Colors are in `THEMES` array (sky gradients, cloud/sun colors); adjust `BOSSES` hull/wing/core colors.
- **Audio tweaking**: Edit `sfx` object functions; change frequencies, durations, filter cutoffs.

When editing, watch for:
- **Canvas dimensions**: Hardcoded as `BASE_W = 480, BASE_H = 720`. Scaling is handled by CSS.
- **Timing**: Game runs at 60 FPS target (delta-time aware in update). Frame-based timers use frame counter; adjust spawn rates in frames.
- **Collision radius**: Hitboxes are rectangles; particles and enemies shrink by a few pixels for visual polish.
- **Weapon cooldown**: Stored per-player in `cooldown` and `rapid` fields (frames).
- **Chain timer**: Decrements each frame; game has no true sprite animation—all motion is procedural.

## Testing the Game

There are no automated tests. Verify changes by:
1. Launch the game in a browser.
2. Test the feature (e.g., new enemy spawn, weapon change, boss behavior).
3. Play through at least one stage on each difficulty.
4. On mobile: Verify touch input and responsive layout.
5. Test accessibility: Tab through menu, enable reduced motion, check mute toggle.

## Session Context Tips

When returning to this project:
- The game is entirely in `index.html`; all edits go there.
- Frame rate is critical—if adding logic, measure its impact on FPS (check `trackPerf()` or browser DevTools).
- Audio context must be resumed after user gesture (`ensureAudio()`); it won't initialize until first input.
- `localStorage` uses `skyfighter.*` keys; clearing these resets settings.
- Mobile testing is important: the game adapts to touch, but desktop keyboard-only must also work.
