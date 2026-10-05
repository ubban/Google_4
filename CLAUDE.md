# Sky Fighter - Claude Development Guide

## Project Overview
**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with HTML5 Canvas. Players control a fighter jet, switch weapons, deploy smart bombs, and battle stage bosses in a fast-paced arcade experience.

**Repository:** `ubban/google_4`  
**Primary Branch:** `main`  
**Development Branch:** `claude/inspiring-planck-21oguh`

---

## Project Structure

```
Google_4/
├── index.html          # Main game file (HTML + CSS + JavaScript)
├── README.md           # Project readme with timestamp
├── CLAUDE.md           # This file - development guide
└── .git/               # Git repository
```

### Current Implementation
- **Single-file application:** All game code is contained in `index.html`
- **Canvas-based rendering:** Uses HTML5 Canvas API for 2D graphics
- **Responsive design:** Supports multiple viewport sizes with viewport-fit=cover
- **Dark theme:** Navy/blue color scheme (#0b1d3a, #05122b)

---

## Technology Stack

- **HTML5 Canvas API** - 2D graphics rendering
- **Vanilla JavaScript** - Game logic, input handling, collision detection
- **CSS3** - Layout and styling with viewport units (clamp, vw/vh)
- **Responsive Web Design** - Mobile-first, safe-area-inset support

---

## Game Mechanics

### Core Features
- **Player Character:** Controllable fighter jet
- **Weapon System:** Multiple weapon types with switching capability
- **Special Attacks:** Smart bombs with tactical deployment
- **Boss Battles:** Stage-based boss encounters
- **HUD Display:** Real-time stats including score, lives, weapon status

### Game Elements
- **Player Aircraft:** Center-screen positioning with crosshair cursor
- **Enemies:** Various enemy types with attack patterns
- **Projectiles:** Bullets, bombs, and enemy fire
- **Collision Detection:** Player-enemy, projectile-enemy, projectile-player interactions
- **Scoring System:** Points for enemy defeats and stage progression

---

## Development Guidelines

### Code Style
- Use clear, descriptive variable and function names
- Keep functions focused on single responsibilities
- Document complex game mechanics with inline comments
- Use consistent indentation (2 or 4 spaces)

### Game Loop Structure
The application follows a typical game loop pattern:
1. **Input handling** - Capture keyboard/mouse/touch input
2. **Update** - Update game state, physics, and entity positions
3. **Render** - Draw all game elements to canvas

### Adding New Features
1. **New Weapon Type:** Update weapon system logic, add weapon sprites/effects
2. **New Enemy Type:** Create enemy class, define movement patterns and attacks
3. **New Stage/Boss:** Add boss definition, attack patterns, and rewards
4. **Visual Effects:** Enhance with particle systems, animations, screen shakes

### Input Handling
- **Keyboard:** Arrow keys or WASD for movement
- **Mouse/Touch:** Aim and fire with crosshair
- **Special Keys:** Space for bombs, key for weapon switching

---

## Building and Running

### Local Development
1. Open `index.html` directly in a modern web browser
2. No build step required - vanilla JavaScript
3. Game runs immediately with full functionality

### Browser Compatibility
- **Required:** HTML5 Canvas support
- **Recommended:** Modern browsers (Chrome, Firefox, Safari, Edge)
- **Mobile:** Touch input supported for mobile browsers

### Testing
- **Manual testing:** Play through all levels and boss battles
- **Edge cases:** Test with different viewport sizes, input methods
- **Performance:** Monitor FPS and CPU usage during intensive scenes

---

## Common Development Workflows

### Adding a New Game Feature
1. Identify the game system affected (input, physics, rendering, etc.)
2. Implement the feature in the appropriate section of the JavaScript
3. Add HUD indicators if user feedback is needed
4. Test across different viewport sizes
5. Commit with descriptive message

### Debugging Gameplay Issues
- Use browser DevTools console for logging
- Check collision detection and entity positioning
- Verify animation frame timing and game loop synchronization
- Test with different input methods (keyboard, mouse, touch)

### Performance Optimization
- Profile rendering with DevTools Performance tab
- Minimize canvas operations and redraws
- Cache frequently-used calculations
- Consider requestAnimationFrame optimization

---

## Git Workflow

### Committing Changes
```bash
git add <files>
git commit -m "Brief description of changes"
git push -u origin claude/inspiring-planck-21oguh
```

### Commit Message Format
- Clear, descriptive summary (50 chars max)
- Details in commit body if needed
- Reference issues/features where applicable
- Include Co-Authored-By attribution as needed

### Branch Management
- Work on designated development branch: `claude/inspiring-planck-21oguh`
- Never push directly to `main` without approval
- Merge back to `main` via pull request when complete

---

## Architecture Notes

### Game State
- Player state: position, health, current weapon, ammunition
- Enemy state: positions, health, attack cooldowns
- Game state: score, level, wave counter, boss phase
- Camera state: viewport and effect variables

### Rendering Pipeline
1. Clear canvas with background gradient
2. Draw background and parallax effects
3. Render all game entities (player, enemies, projectiles)
4. Draw particle effects and animations
5. Render HUD overlay with stats

### Collision System
- Axis-aligned bounding box (AABB) collisions
- Spatial partitioning for performance (if implemented)
- Collision callbacks for damage, powerups, etc.

---

## Future Enhancement Areas
- Multiple difficulty levels
- Score leaderboard system
- Additional weapon types and power-ups
- More enemy varieties and boss encounters
- Sound and music integration
- Mobile optimization improvements
- Save/progress system

---

## Resources and References
- [MDN Canvas API Documentation](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
- [Game Programming Patterns](https://gameprogrammingpatterns.com/)
- [HTML5 Gamedev Resources](https://html5gamedev.com/)

---

**Last Updated:** 2026-10-05  
**Maintained By:** Claude Code Agent  
**Current Version:** 1.0
