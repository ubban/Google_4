# Sky Fighter — Codebase Guide

## Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in vanilla HTML5/Canvas. The game is contained in a single `index.html` file (1669 lines) with embedded CSS and JavaScript.

### Key Features
- Vertical scrolling shmup gameplay
- Multiple weapon types that can be switched
- Smart bomb mechanics
- Stage-based boss encounters
- Real-time HUD with score, lives, bombs, and weapon display
- Responsive canvas scaling for mobile and desktop
- Touch and mouse control support

---

## Project Structure

```
Google_4/
├── index.html      (Complete game implementation)
├── README.md       (Project metadata)
└── .git/
```

**Single-file architecture:** All game code (HTML, CSS, JavaScript) is in `index.html`. No build process, no external dependencies.

---

## Code Architecture

### Three-Layer Structure

1. **HTML & Meta** (Top, ~50 lines)
   - DOCTYPE, charset, viewport configuration
   - Theme color and favicon
   - Container and canvas elements

2. **CSS Styling** (~150 lines)
   - `:root` for color scheme configuration
   - Canvas and HUD styling
   - Dark mode support via `prefers-color-scheme`
   - Animation keyframes for effects (heartBeat, vignette, flash)
   - Responsive sizing with `clamp()` for mobile

3. **JavaScript Game Engine** (~1400 lines)
   - Player ship class and input handling
   - Enemy and projectile systems
   - Collision detection and physics
   - Game state management (menu, playing, game over)
   - Weapon switching and bomb mechanics
   - Stage/boss progression
   - Rendering loop (requestAnimationFrame)

---

## Key Game Systems

### Player Ship
- Positioned at screen bottom, controlled via mouse/touch
- Health system (displayed as hearts in HUD)
- Weapon inventory with switchable types
- Smart bomb resource management

### Weapons
Multiple weapon types available:
- **Basic**: Standard bullet stream
- **Spread**: Wider coverage arc
- **Rapid**: High fire rate
- **Charged**: Powerful single shots (likely)

Can be toggled mid-game; HUD shows current weapon.

### Enemies & Bosses
- Spawned in waves during stages
- Boss encounters at stage completion
- Collision damage and explosion effects
- Drop items (likely health, weapons, bombs)

### Collision & Hit Detection
- Player-bullet to enemy collision
- Enemy bullet to player collision
- Visual feedback: flash effect, vignette pulse on damage
- Knockback and invulnerability frames (likely)

### HUD System
Located at top-center, displays:
- Score (gold text)
- Lives (❤ hearts, grayed when lost)
- Bombs (💣 icons, opacity indicates availability)
- Weapon type (lime green text)
- Multiplier/combo counter (orange text)

---

## Development Notes

### Working with the Code

**Why single-file?**
- Easy to run locally (no server needed)
- Complete game in one portable file
- Suitable for game jams and quick prototypes

**To modify:**
1. Open `index.html` in any editor
2. Keep the three sections (HTML, CSS, JavaScript) intact
3. Test in a browser with `File > Open` or a local server
4. Commit with clear messages describing changes

### Key Areas for Enhancement

1. **Adding new weapon types** → Find weapon switch logic in JavaScript
2. **Tuning game difficulty** → Enemy spawn rates, health pools
3. **New stages/bosses** → Stage progression system
4. **Visual polish** → CSS animations and shader effects
5. **Audio** → Web Audio API integration (not yet present)

### Testing Locally
```bash
# Option 1: Direct file open
open index.html  # macOS
start index.html # Windows
xdg-open index.html # Linux

# Option 2: Simple HTTP server
python3 -m http.server 8000
# Then visit http://localhost:8000/index.html
```

---

## Game Flow

1. **Initialization** → Canvas setup, asset preloading (if any)
2. **Menu State** → Press to start (if implemented)
3. **Playing State** → Main game loop
   - Update player position from input
   - Spawn enemies/bullets
   - Process collisions
   - Update HUD
   - Render all entities
4. **Boss Stages** → Special enemy patterns
5. **Game Over** → Restart or exit

---

## Technical Stack

- **Language**: Vanilla JavaScript (ES6+)
- **Rendering**: Canvas 2D API
- **Input**: Mouse, Touch, Keyboard
- **Animation**: RequestAnimationFrame
- **Styling**: CSS3 (no preprocessor)

**No external libraries** — entirely self-contained.

---

## Browser Compatibility

- **Chrome/Edge**: Full support
- **Firefox**: Full support
- **Safari**: Full support
- **Mobile browsers**: Optimized with viewport meta and touch handlers

Canvas and modern CSS are well-supported across all modern browsers.

---

## Notes for Contributors

- Keep the single-file structure unless there's a strong reason to split
- Maintain responsive design (uses `clamp()` and `vw/vh` units)
- Test on both desktop and mobile before committing
- Preserve the dark theme aesthetic
- Use clear variable names and comments for complex game logic
- Update this CLAUDE.md if architectural changes are made

---

*Last updated: 2026-10-02*
