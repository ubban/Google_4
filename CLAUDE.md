# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application with no external dependencies. The entire game (HTML, CSS, JavaScript) is contained in `index.html` (~1670 lines).

## Development Quick Start

- **Run the game**: Open `index.html` directly in a web browser or use `python -m http.server` to serve locally
- **Testing changes**: Refresh the browser to see updates immediately (no build step needed)
- **Check browser console**: Open DevTools to catch any runtime errors or console output

## Code Architecture

### High-Level Structure

The game runs as a single self-executing function (IIFE) starting at line 423. The architecture follows this pattern:

1. **Initialization** (lines 424–527): DOM element references, game constants (difficulties, enemies, bosses, themes), persistent storage setup
2. **Audio system** (lines 542–676): Web Audio API synthesis for all sound effects (beeps, noise bursts, modulated sounds)
3. **UI/Rendering helpers** (lines 680–750): DOM manipulation for HUD, hearts, bombs, combo notifications, pause, difficulty
4. **Input handling** (lines 767–862): Keyboard, mouse, and touch input (including joystick emulation on mobile)
5. **Game state & update loop** (lines 863–1302): Core simulation (enemy spawning, collision detection, damage, scoring)
6. **Boss logic** (lines 1372–1439): Pattern definitions and boss behavior
7. **Game lifecycle** (lines 1441–1505): End game, pause, stage progression
8. **Canvas rendering** (lines 1507–1638): Drawing planes, bullets, enemies, particles, HUD
9. **Main loop** (lines 1640–1665): requestAnimationFrame tick that calls update() and draw()

### Key Game Objects

- **Player**: Position, dimensions, speed, weapon (vulcan/laser), invulnerability, cooldown
- **Enemies**: Position, type (grunt/interceptor/tank), health, speed pattern (weave), shooting behavior
- **Bullets**: Player bullets (pierce-able lasers or spread vulcan shots) and enemy bullets (straight or radial patterns)
- **Boss**: Complex multi-stage entity with 6 distinct designs and 3-pattern attack sequences per boss
- **Power-ups**: Life, bomb, weapon upgrades collected by collision
- **Particles**: Explosion debris with physics

### Difficulty & Progression

Three difficulties (easy/normal/hard) adjust spawn rates, enemy behavior, lives, bombs, and boss health. 
Progression: 18 kills → boss spawn → stage clear → new theme. Stages cycle through 6 unique sky/cloud/sun color themes.

### Weapon System

**Vulcan** (levels 1–3): Single to triple spread shot. **Laser** (levels 1–3): Single or multiple vertical piercing beams. Swaps between weapons at max level, resetting to level 1.

### Boss Patterns (Lines 1372–1409)

Each of the 6 bosses has 3 attack patterns (radial bursts, aimed spreads, columns, spirals, etc.) selected in order. Patterns include:
- `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`

### Mobile Controls

- **Left 55% of screen**: Virtual joystick for movement
- **Right 45% of screen**: FIRE and BOMB buttons
- Auto-detects touch device and enables touchControls on line 779

### Audio (No External Assets)

All sounds are synthesized using Web Audio API:
- **Oscillators** (sine, square, sawtooth, triangle) with frequency sweeps
- **Noise bursts** for explosions
- **Filters** (lowpass/highpass) for tone shaping
- **Stereo panning** based on screen position for spatial audio

Start audio with `ensureAudio()` (line 545) on first user interaction; stored mute preference survives refreshes.

### Accessibility

- **Reduced motion**: Respects `prefers-reduced-motion` media query; toggleable via UI
- **Screen reader support**: Announcements via `srLive` element on important events (score milestones, game over, stage clear)
- **Semantic HTML**: Buttons, ARIA attributes for live regions

### Persistent Storage

- `skyfighter.bestScore`: High score (number)
- `skyfighter.muted`: Audio mute toggle (0/1)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Motion reduction override (0/1)

All stored/loaded via `localStorage` at startup and updated on user action.

### Performance

Adaptive quality levels (full, medium, low) tracked on lines 474–483 based on frame time averages. Lower quality reduces particle counts and shadow effects on slower devices.

### Collision Detection

Simple axis-aligned rectangle overlap check: `rectsOverlap()` (line 1069). Used for player–enemy, bullets–enemies, bullets–boss, powerups–player.

## Common Edits

**Add/modify difficulty**: Edit `DIFFICULTIES` constant (line 485).
**Adjust enemy spawning**: Modify `spawnTimer`, `spawnEnemy()` (lines 1022–1041), or `ENEMY_TYPES` (line 502).
**Change boss patterns**: Add methods to pattern-firing functions or extend `BOSS_PATTERNS` (line 1372).
**Tweak weapon balance**: Edit weapon firing logic in `fireWeapon()` (line 1124) or cooldown timings.
**Add visual themes**: Extend `THEMES` array (line 493) with new sky gradient, cloud, and sun colors.
**Adjust scoring/multiplier**: Modify `addScore()` (line 1087) and chain logic.
**Fine-tune sound effects**: Edit `sfx` object (line 624) – frequency, duration, modulation.
**Change canvas size or input mapping**: See `BASE_W`/`BASE_H` (line 753) and input handlers (lines 768–862).

## No Build Step

The file is self-contained; there's no bundling, transpilation, or asset pipeline. Edit and reload the browser to test.
