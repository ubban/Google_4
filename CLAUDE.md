# Sky Fighter - Claude Code Guidelines

## Project Overview
Sky Fighter is a Raiden-style vertical shoot-'em-up game built with vanilla JavaScript and HTML5 Canvas. The entire game runs in a single `index.html` file.

## Development Setup

### Running the Game
Open `index.html` in a web browser or use the `/run` command to launch the game directly.

### Testing Changes
- Use the browser's developer console to check for any errors
- Test gameplay mechanics: weapon switching, bombs, enemy patterns, boss fights
- Verify on different screen sizes (mobile, tablet, desktop)
- Check performance on lower-end devices if possible

## Game Architecture

### Canvas-Based Rendering
- Single canvas element for all game rendering
- All game logic is contained within inline `<script>` tags in index.html
- Game loop uses `requestAnimationFrame` for smooth 60 FPS gameplay

### Key Game Systems
- **Player Ship**: Keyboard/mouse-controlled, health system, weapon switching
- **Weapons**: Multiple weapon types with distinct firing patterns
- **Enemies**: Various enemy types with AI patterns
- **Bombs**: Smart bomb system for crowd control
- **Bosses**: Stage bosses with unique attack patterns and behavior
- **Scoring**: Points system with combo multipliers

## Code Organization

All game code is in `index.html`:
1. **CSS** - Styling (dark theme, responsive layout)
2. **HTML** - Game container and HUD elements
3. **JavaScript** - Game engine, entities, physics, and logic

## Common Tasks

### Adding New Enemy Types
- Locate the enemy spawning logic in the game loop
- Define new enemy properties (speed, health, firing pattern)
- Add collision detection for the new enemy type

### Modifying Weapon Systems
- Find weapon configuration in the weapon switching code
- Adjust bullet speed, fire rate, and damage values
- Update visual effects if needed

### Adjusting Game Difficulty
- Modify enemy spawn rates and patterns
- Adjust boss health and attack patterns
- Change score multipliers and progression speed

## Testing & Validation

Always verify:
- Game launches without console errors
- Player controls respond correctly
- Enemies spawn and move as intended
- Collisions register properly
- Bosses function as designed
- HUD displays correct information
- Game performs smoothly without lag

## Browser Compatibility
- Target modern browsers (Chrome, Firefox, Safari, Edge)
- Ensure mobile touch controls work on iOS and Android
- Use requestAnimationFrame for consistent performance

## Performance Considerations
- Minimize DOM manipulation during gameplay
- Use efficient collision detection algorithms
- Manage enemy/bullet object pooling to prevent memory leaks
- Monitor frame rate during complex scenes (boss fights)

## Attribution
Changes made by Claude Code agents should follow the standard commit message format with proper attribution.
