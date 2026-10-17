# Claude.md - Sky Fighter Project

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla JavaScript and HTML5 Canvas. Players control a fighter jet, switch between weapons, drop smart bombs, and battle stage bosses.

**Repository**: ubban/Google_4  
**Tech Stack**: HTML5, CSS3, Vanilla JavaScript (no frameworks)  
**Type**: Browser-based game  

## Repository Structure

```
Google_4/
├── README.md          # Minimal project metadata
├── index.html         # Single-page game application (HTML, CSS, JS combined)
└── .git/              # Version control
```

## Key Technologies

- **Canvas API**: Rendering graphics and game entities
- **Web Audio API**: Sound effects and music (if implemented)
- **Touch/Mouse Events**: Player input handling
- **ES6+ JavaScript**: Game logic and state management

## Game Features

- Vertical scrolling shoot-'em-up gameplay
- Player-controlled fighter jet with crosshair cursor
- Multiple weapon systems with switching capability
- Smart bombs for area damage
- Stage-based boss battles
- HUD displaying stats (score, health, weapons, etc.)
- Touch and mouse input support
- Responsive scaling to fit different screen sizes
- Dark space/sky themed visuals

## Development Workflow

### Running the Game

1. **Local Server** (recommended):
   ```bash
   # Using Python 3
   python3 -m http.server 8000
   # Then open: http://localhost:8000
   
   # Or using Node.js
   npx http-server
   ```

2. **Direct File** (not recommended - may have CORS issues):
   - Simply open `index.html` in a browser

### Code Organization (within index.html)

The single HTML file contains three sections:
1. **HTML Structure** (`<html>`, `<head>`, `<body>` tags)
2. **CSS Styles** (`<style>` block) - Game UI, canvas styling, responsive design
3. **JavaScript** (`<script>` block) - Game engine, entity management, event handling

### Development Guidelines

1. **Single-File Architecture**:
   - Keep all game code in `index.html` for simplicity
   - Organize JavaScript into clear, logical sections with comments
   - Group related functionality (input, collision, rendering, etc.)

2. **Code Style**:
   - Use descriptive variable and function names
   - Comment WHY, not WHAT (readers can understand the code)
   - Keep functions focused and under ~50 lines when possible
   - No frameworks or bundlers required

3. **Performance Considerations**:
   - Cache DOM elements (canvas, container)
   - Use requestAnimationFrame for smooth animation
   - Minimize allocations in the game loop
   - Batch canvas operations when possible

4. **Testing Changes**:
   - Refresh browser after changes (Cmd/Ctrl+Shift+R for hard refresh)
   - Test on multiple screen sizes (phone, tablet, desktop)
   - Verify keyboard and touch controls both work
   - Check HUD visibility on different viewports

## Git Workflow

### Branch Strategy

- **Development Branch**: `claude/inspiring-planck-kxvohi`
- **Main Branch**: `main` (deployment/stable builds)

### Commit Practices

1. Keep commits focused on single features or fixes
2. Write clear, descriptive commit messages:
   - Example: "Add smart bomb mechanic with area damage"
   - Example: "Fix weapon switching responsiveness on touch"
3. Include the Claude Code attribution in commit messages (already configured)

### Push Protocol

```bash
# Push to development branch
git push -u origin claude/inspiring-planck-kxvohi

# Only push to main when code is tested and stable
```

## Common Tasks

### Adding a New Weapon

1. Define weapon properties (damage, fire rate, bullet pattern)
2. Add weapon selection UI to HUD
3. Implement bullet generation in the game loop
4. Add collision detection for weapon hits
5. Test weapon switching and firing

### Modifying Game Balance

- Player health/speed: Look for numeric constants near player initialization
- Enemy stats: Enemy spawn configuration and behavior parameters
- Weapon properties: Individual weapon definition sections
- Always test impact on gameplay difficulty

### Debugging

```javascript
// Browser console for debugging
console.log(gameState); // Check game state
console.log(player);    // Check player properties
debugger;               // Set breakpoints (F12 in DevTools)
```

## Known Conventions

- Canvas coordinate system: (0,0) at top-left
- Positive X: right, Positive Y: down
- Player sprite centered on screen
- Enemies spawn at top of screen, move downward
- Z-index for UI: HUD at z-index 5 to appear above canvas

## Performance Targets

- 60 FPS target (requestAnimationFrame)
- Canvas size: Responsive scaling (max-width: 100%, max-height: 100%)
- Asset loading: All assets embedded or generated inline (no external dependencies)

## No External Dependencies

This is a pure vanilla JavaScript project with:
- No npm packages
- No build tools required
- No module bundlers
- Single HTML file deployment

## Contact & Maintenance

This project is maintained as part of the ubban/Google_4 repository. Updates should follow the git workflow outlined above and be pushed to the designated development branch.

---

*Last updated: 2026-10-17*
