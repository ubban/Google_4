# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game implemented as a single-file HTML/CSS/JavaScript application. The game runs entirely in a web browser with no external dependencies or build tools required.

## Running the Game

Simply open `index.html` in a web browser. The game works on:
- Desktop (mouse + keyboard controls)
- Mobile/tablet (touch controls with on-screen buttons and virtual joystick)

No build step, linting, or testing infrastructure is required—the game runs directly from the HTML file.

## Code Architecture

The entire game logic lives in a single `index.html` file (~1670 lines) with embedded `<style>` and `<script>` tags. The code is organized into functional sections within the IIFE (Immediately Invoked Function Expression):

### 1. **DOM Elements & Setup** (lines 426–461)
Cache references to all UI elements (canvas, HUD, buttons, overlay, touch controls) used throughout the game loop.

### 2. **Constants & Configuration** (lines 463–515)
- `DIFFICULTIES`: Three difficulty presets (easy, normal, hard) controlling spawn rates, enemy speed, HP scaling, and starting lives/bombs.
- `ENEMY_TYPES`: Defines grunt, interceptor, and tank enemies with their visual, behavioral, and scoring properties.
- `BOSSES`: Six unique boss designs with color schemes and HP multipliers.
- `BOSS_PATTERNS`: Attack patterns indexed by boss type (6 bosses × 3 patterns each = varied combat).
- `THEMES`: Six visual themes (Daylight, Sunset, Night, Storm, Arctic, Volcanic) applied per stage.

### 3. **Audio Synthesis** (lines 542–676)
All sound effects are generated procedurally using the WebAudio API—no asset files:
- `beep()`: Oscillator-based tones with frequency sweeps and filtering.
- `noiseBurst()`: Procedural noise for explosive/impact sounds.
- `sfx` object: Pre-composed sound routines (shoot, laser, explode variants, hit, bomb, powerup, boss patterns, UI click, gameover).

### 4. **Input Handling** (lines 767–862)
Three input modes run in parallel; keyboard/gamepad always takes priority over mouse:
- **Keyboard/Gamepad**: Arrow keys + WASD for movement, Space/X/Shift for fire/bomb, P/Esc for pause.
- **Mouse**: Player glides toward pointer; click to fire.
- **Touch**: Virtual joystick on left half of screen (movement), fire/bomb buttons on right half.

### 5. **Game State** (lines 863–911)
Global state variables track player, bullets, enemies, boss, score, lives, bombs, stage, chain multiplier, and frame counter. `resetGame()` initializes a new game.

### 6. **Collision & Entity Management** (lines 1069–1302)
- `rectsOverlap()`: AABB collision detection.
- Separate loops check enemy–bullet, boss–bullet, player–enemy, player–bullet, player–powerup collisions.
- Entities (bullets, enemies, particles) are filtered each frame to remove off-screen objects.

### 7. **Weapon System** (lines 871–1146)
- **Vulcan** (default): Rapid-fire straight shots with spreading at levels 2–3.
- **Laser**: Faster, narrower shots that pierce multiple enemies (pierce cap increases at level 3).
- Weapon switches and levels up from power-ups; UI updates in real-time.

### 8. **Boss System** (lines 1043–1434)
- `spawnBoss()`: Creates boss with design, HP, and telegraph timer.
- `updateBoss()`: Moves boss horizontally, manages telegraph (attack telegraph) → attack pattern cycle.
- Six attack pattern generators (fireRadial, fireColumns, fireSpiralBurst, etc.) produce radial, aimed, or homing bullet patterns.
- Distinct patterns per boss type ensure variety across 6 stages + loop.

### 9. **Rendering** (lines 1507–1638)
- **Background**: Animated sky gradient, sun glow, scrolling clouds (per-theme colors).
- **Entities**: Player plane, enemies, bullets (with glow in high quality), boss, power-ups, particles.
- **Screen shake**: Applied via `ctx.translate()` in draw loop; reduced or disabled in low-quality or reduced-motion modes.
- **Quality adaptation** (lines 473–482): Tracks frame time; reduces particle counts and glow effects at lower quality levels (0→full, 1→medium, 2→low).

### 10. **Game Loop & Timing** (lines 1640–1665)
- `requestAnimationFrame(loop)` at browser refresh rate (~60 FPS on most devices).
- Delta-time (`dt`) capped at 50ms to prevent spiral-out on frame drops.
- `update(dt)` moves all entities and checks collisions.
- `draw()` renders the scene (may skip if paused or not running).

## Key Gameplay Mechanics

- **Score Multiplier**: Kill chain increases multiplier (max 10x). Breaks if player takes damage. `addScore()` multiplies base points.
- **Weapon Progression**: Level 1–3 for each weapon type; cycling between Vulcan and Laser resets level to 1.
- **Power-ups**: Life, bomb, weapon—spawn from defeated enemies at configurable rates.
- **Bomb System**: Limited count per difficulty; smart bomb clears all enemy bullets and damages all enemies or boss.
- **Boss Stages**: After 18 kills in a stage, a boss spawns. Defeat it to advance stage counter and spawn new enemy types.
- **Difficulty Ramp**: Spawn rate increases with frame count; stage 2+ introduces tank enemies; stage 3+ adds interceptors.

## Performance & Optimization

- **Particle reduction**: High quality shows full explosion particles; medium (quality=1) shows 60%; low (quality=2) shows 35%.
- **Shadow/glow**: Disabled at low quality for faster rendering on slow devices.
- **Collision**: Single-pass AABB checks; no spatial hashing needed given typical entity counts (~50 bullets, ~20 enemies per frame).
- **Touch throttling**: Virtual joystick updates only on touch move events, not every frame.

## Storage & Settings

All persistent settings use `localStorage`:
- `skyfighter.bestScore`: High score.
- `skyfighter.muted`: Audio mute state.
- `skyfighter.difficulty`: Last selected difficulty.
- `skyfighter.reducedMotion`: Reduced-motion toggle (respects system preference on first load).

## Accessibility

- **Reduced Motion**: Toggle button (`motionBtn`) disables screen shake, flash, animations on reduced-motion mode.
- **Screen Reader**: Hidden `#srLive` element announces key game events (boss defeat, stage clear, game over, lives remaining).
- **High Contrast**: All text uses dark backgrounds with light text or text shadows; buttons have clear active states.
- **Touch Support**: Full virtual joystick + large buttons for mobile (sizes scale with viewport).

## Debugging & Extending

### Common Tweaks
- **Difficulty balance**: Adjust `spawnBase`, `enemySpeed`, `bossHpMul` in `DIFFICULTIES`.
- **Enemy variety**: Add new enemy types to `ENEMY_TYPES` with `minStage` gate (e.g., a new enemy appears at stage 5).
- **Boss attacks**: Edit or add patterns in `BOSS_PATTERNS`; use helper functions like `fireRadial()`, `fireAimedSpread()` for consistency.
- **Visuals**: Themes are in `THEMES`; add a new theme object to rotate through more stage backgrounds.
- **Audio**: Tweak frequencies, durations, or filters in the `sfx` object; all synth parameters are tunable.

### Testing Changes
1. Save edits to `index.html`.
2. Refresh the browser (Ctrl+Shift+R for hard refresh to bypass cache).
3. Verify on mobile (browser DevTools → device emulation, or actual device).

### Recommended Focal Points for Changes
- **Weapon balance**: Edit `fireWeapon()` (line 1124) and weapon spread/speed constants.
- **Enemy spawn logic**: `spawnEnemy()` (line 1022) controls type distribution and difficulty gating.
- **Boss patterns**: Each boss gets 3 patterns in `BOSS_PATTERNS`; rotate or swap pattern functions to change behavior.
- **Stage progression**: Adjust `KILLS_PER_STAGE` (line 491) to change how many kills trigger a boss.
- **UI/HUD styling**: All HUD styles are in `<style>` block (lines 10–358); adjust via CSS classes.

## File Dependencies

This is a single-file, zero-dependency project. No npm packages, no build tools, no external assets. Everything needed is embedded:
- CSS: Inline styles in `<style>` block.
- JavaScript: All logic in a single `<script>` IIFE.
- Audio: Procedurally generated via WebAudio API.
- Images: Procedural drawing via Canvas 2D API.
