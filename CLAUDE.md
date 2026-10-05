# Google_4 / Sky Fighter

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas and JavaScript. A single-file implementation featuring weapon switching, smart bombs, enemy waves, stage bosses, and responsive mobile-friendly controls.

## Quick Start

- **Run**: Open `index.html` in a browser
- **Dev**: Edit `index.html` directly and reload to see changes
- **Mobile**: Touch controls on phones and tablets

## Architecture

### Single-File Game Engine
The entire game is contained in `index.html`:
- **Canvas Rendering**: HTML5 Canvas for all graphics (2D context)
- **Game Loop**: `requestAnimationFrame` for smooth 60 FPS updates
- **Entity System**: Players, enemies, bullets, explosions as objects in arrays
- **Game States**: Intro, playing, boss, game over, victory screens

### Core Systems
1. **Player**: Keyboard (Arrow Keys, Space) or touch/mouse controls
   - Health system (5 hearts max)
   - Smart bomb resource (limited uses)
   - Weapon switching (Machine Gun, Rapid Fire, Spread Shot)
   - Multiplier mechanic (increases with kills)

2. **Enemies**: Spawned in waves with configurable patterns
   - Bullet behavior (aimed, spreading, random)
   - Experience drops on death (healing and bomb ammo)
   - Boss encounters at stage end

3. **Rendering**: Dark atmospheric sci-fi aesthetic
   - Gradient sky background
   - HUD for stats (health, bombs, weapon, multiplier)
   - Vignette effect on bomb/critical damage
   - Screen flash on hits

## Development

### Editing
- All code is inline in `index.html`
- Look for the `<script>` tag at the end for main game logic
- CSS is in `<style>` for styling and animations

### Key Variables & Functions
- `gameState`: Controls game flow (intro, playing, boss, gameOver, victory)
- `player`: Player object with health, position, weapon, bombs
- `enemies`: Array of active enemies
- `bullets`: Array of active player bullets
- `update()`: Main game update loop
- `render()`: Canvas drawing function
- `spawnWave()`: Generates enemy waves

### Adding Features
When adding new mechanics:
1. Update the entity update/collision logic
2. Add rendering code in `render()`
3. Test on both desktop (keyboard) and mobile (touch)
4. Verify HUD updates if it affects player state

### Testing
- Desktop: Arrow keys + Space for shoot/bomb
- Mobile: Tap to shoot, hold near top for bomb
- Check HUD visibility on various screen sizes
- Verify sound effects (if any) work cross-browser

## File Structure
```
index.html          # Complete game implementation
README.md           # Project info
CLAUDE.md           # This file
```

## Browser Support
- Chrome/Edge: Full support
- Firefox: Full support
- Safari: Full support (iOS 14+)
- Mobile browsers: Touch-optimized

## Notes
- Game uses `localStorage` for potential persistence (if added)
- No external dependencies — pure vanilla JavaScript
- Responsive design uses viewport meta tags and CSS `clamp()`
- Audio can be added via Web Audio API if desired
