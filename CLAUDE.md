# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built with vanilla HTML5 Canvas and JavaScript. The game features weapon switching, smart bombs, and stage bosses.

- **Type**: Web-based game (single-player)
- **Technology**: HTML5 Canvas + Vanilla JavaScript
- **Format**: Single-file implementation (index.html)
- **Purpose**: Classic arcade-style vertical shmup gameplay

## Repository Structure

```
Google_4/
├── index.html          # Complete game implementation (HTML + CSS + JavaScript)
├── README.md           # Project description
└── CLAUDE.md          # This file - codebase documentation
```

## Key Game Features

### Gameplay Mechanics
- **Player Aircraft**: Controllable ship with movement and firing
- **Weapon Systems**: Multiple weapon types with switching capability
- **Smart Bombs**: Special attacks with cooldown mechanics
- **Enemy Waves**: Procedural enemy spawning with difficulty progression
- **Boss Encounters**: Stage-specific boss battles
- **Scoring System**: Points for defeated enemies and bosses

### Visual Style
- **Color Scheme**: Dark blue background with gradient sky effect
- **Resolution**: Responsive canvas that scales to viewport
- **HUD**: Score, lives, health, and weapon indicator display
- **Effects**: Basic particle effects for explosions and impacts

## Code Architecture

The game uses a single-file architecture embedded in `index.html`:

### Structure Sections
1. **HTML Structure** (lines ~1-100)
   - Canvas container for game rendering
   - HUD elements for display information
   - Game state containers

2. **CSS Styling** (lines ~100-300)
   - Layout and positioning
   - Theme colors and visual styling
   - Responsive design rules
   - Safe area insets for mobile browsers

3. **JavaScript Logic** (lines ~300+)
   - **Game Loop**: Canvas rendering and update cycle
   - **Input Handling**: Keyboard and touch controls
   - **Entity Systems**: Player, enemies, bullets, effects
   - **Physics**: Collision detection, movement calculation
   - **State Management**: Game mode, level, scoring
   - **Audio**: Sound effects (via Web Audio API if implemented)

## Development Guidelines

### Adding New Features

#### New Weapon Type
1. Define weapon properties (damage, fire rate, sprite)
2. Add weapon ID to player's weapon list
3. Implement firing logic in the weapon system
4. Add HUD indicator for weapon selection

#### New Enemy Type
1. Create enemy class with sprite, movement pattern
2. Define health and damage values
3. Add to appropriate spawn wave
4. Update collision detection if needed

#### New Boss
1. Design boss sprite and attacks
2. Implement movement and attack patterns
3. Create boss defeat condition and transition logic
4. Add to stage progression

### Performance Considerations
- Keep canvas size reasonable (avoid very large dimensions)
- Batch rendering operations when possible
- Manage entity pooling to reduce garbage collection
- Be mindful of particle effect count
- Profile frame rate on target devices

### Testing
- Test on mobile browsers (iOS Safari, Chrome Mobile)
- Verify touch controls work properly
- Check performance on lower-end devices
- Test boss difficulty and balance

### Code Style
- Use descriptive variable names
- Keep functions focused and modular
- Add comments for complex logic
- Avoid global state where possible
- Use constants for configuration values

## Build & Deployment

No build step required. The game is a single HTML file that can be:
- Opened directly in a browser (`file://` protocol)
- Served via HTTP/HTTPS web server
- Deployed to any static hosting service
- Embedded in other pages via iframe

### Performance Tips
- Minify JavaScript/CSS for production
- Optimize image assets if using external sprites
- Consider service worker for offline gameplay
- Use browser caching headers

## Git Workflow

- Work on feature branches (e.g., `claude/inspiring-planck-x0jldn`)
- Commit changes with clear, descriptive messages
- Push completed features back to the repository
- Use semantic commit messages:
  - `feat: add new feature`
  - `fix: bug fix description`
  - `refactor: code improvement`
  - `perf: performance optimization`

## Common Tasks

### Testing the Game Locally
```bash
# Open in browser (requires local server due to potential CORS issues)
python3 -m http.server 8000
# Then visit: http://localhost:8000
```

### Checking Browser Compatibility
- Target: Chrome/Edge 90+, Firefox 88+, Safari 14+
- Mobile: iOS Safari 14+, Chrome Android latest

### Debugging Tips
- Use browser DevTools Console for JavaScript errors
- Check Network tab for any resource loading issues
- Use Performance profiler to identify bottlenecks
- Test with DevTools device emulation for mobile

## Future Enhancements

Potential areas for expansion:
- [ ] Mobile touch controls refinement
- [ ] Sound/music integration
- [ ] Local high score persistence (localStorage)
- [ ] Difficulty levels selection
- [ ] Additional weapon types
- [ ] More sophisticated enemy AI
- [ ] Boss rush mode
- [ ] Leaderboard system (with backend)
- [ ] Settings menu (volume, controls mapping)
- [ ] Multiple stages/levels

## Attribution & Session Context

**Last Updated**: 2026-09-24  
**Maintained By**: Claude Code Agent  
**Session**: claude/inspiring-planck-x0jldn

This documentation is automatically maintained to reflect the current state of the Sky Fighter codebase.
