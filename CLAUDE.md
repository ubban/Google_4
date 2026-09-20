# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**Sky Fighter** is a single-file HTML5 game — a Raiden-style vertical shoot-'em-up written entirely in vanilla JavaScript and Canvas. The entire game (HTML, CSS, and JS) is contained in `index.html` (~1700 lines).

## Running the Game

No build process required. Simply open `index.html` in a web browser. The game runs immediately.

- **Development**: Use `python -m http.server 8000` or similar to serve locally if testing cross-origin features.
- **Testing**: Open `index.html` directly in the browser or via localhost.

## Project Structure

Since this is a monolithic file:
- **Lines 1–359**: HTML structure & CSS styling
- **Lines 422–1667**: Game logic (wrapped in IIFE)

## Architecture & Key Systems

### Core Game Loop (lines 1640–1652)
- `loop()` uses `requestAnimationFrame` at 60 FPS.
- Calls `update(dt)` for logic, `draw()` for rendering.
- Delta time capped at 50ms to handle frame drops.

### Game State (lines 864–869)
Main state variables:
- `player`: Position, speed, weapon, level, invulnerability
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`
- `boss`: Active boss or null
- `score`, `lives`, `stage`, `chain` (combo), `multiplier`

### Input Handling (lines 767–861)
- **Keyboard**: Arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
- **Mouse**: Movement steers the plane; click fires
- **Touch**: Left zone for movement (joystick), right zone for fire/bomb buttons
- Priority: Keyboard/joystick > mouse (mouse only activates when no other input detected)

### Rendering & Canvas (lines 752–1638)
- Base resolution: 480×720 (scaled with `resize()` on window events)
- Sky themes rotate per stage (6 themes, lines 493–500)
- Screen shake: controlled via `screenShake(magnitude, duration)`
- Quality adaptation: `quality` variable (0=full, 1=medium, 2=low) auto-scales particles & effects based on frame time

### Difficulty System (lines 485–489)
Three presets: **easy**, **normal**, **hard** — affects spawn rate, enemy speed, HP, bomb count, boss HP.
- Stored in localStorage under `skyfighter.difficulty`
- Set via difficulty buttons in overlay

### Audio (lines 542–676)
- **No external assets** — all sound generated via Web Audio API
- `beep()`: Oscillator-based tones with optional frequency sweep, filters, panning
- `noiseBurst()`: White noise filtered for punch
- `sfx` object: Named sounds (shoot, laser, explode*, hit, bomb, powerup, bossHit, stageClear, gameover, uiClick)
- Mute toggle saves to localStorage

### Enemy & Boss Mechanics

**Enemy Types** (lines 502–506):
- `grunt`: Basic, 1 HP
- `interceptor`: Fast, weaves, 1 HP
- `tank`: Slow, 4 HP, appears stage 2+

**Enemy Behavior**:
- `spawnEnemy()` (lines 1022–1041): Randomly picks type based on stage; assigns weave, shoot pattern (spread or aimed)
- Enemies shoot on timers; aimed shots track toward player

**Boss System** (lines 1043–1056):
- 6 boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), cycle per stage
- Each boss has unique attack pattern set (lines 1372–1409)
- Patterns: radial bursts, homing missiles, spirals, columns, teleport + attacks
- Boss enters with telegraph animation; fires pattern, waits, repeats
- Boss health scales by difficulty multiplier and boss design HP multiplier

**Boss Attack Patterns** (lines 1324–1409):
- `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`
- Each pattern defined as a function; `BOSS_PATTERNS` array maps designs to 3-pattern cycles
- New pattern fires on telegraph cooldown (~22 frames); `patternIndex` cycles through

### Weapon System (lines 1124–1146)
- **Vulcan**: Rapid, spreads at level 3
- **Laser**: Slower, pierces through enemies, wider lanes at higher levels
- Levels 1–3 improve spread/lanes; level 3+ can switch weapon type
- Weapon drops from enemies; collected via collision

### Combo & Scoring (lines 1087–1101)
- `chain` counter increments per kill; resets after 2.2s (chainTimer)
- `multiplier` = 1 + floor(chain / 5), capped at 10x
- Visual feedback: combo counter pops every 10 kills

### Particle System (lines 1073–1085)
- `spawnExplosion()`: Creates random velocity particles that fade
- Count scaled by quality level
- Used for enemy deaths, hits, bomb detonations

### UI & HUD
- **Overlay** (lines 401–419): Start screen, pause menu, game-over screen
- **HUD** (lines 367–374): Score, best score, multiplier, weapon, hearts, bombs
- **Boss bar** (lines 376–378): Shows boss name & health during boss fight
- **Stage banner** (lines 381–382): Displays on stage transitions
- **Combo pop** (line 381): Shows kill milestones (10 KILLS!, etc.)
- **Top bar buttons** (lines 384–388): Mute, motion reduction, pause

### Performance Tracking (lines 473–483)
- `trackPerf(dt)`: Maintains 45-frame rolling average of frame times
- Auto-scales quality if avg frame time >25ms (quality++) or <17ms (quality--)
- Affects particle counts, shadow blur, glow intensity

### Storage (lines 463–466)
- `skyfighter.bestScore`: High score persistence
- `skyfighter.muted`: Mute state
- `skyfighter.difficulty`: Last difficulty selected
- `skyfighter.reducedMotion`: Reduced motion preference override

## Common Editing Tasks

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (lines 502–506) with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Add spawn chance condition in `spawnEnemy()` (line 1029)
3. If new death sound, add to `EXPLODE_SFX` map (line 678)

### Add a New Boss
1. Add to `BOSSES` array (lines 508–515) with hull colors, wing color, core color, `hpMul`
2. Add attack pattern array to `BOSS_PATTERNS` (lines 1372–1409)
3. Bosses cycle by stage mod `BOSSES.length`

### Add a New Boss Attack Pattern
- Create function using helpers: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.
- Add to a boss's pattern array in `BOSS_PATTERNS`
- Pattern receives boss object `b`; uses `enemyBullets.push()` to fire

### Add a New Theme
1. Add to `THEMES` array (lines 493–500) with sky gradient stops, cloud/sun color (RGB strings), name
2. Applied via `stage % THEMES.length`

### Add a New Sound
1. Create function in `sfx` object (lines 624–676) using `beep()` or `noiseBurst()`
2. Call via `sfx.soundName()`

### Adjust Difficulty
1. Modify `DIFFICULTIES` object (lines 485–489): spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul
2. Changes take effect next game start

### Modify Collision Detection
- `rectsOverlap(a, b)` (lines 1069–1071): AABB collision, uses `w` and `h` half-widths
- Called for: enemy bullets hitting player, player bullets hitting enemies/boss, enemies/boss hitting player, powerups

### Control Screen Shake
- `screenShake(magnitude, duration)` triggers shake; rendered in `draw()` (lines 1523–1527)
- Reduced motion preference scales magnitude down and caps duration

## Quick Reference: Key Lines

| System | Lines |
|--------|-------|
| HTML/CSS | 1–359 |
| Difficulty presets | 485–489 |
| Enemy types | 502–506 |
| Bosses | 508–515 |
| Themes | 493–500 |
| Audio setup | 542–561 |
| SFX definitions | 624–676 |
| Canvas sizing | 752–765 |
| Input handlers | 767–861 |
| Game state init | 864–911 |
| Main game loop | 1640–1652 |
| Update logic | 1148–1302 |
| Drawing | 1520–1638 |
| Boss patterns | 1372–1409 |
| Boss attack firing functions | 1324–1371 |

## Testing Tips

- Open browser DevTools (F12) and check console for any errors
- Modify `DIFFICULTIES` to test different challenges quickly
- Add `console.log()` in `update()` to debug game state
- Temporarily set `quality = 0` to force high-quality rendering for testing
- Test touch controls on mobile or use browser device emulation (Ctrl+Shift+M)

## Notes

- Game runs at 60 FPS; delta time is capped at 50ms to prevent large jumps on frame drops
- All game state is transient; no level progression saved (only high score)
- LocalStorage keys are prefixed with `skyfighter.` to avoid collisions
- Accessibility: Screen reader announcements via `announce()` (line 468) and `srLive` element for live regions
