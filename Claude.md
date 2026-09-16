# Sky Fighter - Codebase Documentation

## Project Overview
Sky Fighter is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5, CSS3, and JavaScript. Players control an aircraft, switch between different weapons, drop smart bombs, and battle through stages to defeat bosses.

## Architecture

### Technology Stack
- **Frontend**: HTML5, CSS3, JavaScript (ES6+)
- **Graphics**: HTML5 Canvas API
- **Audio**: Web Audio API (if implemented)
- **Input**: Keyboard and Touch Events
- **Responsiveness**: CSS media queries and viewport meta tags

### Project Structure
```
Google_4/
├── index.html       # Single-file game application (HTML + CSS + JS)
├── README.md        # Basic project info
└── Claude.md        # This file - codebase documentation
```

## Key Features
- Vertical scrolling shoot-'em-up gameplay
- Multiple weapon types
- Smart bomb mechanic
- Stage progression with boss battles
- Responsive design for desktop and mobile
- Touch-optimized controls
- Dark space-themed visual style

## Game Mechanics

### Core Components (in index.html)
1. **Canvas Rendering**: Main game loop uses requestAnimationFrame for smooth 60fps gameplay
2. **Player Aircraft**: Controlled via keyboard/touch, has multiple weapon options
3. **Weapon System**: Different damage types and firing patterns
4. **Smart Bombs**: Screen-clearing special attack with limited uses
5. **Enemy Spawning**: Progressive enemy waves per stage
6. **Boss Encounters**: Stage-ending boss battles
7. **HUD System**: Displays score, health, weapons, and stage info
8. **Collision Detection**: Bounding box collision for bullets, enemies, and player

### Visual Design
- Color scheme: Dark blue space background (#05122b, #0d2f5e) with cyan/blue accents
- UI elements rendered in white with text shadows for visibility
- Responsive sizing using clamp() for different screen sizes
- Safe area insets for notch/island-shaped displays

### Input Handling
- Keyboard: WASD for movement, Space for shoot, Shift for bombs
- Touch: Direct click/tap control with crosshair cursor
- Touch-action: none to prevent browser gestures

## Development Guidelines

### Code Style
- Single-file architecture for simplicity and easy deployment
- Inline CSS for styling efficiency
- Vanilla JavaScript (no external frameworks)
- Game loop driven by requestAnimationFrame

### Common Tasks
1. **Adjusting Difficulty**: Modify enemy spawn rates and health values
2. **Tweaking Weapon Balance**: Adjust damage, fire rate, and ammo pool
3. **Changing Visual Style**: Modify gradient backgrounds and color scheme
4. **Adding New Weapon Types**: Implement new firing patterns in weapon logic
5. **Modifying Boss Attacks**: Adjust boss behavior and attack patterns

### Performance Considerations
- Canvas rendering is optimized for mobile (max-width/height constraints)
- Collision detection uses bounding boxes (no pixel-perfect detection)
- Enemy pooling recommended for large enemy counts
- Minimize DOM manipulations outside the game canvas

## Running the Game
1. Open `index.html` in a modern web browser (Chrome, Firefox, Safari, Edge)
2. For local development: use a simple HTTP server (e.g., `python -m http.server`)
3. Supports both desktop and mobile play
4. No build process or dependencies required

## Browser Compatibility
- Modern browsers supporting:
  - HTML5 Canvas API
  - CSS Grid/Flexbox
  - ES6 JavaScript
  - Touch Events
  - RequestAnimationFrame
  - CSS env() for safe areas

## Accessibility
- High contrast colors for visibility
- Touch targets appropriately sized
- Alternative keyboard controls available
- Safe area insets respected for notched devices

## Future Enhancements
- Sound effects and music (Web Audio API)
- Particle effects system
- Save/load game state (localStorage)
- Difficulty settings
- Leaderboard/high scores
- Mobile app packaging (PWA, Electron, Cordova)

## Build & Deployment
- **No build step required** - single HTML file deploys directly
- **Deployment**: Upload `index.html` to web server
- **Static hosting**: Compatible with GitHub Pages, Netlify, Vercel
- **PWA potential**: Can be configured as Progressive Web App with manifest.json

## Notes for AI Assistants
- Entire game logic is contained in `index.html`
- Examine the `<script>` section for game mechanics
- CSS variables (--color-scheme: dark) control visual theme
- Game state managed in local JavaScript objects/variables
- No external API calls or backend required
