# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The game features a player-controlled aircraft navigating vertically through enemy waves, with weapon switching mechanics, smart bomb system, and stage bosses.

- **Type**: Browser-based game
- **Architecture**: Single HTML file with inline CSS and JavaScript
- **Entry Point**: `index.html`
- **Technology Stack**: HTML5, Canvas API, JavaScript (ES6+)
- **No external dependencies** - fully self-contained

---

## Game Mechanics

### Core Features

1. **Player Aircraft**
   - Vertical movement controls (mouse/touch)
   - Health system (3 hit points by default)
   - Visual feedback (vignette effect when damaged)

2. **Weapon System**
   - Multiple weapon types (varies by setup)
   - Weapon switching capability
   - Fire rate and damage variation per weapon
   - On-screen weapon status indicator

3. **Smart Bombs**
   - Limited quantity per stage (typically 3)
   - Screen-wide damage effect
   - Dramatic visual and audio feedback
   - Destroys most enemies and clears projectiles

4. **Enemy Types**
   - Basic wave enemies
   - Stage bosses with health bars
   - Projectile-based attacks
   - Drop items (health, bombs, score multipliers)

5. **Scoring System**
   - Score multiplier mechanism
   - Combo counter display
   - Stage banner presentation
   - Visual effects for achievements

### HUD Elements

The game displays real-time statistics:
- **Hearts**: Health indicator (animated when critical)
- **Bombs**: Smart bomb count
- **Weapon**: Current weapon type and status
- **Multiplier**: Current score multiplier
- **Boss Bar**: Health indicator during boss fights (name + health bar)

---

## Code Structure

### HTML Structure
- `#container`: Main game viewport
- `#gameCanvas`: Canvas element for rendering
- `#hud`: Heads-up display with game statistics
- `#topBar`: Top-right control buttons (menu, settings, etc.)
- `#vignette`: Damage feedback visual effect
- `#flash`: Screen flash effect for impacts/bombs
- `#comboPop`: Floating combo counter text
- `#stageBanner`: Stage introduction banner
- `#bossBarWrap`: Boss health bar container

### Styling Approach
- **Dark theme** with blue/cyan color scheme
- **Responsive design** using CSS `clamp()` for scaling
- **Safe area support** for notched devices
- **Animation keyframes** for:
  - Heart beating (damage state)
  - Vignette pulsing (damage vignette)
  - Flash effects (hit/bomb feedback)
  - Combo pop animation
  - Stage banner presentation

### Game Loop Architecture

The JavaScript implementation follows a typical game loop pattern:

1. **Initialization**: Canvas setup, entity creation, input binding
2. **Update**: 
   - Input processing (mouse/touch movement)
   - Entity updates (position, animation, collision)
   - Game state management
3. **Render**: 
   - Canvas clearing
   - Entity drawing
   - HUD updates
4. **Animation Frame Loop**: Continuous requestAnimationFrame() cycle

### Key Game Objects/Entities

The code manages several entity types:
- **Player**: Controlled aircraft with health, position, active weapon
- **Enemies**: Various enemy types with AI patterns
- **Projectiles**: Player bullets and enemy fire
- **Items**: Drops from defeated enemies (health, bombs, multipliers)
- **Effects**: Visual effects (explosions, hits, vignettes)
- **Bosses**: Special enemies with unique patterns and health bars

---

## Visual Design

### Color Palette
- **Background**: Deep navy (`#05122b`, `#0b1d3a`)
- **Sky Gradient**: Blue to cyan gradient for game area
- **UI**: White text with dark semi-transparent backgrounds
- **Accents**: 
  - Yellow (`#ffd54f`, `#fff176`) for score/UI elements
  - Lime green (`#7CFC00`) for weapon status
  - Orange (`#ff8a65`) for multiplier
  - Red-orange gradient for boss health bar

### Animation Techniques
- **CSS Animations**: Keyframe-based for UI elements
- **Canvas Transforms**: Real-time rendering for game entities
- **Transition Effects**: Smooth opacity/position changes for HUD
- **Blur Effects**: Backdrop filters on UI panels

---

## Input Handling

### Controls
- **Mouse/Touch**: Player aircraft follows pointer position for aiming
- **Click/Tap**: Weapon firing (continuous or single-shot depending on weapon)
- **Keyboard** (if implemented): Alternative control scheme
- **Button clicks**: UI controls for menu navigation

### Touch Optimization
- `touch-action: none` on canvas prevents default scroll behavior
- Removed tap highlight (`-webkit-tap-highlight-color: transparent`)
- Viewport configured for mobile: `maximum-scale=1, user-scalable=no`

---

## Performance Considerations

1. **Canvas Rendering**: Direct 2D context rendering for fast graphics
2. **No DOM manipulation during gameplay**: Game state isolated to Canvas
3. **Asset management**: No external asset loading; all visuals generated
4. **Animation optimization**: Using CSS animations where possible, Canvas for dynamic elements

---

## Development Notes

### Adding New Features

**New Weapon Type:**
- Define weapon properties (fire rate, damage, spread pattern)
- Implement drawing code in render function
- Add collision detection logic for new projectile type
- Update weapon switching UI

**New Enemy Type:**
- Create enemy object with position, health, AI pattern
- Implement movement and firing logic
- Add collision detection
- Design drop item behavior

**New Game Stage:**
- Define enemy waves
- Create boss encounter with health/patterns
- Set stage configuration (difficulty, enemy types)
- Add stage banner presentation

### Key Implementation Patterns

1. **Entity Management**: Likely uses arrays to track active entities
2. **Collision Detection**: Distance-based or AABB checking
3. **State Management**: Game states (menu, playing, boss, game-over)
4. **Visual Feedback**: Flash effects, vignettes, screen shake (if implemented)

---

## Testing & Running

### How to Run
1. Open `index.html` in any modern web browser
2. Game starts immediately on page load
3. Compatible with:
   - Desktop browsers (Chrome, Firefox, Safari, Edge)
   - Mobile browsers (iOS Safari, Chrome Mobile)
   - Tablets with touch support

### Browser Requirements
- HTML5 Canvas support
- ES6 JavaScript support
- CSS Grid/Flexbox support
- Touch events API (for mobile)

---

## File Statistics

- **index.html**: ~59 KB (all-in-one game implementation)
- **README.md**: Minimal (project title only)
- **Total source files**: 1 (everything in index.html)

---

## Future Enhancement Ideas

1. **Persistent storage**: LocalStorage for high scores/settings
2. **Mobile controls**: On-screen joystick UI
3. **Audio**: Sound effects and background music
4. **Advanced visuals**: Particle effects, screen shake, advanced animations
5. **Difficulty levels**: Variable difficulty progression
6. **Achievements**: Badge/achievement system
7. **Leaderboards**: Online high score tracking
8. **Level editor**: Custom stage creation

---

## Notes for Contributors

- **Single file design**: Maintain all-in-one structure for portability
- **Performance first**: Test on mobile devices before committing
- **Accessibility**: Ensure touch and keyboard controls work well
- **Visual consistency**: Follow the dark blue/cyan color scheme
- **Code organization**: Keep JavaScript organized in logical sections despite single file

---

**Last Updated**: 2026-10-01
**Project Status**: Active development
**Version**: 1.0 (Raiden-style vertical shoot-'em-up)
