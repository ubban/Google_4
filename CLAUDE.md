# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in HTML/JavaScript. It's a single-file project (`index.html`) with no build system, external dependencies, or tooling. The game runs directly in any modern browser.

## Development

### Running the Game

Open `index.html` in a web browser. The game starts with a menu overlay where you can select difficulty and begin playing.

### No Build/Test/Lint Commands

This project doesn't use any build system, testing framework, or linter. Changes are instant—just edit `index.html` and refresh the browser.

## Architecture

The game is organized as one monolithic ~1670-line HTML file with embedded CSS and JavaScript. The code structure within the `<script>` tag follows this pattern:

### Core Systems

1. **Game State** (lines 863–910)
   - Player, bullets, enemies, enemy bullets, particles, power-ups, boss
   - Score, lives, multiplier chain, stage progression
   - `resetGame()` initializes all state for a new game

2. **Update Loop** (lines 1148–1302)
   - `update(dt)` handles input, movement, collision detection, and game logic
   - Runs every frame if `running && !paused`
   - Collision detection between: player + enemies, player + bullets, bullets + enemies, bullets + boss

3. **Rendering** (lines 1520–1638)
   - `draw()` renders all game elements using Canvas 2D
   - Sky gradient with parallax clouds
   - Planes (player and enemies), bullets, boss, particles, power-ups
   - HUD updates via DOM elements (score, lives, bombs, weapon, multiplier)
   - Screen shake effect for impacts (reduced in low-performance or reduced-motion modes)

4. **Audio** (lines 542–676)
   - WebAudio synth (no external audio files)
   - `beep()` for pitched tones, `noiseBurst()` for noise
   - Muted state saved to localStorage

5. **Input** (lines 767–861)
   - Keyboard: arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
   - Mouse: move to steer, click to fire
   - Touch: left side for movement (joystick), right buttons for fire/bomb

### Enemy & Boss System

- **Enemy types** defined in `ENEMY_TYPES` (lines 502–506): grunt, interceptor, tank with varying HP, speed, and scoring
- **Boss patterns** (lines 1372–1409): 6 unique boss designs with complex attack patterns (radial, aimed spreads, spirals, homing, etc.)
- **Difficulty system** (lines 485–489): Easy/Normal/Hard change spawn rates, enemy speed, shoot chance, lives, and HP multipliers

### Power-Up & Weapon System

- **Weapons**: Vulcan (spread shots) and Laser (penetrating beams)
- **Levels**: 1–3 progression; at level 3, collecting power-ups switches weapon type
- **Power-up types**: life (health), bomb (smart bomb), weapon (upgrade)
- Weapon properties affect fire rate and spread pattern (lines 1124–1146)

### Stage Progression

- Each stage requires 18 enemy kills (`KILLS_PER_STAGE`)
- After kills, a boss appears
- Defeating boss advances stage, increases difficulty, changes theme

### Theme System

- 6 visual themes (lines 493–500) with different sky gradients, cloud colors, sun colors
- Theme cycles with stage progression

## Key Implementation Details

- **Collision detection**: Simple axis-aligned bounding box (AABB) overlap via `rectsOverlap()`
- **Scoring & chain multiplier**: Kill streak extends combo timer; missing for 2.2 seconds breaks chain. Multiplier ranges 1–10x based on chain count
- **Adaptive performance** (lines 473–483): Tracks frame time; adjusts render quality (full, medium, low) to maintain performance
- **Reduced motion support**: Respects `prefers-reduced-motion` media query; disables animations on user toggle
- **Canvas sizing** (lines 752–765): Fixed 480×720 logical canvas, scaled to fit window up to 1.6x
- **LocalStorage**: Persists best score, mute state, difficulty, reduced-motion preference

## Common Edits

- **Adjust difficulty**: Modify `DIFFICULTIES` object values (spawn rates, speeds, HP)
- **Add/modify enemy type**: Add to `ENEMY_TYPES`, update spawn logic
- **New boss design**: Add to `BOSSES`, define pattern in `BOSS_PATTERNS`
- **Change weapon behavior**: Edit `fireWeapon()`, `fireRadial()`, etc.
- **Tweak visual theme**: Modify `THEMES` or gradient colors in `draw()`
- **Balance scoring**: Change base scores in `ENEMY_TYPES`, `addScore()` calls
- **Update SFX**: Edit `beep()` or `noiseBurst()` parameters in `sfx` object

## Performance Notes

- Quality setting automatically downgrades visual effects (shadows, glows) on slower devices
- Particle count scales with quality level
- Frame cap is ~50ms to handle 60 FPS target with headroom
- WebAudio context is lazily initialized and can be disposed

## Accessibility

- Screen reader announcements via `announce()` function updating `#srLive` aria-live region
- Reduced motion toggle respects and can override system preference
- Button titles and labels present for UI controls
- Full keyboard support for all gameplay

## Testing

Use the browser's developer tools to:
- Test different viewport sizes (DevTools → Device Toolbar)
- Monitor performance (DevTools → Performance tab, record gameplay)
- Check accessibility (DevTools → Lighthouse)
- Debug game state in console (e.g., `player`, `score`, `lives`)
