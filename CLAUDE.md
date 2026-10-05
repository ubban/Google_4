# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-page HTML5 application. It's a classic arcade-inspired game featuring:
- Vertical scrolling gameplay with a player-controlled aircraft
- Multiple weapons with upgrade levels
- Smart bombs for special attacks
- Progressive difficulty stages with boss encounters
- Touch and mouse/keyboard controls
- Responsive design supporting desktop and mobile platforms

**Repository**: ubban/Google_4  
**Primary File**: `index.html` (~1,669 lines)  
**Technology**: Pure HTML5, CSS3, JavaScript (no external dependencies)

---

## Architecture

### Single-File Design
The entire game is implemented in `index.html` with three main sections:
1. **HTML Structure** - Canvas, HUD elements, overlays, controls
2. **CSS Styling** - Game aesthetics, animations, responsive design
3. **JavaScript Logic** - Game engine, entity management, physics, collision detection

### Key Game Systems

#### 1. **Rendering Pipeline**
- Canvas-based 2D graphics using CanvasRenderingContext2D
- Gradient backgrounds with parallax effects
- Sprite-based rendering for all entities
- Real-time HUD updates (health, score, weapon info)
- Post-processing effects: vignette, flash animations

#### 2. **Entity Management**
Game entities include:
- **Player**: Configurable movement speed, weapons, health, bombs
- **Enemies**: Basic enemies, elite enemies, bosses with health bars
- **Projectiles**: Player bullets, enemy bullets with varied patterns
- **Explosions/Particles**: Visual feedback for hits, bombs, stage clear
- **Power-ups**: Weapon upgrades, health, bomb refills

#### 3. **Collision Detection**
- AABB (Axis-Aligned Bounding Box) collision system
- Entity bounds calculated from visual representations
- Separate collision checks for:
  - Player vs. enemy bullets
  - Player bullets vs. enemies
  - Player vs. power-ups
  - Boss hit detection

#### 4. **Game State Management**
State includes:
- `PLAYING` - Active gameplay
- `PAUSED` - Game pause (keyboard only)
- `GAME_OVER` - End state with restart options
- Stage progression tracking
- Score, combo multiplier, best score persistence

#### 5. **Weapon System**
Multiple weapons with upgrade levels (1-3):
- **Rifle**: Default balanced weapon
- **Cannon**: Single powerful shot
- **Spread**: Multiple angled shots
- **Laser**: Concentrated beam
- **Burst**: Rapid fire rounds

#### 6. **Input Handling**
- **Mouse/Keyboard**: WASD/Arrow keys for movement, Spacebar for shoot, E for bombs
- **Touch**: Virtual joystick for movement, tap buttons for shoot/bomb
- **Device Orientation**: Motion sensors for tilt-based aiming (iOS)
- **Pause**: P key to pause/unpause
- **Mute**: M key to toggle sound

#### 7. **Audio System**
- Sound effects for shoots, explosions, pickups, stage complete
- Background music playback
- User-controlled mute toggle
- Web Audio API with fallback to HTML5 audio elements

#### 8. **Persistence**
LocalStorage keys:
- `sky_fighter_best` - High score
- `sky_fighter_mute` - Audio mute state
- `sky_fighter_diff` - Difficulty selection
- `sky_fighter_reduced_motion` - Accessibility setting

#### 9. **Difficulty System**
Three difficulty levels:
- **Easy**: Reduced enemy spawning, simple patterns
- **Normal**: Balanced gameplay
- **Hard**: Increased enemy count, complex bullet patterns, tougher bosses

#### 10. **Stage System**
Progressive stages with:
- Incrementally harder enemies
- Boss encounters at stage conclusions
- Stage banners and visual progression
- Score multipliers based on stage completion speed

---

## Code Structure Breakdown

### HTML Elements (ID References)
- `#gameCanvas` - Main rendering canvas
- `#container` - Full-screen game container
- `#overlay` - Game over/start screen
- `#hud` - Heads-up display (stats, score, health)
- `#gameContainer` - Parent wrapper
- `#touchControls` - Mobile joystick and buttons
- `#moveZone` - Touch movement area
- `#fireBtn`, `#bombBtn` - Action buttons
- `#bossBarWrap`, `#bossBarFill` - Boss health indicator

### JavaScript Sections
1. **Global State Variables** (~50-100 lines)
   - Canvas and context references
   - DOM element cache
   - Game state flags
   - Entity arrays (enemies, bullets, particles)
   - Player state object

2. **Utility Functions** (~200-300 lines)
   - Random number generation
   - Distance/collision calculations
   - Audio management (play sounds)
   - LocalStorage helpers (saveScore, loadScore)

3. **Game Loop** (~100-150 lines)
   - `update()` - Logic updates each frame
   - `draw()` - Rendering
   - `gameLoop()` - RAF loop coordinator
   - Timing and delta calculations

4. **Player Logic** (~300-400 lines)
   - Movement calculation
   - Weapon firing and cooldowns
   - Bomb mechanics
   - Health and damage handling
   - Input event listeners

5. **Enemy System** (~400-500 lines)
   - Enemy spawning based on stage/difficulty
   - Movement patterns
   - Bullet firing behaviors
   - Boss entity definition
   - Collision responses

6. **Particle/Visual Effects** (~200 lines)
   - Explosion particle generation
   - Combo pop animations
   - Vignette and flash effects
   - Text rendering (score popups)

7. **Collision & Physics** (~150-200 lines)
   - Bounding box calculations
   - Collision detection loops
   - Damage application
   - Power-up collection

8. **UI & State Management** (~200-300 lines)
   - Game over screen display
   - HUD updates (score, lives, weapon)
   - Stage transitions
   - Start/restart logic
   - Difficulty selection

9. **Event Handlers** (~100-150 lines)
   - Keyboard events (movement, shoot, bomb)
   - Touch events (virtual joystick)
   - Device orientation (tilt aiming)
   - Button click handlers

---

## Development Guidelines

### Adding New Features

#### New Weapon Type
1. Add weapon definition to weapon list
2. Create firing logic in `player.fire()` update
3. Add visual rendering in player draw section
4. Update HUD weapon display

#### New Enemy Type
1. Define enemy object in spawning logic
2. Implement movement pattern in entity update
3. Add bullet firing behavior
4. Create collision damage response
5. Add visual rendering

#### New Visual Effect
1. Create particle generator function
2. Add update and draw logic to main game loop
3. Integrate into appropriate game event (hit, pickup, explosion)

### Performance Considerations
- Canvas rendering is single-threaded; keep entity counts reasonable
- Update collision only for active on-screen entities
- Pool particles to avoid allocation overhead
- Minimize DOM mutations; batch HUD updates

### Mobile Optimization
- Touch controls automatically show on non-desktop devices
- Motion sensor integration for enhanced aiming
- Reduced motion detection for accessibility
- Virtual joystick provides analog movement feel
- Full-screen and safe-area inset support

---

## Browser Compatibility

- **Modern Browsers**: Full support (Chrome, Firefox, Safari, Edge)
- **Mobile**: iOS Safari, Chrome Android, Samsung Internet
- **Fallbacks**:
  - Audio element for Web Audio API unsupported browsers
  - Touch events for mouse input on touch devices
  - Orientation API for motion sensing

---

## Testing & Debug Notes

### Console Commands (Add to Dev Tools for Testing)
```javascript
// Examples of useful debugging:
game.player.health = 10;  // Set player health
game.stage = 5;           // Jump to stage
game.score = 50000;       // Set score
game.stage_clear = true;  // Trigger stage clear
```

### Known Limitations
- No level editor (stages hardcoded)
- Single player only
- No online leaderboard (localStorage only)
- Audio may not autoplay on mobile (user gesture required)
- Motion sensing limited to iOS

---

## File Checklist for Contributors

When modifying the codebase:
- [ ] Test on desktop (Chrome, Firefox, Safari)
- [ ] Test on mobile (iOS, Android)
- [ ] Verify touch controls responsiveness
- [ ] Check localStorage persistence works
- [ ] Test all difficulty levels
- [ ] Verify score and best score tracking
- [ ] Test audio mute/unmute
- [ ] Check collision detection after changes
- [ ] Validate responsive layout at different screen sizes

---

## Future Enhancement Ideas

1. **Level Editor** - Allow custom stage creation
2. **Leaderboard** - Cloud-based high scores
3. **Power-ups** - Shield, rapid fire, slow-motion
4. **Bosses** - More varied boss patterns
5. **Achievements** - Unlock system
6. **Replays** - Record and playback gameplay
7. **Music/Sound** - Procedural audio or expanded SFX
8. **Themes** - Dark/light mode, color schemes
9. **Tutorial** - Guided first-time player experience
10. **Controller Support** - Gamepad API integration

---

## Last Updated
2026-10-05

## Maintainer
Claude Haiku 4.5
