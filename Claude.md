# Claude.md — Sky Fighter Codebase Guide

## Project Overview
**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-page HTML/CSS/JavaScript application. Players control an aircraft, switch between weapons, deploy smart bombs, and defeat stage bosses.

## Project Structure
- **index.html** — Single file containing HTML, CSS, and JavaScript
  - HTML: Game container and HUD elements (lives, bombs, weapon, multiplier)
  - CSS: Styling for game canvas, HUD, animations (heartbeat, vignette, flash effects)
  - JavaScript: Complete game engine including player, enemies, projectiles, bosses, physics

## Technology Stack
- **Vanilla HTML5**: No frameworks; pure browser Canvas API for rendering
- **CSS3**: Grid/flex layouts, animations, backdrop filters, CSS variables via `:root`
- **Vanilla JavaScript**: ES6+ syntax, requestAnimationFrame for game loop, touch/mouse input handling

## Key Game Mechanics
1. **Player Control**: WASD or arrow keys for movement, mouse/touch aim, click/tap to shoot
2. **Weapons**: Fire primary weapon continuously; cycle through weapon types
3. **Smart Bombs**: Limited resource; clears enemies in radius and grants temporary immunity
4. **Enemy Waves**: Progressively harder enemies spawn across stages
5. **Boss Fights**: Stage bosses with distinct attack patterns and phases
6. **Lives & Score**: Lives indicated by hearts; multiplier increases on combo kills

## Important Code Patterns
- **Color Scheme**: Dark blue base (#05122b, #0d2f5e) with accent colors (yellow #ffd54f, lime #7CFC00, orange #ff8a65)
- **HUD Display**: Stats show in top-center with semi-transparent dark background and blur effect
- **Visual Feedback**: Flash effects, vignette pulse, and animations for hits/bombs
- **Responsive Design**: Uses `clamp()` for scalable font sizing across screen sizes
- **Safe Area Insets**: Supports notched devices via `env(safe-area-inset-top)`

## Game Loop & Rendering
The game likely uses `requestAnimationFrame` for a 60fps loop:
1. Process input (keyboard, mouse, touch)
2. Update game state (player position, projectiles, enemies, physics)
3. Handle collisions and damage
4. Render via Canvas
5. Update HUD display

## Development Guidelines
### Adding New Features
- Keep the single-file architecture intact for simplicity
- Use semantic class and ID names (e.g., `#gameCanvas`, `.stat`)
- Follow existing CSS conventions (use `:root` for colors, `clamp()` for responsive sizing)
- Maintain mobile-first responsive design

### UI Changes
- HUD elements use flexbox with `gap` for spacing
- Animations are CSS-based with `@keyframes` for performance
- Use `pointer-events: none` on overlay elements to allow canvas interaction

### Enemy/Boss Design
- Enemies likely have position, velocity, health, and render logic
- Bosses may have multiple phases with attack pattern changes
- Collision detection handles projectile-to-enemy and enemy-to-player interactions

### Testing & Debugging
- Test on desktop (keyboard input) and mobile (touch input)
- Verify responsive scaling at multiple viewport sizes
- Check performance on devices with lower refresh rates
- Test with safe area insets (notched devices)

## Performance Considerations
- Canvas rendering is efficient for 2D sprite-based games
- Limit active projectiles and enemies to maintain 60fps
- Use `touch-action: none` and `-webkit-tap-highlight-color: transparent` for mobile performance

## Future Enhancements
- Persist high scores using localStorage
- Add sound effects and background music (Web Audio API)
- Implement difficulty levels or endless mode
- Add more weapon types and power-ups
- Create level selection menu
- Add controller support (Gamepad API)

---

*Last updated: 2026-09-27*
