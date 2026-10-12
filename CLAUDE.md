# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The game features dynamic combat with weapon switching, smart bombs, progressive stages, and challenging boss encounters. It's designed to be fully responsive and playable on both desktop and mobile devices.

**Tech Stack:**
- Pure HTML5 Canvas for rendering
- Vanilla JavaScript (no external dependencies)
- CSS3 for styling and animations
- Mobile-optimized with touch controls

**Repository:** `ubban/Google_4`

## Getting Started

### How to Play

1. **Open the Game:** Open `index.html` in any modern web browser
2. **Desktop Controls:**
   - **Move:** Arrow keys or WASD
   - **Aim/Shoot:** Mouse position (crosshair follows cursor)
   - **Shoot:** Click or spacebar
   - **Switch Weapon:** Z (previous) or X (next)
   - **Smart Bomb:** C
3. **Mobile Controls:**
   - **Move:** Tap and drag your ship
   - **Aim/Shoot:** Tap anywhere on screen to shoot
   - **Switch Weapon:** On-screen buttons or swipe gestures
   - **Smart Bomb:** Button in UI

## Game Mechanics

### Player Ship
- Starts with 3 hearts (health points)
- Takes damage from enemy fire and collisions
- Game ends when all hearts are lost
- Player position updates based on arrow keys/WASD or touch input

### Weapons System
Cycle between weapons using Z/X or on-screen buttons:
- **Bullet:** Standard rapid-fire weapon
- **Laser:** More powerful, pierces through enemies
- **Spread:** Fires multiple projectiles in a cone pattern

Each weapon has different damage values and fire rates, displayed in the HUD.

### Smart Bombs
- Limited supply (shown as bomb indicators in HUD)
- Clear all on-screen enemies and projectiles
- Can be used strategically for dangerous situations
- Visual feedback with full-screen flash animation

### Combo System
- Kill enemies in quick succession to build multiplier
- Multiplier displayed in orange on HUD
- Higher multipliers increase score value per enemy
- Visual feedback with floating "+Combo" text
- Resets if player takes damage or goes too long without kills

### Scoring
- Base points for defeating enemies (varies by enemy type)
- Multiplied by current combo count
- Final score displayed on game over screen

### Stage Progression
- Game divided into multiple stages
- Each stage starts with stage banner ("STAGE 1", etc.)
- Boss appears at end of each stage
- Stage banner animations and transitions handled automatically

### Boss Battles
- Larger health bar displayed at top when boss appears
- Boss name shown above health bar
- Visual indicators for critical damage (red vignette effect)
- Defeat boss to advance to next stage
- Each boss has unique behavior and attack patterns

## Game Architecture

### Single-File Structure
All code is embedded in `index.html`:
- **Head section:** Meta tags, styling (CSS), favicon
- **Body section:** Canvas element, HUD elements, UI containers
- **Script section:** Complete game logic in JavaScript

### Key Game Objects

**Game State:**
- `score`: Current score with combo multiplier
- `combo`: Current kill combo count
- `health`: Player health (hearts)
- `bombs`: Remaining smart bombs
- `weapon`: Current weapon selection
- `stage`: Current stage number
- `gameOver`: Game state flag

**Game Entities:**
- `player`: Ship object with position, size, weapon state
- `enemies[]`: Array of active enemies
- `projectiles[]`: Array of active projectiles (player and enemy)
- `particles[]`: Visual effects (explosions, text popups)

### Rendering Pipeline
1. Clear canvas with gradient background
2. Draw game entities (player, enemies, projectiles)
3. Update particle effects
4. Update HUD (score, health, weapon, combo)
5. Handle boss health bar
6. Process user input and physics

### Game Loop
- Runs at 60 FPS using `requestAnimationFrame`
- Each frame: update, render, handle collisions, check game state
- Collision detection between: player-enemy, player-projectile, projectile-enemy

## User Interface Elements

### HUD (Heads-Up Display)
Located at top-center of screen:
- **Score:** Current score with combo multiplier indicator
- **Health:** Heart icons showing remaining lives
- **Weapon:** Current weapon name and type
- **Bombs:** Bomb indicators showing available smart bombs

### Boss Bar
Appears only during boss fights:
- Boss name centered above health bar
- Health bar with gradient fill (red-orange)
- Smooth width transitions as damage is taken

### Status Effects
- **Red Vignette:** Pulses when player health is critical (1 heart remaining)
- **Flash:** White screen flash on taking damage or using bomb
- **Combo Popup:** Yellow floating text showing combo count
- **Stage Banner:** Large animated text announcing stage start

### Top-Right Controls
- Settings button (gear icon)
- Menu button (hamburger menu)
- Responsive sizing based on viewport

## Visual Design

### Color Scheme
- **Background:** Dark blue (#05122b) - calm, space-like atmosphere
- **Canvas:** Gradient blue sky (from dark to light)
- **UI:** Dark semi-transparent elements with light borders
- **Accents:** Gold/yellow for scores, green for weapons, orange for multipliers
- **Critical State:** Red tinted vignette effect

### Animations
- Heart beat animation when critical health
- Smooth combo popup fade and scale
- Stage banner slide-in and fade-out
- Boss bar width transition on damage
- Full-screen flash effects for damage and bombs

### Responsive Design
- Uses `clamp()` for font sizing across all viewport sizes
- Responsive layout using flexbox
- Canvas scales to fit device screen
- Safe area insets for notched devices

## Mobile Optimization

- Touch event handling for movement and shooting
- Tap-to-shoot mechanics
- Haptic feedback ready (placeholder)
- No horizontal scrolling
- Viewport meta tag prevents zoom
- Dark color scheme reduces eye strain

## Development Notes

### Adding New Features

**New Enemy Type:**
1. Create enemy object in enemy spawning code
2. Define sprite/rendering in canvas draw section
3. Add collision logic in collision detection
4. Set health, behavior, and drop rewards

**New Weapon:**
1. Add weapon type to weapon array
2. Update weapon cycle logic
3. Define firing pattern in shoot() function
4. Update HUD display logic
5. Balance damage and fire rate

**New Stage:**
1. Add stage configuration with enemy waves
2. Create boss object with health and behavior
3. Update stage progression logic
4. Add stage banner display

### Performance Considerations
- Uses `requestAnimationFrame` for smooth 60 FPS
- Object pooling not implemented (could improve performance)
- Canvas rendering is hardware-accelerated
- Mobile touch events debounced appropriately

### Known Limitations
- Single file makes organization challenging for large projects
- No persistent save data
- Audio would require additional implementation
- Particle system is simple (no advanced effects)

## Customization

You can customize the game by modifying:
- **Colors:** Update CSS variables in `:root` style section
- **Difficulty:** Adjust enemy spawn rates, health, damage values
- **Game Balance:** Modify weapon damage, fire rates, bomb effects
- **UI Layout:** Adjust HUD positioning and sizing in CSS

## Browser Compatibility

- Chrome/Edge: Full support
- Firefox: Full support
- Safari: Full support
- Mobile browsers: Full support with touch controls

## Future Enhancement Ideas

- Sound effects and background music
- Multiple difficulty levels
- Leaderboard/high score saving (localStorage)
- Power-ups and special items
- More weapon variety
- Animated backgrounds
- Pause functionality
- Tutorial/how-to screen
- Additional stages and bosses
- Different game modes

---

**Last Updated:** October 2026
