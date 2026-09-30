# Claude.md — Sky Fighter Game Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML/JavaScript and Canvas. The game features:
- Player-controlled aircraft with weapon switching
- Enemy waves and stage bosses
- Smart bomb mechanics
- Score and health tracking
- Mobile-responsive design with touch support

The entire game is contained in a single `index.html` file (~58KB) with embedded CSS and JavaScript.

## Architecture

### Game Structure
- **Canvas Rendering**: 2D Canvas-based graphics at dynamic resolution scaling
- **Entity System**: Player, enemies, bullets, explosions as managed entities
- **Input Handling**: Keyboard and touch/pointer support with multi-touch awareness
- **State Management**: Game state (menu, playing, game-over, boss fight)
- **Audio System**: Sound effects and music (if implemented)

### Key Game Variables
- `player`: Main player aircraft object with health, weapons, position
- `enemies`: Array of active enemy entities
- `bullets`: Array of active projectiles (player bullets and enemy fire)
- `explosions`: Particle effects and explosion animations
- `gameState`: Current game phase (START, PLAYING, BOSS, GAMEOVER, WIN)
- `score`, `level`: Game progression tracking
- `gameWidth`, `gameHeight`: Canvas dimensions (responsive to viewport)

### HTML Structure
- `#container`: Fixed-position fullscreen wrapper
- `#gameCanvas`: Main rendering surface
- `#hud`: Heads-up display showing health, bombs, weapon, score
- `#vignette`: Damage/critical hit visual effect
- `#flash`: Screen flash effects

## Development Guidelines

### Code Style
- Use clear, descriptive variable names
- Keep functions focused on a single responsibility
- Use comments for non-obvious logic only
- Maintain consistent indentation (2 spaces in the embedded code)

### Making Changes
1. **Locate the JavaScript section** in `index.html` (after `<script>` tag)
2. **Identify the relevant function or entity** (player, enemies, bullets, etc.)
3. **Make targeted changes** — avoid refactoring unrelated code
4. **Test in browser** before committing:
   - Verify visual appearance
   - Test player movement and shooting
   - Check enemy behavior
   - Test collision detection
   - Verify HUD updates

### Common Tasks

#### Adding a New Enemy Type
1. Add enemy factory function in the game script
2. Define sprite/appearance (using canvas drawing)
3. Set movement pattern and attack behavior
4. Add to enemy spawn list for appropriate levels
5. Test collision detection and scoring

#### Modifying Weapons
1. Find `player.weapon` configuration
2. Adjust bullet speed, spread, damage, or rate of fire
3. Update visual feedback (muzzle flash if applicable)
4. Test balance against enemies

#### Adjusting Game Balance
- Enemy health: Modify individual enemy object health values
- Player damage: Adjust bullet damage values
- Spawning rates: Modify enemy spawn functions
- Difficulty scaling: Adjust per-level multipliers

### Testing Checklist
- [ ] Game starts and displays menu
- [ ] Player can move in all directions
- [ ] Shooting works and bullets behave correctly
- [ ] Enemies spawn and move as expected
- [ ] Collision detection works (player hits, enemy hits)
- [ ] Health/bombs display correctly
- [ ] Score updates properly
- [ ] Level progression works
- [ ] Boss fight triggers at correct level
- [ ] Game-over screen appears on death
- [ ] Responsive design works on mobile/tablet
- [ ] No console errors

## Git Workflow

### Branch Policy
- Develop all changes on the designated branch: `claude/inspiring-planck-4oydpw`
- Always commit changes with descriptive messages

### Commit Message Format
```
<type>: <description>

[optional body with details]

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
Claude-Session: <session-url>
```

**Types:**
- `feat:` New game feature or mechanic
- `fix:` Bug fix or gameplay adjustment
- `balance:` Difficulty or gameplay tuning
- `ui:` HUD or visual changes
- `perf:` Performance optimization
- `refactor:` Code organization (avoid unnecessary changes)
- `docs:` Documentation updates

### Example Commits
```
feat: add power-up system for rapid fire
fix: prevent player from moving off-screen
balance: increase boss health by 25%
ui: improve weapon indicator visibility
```

## Performance Notes

- Canvas rendering is CPU-based; optimize entity counts for lower-end devices
- Particle effects (explosions) can impact performance with many active entities
- Touch event handling requires `preventDefault()` to avoid scrolling
- CSS `pointer-events: none` prevents canvas from interfering with mouse events

## Responsive Design

- Game uses `viewport-fit=cover` for notched devices
- Canvas scales with viewport while maintaining aspect ratio
- HUD uses `clamp()` for responsive font sizing
- Touch-action set to `none` prevents browser gestures

## File Organization
```
index.html — Complete game (HTML + CSS + JS)
README.md  — Project metadata
Claude.md  — This file
```

## No External Dependencies
The game uses only vanilla JavaScript and HTML5 Canvas—no frameworks or libraries required. This keeps it lightweight and easy to modify.

## Future Enhancement Ideas
- Additional enemy types and bosses
- New weapon types
- Power-up system
- High score persistence (localStorage)
- Sound effects and music
- Particle effects library
- Mobile app wrapper

## Quick Reference

### Canvas Coordinate System
- Origin (0,0) at top-left
- X increases rightward
- Y increases downward
- Use `gameWidth` and `gameHeight` for boundary checks

### Event Listeners
- `keydown`/`keyup`: Keyboard controls
- `mousemove`/`pointerdown`/`pointerup`: Mouse and touch input
- `touchstart`/`touchmove`/`touchend`: Touch-specific (if separate handling needed)

### Rendering Loop
- Called via `requestAnimationFrame()` for smooth 60fps gameplay
- Updates game state
- Draws all entities to canvas
- Updates HUD display

---

**Last Updated:** 2026-09-30
**Project:** Sky Fighter — Raiden-style Shoot-'em-Up
