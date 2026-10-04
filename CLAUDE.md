# Sky Fighter: Raiden-style Vertical Shoot-em-up

## Project Overview

**Sky Fighter** is a browser-based, Raiden-style vertical shoot-em-up game implemented in pure JavaScript with HTML5 Canvas. Players control a fighter jet, switch between different weapons, drop smart bombs, and progress through stages by defeating increasingly challenging enemies and boss encounters.

**Key Stats:**
- Single HTML file (index.html) - ~58KB
- No external dependencies (vanilla JavaScript)
- Fully responsive canvas-based rendering
- Touch and keyboard controls
- Cross-browser compatible

## File Structure

```
/
├── index.html          Main game file (all HTML, CSS, and JavaScript combined)
├── README.md           Project metadata
└── CLAUDE.md          This file - codebase documentation
```

## Game Architecture

### Core Game Loop
The game uses a requestAnimationFrame-based render loop that:
1. Processes input (keyboard/touch)
2. Updates game state (player, enemies, bullets, particles)
3. Renders the scene to canvas
4. Manages game phases (menu, playing, pause, game over)

### Key Game Systems

**Player System:**
- Position tracking and boundary checking
- Health/armor system
- Multiple weapon types (varies by implementation)
- Smart bomb ability with cooldown
- Animation and particle effects on damage/death

**Enemy System:**
- Spawning waves based on stage progression
- Individual AI patterns for enemy movement
- Bullet patterns and attack sequences
- Boss encounters with multi-phase behavior
- Drop rewards (health, weapon pickups, score bonuses)

**Combat System:**
- Player bullet collision with enemies
- Enemy bullet dodging and collision with player
- Damage calculation and health tracking
- Weapon switching mechanics
- Visual/audio feedback for hits and kills

**UI/HUD:**
- Score display (centered top)
- Health/armor status
- Current weapon indicator
- Wave/stage information
- Game state overlays (menu, pause, game over)

**Graphics:**
- Canvas-based 2D rendering
- Gradient background (sky blue effect)
- Sprite-like rectangles for game objects
- Particle effects for explosions and impacts
- Text rendering for UI elements

## Running the Game

1. **Open in Browser:** Simply open `index.html` in any modern web browser (Chrome, Firefox, Safari, Edge)
2. **Local Server (Optional):** For development, run a local HTTP server:
   ```bash
   python3 -m http.server 8000
   # or
   npx http-server
   ```
   Then visit `http://localhost:8000`

## Controls

- **Keyboard:**
  - Arrow Keys: Move player
  - Z/Space: Fire weapon
  - X: Switch weapon
  - B: Drop smart bomb
  - P: Pause/Resume
  - Enter: Start game

- **Touch/Mobile:**
  - Tap and drag to move
  - Tap to fire
  - Double-tap to drop bomb
  - Swipe for weapon switch

## Development Notes

### Code Organization
The entire game is contained within `<script>` tags in index.html. The structure follows:
- CSS styling for layout and theming
- Game configuration constants (difficulty, spawn rates, physics)
- Game state object (player, enemies, bullets, UI state)
- Input handling (keyboard, touch, mouse)
- Update/render functions for each game system
- Game loop orchestration

### Key Implementation Details

**Canvas Rendering:**
- Fixed canvas size relative to viewport
- Scaling handled through CSS transforms
- Safe area inset support (notch devices)
- Dark theme with gradient sky background

**Performance Considerations:**
- Object pooling for bullets/particles to reduce GC pressure
- Efficient collision detection (bounding box checks)
- Sprite animation via frame counters
- Limited particle count with early cleanup

**Mobile Optimization:**
- Touch-action CSS property prevents browser scrolling
- Viewport settings prevent zoom/scaling issues
- Responsive HUD font sizing (clamp)
- Safe area insets for notched devices

### Extension Points

To add new features, consider:
1. **New Enemy Types:** Define AI patterns in the enemy update logic
2. **New Weapons:** Add weapon configurations and firing patterns
3. **New Stages/Bosses:** Define spawn sequences and boss behavior
4. **Sound/Music:** HTML5 Audio API integration (currently visual-only)
5. **Persistence:** LocalStorage for high scores and settings
6. **Mobile App:** Wrap in Electron or Cordova for app stores

## Testing

There are no automated tests. Manual testing should cover:
- Gameplay mechanics on desktop (keyboard)
- Gameplay mechanics on mobile (touch)
- Various screen sizes (responsive design)
- Different browsers (Chrome, Firefox, Safari, Edge)
- Performance on low-end devices
- Boss defeat and stage progression

## Known Limitations

- No sound/music (visual-only game)
- No persistence (scores reset on refresh)
- Single stage implementation (can add more stages)
- No pause menu details
- Limited enemy variety
- No difficulty selection menu

## Future Enhancements

- [ ] Add audio (background music, sound effects)
- [ ] Save high scores to localStorage
- [ ] Multiple stages with increasing difficulty
- [ ] More weapon variety
- [ ] Power-up system
- [ ] Local multiplayer support
- [ ] Mobile app wrapper
- [ ] Leaderboard integration

## Git Workflow

- **Main branch:** Stable, released version
- **Feature branches:** New features or bug fixes (e.g., `claude/inspiring-planck-3ydp2t`)
- **Commit messages:** Clear, descriptive messages for feature additions
- **Testing before merge:** Verify game runs and plays smoothly

## Browser Compatibility

Tested and working on:
- Chrome 90+
- Firefox 88+
- Safari 14+
- Edge 90+

Requires:
- HTML5 Canvas support
- Modern JavaScript (ES6)
- Touch Events API (for mobile)

## Notes for Claude Code Agents

When making changes to this project:
1. The entire game logic is in a single `<script>` block in index.html
2. All game state is managed in a single `gameState` object
3. The render loop is managed by `gameLoop()` and `render()`
4. Test changes by opening index.html in a browser
5. Keep the file under 100KB for optimal performance
6. Maintain the dark theme for aesthetic consistency
7. Always test on both desktop and mobile viewports

---

**Last Updated:** 2026-10-04  
**Project Type:** Browser Game (Vanilla JavaScript)  
**Maintenance:** Active Development
