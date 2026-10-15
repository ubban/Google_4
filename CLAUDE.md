# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML/CSS/JavaScript application. The game runs entirely in the browser with no build system or external dependencies.

## Running the Game

The game is a single HTML file and requires no build step. Open `index.html` in a web browser to play:

```bash
# No build required — simply open the file:
# open index.html  # macOS
# xdg-open index.html  # Linux
# start index.html  # Windows
```

The game is fully playable on desktop and mobile with:
- **Desktop**: Mouse steering (glide toward pointer) + click/Space to fire + X/Shift to bomb
- **Mobile**: Touch joystick on left + tap fire/bomb buttons
- **Keyboard**: Arrow keys or WASD to move, Space to fire, X or Shift to bomb

## Codebase Architecture

The entire game logic lives in a single `index.html` file (~1700 lines) organized as an IIFE (Immediately Invoked Function Expression) containing:

### Major Systems

**Input Handling** (lines ~700–860)
- Multi-input system: keyboard (arrows/WASD), mouse (steering toward pointer), touch (virtual joystick)
- Joystick uses a constrained drag model on mobile's left side
- Mouse steering applies velocity-based movement when mouse is active (keyboard takes priority)
- All inputs update player position in the `update()` loop

**Game State** (lines ~863–910)
- Central state variables: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Game flow: `running`, `paused`, `gameOver` flags
- Score tracking: `score`, `chain`, `multiplier`, `stage`, `kills`
- `resetGame()` initializes all state for a new game

**Rendering & Performance** (lines ~450–650, ~913–1020)
- Canvas-based drawing with adaptive quality (quality 0=high detail, 1=medium, 2=low)
- `trackPerf()` monitors frame time and adjusts quality to maintain ~40–60fps
- Custom drawers: `drawPlane()` (player ship), `drawBossShip()` (boss enemies), and particle rendering
- Theme system: 5 visual themes with sky gradients, cloud colors, and sun colors (lines ~500–510)
- Screen effects: vignette overlay, flash effect, screen shake for impact feedback

**Game Loop** (lines ~1148–1400+)
- `update(dt)` handles player movement, weapon fire, collision detection, enemy/bullet updates
- `draw()` renders frame with background gradient, all entities, HUD overlays
- Runs via `requestAnimationFrame` for smooth 60fps (adaptive to device)
- Collision detection uses simple AABB: `rectsOverlap(a, b)`

**Entities & Spawning**
- **Player** (lines ~877): position, size (40×40), speed, weapon type/level, invulnerability timer
- **Enemies** (lines ~1022–1041): three types (grunt, tank, interceptor) with stats in `ENEMY_TYPES` (lines ~502–520). Enemies weave, shoot in patterns (spread or aimed), and drop power-ups on death
- **Boss** (lines ~1043–1056, ~970–1020): one per stage, with phase-based attack patterns stored in `BOSSES` array. Boss HP scales with difficulty and stage
- **Bullets & Particles**: simple velocity-based movement and decay
- **Power-ups**: life, bombs, weapon upgrades—spawn randomly on enemy death

**Combat System**
- **Weapons** (lines ~1124–1146): Vulcan (spread shot, faster fire) vs Laser (piercing, slower). Each has 3 levels affecting spread/damage
- **Weapon upgrades**: drop from enemies, cycle weapon or increase level
- **Bombs** (lines ~1103–1122): clear screen, damage boss, add combo score
- **Combo/Multiplier** (lines ~1087–1101): kill streak mechanic—chain resets on 2.2s idle, multiplier increases every 5 kills (up to 10x)

**Difficulty & Progression**
- Three modes in `DIFFICULTIES` (lines ~485–489): Easy, Normal (default), Hard
- Difficulty affects: enemy spawn rate, enemy speed, shoot chance, player lives/bombs, boss HP
- Progression: 18 kills per stage, then boss battle. 5 boss designs cycle through stages
- Difficulty, best score, mute state, reduced-motion preference saved to `localStorage`

**UI & Accessibility** (lines ~300–470)
- HUD displays score, best score, multiplier, weapon level, hearts/bombs
- Boss health bar (shown during boss fights)
- Screen reader announcements via `aria-live` region (lines ~470)
- Reduced motion toggle respects user preference (disables screen shake, vignette pulse, flash effects)

## Key Development Notes

### Single-File Design
- No modularization; all state is closure-scoped within the IIFE
- Reduces coupling but makes the file longer; additions go where logically related code lives
- No imports/exports; modify constants and functions directly

### Coordinate System
- Canvas origin (0,0) at top-left
- Y increases downward (player starts near bottom)
- Collision checks use bounding boxes centered on entity position: `Math.abs(a.x - b.x) < (a.w + b.w) / 2`

### Performance Tuning
- Particle count auto-reduces on slower devices (line ~1074)
- Shadow blur disabled on lowest quality (line ~1002)
- Quality 0 (desktop) renders full detail; quality 2 (mobile) skips expensive effects
- If changes cause frame drops, check `trackPerf()` thresholds (lines ~476–483) or reduce particle count

### Adding Features
- **New enemy type**: Add to `ENEMY_TYPES`, update spawn logic in `spawnEnemy()`
- **New weapon**: Add to `fireWeapon()` branches and `weaponLabel()`
- **New visual effect**: Create a draw function and call from `draw()` loop
- **New stage mechanic**: Hook into `stage` variable and difficulty config
- **Audio**: Call `sfx.bomb()`, `sfx.shoot()`, `sfx.laser()` as needed; mute state checked internally

### Mobile Considerations
- Touch events use `preventDefault()` to avoid unwanted scrolling/zoom
- Virtual joystick (base + stick) appears on left; fire/bomb buttons on right
- Canvas resizes/scales via CSS `max-width: 100%; max-height: 100%;` to fit viewport
- Safe area insets respected for notches: `env(safe-area-inset-top)` etc. in HUD positioning

### Testing
No automated test suite; manual testing covers:
- Each difficulty mode + boss progression
- All three control methods (keyboard, mouse, touch)
- Weapon switches and level progression
- Bomb usage and enemy clear
- Screen resize (desktop window, mobile orientation)
- Reduced motion toggle (no flashing/shaking)

## Configuration & Constants

- **Difficulty params** (lines ~485–489): spawn rate, enemy speed, player lives, bomb count, boss HP multiplier
- **Enemy types** (lines ~502–520): speed, HP, fire rate, score, color
- **Boss designs** (lines ~520+): name, colors, HP multiplier
- **Themes** (lines ~500–510): sky gradient, cloud/sun colors, name
- **Stage progression**: `KILLS_PER_STAGE = 18` (line ~490)
- **Weapon fire delay**: Vulcan 0.15s, Laser 0.1s (line ~1177)

## Browser APIs Used

- **Canvas 2D** (`ctx.fillStyle`, `ctx.beginPath()`, etc.)
- **localStorage** (best score, mute, difficulty, reduced motion)
- **requestAnimationFrame** (game loop)
- **Web Audio** (if present; graceful fallback for mute)
- **Touch Events** (`touchstart`, `touchmove`, `touchend`)
- **Keyboard Events** (`keydown`, `keyup`)
- **Mouse Events** (`mousemove`, `mousedown`)
