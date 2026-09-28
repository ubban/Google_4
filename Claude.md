# Claude.md - Sky Fighter Game Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with HTML5 Canvas and JavaScript. The game features:
- Fast-paced vertical scrolling gameplay
- Weapon switching mechanics
- Smart bomb power-ups
- Stage bosses
- Progressive difficulty

This is a single-file web application (`index.html`) that runs entirely in the browser.

## Project Structure

```
Google_4/
├── README.md          # Project metadata
├── index.html         # Main game file (all game logic, styles, and HTML)
└── Claude.md          # This file
```

## Game Features

- **Player Aircraft**: Control via mouse/touch with weapon switching
- **Weapon System**: Multiple weapon types with switch capability
- **Smart Bombs**: Screen-clearing power-ups with limited uses
- **Boss Encounters**: Stage bosses with unique attack patterns
- **Visual Effects**: Gradient backgrounds, smooth animations, particle effects
- **Responsive Design**: Works on desktop and mobile devices
- **Dark Theme**: Night-sky aesthetic with neon accents

## Code Architecture

The game is implemented as a single HTML file with:
1. **HTML Structure**: Canvas element and HUD overlay
2. **CSS Styling**: Game container, HUD display, responsive design
3. **JavaScript Game Engine**: 
   - Canvas rendering loop
   - Input handling (mouse/touch)
   - Game state management
   - Entity systems (player, enemies, projectiles, particles)
   - Collision detection
   - Score tracking

## How Claude Code Can Help

Claude Code is well-suited for working on this project:

### Code Analysis & Exploration
- Search for specific game mechanics or functions
- Understand the game flow and architecture
- Locate bug-related code sections
- Trace entity behavior and collision systems

### Development Tasks

#### Bug Fixes
- Fix gameplay issues (collision detection, movement, firing)
- Correct visual glitches or rendering problems
- Resolve physics/animation timing issues

#### Feature Implementation
- Add new weapon types
- Implement new enemy patterns
- Create additional boss encounters
- Add new visual effects or animations

#### Game Balance
- Adjust difficulty curves and enemy spawn rates
- Tune weapon damage and fire rates
- Balance boss health and attack patterns
- Modify score multipliers and bonuses

#### Optimization
- Improve canvas rendering performance
- Optimize collision detection algorithms
- Reduce memory usage for particle systems
- Improve input responsiveness

## Development Workflow

### 1. Testing Changes Locally
```bash
# The game runs in any modern browser
# Open index.html directly or serve via HTTP
python3 -m http.server 8000
# Then visit http://localhost:8000
```

### 2. Making Changes
- Edit the JavaScript, CSS, or HTML sections in `index.html`
- Test gameplay mechanics immediately in browser
- Use browser DevTools console for debugging

### 3. Git Workflow
```bash
# Current branch: claude/inspiring-planck-5bcdzy
git add index.html
git commit -m "Feature/fix description"
git push -u origin claude/inspiring-planck-5bcdzy
```

## Common Development Tasks

### Testing the Game
1. Open `index.html` in a web browser
2. Test gameplay:
   - Player movement (mouse/touch)
   - Weapon firing and switching
   - Collision detection
   - Score tracking
   - Boss encounters
3. Check visual quality:
   - Smooth animations
   - No rendering artifacts
   - Proper HUD display on various screen sizes

### Debugging
- Use browser DevTools (F12 or right-click → Inspect)
- Check Console for JavaScript errors
- Use Network tab to verify no resource loading issues
- Use Performance tab to monitor rendering frame rate

### Common Issues & Solutions
- **Poor performance**: Check if too many particles/entities, optimize loops
- **Input not responsive**: Verify event listeners and input handling code
- **Visual glitches**: Check canvas clear calls and draw order
- **Boss not spawning**: Verify spawn logic and stage progression

## Game Balance Parameters

Key values to adjust for game balance (typically in JavaScript):
- Enemy spawn rates and patterns
- Enemy health and damage values
- Player weapon damage and fire rates
- Bomb cooldown and availability
- Score multipliers and bonuses
- Difficulty progression curve

## Browser Compatibility

The game works in all modern browsers supporting:
- HTML5 Canvas API
- ES6 JavaScript
- Touch Events API
- Mouse Events API

Tested/supported:
- Chrome/Chromium 90+
- Firefox 88+
- Safari 14+
- Edge 90+

## Performance Considerations

- Canvas rendering at 60 FPS target
- Entity pooling for bullets and particles
- Efficient collision detection (spatial hashing or grid-based)
- Minimal DOM manipulation

## Future Development Ideas

- Additional weapon types and upgrades
- Multiplayer/co-op mode
- Sound effects and music integration
- Leaderboard system
- Mobile app packaging (Cordova/Capacitor)
- Advanced visual effects (shaders)
- Story mode with cinematics

## Getting Help with Claude Code

When asking Claude Code for help with this project:
1. **Be specific**: "Fix collision detection for boss bullets" vs. "The game isn't working"
2. **Describe behavior**: What should happen vs. what actually happens
3. **Provide context**: Which feature/mechanic is affected
4. **Test locally first**: Verify the issue before reporting

Claude Code can:
- Analyze code and identify problems
- Suggest implementation approaches
- Write and test code changes
- Debug gameplay issues
- Optimize performance bottlenecks

---

**Last Updated**: 2026-09-28
**Project Type**: Web-based JavaScript Game
**Main File**: index.html (~1600 lines of combined HTML/CSS/JS)
