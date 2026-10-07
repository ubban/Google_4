# Sky Fighter - Codebase Documentation

## Project Overview

Sky Fighter is a Raiden-style vertical shoot-'em-up game built entirely in vanilla JavaScript and HTML5. It's a single-page application served via `index.html` that runs in a web browser with full support for desktop and mobile devices.

**Key Characteristics:**
- Pure vanilla JavaScript (no frameworks)
- Canvas-based rendering
- Responsive design with viewport scaling
- Mobile-first with touch controls
- Self-contained in a single HTML file (~1669 lines)

## Architecture

### Single-File Structure
The entire game is contained in `index.html` with:
- **HTML markup**: Game canvas container and heads-up display (HUD) elements
- **CSS styling**: Dark-themed game UI with responsive typography and layout
- **JavaScript game logic**: All game mechanics, entities, and rendering

### Core Game Systems

#### 1. **Rendering**
- Canvas-based 2D rendering with `requestAnimationFrame`
- Game canvas sizes responsively to fit the viewport
- HUD (heads-up display) overlays game state information
- Dark gradient background (blue tones) simulating sky

#### 2. **Game Entities**
- **Player ship**: Positioned at bottom center, controlled via mouse/touch
- **Enemies**: Multiple enemy types with scripted patterns
- **Projectiles**: Player bullets and enemy fire
- **Bombs**: Smart bomb power-ups for crowd control
- **Stage bosses**: Multi-phase enemy encounters

#### 3. **Game State**
- Score tracking
- Lives/health management
- Weapon selection and ammunition
- Game over / victory conditions
- Multi-stage progression

#### 4. **Controls**
- **Desktop**: Mouse movement to aim, click/spacebar to shoot
- **Mobile**: Touch/pointer movement to aim, tap to shoot
- Weapon switching (likely via keyboard or UI)
- Bomb activation

#### 5. **Visual Effects**
- Explosions and particle effects
- Enemy death animations
- Weapon feedback (muzzle flash, etc.)
- Screen flash on damage

### Styling & Theme
- **Color scheme**: Dark blue (`#05122b`, `#0d2f5e`) with cyan accents (`#1976d2`, `#7fc1ff`)
- **Typography**: Responsive font sizing using CSS clamp
- **Layout**: Flexbox-based centering and HUD arrangement
- **Safe area awareness**: Respects device safe areas (notches, home indicators)

## Development Guidelines

### Coding Standards
- Vanilla JavaScript (ES6+)
- Prefer modern Web APIs (e.g., `pointer` events for cross-device input)
- Use CSS custom properties (`:root` variables) for theming
- Responsive design with CSS clamp and viewport units

### Key Features to Maintain
- **Cross-platform compatibility**: Works on desktop and mobile
- **Performance**: Optimize canvas rendering and entity updates
- **Accessibility**: Text contrast, touch-friendly hit areas
- **Responsive**: Adapts to any screen size and orientation

### Common Tasks

#### Adding a New Enemy Type
1. Define enemy properties (position, speed, health, sprite)
2. Implement update logic (movement pattern, attack timing)
3. Add rendering code to canvas
4. Integrate into enemy spawning system

#### Adding a New Weapon
1. Define weapon properties (fire rate, damage, bullet speed)
2. Implement firing mechanism (pattern, trajectory)
3. Add visual feedback (sprites, sounds if applicable)
4. Wire up weapon selection UI

#### Implementing a New Stage/Boss
1. Design enemy waves and boss patterns
2. Implement stage progression logic
3. Add victory/defeat conditions
4. Update game state management

## File Structure
```
Google_4/
├── index.html          # Main game file (all code)
├── README.md          # Project metadata
└── CLAUDE.md          # This documentation
```

## Deployment
The game is a static single-file application. Simply serve `index.html` over HTTP(S). No build step or backend required.

## Browser Compatibility
- Modern browsers with ES6 support
- Canvas API support required
- Touch Events API for mobile support
- Pointer Events API for cross-device input

## Performance Considerations
- Canvas rendering optimized with single draw pass
- Entity pooling may improve performance for high entity counts
- Consider WebWorker offloading for complex calculations
- Profile on low-end mobile devices

## Future Enhancement Areas
- Sound effects and music
- Mobile UI improvements (on-screen buttons)
- Level/stage editor
- Leaderboards/persistence
- Difficulty settings
- Power-up variety expansion
