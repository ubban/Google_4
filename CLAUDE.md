# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file web application. The game features:
- Vertical scrolling shooter mechanics
- Weapon switching system
- Smart bomb power-ups
- Stage bosses
- Progressive difficulty
- Mobile-optimized responsive design
- Dark sci-fi theme with dynamic visuals

The entire game is contained in `index.html` (~59KB), making it self-contained and easy to deploy.

## Architecture

### Single-File Structure
- **index.html**: Complete game implementation
  - HTML5 structure
  - Embedded CSS for styling and animations
  - Inline JavaScript game engine
  - Canvas-based rendering system

### Game Engine Components

#### Core Systems
1. **Rendering Engine**: Uses HTML5 Canvas for game graphics and animations
2. **Game Loop**: Manages update and render cycles (typically 60 FPS target)
3. **Input Handling**: Touch and mouse support for cross-platform play
4. **Entity System**: Manages game objects (player, enemies, projectiles, effects)
5. **Collision Detection**: Spatial checks for combat interactions

#### Game Features
- **Player Entity**: The controllable sky fighter aircraft
- **Weapon System**: Multiple weapon types with switching mechanics
- **Enemy AI**: Procedural enemy patterns and behaviors
- **Boss Encounters**: Stage-specific boss entities with unique attack patterns
- **Particle Effects**: Visual feedback for collisions, explosions, and power-ups
- **HUD System**: Real-time display of score, health, bombs, and weapon status

### Visual Design
- **Theme**: Dark sci-fi with blue gradient sky
- **Color Scheme**: Blues (#0d2f5e, #1e6fd8), white text, accent golds (#ffd54f) and greens (#7CFC00)
- **Effects**: 
  - Vignette effect on critical health
  - Screen flash on hits/bombs
  - Heart beat animation for low health warning
  - Gradient background with parallax scrolling

### User Interface
- **HUD Elements** (positioned top-center):
  - Health hearts (❤️) with critical animation
  - Smart bombs remaining (💣)
  - Current weapon indicator
  - Score/multiplier display
  - Safe area padding for notch displays
- **Responsive Design**: Scales from mobile to desktop using `clamp()` for font sizes
- **Touch-Optimized**: Includes viewport settings for maximum display and no zoom

## Key Game Mechanics

### Player Controls
- **Movement**: Touch/mouse position relative to game area
- **Weapon Fire**: Automatic when gameplay is active
- **Weapon Switch**: Cycle through available weapons (design determines hotkey)
- **Smart Bomb**: Deploy limited-use screen-clearing attack

### Combat System
- **Health**: Player starts with 3-5 hearts; loses one per enemy/bullet hit
- **Weapons**: Different damage, fire rate, and spread patterns
- **Multiplier**: Kill combo system that increases score multiplier
- **Invincibility Frames**: Temporary protection after taking damage

### Enemy AI
- **Spawning**: Wave-based or progressive spawning tied to score milestones
- **Behaviors**: Direct attacks, spread patterns, spinning obstacles
- **Boss Patterns**: Multi-phase encounters with telegraphed attacks

### Progression
- **Stages**: Multiple distinct levels with visual themes
- **Difficulty Scaling**: Enemy count, speed, and attack patterns increase with score
- **Boss Gates**: Stage transitions locked behind boss defeats

## Development Guidelines

### Code Style
- **Naming**: Descriptive camelCase for variables and functions
- **Organization**: Logical grouping of related game systems
- **Comments**: Add explanatory notes for non-obvious game mechanics, AI patterns, or performance optimizations
- **Performance**: Minimize allocations in game loop; reuse objects where possible

### Adding Features

#### New Weapons
1. Define weapon properties (damage, fireRate, spread, visual)
2. Add firing logic to player update method
3. Register weapon in weapon system
4. Add HUD display update

#### New Enemy Types
1. Create entity object with position, velocity, health, behavior
2. Implement update method with movement and attack logic
3. Register in enemy spawn system
4. Add collision handler for player projectiles

#### New Visual Effects
1. Create particle or effect entity
2. Add to effects rendering layer
3. Tie to combat events (hit, explosion, power-up)
4. Set lifetime and fade logic

### Testing Guidelines
- **Manual Testing**: Open `index.html` in a modern browser and test:
  - Player movement responsiveness across different screen sizes
  - Weapon mechanics and switching
  - Enemy spawning and patterns
  - Collision detection accuracy
  - Boss encounters and win conditions
  - Mobile touch controls (test on actual devices if possible)
- **Responsive Design**: Test at various viewport sizes (mobile, tablet, desktop)
- **Performance**: Monitor frame rate (target 60 FPS) with DevTools Performance tab

## File Structure

```
Google_4/
├── index.html          # Complete game implementation
├── README.md           # Project introduction
├── CLAUDE.md           # This file - development documentation
└── .git/               # Git version control
```

## Running the Game

1. **Local Development**: Open `index.html` directly in a web browser
2. **HTTP Server** (recommended): Serve via `python -m http.server` or similar for network requests
3. **Mobile Testing**: Use device on same network or expose via tunnel
4. **Deployment**: Single-file design allows easy hosting on any static web server

## Common Tasks

### Starting Fresh on a Feature
1. Create a feature branch from main: `git checkout -b feature/weapon-name`
2. Edit the `index.html` file with the feature implementation
3. Test thoroughly in browser across device sizes
4. Commit with clear message describing the change
5. Push to feature branch and open PR for review

### Debugging Tips
- **DevTools Console**: Check for JavaScript errors and debug logs
- **Canvas Drawing**: Use browser DevTools to inspect rendered canvas state
- **Performance Profiling**: Use Performance tab to identify bottlenecks
- **Network Tab**: Verify asset loading (favicon, etc.)

## Deployment

The `index.html` file is deployment-ready. Deploy by:
1. Committing changes to main branch
2. Copying `index.html` to web server
3. Serving with any static web server (Apache, Nginx, GitHub Pages, etc.)
4. Optional: Minify CSS/JavaScript if concerned about size

## Performance Considerations

- **Canvas Rendering**: Optimize by reducing draw calls and reusing paths
- **Object Pooling**: Reuse enemy and projectile objects instead of creating new ones
- **Collision Optimization**: Use spatial partitioning for large numbers of entities
- **Animation**: Use CSS transforms and requestAnimationFrame for smooth 60 FPS
- **Memory**: Monitor entity count; clean up destroyed objects promptly

## Browser Compatibility

Target modern browsers with:
- ES6 support
- Canvas API
- Touch Events API
- CSS Grid/Flexbox
- CSS Transform animations

Tested on:
- Chrome/Chromium (latest)
- Firefox (latest)
- Safari (latest)
- Edge (latest)

## Future Enhancement Ideas

- [ ] Leaderboard system (local storage or backend)
- [ ] Multiple game modes (endless, time attack, boss rush)
- [ ] Sound effects and music system
- [ ] Difficulty settings
- [ ] Weapon customization/upgrades
- [ ] Multiple player characters with unique abilities
- [ ] Replay system
- [ ] Achievements/badges

---

**Last Updated**: 2026-10-11  
**Maintained by**: Claude Code Agent
