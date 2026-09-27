# Claude.md — Google_4 / Sky Fighter

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented in vanilla HTML5 and JavaScript. The game runs in a single `index.html` file with embedded styles and JavaScript, requiring no build tools or external dependencies beyond the browser.

- **Type**: Single-file web game
- **Technology**: HTML5 Canvas, vanilla JavaScript (ES6+)
- **Target**: Modern web browsers (Chrome, Firefox, Safari, Edge)
- **Platforms**: Desktop and mobile (touch-optimized)

## Game Features

- Vertical scroll shooter with stage-based progression
- Player ship with weapon system (cycling through weapon types)
- Enemy spawning and AI patterns
- Boss encounters at stage end
- Smart bomb mechanic for clearing screen
- Score/multiplier system
- Health/lives system
- Responsive HUD with stats display
- Mobile touch controls and desktop keyboard/mouse controls

## Development Workflow

### Branch Strategy

All development occurs on the designated feature branch: `claude/inspiring-planck-n0hmbe`

- Make all changes on this branch
- Commit with clear, descriptive messages
- Push changes with `git push -u origin <branch-name>`
- Do NOT push to other branches without explicit permission

### Running the Game

1. Open `index.html` in a modern web browser
2. Game starts automatically on page load
3. Use keyboard/mouse on desktop: arrow keys to move, spacebar to shoot
4. Use touch on mobile: tap to move player, hold to auto-aim
5. Press 'P' to pause/resume

### File Structure

```
Google_4/
├── index.html          # Main game file (HTML + CSS + JavaScript)
├── README.md           # Brief project info
└── Claude.md           # This file
```

## Code Organization (in index.html)

The single HTML file contains three main sections:

1. **Head Section**
   - Meta tags (charset, viewport, theme color)
   - Inline CSS for styling and animations
   - SVG favicon definition

2. **Body Section**
   - Game container with canvas element
   - HUD overlay for stats display
   - Vignette and flash effect layers

3. **Script Section**
   - Game engine and core logic
   - Canvas rendering pipeline
   - Input handling (keyboard/mouse/touch)
   - Entity system (player, enemies, bullets, explosions)
   - Stage/level progression
   - Audio system (if implemented)

## Coding Conventions

### Style Guidelines

- Use ES6+ syntax (const/let, arrow functions, destructuring)
- Consistent 2-space indentation
- Clear variable/function naming (camelCase for variables/methods, PascalCase for classes)
- Inline comments only when logic is non-obvious
- No external dependencies or frameworks

### Canvas Rendering

- Use a single `requestAnimationFrame` loop for frame updates
- Render order: background → entities → effects → HUD
- Clear canvas each frame with appropriate background color
- Use `canvas.getContext('2d')` for 2D rendering

### Entity System

- Entities tracked in arrays (player, enemies, projectiles, explosions)
- Each entity has position, velocity, and health properties
- Update and render methods called each frame
- Collision detection via distance or AABB checks
- Dead entities removed from arrays after render

### Input Handling

- Keyboard: arrow keys for movement, spacebar for shoot, 'P' for pause
- Mouse: movement follows cursor, click to shoot
- Touch: tap position sets target, auto-aim at tap point
- Use event listeners for input capture

### Performance

- Minimize DOM manipulation (single canvas element)
- Pool objects where possible (bullets, explosions)
- Limit simultaneous entities (enemies, projectiles)
- Use requestAnimationFrame, not setInterval/setTimeout for rendering

## Commit Message Guidelines

Write clear, descriptive commit messages following this format:
```
<type>: <subject>

<optional body explaining why, not what>
```

Types:
- `feat`: New game feature (weapon type, enemy type, stage)
- `fix`: Bug fix
- `perf`: Performance optimization
- `refactor`: Code restructuring without behavior change
- `style`: CSS/visual changes only
- `docs`: Documentation updates (Claude.md, comments)

Example:
```
feat: add triple-shot weapon with spread pattern

Implements a new weapon that fires three bullets in a spread formation,
allowing wider coverage at the cost of reduced forward penetration.
```

## Testing

- Manual testing in browser (open index.html)
- Test on multiple browsers and screen sizes
- Test keyboard, mouse, and touch controls
- Test game states: normal, paused, game over, stage complete
- Monitor browser console for errors

## Performance Targets

- Target 60 FPS on modern devices
- Keep canvas render time under 16ms per frame
- Maintain reasonable memory usage (no memory leaks)
- Support devices down to mid-range mobile phones

## Git Workflow

```bash
# Ensure you're on the correct branch
git checkout claude/inspiring-planck-n0hmbe

# Make changes to index.html
# Test changes in browser

# Stage and commit
git add index.html
git commit -m "feat: description of change"

# Push to remote
git push -u origin claude/inspiring-planck-n0hmbe
```

## Common Tasks

### Adding a New Enemy Type
1. Define enemy properties (speed, health, spawn pattern)
2. Add to enemy spawning logic in stage definition
3. Implement collision detection with player bullets
4. Test enemy movement and interaction

### Implementing a New Weapon
1. Define weapon properties (damage, fire rate, projectile speed)
2. Add to weapon cycling system
3. Implement firing pattern (single, spread, homing, etc.)
4. Test effectiveness and balance

### Creating a New Stage
1. Define stage progression (enemy waves, boss)
2. Implement spawn patterns and timing
3. Design boss AI and attack patterns
4. Test difficulty balance

## Known Limitations & Considerations

- Single-file design limits modularity but ensures portability
- No persistent storage (scores not saved between sessions)
- Audio implementation may vary by browser and device
- Mobile performance depends on device capabilities
- Touch controls optimized for portrait orientation

## Browser Compatibility

- Modern browsers: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- Requires HTML5 Canvas and ES6 JavaScript support
- Fallback graceful degradation if not available

## Attribution

Development tracked in Claude Code sessions. See git history for authorship details.
