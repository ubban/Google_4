# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. It's a self-contained single-file HTML/CSS/JavaScript game that runs directly in the browser with no build process, dependencies, or external assets.

## Architecture & Code Structure

The `index.html` file contains the entire game. The embedded JavaScript is organized into logical sections:

### State Management
- **Game state**: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- **Scoring**: `score`, `chain`, `chainTimer`, `multiplier` (combo system)
- **Game modes**: `running`, `paused`, `gameOver`, `stage`, `stageKills`
- **Difficulty**: Three tiers (easy/normal/hard) stored in `DIFFICULTIES` object; affects spawn rates, enemy behavior, lives, bombs, and boss HP

### Core Subsystems

**Input System**
- Keyboard (arrow keys/WASD for movement, Space for fire, X/Shift for bomb, P/Esc for pause)
- Mouse steering (optional; when active, plane glides toward pointer; keyboard takes priority)
- Touch input with joystick on left side, fire/bomb buttons on right (for mobile devices)

**Audio (Web Audio API)**
- Procedural synth-based sound effects (no external audio files)
- Master gain, compressor, oscillators, filters, noise bursts
- Mute toggle persisted to localStorage
- Pan effects for spatial audio (left/right based on screen position)

**Rendering**
- Canvas-based (480×720 px, responsive scaling)
- Sky gradient that changes per stage (`THEMES` array)
- Planes drawn via path operations (`drawPlane` for player/enemies, `drawBossShip` for bosses)
- Adaptive quality setting: tracks frame time and reduces glow/particle count on slower devices

**Gameplay Systems**

- **Spawning**: Enemies spawn at configurable intervals; scales difficulty over time
- **Weapons**: Two types—Vulcan (spread shot) and Laser (focused beam with piercing). Levels 1-3, then weapon swap
- **Enemies**: Three types (grunt, interceptor, tank) with different HP, speed, and shooting patterns; appear when stage >= `minStage`
- **Boss Encounters**: Spawn after 18 kills per stage; each of 6 boss types (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) has unique attack patterns
  - Boss patterns defined in `BOSS_PATTERNS` array, indexed by boss type
  - Patterns fire bullets in various formations (radial bursts, aimed spreads, homing missiles, etc.)
  - Boss telegraphs attacks before firing
- **Bombs**: Smart bomb clears all on-screen enemies and enemy bullets, deals damage to boss, refunds score
- **Power-ups**: Life, bomb, and weapon pickups drop after enemy death
- **Combo/Multiplier**: Chain bonus for consecutive kills resets on player hit; x1 to x10 multiplier based on chain length

### Data Structures

- **DIFFICULTIES**: Spawn rates, enemy speed multiplier, shoot chance, lives, bombs, boss HP multiplier
- **THEMES**: Per-stage sky color gradients, cloud colors, sun colors, and theme names
- **ENEMY_TYPES**: Properties for each enemy type (size, HP, speed, color, score, minimum stage)
- **BOSSES**: Boss designs with hull colors, core colors, and HP multiplier
- **BOSS_PATTERNS**: Arrays of attack functions for each boss type

### Persistence

Uses `localStorage` for:
- Best score (`skyfighter.bestScore`)
- Mute setting (`skyfighter.muted`)
- Difficulty selection (`skyfighter.difficulty`)
- Reduced motion preference (`skyfighter.reducedMotion`)

## Running & Testing

**No build process required.** Open `index.html` directly in any modern browser (desktop or mobile).

**To test a change:**
1. Open `index.html` in a browser
2. Play a few rounds to verify gameplay feel, enemy spawning, boss behavior, audio, mobile controls
3. Test difficulty modes (Easy, Normal, Hard)
4. Check pause/resume, muting, motion-reduction toggle work
5. Verify all weapon types and power-ups spawn and function correctly

**Mobile testing:**
- Use Chrome DevTools device emulation or test on actual device
- Verify touch joystick responds smoothly; fire and bomb buttons are positioned correctly

## Key Game Tuning Parameters

- **BASE_W / BASE_H**: Canvas dimensions (480×720 px)
- **KILLS_PER_STAGE**: How many enemies to kill before boss spawns (currently 18)
- **Spawn rates**: `spawnBase`, `spawnMin`, `difficultyRamp` in DIFFICULTIES
- **Boss HP formula**: `(70 + stage * 35) * bossHpMul * designHpMul`
- **Enemy speeds & shoot chances**: Vary per difficulty
- **Weapon balance**: Fire rate (laser 0.1s, vulcan 0.15s), spread patterns, pierce counts
- **Animation frame times & durations**: Combo pop (0.6s), stage banner (2s), flash effects (0.25s/0.5s)

## Development Notes

- **Performance**: Adaptive quality system reduces particles, shadows, and glow when FPS dips
- **Accessibility**: Screen reader announcements, reduced-motion support (disables animations when `prefers-reduced-motion` is set)
- **CSS**: Responsive design uses clamp() for font sizes and layout; safe-area insets for notched devices
- **No dependencies**: Pure vanilla JS, HTML, CSS—no frameworks or external libraries
- **Canvas optimization**: Clip/cull bullets and enemies off-screen; avoid excessive shadowBlur in low quality mode

## Common Changes

- **Adjust difficulty**: Edit DIFFICULTIES object values
- **Add a new boss**: Add to BOSSES array and corresponding pattern array in BOSS_PATTERNS
- **Change enemy behavior**: Modify spawnEnemy(), ENEMY_TYPES, or enemy update logic
- **Tune weapon feel**: Adjust fireWeapon(), fire rates, spread angles
- **Modify themes**: Edit THEMES array (sky gradients, cloud colors, etc.)
- **Sound tweaks**: Edit sfx object functions (frequencies, durations, filter types)
