# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file web application. The entire game is contained in `index.html` (~1669 lines) with embedded CSS and JavaScript.

**Live at:** `https://github.com/ubban/Google_4`

## Quick Start

- **Run the game:** Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge)
- **No build step required** — it's pure HTML/CSS/JavaScript

## Codebase Structure

The single `index.html` file contains:

### HTML Section (lines ~1–20)
- Meta tags, viewport configuration, favicon (inline SVG), and page title
- Container and canvas elements for game rendering
- HUD (heads-up display) elements for stats, weapons, bombs, and lives

### CSS Section (lines ~20–150)
- **Dark theme** with `color-scheme: dark`
- **Canvas styling:** gradient background, shadow, responsive sizing
- **HUD styling:** stats display with glassmorphism (semi-transparent cards with blur)
- **Animations:** heart beat (low health warning), vignette pulse (hit effect), flash effects (damage/bomb feedback)
- **Safe area insets** for mobile notch handling

### JavaScript Section (lines ~150–1669)
The game engine is organized as a single large script with no external dependencies:

1. **Game State & Config** — resolution, difficulty, game speed
2. **Asset Definitions** — sprite/sprite sheet info (enemies, player, effects)
3. **Entity Classes:**
   - `Player` — movement, health, weapons, bombs
   - `Enemy` — basic and boss enemies with patterns and behavior
   - `Projectile` — bullets (player and enemy) with collision
   - `Effect` — visual effects (explosions, impacts)
4. **Game Manager** — game loop, entity management, collision detection, scoring
5. **Input Handling** — keyboard, mouse, and touch controls
6. **Rendering** — sprite-based drawing using canvas

## Key Architectural Patterns

### Sprite-Based Rendering
- All visuals are drawn from an embedded sprite sheet (base64 PNG) using canvas `drawImage()` with source/destination coordinates
- Sprites are defined with `{x, y, width, height}` rects in a lookup object

### Entity Management
- Entities (player, enemies, projectiles, effects) are stored in separate arrays
- Game loop iterates over arrays, updates state, renders, and removes dead entities
- Collision detection uses bounding box (AABB) checks between entity rectangles

### Weapon System
- Player has multiple weapon types (slot 0-2) that can be switched with **1/2/3** keys
- Each weapon has different fire patterns, damage, and cooldown
- Weapons can have variations (e.g., spread shots, rapid fire, beams)

### Game Loop
```
requestAnimationFrame() → update all entities → collision checks → render → repeat
```

### Controls
- **WASD or Arrow Keys:** Move player
- **Left Mouse / Touch:** Fire weapon
- **1/2/3:** Switch weapon
- **Space / Right Mouse:** Drop smart bomb (limited per stage)
- **Mouse/Touch:** Aim (weapon fires toward cursor/touch point)

## Common Development Tasks

### Adding a New Enemy Type
1. Add sprite definition to the enemy sprites object (find `enemySprites`)
2. Create a new enemy class or extend `Enemy` class with custom behavior
3. Modify the spawner logic to instantiate your new enemy at appropriate waves
4. Test in-game and adjust movement patterns, health, and damage values

### Adding a New Weapon
1. Add sprite definition if it has unique visuals
2. Create weapon config with `{dmg, rof, spread, ...}` in weapon definitions
3. Implement firing logic in the `fireWeapon()` or weapon-switch section
4. Assign hotkey binding (1/2/3 keys) in input handler

### Tweaking Game Balance
- **Player health:** Search for `playerHealth` or `maxHealth` in Player class
- **Enemy difficulty:** Adjust enemy spawn rate, health, damage in spawner logic
- **Weapon damage:** Modify `dmg` values in weapon definitions
- **Game speed:** Adjust `gameSpeed` constant at the top

### Testing Mobile
- Open `index.html` on a mobile device or use browser DevTools device emulation
- Game supports touch controls (drag to move, tap to fire, gestures for bombs)
- Viewport is fully responsive with safe area insets for notched devices

## Performance Considerations

- **No external dependencies** — entire game runs in a single HTML file
- **Canvas rendering** is efficient for 2D sprite-based graphics
- **Entity pooling** could be added if performance degrades (currently not used)
- **Sprite atlas** is embedded as base64, so no extra network requests

## Browser Compatibility

- Modern browsers (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+)
- Requires `requestAnimationFrame`, canvas 2D context, touch/mouse event handling
- No transpilation needed; uses ES6 syntax throughout

## Git Workflow

- Main branch: `main`
- Feature branches: `claude/*` for Claude Code agents
- Commit messages should describe game mechanic or UI changes clearly
