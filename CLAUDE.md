# Sky Fighter

A Raiden-style vertical shoot-'em-up game featuring weapon switching, smart bombs, and boss battles.

## Project Overview

Sky Fighter is a browser-based arcade game written in vanilla HTML/CSS/JavaScript. Players control a fighter jet and must defeat waves of enemies and challenging boss encounters.

## Project Structure

- **index.html** - Single file containing all HTML markup, CSS styles, and JavaScript game logic
  - HTML structure: Game canvas container, HUD overlay, visual effects (flash, vignette)
  - Inline CSS: Responsive styling with viewport adaptation, animations, and game-specific styling
  - Inline JavaScript: Complete game engine including rendering, collision detection, physics, and game state management

## Game Architecture

### Canvas and Rendering
- Uses HTML5 Canvas API for game rendering
- Game canvas is responsive and centered in the viewport
- Gradient background (blue sky theme) with dynamic effects

### HUD System
- Top-center overlay displaying:
  - Player health (hearts: ❤️ for alive, dimmed for lost)
  - Smart bombs remaining (💣 indicators)
  - Current weapon type and ammo
  - Score multiplier
- All HUD elements use semi-transparent styling with backdrop blur for visual clarity

### Player Mechanics
- **Controls**: Mouse/pointer movement to aim and move, click to shoot
- **Health System**: 3-hit points (shown as hearts), flashing animation when critical
- **Weapons**: Multiple weapon types with switching capability
  - Each weapon has different firing patterns and ammo
- **Smart Bombs**: Limited-use screen-clearing attacks (max 3)

### Enemy and Boss System
- Waves of enemies with varying patterns
- Stage bosses with unique attack patterns and health pools
- Progressive difficulty increase across stages

### Visual Effects
- **Flash Effect**: White screen flash on hit (0.25s)
- **Bomb Flash**: Larger white flash for smart bomb detonation (0.5s)
- **Vignette**: Red pulsing vignette overlay when player takes damage
- **Hit Animations**: Screen shakes and visual feedback for combat

## Key Features

1. **Weapon System**: Switch between different weapon types during gameplay
2. **Smart Bomb Mechanic**: Screen-clearing special attack with limited uses
3. **Score Multiplier**: Increases with consecutive hits, resets on miss
4. **Stage Progression**: Multiple stages with escalating difficulty
5. **Boss Encounters**: Challenging boss fights at stage completion
6. **Mobile Support**: Touch controls and viewport optimization for mobile devices

## Development Notes

### Game Loop
The game uses a typical canvas-based game loop with:
- Input handling (mouse/touch)
- Entity updates (player, enemies, projectiles)
- Collision detection
- Rendering to canvas
- HUD updates

### Responsive Design
- Uses CSS `clamp()` for scalable fonts and spacing
- Viewport fit for mobile notches/safe areas
- Max-width/height constraints for canvas to maintain aspect ratio

### Accessibility
- High contrast UI elements
- Large touch targets for mobile devices
- Text shadows for readability on gradient backgrounds

## Running the Game

Open `index.html` in a modern web browser. The game will auto-start with the game canvas filling the available screen space.

## Browser Compatibility

Requires:
- HTML5 Canvas API
- Modern JavaScript (ES6+)
- CSS 3 (backdrop-filter, custom properties, grid/flex)
- Touch event API (for mobile)

## Future Development Considerations

- Audio system (sound effects and music)
- Persistent high score tracking
- Game settings/difficulty selection
- Mobile-specific UI improvements
- Additional weapon types and enemy variety
- Tutorial/how-to-play screen
