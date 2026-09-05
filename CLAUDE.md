# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in vanilla HTML5 Canvas and JavaScript. It's a single-file game (index.html) with no external dependencies or build process. The game runs directly in a web browser with full support for keyboard, mouse, and touch input.

## Architecture Overview

The entire game is contained within a single `<script>` block in index.html. The codebase follows a functional architecture with these key systems:

### Core Systems

1. **Game State** (lines 863-869): Tracks player position, lives, score, stage, enemies, bullets, boss, and game running state
2. **Rendering Pipeline** (lines 1520-1638): Canvas drawing with sky gradient, clouds, entities, particles, and UI
3. **Game Loop** (lines 1640-1653): `requestAnimationFrame` driven update/render cycle with delta-time
4. **Input Handling** (lines 767-861): Unified keyboard, mouse, and touch input system with adaptive steering
5. **Audio System** (lines 542-676): WebAudio API synthesizer with no external sound files—all sounds are procedurally generated
6. **Entity Update** (lines 1148-1302): Movement, collision detection, spawning, and lifecycle for player, bullets, enemies, powerups

### Entity Types and Data Structures

- **Player**: Position, dimensions, weapon state, level, cooldown, invulnerability timer
- **Enemies**: Three types (grunt, interceptor, tank) with HP, speed modifiers, attack patterns, position
- **Bullets**: Player bullets (hitscan) and enemy bullets (projectiles); support pierce/laser variants
- **Boss**: Massive health, position, attack telegraph timer, pattern index, design reference
- **Particles**: Transient explosion effects with velocity and alpha fade
- **Power-ups**: Life, bomb, or weapon types that float down and apply effects

### Key Game Mechanics

- **Difficulty Scaling** (lines 485-489): Three difficulties (easy/normal/hard) modify spawn rates, enemy speed, and boss HP
- **Weapon System** (lines 871-1146): Two weapons (Vulcan with spread, Laser with pierce) that upgrade 1-3 times, then toggle
- **Combo Multiplier** (lines 1087-1101): Score multiplier increases with consecutive kills, resets after 2.2s without a kill
- **Boss Patterns** (lines 1372-1409): Six bosses with unique telegraph/attack phases; patterns rotate through radial, aimed, and spread attacks
- **Stage Progression** (lines 491, 1204-1207): Advance to next stage after defeating boss; visual themes cycle through THEMES array

### Performance Adaptation

Lines 473-483 implement adaptive quality (full/medium/low) based on frame time tracking. This reduces particle count and shadow effects on slower devices.

## Running and Testing the Game

**No build process required.** Simply:
- Open `index.html` in a web browser
- Click "Start Game" to begin
- Game saves best score, difficulty, mute state, and reduced-motion preference to localStorage

**Testing specific features:**
- Difficulty: Use on-screen buttons or modify `DIFFICULTIES` object
- Weapons: Look for weapon powerups or edit `player.weapon` and `player.level` in `resetGame()`
- Boss patterns: Change `stage` variable or advance through stages by clearing enemies
- Performance: Modify `quality` variable to test adaptive rendering
- Audio: Click mute button or set `muted = true`

## Key Code Sections

| Section | Lines | Purpose |
|---------|-------|---------|
| Constants & themes | 485-515 | Difficulty presets, enemy types, boss designs, visual themes |
| Audio setup & SFX | 542-676 | WebAudio context, beep/noise functions, 12+ sound effect definitions |
| Input system | 767-861 | Keyboard, mouse, touch; joystick logic; mouse steering |
| Entity drawing | 913-1020 | Player plane, boss ship, clouds with parallax |
| Update logic | 1148-1302 | Movement, spawning, collision detection, score application |
| Boss AI | 1372-1439 | Attack pattern definitions (8 pattern types × 6 bosses) and execution |
| Game state | 1469-1504 | Game over, pause, reset flows |

## Design Patterns and Conventions

- **Entity lifecycle**: Create → update position/state → check collisions → remove if dead
- **Audio playback**: All SFX go through `beep()` (oscillator) or `noiseBurst()` (white noise); call `ensureAudio()` before playing to handle browser autoplay restrictions
- **Collision**: Simple AABB overlap in `rectsOverlap()` (line 1069); checks are O(n²) but acceptable for ~50 entities
- **Canvas transforms**: Most drawing saves/restores context to isolate transforms
- **Persistence**: Four localStorage keys (STORAGE_* constants) for best score, mute, difficulty, reduced motion

## Modification Tips

- **Adding enemy types**: Add entry to ENEMY_TYPES object (line 502), adjust spawn logic in `spawnEnemy()` if needed
- **New weapon**: Duplicate weapon branch in `fireWeapon()` (line 1124), add weapon name to `weaponLabel()`, add level-up logic
- **Boss customization**: Add new entry to BOSSES array (line 508); define 3 attack patterns in BOSS_PATTERNS; each pattern is a function that calls `fireRadial()`, `fireAimedSpread()`, etc.
- **Visual themes**: Add theme object to THEMES array (line 493) with sky colors, cloud/sun colors, and name
- **Sound changes**: Modify or add SFX in the `sfx` object (line 624); parameters are freq, duration, waveform type, volume, and options like sweep/filter
- **UI/HUD**: Modify #hud, #overlay, #topBar styles in `<style>` block; track which DOM elements are referenced in the script

## Common Development Tasks

**Rebalance difficulty:**
- Modify `DIFFICULTIES` object spawn rates, enemy speeds, shoot chances, lives, bomb counts, boss HP multipliers
- Difficulty also affects `spawnTimer` calculation (line 1201)

**Adjust performance quality:**
- Quality thresholds are in `trackPerf()` (lines 476-482)
- Quality affects particle count (line 1074), shadow blur (lines 1002, 1583, 1594)

**Debug entity state:**
- Most update happens in `update()` function; add console.log for player, enemies, boss, bullets
- Use browser DevTools to inspect canvas and verify collision boxes visually

**Accessibility:**
- Reduced motion: toggle via button or honor `prefers-reduced-motion` media query (line 520)
- Screen reader announcements go through `announce()` function (line 468) which updates #srLive element
- Arrow/WASD/Space keys in addition to mouse for input

## Storage Keys

Games uses localStorage with these keys (defined as constants):
- `skyfighter.bestScore`: Best score ever achieved
- `skyfighter.muted`: Mute state (0 or 1)
- `skyfighter.difficulty`: Last selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Reduced motion preference (0 or 1)

Clear storage in DevTools Console to reset: `localStorage.clear()`
