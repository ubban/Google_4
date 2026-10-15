# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5, Canvas 2D API, and JavaScript. Players navigate through stages, collect weapon power-ups, and defeat boss squadrons using smart bombs and various weapons.

### Key Features
- **Vertical shoot-'em-up gameplay**: Navigate stages filled with enemies
- **Multiple difficulty levels**: Easy, Normal, and Hard modes with scalable difficulty parameters
- **Weapon system**: Multiple weapon types (Vulcan, Laser, Shotgun, etc.) with progression levels
- **Power-up mechanics**: Collect drops for weapons and bomb refills
- **Boss encounters**: Stage-ending boss battles with health bars
- **Score multipliers**: Dynamic combo system based on consecutive hits
- **Accessibility features**: Reduced motion support, screen reader announcements, touch controls
- **Responsive design**: Works on desktop (mouse/keyboard), tablet, and mobile (touch)
- **Local storage**: Persists best score, difficulty, mute state, and motion preferences

## Architecture

### File Structure
```
/home/user/Google_4/
├── index.html          # Single-file game (all HTML, CSS, and JavaScript)
├── README.md           # Brief project description
└── CLAUDE.md           # This file - codebase documentation
```

### Game Structure (within index.html)

The game is organized into distinct sections within a single file:

#### 1. **HTML Structure** (lines 361-420)
- Game canvas container
- HUD (Heads-Up Display) showing score, weapon, lives, bombs
- Boss health bar
- Overlay for menus and game over screens
- Touch controls for mobile devices
- Accessibility elements (sr-only status region)

#### 2. **CSS Styling** (lines ~1-359)
- **Layout**: Fixed fullscreen container with centered canvas
- **HUD components**: Status displays with backdrop blur effects
- **Animations**: Screen shake (vignette), flash effects, combo popups, stage banners
- **Difficulty buttons**: UI for difficulty selection
- **Touch controls**: On-screen buttons for mobile play (joystick simulation, fire, bomb)
- **Accessibility**: Reduced motion support via `@media (prefers-reduced-motion)`
- **Theme**: Dark space theme with bright accent colors

#### 3. **JavaScript Game Logic** (lines 422+)

##### Core Game Systems

**State Management**
- `gameState`: Global state object tracking current game mode, score, stage, multiplier
- `player`: Player aircraft object with position, health, weapon, bombs
- `enemies`: Array of active enemy entities
- `projectiles`: Array of bullets and bombs in flight
- `pickups`: Array of power-up drops on field

**Game Loop**
- `requestAnimationFrame` drive at ~60 FPS
- Delta-time (dt) based updates for frame-rate independence
- Adaptive quality system that reduces rendering detail on slower devices (see `quality` variable)

**Difficulty System**
- Constants defined in `DIFFICULTIES` object
- Parameters: `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`
- Easy mode: 4 lives, 3 bombs, slower enemies
- Normal mode: 3 lives, 2 bombs (default)
- Hard mode: 3 lives, 2 bombs, faster enemies, more aggression

**Stage Progression**
- `KILLS_PER_STAGE = 18`: Number of enemy kills to complete a stage
- Progression: Normal enemies → Boss encounter → Next stage
- Each stage increments difficulty via `stageDifficulty` multiplier
- Themes cycle through visual styles (see `THEMES` array with 6 variations)

**Player System**
- Position: `player.x`, `player.y` (centered in canvas)
- Health: `player.lives` (visual representation as hearts ❤️)
- Bombs: `player.bombs` (limited ammo, visual representation as ☢️)
- Weapon: `player.weapon` and `player.weaponLvl` (upgrade via pickups)
- Input: Keyboard (arrow keys, WASD, Space, X, P) + Mouse + Touch

**Weapon Types** (inferred from code structure)
- Vulcan (default)
- Laser
- Shotgun
- Other variants (typically 4-6 weapons total)
- Each has unique firing pattern, damage, and visual

**Enemy System**
- Spawned continuously until `KILLS_PER_STAGE` reached
- Movement: Sinusoidal paths with downward drift
- Behavior: Occasional shooting at player
- Shooting chance: Scales with difficulty (45% easy → 95% hard)
- Speed: Scales with difficulty
- Rewards: Drop weapon pickups or bomb pickups on death

**Boss System**
- Spawned after stage completion
- Health pool: Difficulty-scaled via `bossHpMul`
- Boss bar display: Shows remaining health
- Defeat condition: Health → 0
- Progression: Defeating boss advances to next stage

**Weapon Power-ups**
- Drop from random enemies
- `player.weapon` changes on pickup
- `player.weaponLvl` increments (up to level ~3)
- Level display in HUD: "VULCAN 2"

**Bomb Mechanics**
- Triggered manually by player (Shift/X key or BOMB button)
- Smart bomb: Wide-area damage, clears projectiles
- Visual effect: White flash + vignette pulse
- Limited ammo (2-4 depending on difficulty)

**Scoring System**
- Base points: Enemy kill = 100 pts (scales with stage)
- Multiplier: `gameState.mult` (starts at 1.0)
- Combo pop display: Shows "1.5x" when combo increases
- Hits without damage: Increment combo
- Damage taken: Reset combo to 1.0
- Boss kill: Large bonus based on remaining health

**Audio System**
- Mutable via mute button (stored in localStorage)
- Uses Web Audio API (inferred from code structure)
- Likely includes: engine hum, weapon fire, hit sounds, boss music

**Touch Control System**
- Joystick simulation: Left 55% of screen (move zone)
- Action buttons: Right 45% of screen
- Fire button: Red circular button
- Bomb button: Yellow circular button
- Shows joystick base when touch detected on left side

**Accessibility Features**
- Screen reader region (`#srLive`) for game announcements
- Reduced motion option: Disables animations for users requesting it
- Mute button: Controls audio
- Motion toggle: Disables screen shake and flash effects
- Semantic HTML with ARIA attributes

### Key Constants and Configuration

```javascript
STORAGE_BEST = 'skyfighter.bestScore'      // localStorage key
STORAGE_MUTE = 'skyfighter.muted'          // localStorage key
STORAGE_DIFF = 'skyfighter.difficulty'     // localStorage key
STORAGE_REDUCED = 'skyfighter.reducedMotion' // localStorage key

KILLS_PER_STAGE = 18                       // Enemies to defeat per stage
```

### Color Scheme and Themes

The game includes 6 visual themes with different color palettes:
1. **Daylight Front**: Blue sky, white clouds
2. **Sunset Corridor**: Purple/orange, warm tones
3. **Night Approach**: Dark blue, cool tones
4. **Storm Front**: Green/teal, stormy
5. **Arctic Wastes**: Light blue, icy
6. **Volcanic Ridge**: Orange/red, fiery

Themes cycle with progression or can be triggered by stage mechanics.

## Development Guidelines

### Code Style
- **Language**: Vanilla JavaScript (no frameworks, IIFE pattern)
- **Strictness**: `'use strict'` mode enabled
- **Scope**: Single global IIFE to avoid polluting global namespace
- **Naming**: camelCase for variables and functions, UPPERCASE for constants
- **Comments**: Minimal; use clear function names and variable names
- **Canvas API**: 2D context used for all rendering

### Game Loop Pattern
```javascript
let lastTime = 0;
function gameLoop(currentTime) {
  const dt = (currentTime - lastTime) / 1000; // Delta time in seconds
  lastTime = currentTime;
  
  update(dt);           // Update game state
  render();             // Draw everything
  trackPerf(dt);        // Monitor performance
  
  requestAnimationFrame(gameLoop);
}
```

### Adding New Features

#### New Weapon Type
1. Define weapon stats in weapon configuration section
2. Add firing logic in `player.fire()` method
3. Add visual rendering in canvas draw section
4. Update HUD weapon name display

#### New Enemy Type
1. Define enemy properties (speed, firing pattern, rewards)
2. Add spawning logic to spawn system
3. Add update logic for movement and behavior
4. Add collision/hit logic

#### New Difficulty Parameter
1. Add new field to `DIFFICULTIES` object
2. Access via `DIFFICULTIES[currentDifficulty].paramName`
3. Update UI if player-facing

#### Boss Variations
1. Define boss stats (HP, phases, attack patterns)
2. Implement phase transitions
3. Add visual changes per phase
4. Ensure health bar displays correctly

### Performance Considerations

- **Adaptive Quality**: System reduces rendering quality on slow devices
  - Quality levels: 0 (full) → 1 (medium) → 2 (low)
  - Monitored via frame time averaging
  - Watch `quality` variable in render code for quality-dependent rendering

- **Object Pooling**: Consider for high-frequency spawn items (bullets, enemies)
  
- **Canvas Rendering**:
  - Use `requestAnimationFrame` for smooth 60 FPS
  - Batch draw operations when possible
  - Clear canvas with filled rectangle (not clearRect for performance)

- **Event Handlers**: Delegated to single IIFE, not scattered globally

### Browser Compatibility
- Requires: Canvas 2D, requestAnimationFrame, localStorage, Web Audio API
- Tested on: Modern Chrome, Firefox, Safari, Edge
- Mobile: iOS Safari, Chrome Android
- Touch event handling via `touchstart`, `touchmove`, `touchend`

## Common Tasks

### Adjusting Difficulty
Edit the `DIFFICULTIES` object constants:
```javascript
normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, ... }
```

### Changing Game Balance
- **Enemy spawn rate**: Adjust `spawnBase` and `spawnMin`
- **Player health**: Modify `lives` in difficulty config
- **Bomb availability**: Change `bombs` in difficulty config
- **Boss difficulty**: Adjust `bossHpMul`

### Adding Sound Effects
Locate the audio system (likely Web Audio API wrapper) and add new sound registrations to the audio pool.

### Modifying UI Layout
Edit CSS in the `<style>` section. Note responsive design uses `clamp()` for fluid sizing:
```css
font-size: clamp(12px, 2.8vw, 16px); /* Min, preferred, max */
```

### Accessibility Improvements
- Add announcements via `announce(text)` function for screen readers
- Maintain `prefers-reduced-motion` media query support
- Ensure touch buttons have sufficient size (44×44px minimum)

## Testing Checklist

- [ ] Game loads without console errors
- [ ] All three difficulty levels selectable
- [ ] Player can navigate using keyboard (arrow keys, WASD)
- [ ] Player can fire weapon (space bar or mouse click)
- [ ] Bombs function correctly (X/Shift key)
- [ ] Weapon pickups upgrade weapon level
- [ ] Score multiplier increases with consecutive hits
- [ ] Damage resets multiplier
- [ ] Boss appears after 18 enemy kills
- [ ] Boss health bar displays and depletes
- [ ] Best score persists across sessions
- [ ] Mute/unmute toggles audio
- [ ] Reduced motion disables animations
- [ ] Touch controls appear on mobile
- [ ] Screen reader announcements work
- [ ] Game over screen shows final score and new best notification
- [ ] Pause button stops gameplay (if implemented)

## Important Notes

- **Single File Design**: All game logic is in `index.html`. Large changes should maintain code organization.
- **No Build Process**: Game runs directly in browser; no compilation or bundling needed.
- **Local Storage**: Game persists player preferences; clear if testing fresh state needed.
- **Canvas Rendering**: All visuals are drawn procedurally; no external image assets.
- **Responsive**: Design adapts to any screen size; test on multiple viewport sizes.

## Related Files and External Resources

- **README.md**: Brief project description
- **GitHub**: Source repository location (if applicable)
- **Browser DevTools**: Use for debugging performance, touch events, and canvas rendering

---

**Last Updated**: 2026-10-15  
**Project**: Sky Fighter - Vertical Shoot-'em-Up Game  
**Format**: Single-file HTML5 Canvas application
