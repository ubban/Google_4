# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game built as a single-file HTML5 canvas application. All code (HTML structure, CSS styling, and JavaScript game logic) is contained in `index.html` (~1670 lines).

## Architecture & Key Systems

### Game Loop & State Management

The main game loop runs via `requestAnimationFrame`. The `update(dt)` function handles:
- Player movement and firing
- Enemy spawning and behavior
- Bullet/projectile logic
- Collision detection
- Combo tracking and score multiplier
- Pause state

Game state includes:
- `gameRunning`, `gamePaused`, `gameOver` flags
- Current `score`, `combo`, `comboDuration`, `lives`, `bombs`
- `waves` array (stages/difficulty progression)
- `enemies`, `bullets`, `explosions`, `pickups` arrays
- `currentTheme`, `currentBoss`, `stageClear`

### Rendering Pipeline

Canvas rendering happens in a specific order:
1. Background sky (theme-dependent gradient)
2. Clouds and sun (visual effects)
3. Projectiles (bullets, bombs)
4. Enemies and boss
5. Explosions and visual effects
6. HUD overlay (score, lives, multiplier, weapon info)

Key rendering functions:
- `drawPlane(x, y, w, h, color, darkColor, flame)` — Draws player and enemy ships
- `drawBossShip(b)` — Draws boss with hull, wings, and core
- Particle effects via `spawnExplosion()` for visual impact

### Input Handling

**Keyboard**: Arrow keys move; Space fires; E for bombs; P pauses
**Touch/Mobile**:
- Virtual joystick in bottom-left (move zone)
- Fire button in bottom-right
- Bomb button in top-right corner
- Handles both `pointerdown`/`pointermove`/`pointerup` and fallback `touchstart`/`touchmove`/`touchend`

### Audio System

All sound is synthesized via Web Audio API (no external audio files):
- `beep(freq, dur, type, vol, opts)` — Tonal sounds with frequency sweep, filtering, panning
- `noiseBurst(dur, vol, opts)` — Noise-based sounds (explosions, impacts)
- `masterGain` controls overall mute state; audio context is lazily initialized on first user interaction
- Stereo panning based on screen position for spatial audio

### Difficulty & Progression

Three difficulty presets control:
- Enemy spawn rate and minimum spawn interval
- Enemy speed and shoot chance
- Player starting lives and bomb count
- Boss HP multiplier

Waves/stages are tracked by `KILLS_PER_STAGE = 18` enemy kills to advance. Boss battles occur after stage completion.

### Weapons & Power-ups

**Weapons**: Gun (base), Spread (multi-bullet), Wave (arcing), Rocket (slow, high damage)
- Each weapon has 3 levels (I, II, III)
- Power-ups spawn randomly from defeated enemies
- `applyPowerUp(type)` handles weapon upgrade, health, and bomb pickups

**Bomb Mechanic**: 
- Limited count per life (2 or 3 depending on difficulty)
- Clears all bullets and deals massive damage to all enemies
- `useBomb()` handles invulnerability frames and visual effect

### Enemy Types & Boss Patterns

**Enemy Types** (defined in `ENEMY_TYPES`):
- `grunt` — Basic red enemy
- `interceptor` — Fast, orange, erratic movement
- `tank` — Durable purple enemy (minStage: 2)

**Bosses** (defined in `BOSSES` array):
- 6 boss types with unique color schemes and HP multipliers
- Boss appears after 18 enemy kills per stage
- Randomly selected from pool
- Boss fires 3 projectiles per interval when active

### Combo System & Scoring

Score multiplier increases with consecutive hits (no enemy damage taken). Hitting an enemy increments combo; taking damage breaks it. Visual feedback:
- Combo counter in HUD
- `showComboPop(text)` animates combo messages on screen

## Running & Testing

Since there's no build process, development is direct:

1. **Run the game**: Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge)
2. **Test locally**: Use `python -m http.server 8000` (or similar) to serve the file if needed for full functionality
3. **Check on mobile**: Use browser dev tools responsive mode or physical device; touch controls are responsive to viewport size
4. **Performance**: Adaptive quality system (`quality` variable) reduces rendering detail if frame time exceeds ~25ms

## Common Development Tasks

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` object (define `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optionally `weave`)
2. Modify `spawnEnemy()` to include the new type in the pool based on stage
3. If special behavior is needed (weaving, firing pattern), add logic in `update(dt)` enemy loop

### Adding a New Weapon or Power-up

1. Extend weapon definitions in the `update()` function where weapons are tracked
2. Add a firing pattern function (e.g., `fireRadial()`, `fireColumns()`) or extend existing ones
3. Modify `applyPowerUp()` if it's a power-up type

### Adding a New Boss

1. Add to `BOSSES` array with name, colors (hullTop, hullBottom, wing, core), and `hpMul`
2. Boss is automatically selected from pool; extend `drawBossShip()` if custom shape is needed
3. Boss firing pattern uses `fireRadial()` or modify `update()` for custom attacks

### Tweaking Difficulty or Balance

- Adjust `DIFFICULTIES` constants for spawn rates, enemy speeds, enemy shoot chance, starting lives/bombs, boss HP
- Adjust `KILLS_PER_STAGE` to change when bosses appear
- Modify enemy `score` in `ENEMY_TYPES` or base score in `addScore()`

### Visual Theme Changes

- Add/edit entries in `THEMES` array (sky gradient colors, cloud color, sun color, display name)
- Theme is randomly selected at game start; modify `const themeIdx = Math.floor(Math.random() * THEMES.length);` if needed
- Canvas drawing uses theme colors via `theme.sky`, `theme.cloud`, `theme.sun`

### Tweaking Controls

- Keyboard: Modify `keydown`/`keyup` event listeners
- Touch: Joystick sensitivity and fire button layout are in `joystickStart()`, `joystickMove()`, and touch event handlers

## Performance Considerations

- **Canvas size**: Set by `resize()` based on window dimensions; clamped aspect ratio preserves gameplay feel
- **Particle count**: Explosions spawn 15–40 particles; control via `spawnExplosion()` count parameter
- **Enemy/bullet caps**: No hard limits, but performance degrades with many projectiles; consider culling off-screen objects if adding heavy effects
- **Adaptive quality**: If FPS drops, `quality` auto-increases, reducing particle count and effect complexity
- **Audio**: Lazily initialized and muted by default; no memory leak risk

## Storage & Persistence

Uses `localStorage` for:
- `skyfighter.bestScore` — Best score ever
- `skyfighter.muted` — Mute state
- `skyfighter.difficulty` — Last selected difficulty
- `skyfighter.reducedMotion` — User preference for motion reduction

Keys are defined at top of script for easy reference and updates.

## Mobile & Accessibility

- Responsive canvas sizing with `max-width: 100%` and viewport meta tag
- Touch controls scale to viewport via `clamp()` CSS functions
- Reduced motion support via `prefers-reduced-motion` media query and toggle
- Screen reader support via `#srLive` ARIA live region for announcements
- Safe area insets (`env(safe-area-inset-*)`) for notch compatibility

## Key Files & Line Ranges

- **HTML structure & CSS**: Lines 1–421
- **JavaScript setup & state**: Lines 422–530
- **Audio functions**: Lines 542–670
- **UI & visual effects**: Lines 680–740
- **Game logic (update loop)**: Lines 1148–1303
- **Rendering**: Lines 1350–1669

## Notes for Future Development

- The single-file structure keeps the game portable and easy to share; keep it that way unless refactoring into modules for maintainability
- All visuals and sounds are generated (no external assets), making the file self-contained
- Canvas coordinate system: (0,0) is top-left; y increases downward
- Combo multiplier and score progression can feel tight; test balance changes thoroughly before committing
- Boss AI is simple (radial/aimed spread patterns); more complex patterns can add challenge
