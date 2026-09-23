# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game implemented as a single HTML5 file with embedded JavaScript and CSS. The entire game (logic, rendering, audio, UI) is contained in `index.html`.

The game features:
- Vertical scrolling gameplay with player-controlled aircraft
- Multiple weapon types (guns, lasers) with ammo management
- Smart bombs with screen-clearing capability
- Progressive stages with unique visual themes
- Boss encounters and stage progression
- Score tracking with persistent best-score storage
- Sound effects and screen shake effects
- Full keyboard and mouse/touch support

## Running and Testing

### Open the Game Locally
Simply open `index.html` in any modern web browser:
```
# In browser: File → Open → index.html
# Or serve locally if needed:
python3 -m http.server 8000
# Then visit: http://localhost:8000
```

### Testing Workflow
1. Make changes to `index.html`
2. Save the file
3. Refresh the browser (Ctrl+R / Cmd+R)
4. Test gameplay features: movement, weapons, enemies, bosses, scoring

### Quick Test Checklist for Changes
- Does the game still start without console errors?
- Does player movement respond correctly to keyboard/mouse?
- Do weapons fire and hit detection work as expected?
- Do screen effects (shake, flash) trigger appropriately?
- Does pause/resume work?
- Does game-over state trigger correctly?
- Is the score display and best-score persistence working?

## Code Architecture

The single HTML file is organized into these logical sections (search by these patterns to navigate):

### 1. **HTML & Meta Tags** (top of file)
- Viewport and responsive meta tags for mobile support
- Favicon and theme color
- Canvas container and HUD elements

### 2. **CSS Styling**
- Dark theme styling with glassmorphism effects for UI
- Canvas fullscreen layout
- HUD stat display with flexible spacing (responsive to viewport)
- Animations for screen effects (vignette pulse, heart beat, flash)
- Mobile-friendly responsive font sizing using `clamp()`

### 3. **Game Configuration & Constants** (after `<script>`)
- `THEMES`: Array of stage themes (sky colors, sun, clouds)
- `STORAGE_BEST`: Key for localStorage best-score persistence
- Canvas dimensions and FPS settings
- Entity dimensions (player, enemy, bullet sizes)

### 4. **Audio System (SFX)**
- Web Audio API object (`sfx`) with methods: `shoot()`, `hit()`, `bomb()`, `levelUp()`, `gameover()`, `uiClick()`
- Generates tones programmatically rather than using external audio files
- Frequency/duration tuning for arcade-style sounds

### 5. **Game State**
- `player`: Position, velocity, health, weapon type, bomb count, invulnerability frames
- `bullets`, `enemies`, `bosses`: Pooled/managed arrays
- `score`, `stage`, `frame` (game tick counter)
- Pause/game-over/running flags

### 6. **Input Handling**
- `keys`: Object tracking pressed keys (WASD, arrow keys, Space, Enter, P)
- Mouse/touch tracking for targeting crosshair
- Button click handlers for play/pause/resume

### 7. **Core Game Loop** (`animate()` function)
The frame-by-frame game loop handles:
1. Tick game state if not paused
2. Update player position based on input
3. Fire weapons if holding Space
4. Update and draw bullets
5. Spawn and update enemies/bosses
6. Check collisions (bullets vs enemies, player vs enemies, bombs)
7. Spawn powerups and handle collection
8. Draw everything via `draw()` function
9. RequestAnimationFrame callback for next frame

### 8. **Collision & Damage**
- Separate functions for player-enemy collision, bullet-enemy collision, bomb effects
- Invulnerability frames prevent rapid repeated damage
- Damage numbers spawned as visual feedback

### 9. **Drawing Functions**
- `draw()`: Main renderer—clears canvas, applies screen shake, draws background gradient, clouds, entities
- `drawPlane()`: Draws the player or enemy aircraft sprite
- `drawBoss()`: Boss-specific rendering with health bar
- `drawClouds()`: Parallax scrolling cloud background

### 10. **Enemy & Boss AI**
- Enemy pattern arrays (`PATTERNS`) define movement behavior
- Boss spawning tied to stage progression
- Boss health, attack patterns, and phase transitions

### 11. **UI & Overlays**
- HUD elements: score, stage, health (hearts), bombs, weapon type, multiplier
- Overlay screen for start/pause/game-over states
- Screen effects: vignette pulse (damage), white flash (hit/bomb), screen shake

### 12. **Audio & Feedback**
- Sound effects for actions (shoot, hit, bomb, level up, game over, UI click)
- Screen shake on damage and bomb detonation
- Flash effects on hit and bomb explosion
- Damage numbers as visual feedback

## Key Data Structures

### Player Object
```javascript
{
  x, y,              // Position
  vx, vy,            // Velocity
  w, h,              // Width/height (collision box)
  health,            // Lives remaining
  weapon,            // 'gun' or 'laser'
  bombs,             // Smart bomb count
  invuln,            // Invulnerability frames counter
  shootCooldown      // Rate limit for firing
}
```

### Enemy/Bullet Objects
Arrays (`enemies`, `bullets`) contain objects with position, velocity, health, type, and damage properties.

### Boss Objects
Enemies with large health pools, attack patterns, and special rendering.

## Common Development Tasks

### Adding a New Weapon Type
1. Add weapon name to player's `weapon` property options
2. Add firing logic in the input handler (around the Space key firing code)
3. Create bullet spawn/velocity pattern for the weapon
4. Add drawing code in the `draw()` function if visually distinct
5. Adjust ammo/cooldown values for balance

### Adjusting Difficulty
- Enemy spawn rates: Look for `if (frame % X === 0)` spawn conditions
- Enemy health values: In enemy objects
- Player health: Initial `player.health` value
- Bullet damage: In collision detection functions

### Adding a New Stage Theme
1. Add entry to `THEMES` array with `{ sky: [...], sun: '...', cloud: '...' }`
2. Increase `MAX_STAGE` or ensure boss spawning wraps correctly
3. Test the stage progression in gameplay

### Tweaking Game Feel
- Screen shake: Adjust `screenShake()` duration/magnitude parameters
- Weapon cooldown: Modify `shootCooldown` rates
- Player speed: Adjust velocity increment in movement code
- Enemy bullet speeds: Modify velocity values in firing patterns

## Browser Compatibility

- Requires modern browser with Canvas, Web Audio API support
- Tested on Chrome, Firefox, Safari (desktop and mobile)
- Mobile touch controls work via `touch` event listeners
- Fallback for no Web Audio support (silent mode)

## Performance Notes

- Single canvas render target keeps draw calls minimal
- Object pooling for bullets/enemies (reuse rather than create/destroy)
- RequestAnimationFrame for 60 FPS throttle
- Frame counter (`frame`) used instead of timestamps for deterministic updates
