# Sky Fighter - Claude Code Guide

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas and JavaScript. This document guides Claude in understanding and extending the codebase.

## Project Overview

Sky Fighter is a single-file browser game featuring:
- **Gameplay**: Vertical scrolling shooter with weapon switching, smart bombs, and boss encounters
- **Technology**: Pure HTML5 Canvas (no frameworks), vanilla JavaScript
- **Platform**: Web browser with responsive design and touch support

## Project Structure

```
Google_4/
├── index.html          # Single-file game (HTML, CSS, and inline JavaScript)
├── README.md           # Basic project info
└── .git/               # Version control
```

The entire game is contained in `index.html`:
- **HTML structure**: Canvas container with HUD elements (hearts, bombs, weapon status, multiplier)
- **CSS styling**: Responsive design with `clamp()` for scaling, dark theme, animations
- **JavaScript**: Game loop, entity management, physics, rendering

## Key Game Concepts

### Game Loop
- `animate()`: Main animation frame loop that updates and renders game state
- Fixed 60 FPS target using `requestAnimationFrame`

### Entities
- **Player**: `player` object - position, health, weapon, bombs
- **Bullets**: Array of active bullet objects with position, velocity, damage
- **Enemies**: Array of active enemies with health, bullets, movement patterns
- **Effects**: Explosions, hits, visual feedback

### Weapon System
- Multiple weapon types (Vulcan, Spread, Laser, Homing)
- Weapon switching and ammunition
- Fire rate and damage per weapon

### Enemy System
- Basic enemies with movement patterns
- Boss enemies with health bars and special patterns
- Different bullet patterns for variety

### Visual Effects
- Screen vignette on damage (red flash around edges)
- White flash on hit/bomb detonation
- Smooth transitions and animations

## Development Guidelines

### Code Style
- Use vanilla JavaScript (no transpilers needed)
- Keep the file size reasonable for web delivery
- Favor performance for 60 FPS gameplay
- Use canvas for all rendering (no DOM animations during gameplay)

### Adding Features
1. **New Weapons**: Add weapon definition to weapon system, implement firing logic
2. **New Enemies**: Define enemy patterns and bullet behaviors
3. **Game Mechanics**: Modify player update loop, collision detection
4. **Visual Effects**: Add to canvas rendering pipeline before player/enemies drawn

### Performance Considerations
- Minimize object allocations in game loop
- Reuse objects when possible (object pooling)
- Cache canvas context and computed values
- Profile with browser DevTools if frame rate drops

### Responsive Design
- Uses CSS `clamp()` for fluid scaling across devices
- Touch controls for mobile (pointer events)
- Fits to viewport with max scaling

## How Claude Can Help

### Code Improvements
- **Bug fixes**: Debug gameplay issues, collision problems, visual glitches
- **Performance**: Optimize entity management, rendering pipeline
- **Game balance**: Adjust weapon damage, enemy health, difficulty curves
- **UI enhancements**: Improve HUD visibility, add new stat displays

### Feature Development
- **New weapons**: Design and implement firing patterns and behaviors
- **New enemies**: Create unique movement patterns and attack styles
- **Boss encounters**: Design complex multi-phase boss fights
- **Power-ups**: Add collectible items that affect gameplay

### Game Design
- **Difficulty balancing**: Tune spawning rates, enemy health, weapon power
- **Game feel**: Improve juice (screenshake, feedback, sounds) - currently limited to visual effects
- **Progression**: Add stages, difficulty modes, scoring systems

### Refactoring
- **Code organization**: If file becomes too large, help modularize while keeping single-file structure
- **Readability**: Improve variable names, add comments for complex logic
- **Structure**: Improve game state management and entity systems

## Common Tasks

### Testing Changes
1. Open `index.html` in a browser
2. Test gameplay mechanics by hand
3. Check responsive behavior at different viewport sizes
4. Verify touch controls on mobile devices

### Git Workflow
- Commit gameplay changes with clear messages describing what changed
- Include before/after descriptions for balance changes
- Push changes to `claude/inspiring-planck-n9fzsm` branch

### Debugging
- Use browser DevTools (F12) for console logs and stepping
- Check for JavaScript errors in console
- Monitor canvas rendering performance in DevTools

## Project Goals

Current focus areas:
- Core gameplay polish and balance
- Visual feedback and game feel
- Responsive design across devices
- Performance optimization

## Dependencies

- **None** - Pure vanilla HTML5 Canvas and JavaScript
- **Browser APIs Used**: Canvas 2D Context, RequestAnimationFrame, Pointer Events, Touch Events

---

Last updated: 2026-10-04
