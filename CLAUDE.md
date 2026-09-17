# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 Canvas application. Players control a fighter plane, switch weapons, drop smart bombs, and battle through stages with boss encounters.

## Running the Game

Open `index.html` in any modern web browser. The game runs entirely client-side with no build step or external dependencies required.

- **Local**: Double-click `index.html` or drag into browser
- **Web server**: `python3 -m http.server` then navigate to `localhost:8000/index.html`
- **Live testing**: Use the `/run` skill to launch and test

## Architecture

The entire game is self-contained in `index.html` (~1600 lines):

- **HTML (lines 1-421)**: DOM elements for canvas, HUD, buttons, overlays, touch controls
- **CSS (lines 10-359)**: Game styling, animations (heartbeat, combo pulses, flashes), responsive design with CSS clamp
- **JavaScript (lines 422+)**: Full game engine

### Key Game Systems

1. **Game State**: `gameState` object manages lives, score, multiplier, stage, difficulty
2. **Entity System**: Arrays for player, enemies, bullets, explosions; updated each frame
3. **Rendering**: Canvas-based 2D rendering; `renderFrame()` called at 60fps via `requestAnimationFrame`
4. **Physics**: Collision detection (AABB), movement with boundary checking
5. **Difficulty**: 3 tiers (`easy`, `normal`, `hard`) affect enemy spawn rates, speed, shoot chance, boss HP
6. **Input Handling**: Keyboard (arrows, WASD, space, X), mouse movement+click, touch controls with virtual joystick
7. **Audio**: Muted state tracked in localStorage; sound effects would play via Web Audio API if implemented
8. **Persistence**: Best score, mute state, difficulty, and reduced-motion preference saved to localStorage

### Stage/Boss Structure

- Players complete kills (`KILLS_PER_STAGE = 18`) to advance stages
- Each stage increments base `stage` counter and changes visual theme
- Boss spawns at stage 5, 10, 15, etc. (every 5th stage)
- Boss health scales with difficulty (`bossHpMul`)

### Themes

6 visual themes cycle through stages, each with unique sky gradient colors and cloud styling:
- Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge

## Modifying the Game

### Adding Features

**New Weapons**: Add to the weapon definitions object (find `const weapons = {...}` section), set `id`, `fireRate`, `bulletDamage`, `name`. Update weapon-switching logic.

**New Enemy Types**: Define in enemy spawning code, set `type`, `hp`, `speed`, `fireChance`. Update render logic for visual distinction.

**New Stages/Difficulty Tiers**: Add to `DIFFICULTIES` object (lines 485-489), or modify `THEMES` array (lines 493-500).

### Key Constants to Modify

- `KILLS_PER_STAGE = 18`: How many kills needed to advance stage
- Difficulty settings in `DIFFICULTIES` object (spawn rates, enemy speed, boss HP multiplier)
- `THEMES` array: Visual styling (sky gradients, cloud color, sun color)

### Finding Key Code Sections

Use Ctrl+F in the file:

- `const gameState` - Main state object
- `function renderFrame()` - Main render loop
- `function update(dt)` - Physics and game logic
- `function spawnEnemy()` - Enemy spawning
- `canvas.addEventListener('mousemove'` - Input handling
- `const weapons` - Weapon definitions
- `const THEMES` - Visual themes

## Accessibility & User Preferences

The game respects:
- **Reduced Motion** (`prefers-reduced-motion`): Disables animations/screen shake/vignette/flash effects
- **Screen Reader**: Live region (`#srLive`) announces game events
- **Touch Controls**: Virtual joystick and buttons for mobile; auto-enables on touch device detection
- **Mobile Scaling**: Responsive HUD with CSS clamp for various screen sizes

Controls:
- **Desktop**: Mouse to aim, click or space to fire, X or Shift for bomb, P or Esc to pause
- **Mobile**: Drag left side to move, tap FIRE/BOMB buttons

## Development Notes

- **No Build Step**: Direct browser execution; any changes to `index.html` take effect on page reload
- **Performance Tracking**: Adaptive quality system (`quality` variable) reduces particle/effect detail on lower-end devices
- **Canvas Size**: Dynamically set to window size and maintains aspect ratio
- **localStorage Keys**:
  - `skyfighter.bestScore`: High score
  - `skyfighter.muted`: Sound state
  - `skyfighter.difficulty`: Last selected difficulty
  - `skyfighter.reducedMotion`: Accessibility preference

## Testing

No automated tests currently exist. Manual testing checklist:
- [ ] Game runs on desktop (Chrome, Firefox, Safari)
- [ ] Game runs on mobile (iOS Safari, Chrome mobile) with touch controls
- [ ] Weapon cycling and firing work correctly
- [ ] Boss health bar updates and boss defeats register
- [ ] Score multiplier increments on consecutive kills
- [ ] Bombs clear enemies and provide visual feedback
- [ ] Pause/resume works
- [ ] Settings (mute, motion, difficulty) persist across sessions
- [ ] Accessibility features work (reduced motion, screen reader)
