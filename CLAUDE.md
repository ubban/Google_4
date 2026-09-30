# Claude.md — Sky Fighter Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 web application. The game features:

- Dynamic vertical scrolling gameplay
- Player ship with multiple weapon systems (vulcan, laser, homing missiles)
- Smart bomb mechanic
- Boss encounters and stage progression
- Mobile-responsive design with touch controls
- Canvas-based 2D graphics

**Repository**: `ubban/google_4`  
**Primary File**: `index.html` (single file containing all HTML, CSS, and JavaScript)  
**Development Branch**: `claude/inspiring-planck-qryip8`

## Project Structure

```
Google_4/
├── index.html          # Single-file game implementation (all code)
├── README.md           # Project description
└── CLAUDE.md          # This file
```

## Code Architecture

The game is implemented as a single HTML file with inline CSS and JavaScript. The main sections are:

### 1. **HTML Structure** (lines 1-500+)
- Canvas container for rendering
- HUD (heads-up display) for game statistics
- Vignette and flash effects for visual feedback

### 2. **CSS Styling** (lines 10-150+)
- Dark theme with blue gradient backgrounds
- Responsive design (clamp functions for scaling)
- Animations for HUD effects (heartBeat, vignettePulse, flash effects)
- Mobile-friendly viewport settings

### 3. **JavaScript Game Logic** (main bulk of file)

#### Game State
- Canvas and rendering context
- Player ship state (position, health, weapons, bombs)
- Enemies, bullets, and effects
- Score and multiplier tracking

#### Core Systems
- **Game Loop**: `requestAnimationFrame`-based update/render cycle
- **Input Handling**: Keyboard and touch input for player control
- **Entity Management**: Spawning and updating enemies, projectiles, effects
- **Collision Detection**: Simple AABB collision checks
- **Boss Encounters**: Special boss entities with unique patterns
- **Weapons**: Multiple weapon types with cooldown systems
- **Particle Effects**: Visual feedback for impacts, explosions

#### Key Game Mechanics
- **Weapon Switching**: Cycle through vulcan, laser, and homing missiles
- **Smart Bombs**: Limited-use area-of-effect attacks
- **Stage Progression**: Score-based advancement through stages
- **Difficulty Scaling**: Enemy patterns and spawn rates increase per stage
- **Multiplier System**: Score multiplier increases with consecutive hits

## Development Guidelines

### Code Style
- **Language**: Vanilla JavaScript (ES6+)
- **No external dependencies**: Pure HTML5 Canvas and Web APIs
- **Single file structure**: Keep all code in `index.html`
- **Naming conventions**:
  - Variables and functions: camelCase
  - Constants: UPPERCASE_WITH_UNDERSCORES
  - Game entities: descriptive names (player, enemies, projectiles)

### Responsive Design
- Use CSS `clamp()` for responsive font sizing and spacing
- Test on mobile devices (touch controls must work)
- Viewport includes `viewport-fit=cover` for notched devices
- HUD uses flexbox for responsive layout

### Performance Considerations
- Minimize DOM manipulation (single canvas element)
- Use efficient collision detection (spatial partitioning if adding many entities)
- Reuse objects where possible (pooling for bullets/effects)
- Profile with browser DevTools before optimizations

### Testing
1. Test on desktop browsers (Chrome, Firefox, Safari)
2. Test on mobile devices (iOS Safari, Android Chrome)
3. Verify touch controls and screen orientation handling
4. Check performance: maintain 60 FPS on target devices

## Making Changes with Claude Code

### Adding Game Features
When adding new features (weapons, enemies, stages):
1. Define the feature's data structure (entity type)
2. Add initialization in game setup
3. Implement update logic in the main game loop
4. Add rendering code in the draw function
5. Test thoroughly on desktop and mobile

### Modifying Gameplay
When balancing gameplay (difficulty, weapon power, enemy patterns):
- Adjust spawn rates and patterns in enemy update functions
- Modify weapon damage and cooldowns in weapon systems
- Tweak player health and bomb counts via constants
- Test across multiple difficulty levels/stages

### UI/HUD Changes
When updating the HUD display:
- Modify the HTML structure (lines with `#hud`, `#hearts`, `#bombs`)
- Update CSS styles in the style section
- Update JavaScript variables that render HUD values
- Ensure responsive behavior with mobile screens

### Optimization Tasks
When improving performance:
- Profile with Chrome DevTools (Performance tab)
- Identify bottlenecks in update and draw loops
- Consider object pooling for frequently created/destroyed entities
- Test FPS before and after changes

## Git Workflow

- **Default branch**: `main` (or `master`)
- **Development branch**: `claude/inspiring-planck-qryip8`
- **Commit messages**: Clear, descriptive messages describing the change
  - Example: "Add homing missile weapon system"
  - Example: "Improve boss encounter difficulty scaling"
- **Push changes**: Always push to the development branch
- **Pull requests**: Create PR only when explicitly requested by the user

## Common Tasks

### Debug Game State
```javascript
// Open browser console (F12) and check:
console.log({player, enemies, score, stage, gameState});
```

### Test Weapon Damage
- Temporarily modify weapon damage values
- Run multiple test plays to verify balance

### Adjust Enemy Patterns
- Find enemy update function
- Modify movement patterns or spawn logic
- Verify difficulty curve across stages

## Asset Usage

- **Images**: None (pure Canvas drawing)
- **Fonts**: System fonts (Segoe UI, Roboto, Arial fallback)
- **Colors**: HEX codes defined in CSS (dark blue theme)
- **Audio**: Not currently implemented

## Known Limitations

- Single-file structure (all code in one HTML file)
- No sound effects or background music
- Limited UI (no main menu, settings screen)
- Fixed canvas resolution with scaling

## Future Enhancement Ideas

- Sound effects and music
- Main menu and pause screen
- Leaderboard system
- Additional weapon types
- Boss variations
- Story/narrative elements
- Difficulty settings

---

**Last Updated**: 2026-09-30  
**Maintained by**: Claude Code Agent
