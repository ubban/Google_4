# Sky Fighter: Codebase Guide for Claude Code

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in HTML/JavaScript. Players control a fighter jet, switching weapons, deploying smart bombs, and battling stage bosses. The game runs in a single `index.html` file with no external dependencies.

### Key Features
- Responsive design that works on desktop, tablet, and mobile
- Multiple weapons system with switching capability
- Smart bomb power-up mechanics
- Stage-based boss encounters
- Smooth animations and game physics
- Touch and mouse input support
- Dark theme optimized for gameplay

## Project Structure

### File Organization
- `index.html` - Main entry point containing all HTML, CSS, and JavaScript
- `README.md` - Project metadata
- No build tools or dependencies required

### Code Organization Within index.html

The single HTML file is organized into logical sections:

1. **HTML Structure**
   - Container and canvas element for game rendering
   - HUD (heads-up display) for showing game stats
   - Status text overlay

2. **CSS Styling**
   - Root color scheme and layout
   - Game canvas styling
   - HUD layout using flexbox
   - Stat containers
   - Responsive design using clamp() for fluid scaling
   - Dark theme with game-specific gradients

3. **JavaScript Game Engine**
   - Game state management
   - Input handling (keyboard and touch)
   - Entity system (player, enemies, projectiles, etc.)
   - Collision detection
   - Rendering pipeline
   - Game loop and animation frame handling

## Development Guidelines

### Code Style
- Use modern vanilla JavaScript (no frameworks or transpilation)
- Keep code self-contained within the HTML file
- Use descriptive variable and function names
- Prefer const/let over var
- Use arrow functions for consistency

### Game Architecture

#### Core Game Loop
The game runs on `requestAnimationFrame` for smooth 60 FPS gameplay. The main loop handles:
1. Input processing
2. Entity updates
3. Collision detection
4. Rendering

#### Entity System
- Player: Main character controlled by user
- Enemies: Various enemy types with behaviors
- Projectiles: Bullets and bombs
- Particles: Visual effects

#### Physics & Collision
- Bounding box collision detection for entities
- Canvas boundary checking for movement
- Damage and health systems

#### Input Handling
- Keyboard support: Arrow keys or WASD for movement, Space for shooting
- Touch/Mouse support: Tap location for movement and shooting
- Safe area insets for mobile notches and status bars

### Performance Considerations

1. **Canvas Rendering**
   - Clear canvas each frame
   - Batch render similar entity types
   - Use requestAnimationFrame for optimal performance
   - Optimize hitbox calculations

2. **Entity Management**
   - Reuse entity objects when possible
   - Clean up destroyed entities to prevent memory leaks
   - Use arrays for efficient entity storage and iteration

3. **Mobile Optimization**
   - Prevent tap highlight color
   - Disable user select
   - Use max-width/max-height on canvas to fit smaller screens
   - Support touch-action: none for custom input handling

## Building and Testing

### Running the Game
1. Open `index.html` in any modern web browser (desktop, tablet, or mobile)
2. No build step or server required
3. Game runs entirely client-side

### Testing
- Test on multiple browsers (Chrome, Firefox, Safari, Edge)
- Test on various screen sizes and devices
- Verify touch input on mobile devices
- Check performance on lower-end devices

### Deployment
1. Serve `index.html` on any static file server
2. No build process needed
3. All assets are inline or use data URIs

## Making Changes

### Adding New Features
- Keep all new code within the `<script>` tag in `index.html`
- Follow the existing code organization
- Test thoroughly across browsers and devices
- Ensure no external dependencies are introduced

### Modifying Game Mechanics
- Game parameters (speeds, health, etc.) are typically defined early in the code
- Search for existing mechanic implementations to understand patterns
- Test gameplay impact thoroughly

### Styling Updates
- All CSS is in the `<style>` tag within the HTML head
- Use CSS variables for colors and sizing where possible
- Maintain responsive design using clamp() and viewport units
- Keep dark theme consistent

## Browser Support

- Modern browsers with ES6 support
- Canvas API support required
- Touch Events API for mobile
- Pointer Events support beneficial
- Safe Area Insets support for notched devices

## Performance Targets

- Smooth 60 FPS gameplay on modern devices
- Fast load time (single file, no external resources)
- Minimal CPU/GPU usage
- Responsive input handling with &lt;50ms latency

## Claude Code Specific Notes

When working with this codebase using Claude Code:

1. **Single File Complexity**: All code is in `index.html`. Be prepared to handle large file edits.
2. **No Rebuilds**: Changes take effect immediately when the file is saved and the browser is refreshed.
3. **Rapid Testing**: Use the browser's developer tools (DevTools) to debug issues in real-time.
4. **Search Strategy**: When looking for specific functionality, search for function names or variable definitions within the single file.
5. **Testing**: Always test changes in a browser before considering work complete. The game's visual and gameplay behavior is the source of truth.

## Common Tasks

### Adding a New Enemy Type
1. Define enemy properties (speed, health, behavior)
2. Create spawn logic at appropriate stage points
3. Implement collision detection with player weapons
4. Test spawn rates and difficulty progression

### Implementing a New Weapon
1. Define projectile properties
2. Create firing mechanism tied to weapon switch system
3. Implement damage calculations
4. Balance with other weapons

### Adjusting Game Balance
1. Modify enemy stats (health, speed, damage)
2. Adjust player movement/firing speed
3. Tweak spawn rates and patterns
4. Test difficulty curve across stages

---

*Last Updated: 2026-09-30*
