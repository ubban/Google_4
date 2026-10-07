# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla JavaScript and HTML5 Canvas. Players control an aircraft to defeat enemies, switch weapons, and face stage bosses in an arcade-style action game.

### Key Features
- **Vertical Scrolling Gameplay**: Classic arcade shoot-'em-up mechanics
- **Weapon System**: Switch between different weapon types
- **Smart Bombs**: Special attacks to clear the screen
- **Boss Battles**: Stage bosses with unique patterns
- **Responsive Design**: Works on desktop and mobile devices
- **Dark Space Theme**: Blue gradient sky with sci-fi aesthetic

## Project Structure

```
Google_4/
├── README.md           # Project title and timestamp
├── index.html          # Main game file (all-in-one)
├── .git/               # Git repository
└── Claude.md           # This file
```

## Technology Stack

- **HTML5 Canvas**: Graphics rendering
- **Vanilla JavaScript**: Game logic and mechanics
- **CSS3**: Styling and responsive layout
- **No External Dependencies**: Standalone single-file game

## Architecture Overview

The `index.html` file contains:

### HTML Structure
- Single container div with game canvas
- HUD display for score, health, wave info, and weapon status
- Touch-action: none for mobile optimization
- Safe area insets for notched devices

### Game Canvas
- Responsive canvas (max-width: 100%, max-height: 100%)
- Background: Linear gradient from dark blue to light sky
- Crosshair cursor
- Touch controls support

### CSS Features
- Dark color scheme (#05122b, #0d2f5e)
- Responsive typography using clamp()
- Fixed positioning for HUD overlay
- Safe area inset support for mobile
- Text shadows for readability
- Flexbox layout for responsive design

### JavaScript Game Logic (Embedded in HTML)
The game engine includes:

1. **Game State Management**
   - Score, health, wave tracking
   - Player position and velocity
   - Enemy/projectile management

2. **Input Handling**
   - Keyboard controls (arrow keys, space)
   - Mouse/touch controls (aim and fire)
   - Weapon switching

3. **Rendering System**
   - Canvas drawing pipeline
   - HUD rendering
   - Sprite animations

4. **Game Objects**
   - Player aircraft
   - Enemies (regular and bosses)
   - Projectiles (player and enemy)
   - Visual effects

5. **Game Loop**
   - Update game state
   - Handle collisions
   - Render frame
   - RequestAnimationFrame for smooth 60fps

## Color Palette

- **Primary Dark**: #05122b (background)
- **Dark Sky**: #0d2f5e, #1e6fd8
- **Light Sky**: #7fc1ff
- **Accent Blue**: #1976d2 (player ship)
- **Text**: #ffffff
- **Shadow**: rgba(0,0,0,0.5-0.8)

## Development Guidelines

### Code Style
- Vanilla JavaScript without frameworks
- Single-file architecture for simplicity
- Inline CSS and JS
- Mobile-first responsive design

### Mobile Optimization
- Touch-action: none on canvas
- Viewport meta tags for proper mobile display
- Safe area insets for notched devices
- Maximum-scale: 1 for better mobile feel
- User-scalable: no to prevent zoom issues

### Performance Considerations
- Canvas rendering for optimal performance
- RequestAnimationFrame for smooth animation
- Efficient collision detection
- Object pooling for projectiles and enemies

### Browser Compatibility
- Modern browsers with Canvas support
- Flexbox support
- CSS Grid support (if used)
- Mobile browsers (iOS Safari, Chrome Android)

## Running the Game

### Local Development
1. Open `index.html` directly in a web browser
2. No build process or server required
3. Works offline

### Testing
- Manual testing in web browser
- Test on both desktop and mobile devices
- Test keyboard and mouse/touch controls
- Test different viewport sizes

### Deployment
- Copy `index.html` to any web server
- No build step required
- Works as standalone HTML file

## Git History

- **Latest**: `c4dc8f6` - Add Sky Fighter: Raiden-style vertical shoot-em-up game
- **Initial**: `4efdad1` - Initial commit

## Future Enhancement Ideas

- Sound effects and background music
- Multiple levels/stages
- Power-ups system
- High score persistence (localStorage)
- Difficulty levels
- Leaderboard
- More enemy types and patterns
- Enhanced visual effects (particles, explosions)

## Notes for Claude

- The entire game is contained in a single HTML file (~59KB)
- No external dependencies or build tools required
- All game logic is in inline `<script>` tags
- CSS is inlined in `<style>` tags
- Mobile-friendly with touch controls
- Performance-optimized for 60fps gameplay
- Theme color set for dark mode: #0b1d3a

When making changes:
1. Keep the single-file architecture intact
2. Test responsive behavior on mobile and desktop
3. Maintain performance (60fps target)
4. Preserve the dark theme aesthetic
5. Test all controls (keyboard, mouse, touch)
6. Verify safe area insets work on notched devices
