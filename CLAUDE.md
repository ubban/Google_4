# Sky Fighter — Codebase Documentation

## Project Overview
Sky Fighter is a Raiden-style vertical shoot-em-up game built with vanilla HTML5 Canvas. It's a single-file web application featuring:
- Vertical scrolling gameplay
- Player ship with weapon switching
- Enemy waves with stage bosses
- Smart bombs and power-ups
- Full responsive/mobile support

## Technology Stack
- **Language**: JavaScript (ES6+)
- **Graphics**: HTML5 Canvas
- **Framework**: None (vanilla implementation)
- **File Structure**: Single `index.html` file containing HTML, CSS, and JavaScript

## Key Game Features
- **Player Controls**: Arrow keys or WASD for movement, Space to fire, Shift for smart bomb
- **Weapons**: Multiple weapon types with switching mechanics
- **Enemies**: Various enemy types with distinct behaviors and bosses
- **Responsive Design**: Works on desktop and mobile with touch controls
- **Sound Effects**: Audio feedback for gameplay events

## Development Guidelines

### Code Organization
The entire game is implemented within `index.html` in three sections:
1. **HTML**: Canvas element and UI container
2. **CSS**: Game styling, responsive design, theme
3. **JavaScript**: All game logic (game state, player, enemies, collisions, rendering)

### Architecture Notes
- **Canvas Rendering**: All graphics drawn to HTML5 Canvas in real-time
- **Game Loop**: requestAnimationFrame for smooth 60fps gameplay
- **Collision Detection**: Bounding box collision system for bullets and enemies
- **Responsive Canvas**: Scales to fit viewport while maintaining aspect ratio

### Important Implementation Details
- Touch controls are fully supported with `touch-action: none`
- Safe area insets considered for notched devices
- Dark theme optimized for gaming atmosphere
- Crosshair cursor for targeting feedback

## Git Workflow
- **Branch Convention**: Feature/fix branches follow pattern `claude/[descriptive-name]`
- **Commits**: Clear, descriptive messages explaining gameplay or technical changes
- **Testing**: Manual testing in browser required before commits
- **Current Branch**: `claude/inspiring-planck-8forxn` - main development branch

## How to Work on This Project

### Running the Game
1. Open `index.html` directly in a web browser
2. Game should load and display immediately
3. No build process or server required

### Making Changes
1. Edit the JavaScript section within `index.html`
2. Test changes in browser (hard refresh if needed)
3. Commit changes with clear messages describing the gameplay/feature change
4. Push to the designated branch

### Common Tasks
- **Balancing Gameplay**: Adjust enemy spawn rates, damage values, and player stats in the JavaScript game state
- **Adding Features**: Implement new weapon types, enemy behaviors, or visual effects in the JavaScript section
- **Bug Fixes**: Test thoroughly in multiple browsers and devices before committing

## Code Style
- Use clear variable names for game objects and entities
- Keep game state organized in the main game object
- Add comments for complex collision or rendering logic
- Maintain responsive design considerations for mobile

## Mobile/Responsive Considerations
- Game canvas scales responsively while maintaining aspect ratio
- Touch controls work alongside keyboard controls
- Safe area insets used for notched devices
- Viewport settings ensure proper scaling on mobile

## Performance Notes
- Canvas rendering happens in requestAnimationFrame loop
- Collision detection uses simple bounding box system
- Assets are all inline (no external file dependencies)
- Game runs client-side only (no server required)
