# Sky Fighter - Claude Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely with HTML5 Canvas and vanilla JavaScript. The game features a player-controlled aircraft, weapon switching mechanics, smart bombs, enemy waves, and boss encounters.

**Repository:** `ubban/google_4`  
**Language:** JavaScript (HTML5 Canvas)  
**Type:** Single-file web game  
**Status:** Active development

## Project Structure

```
Google_4/
├── index.html          # Complete game implementation (all code in one file)
├── README.md          # Basic readme
└── .git/              # Version control
```

The entire game logic is contained within `index.html`, including:
- HTML structure
- CSS styling
- JavaScript game engine and logic

## Game Architecture

### Core Systems

1. **Game Engine**
   - Canvas-based rendering at 60 FPS
   - Physics system for projectiles and collision detection
   - Wave/spawning system for enemies
   - Boss battle system

2. **Player Ship**
   - Keyboard and mouse/touch control support
   - Health/damage system
   - Weapon inventory with switching mechanics
   - Smart bomb mechanic with cooldown

3. **Weapon System**
   - Multiple weapon types (bullets, spread shots, lasers, etc.)
   - Weapon switching with keys
   - Projectile management and pooling

4. **Enemy System**
   - Enemy wave generation
   - Boss entities with special patterns
   - Collision detection and damage

5. **UI System**
   - HUD displaying score, health, weapon, and stats
   - Game state screens (title, pause, game over)
   - Touch/mobile control UI elements

### Game States
- **Title Screen** - Menu and start
- **Playing** - Active gameplay
- **Paused** - Pause menu
- **Game Over** - End state with score display
- **Boss Battle** - Special boss encounter logic

## Development Guidelines

### Code Organization

The single HTML file uses a modular approach within JavaScript:
- Game state management at the top level
- Entity classes for Player, Enemy, Boss, Projectile
- Separate update and render loops
- Event listeners for input handling

### Key Variables and Functions to Know

**Game State:**
- `gameState` - Current game state (TITLE, PLAYING, PAUSED, GAME_OVER, BOSS)
- `score`, `wave`, `health` - Player progression metrics
- `enemies[]`, `projectiles[]` - Active entities

**Main Loop:**
- `gameLoop()` - Main animation frame loop
- `update()` - Update game logic
- `render()` - Draw everything to canvas

**Input Handling:**
- Keyboard: W/A/S/D for movement, SPACE for weapon switch, B for smart bomb
- Mouse: Position-based aiming, click for shoot
- Touch: On-screen controls for mobile

### Development Workflow

1. **Local Testing:** Open `index.html` in a browser
2. **Editing:** Modify code directly in `index.html`
3. **Commits:** Use descriptive commit messages for gameplay changes
4. **Branch:** Work on `claude/inspiring-planck-nmvok3` branch

### Performance Considerations

- Canvas rendering is the main performance bottleneck
- Projectile and enemy pooling helps reduce GC pressure
- Collision detection is O(n²) - monitor when many entities are active
- Consider using RequestAnimationFrame for smooth 60 FPS (already implemented)

### Common Tasks

#### Adding a New Weapon Type
1. Define weapon stats in the weapon system constants
2. Add rendering logic in the weapon draw section
3. Update weapon switching logic to include the new type
4. Test weapon balance and visuals

#### Adding Enemy Waves
1. Modify the wave spawning logic
2. Define enemy patterns and spawn positions
3. Update wave progression and difficulty scaling
4. Test enemy AI and bullet patterns

#### Adjusting Game Balance
- Player health/damage taken values
- Weapon damage and fire rates
- Enemy health and spawn rates
- Score multipliers and progression

## Code Style

- Use camelCase for variables and functions
- Use SCREAMING_SNAKE_CASE for constants
- Keep functions focused and single-responsibility
- Comment complex physics or algorithms
- Maintain consistent indentation (2 spaces)

## Git Workflow

- **Branch:** Always develop on `claude/inspiring-planck-nmvok3`
- **Commits:** Write clear messages describing what changed and why
- **Push:** Push to the designated branch when work is complete
- **No PRs:** Unless explicitly requested

## Testing

- Manual testing in browser (Chrome, Firefox, Safari)
- Mobile/touch testing on actual devices when possible
- Performance profiling in DevTools if frame rate issues occur
- Regression testing for existing features after changes

## Known Issues & TODOs

(Update as issues are discovered)

## Resources

- HTML5 Canvas API: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API
- Game Development Patterns: Standard entity-component patterns
- Raiden Game Inspiration: Classic vertical shmup mechanics

## Contact & Attribution

Maintained by the Claude Code development agent.  
Current session: https://claude.ai/code/session_011oXitYr9TjbYGkWuEBRQim

---

**Last Updated:** 2026-10-14
