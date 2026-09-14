# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single, self-contained HTML file. The entire game (HTML structure, CSS styling, and JavaScript logic) is embedded in `index.html`, with no external dependencies or assets—all audio is synthesized using the WebAudio API.

## Game Architecture

### Core Game Loop & Rendering
The game runs on a `requestAnimationFrame`-based loop that:
1. Updates game state (player position, enemies, projectiles, etc.)
2. Renders the canvas with dynamic backgrounds (sky gradient)
3. Updates the HUD (score, lives, weapon level, etc.)
4. Handles collision detection

The canvas is set to responsive sizing (scales with viewport) and uses a gradient sky background that varies by stage theme.

### Game States & Flow
- **Menu/Start**: Overlay card with score display, difficulty selection, and start button
- **Playing**: Active game loop with enemies spawning, player shooting, and collision detection
- **Paused**: Game loop paused; resume on button press or escape key
- **Game Over**: Overlay shows final score, best score, and restart button

### Key Game Subsystems

#### Enemy Spawning & Difficulty
Three difficulty presets control enemy spawn rate, speed, shoot chance, and lives:
- `easy`: Slower spawns, weaker enemies, 4 lives, 3 bombs
- `normal`: Balanced (default), 3 lives, 2 bombs
- `hard`: Aggressive spawns, fast enemies, 3 lives, 2 bombs

Enemy types include `grunt` (basic), `interceptor` (fast, weaving), and `tank` (slow, durable). New enemy stages unlock at `minStage` thresholds defined in `ENEMY_TYPES`.

#### Boss System
Six boss variants (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) spawn after eliminating `KILLS_PER_STAGE` (18) enemies. Each boss has unique colors, HP multiplier, and attack patterns. Boss health bar displays at top-center during boss fights.

#### Weapon System
The player starts with the VULCAN weapon and can pick up weapon upgrades (LASER, PLASMA) from defeated enemies. Weapons have:
- Fire rate and projectile behavior (spread, homing, etc.)
- Visual and audio feedback
- Level system (e.g., VULCAN lvl 1–6)

#### Scoring & Multiplier
- Base score from enemy kills
- Kill multiplier increases with consecutive hits; resets on player damage
- Combo pop animation displays multiplier milestones

#### Difficulty Scaling
As game progresses (by stage), spawn rates decrease and enemy health increases. A quality-adjustment system monitors frame times and scales rendering quality (visual detail) to maintain 60 FPS on slower devices.

### Themes
Six visual themes (Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge) define sky gradients and cloud colors; selected randomly each run.

## UI & Input

### Desktop Controls
- **Arrow Keys** or **WASD**: Move player
- **Space**: Fire weapon
- **B**: Drop bomb
- **Escape**: Pause/Resume
- **S**: Mute/Unmute audio
- **M**: Toggle reduced motion effects

### Touch Controls
On mobile, two zones appear:
- Left zone (55% width): Movement input; shows optional joystick visual on touch
- Right zone (45% width): Fire and Bomb buttons

### Accessibility
- **Reduced Motion toggle**: Disables screen shake, flash effects, and animations; respects browser `prefers-reduced-motion`
- **Screen reader announcements**: Lives lost, bombs used, combos, stage transitions announced via `#srLive` (aria-live region)
- **Mute button**: Toggles all audio

## File Structure & Code Organization

The entire game is in `index.html`:
1. **Lines 1–359**: CSS styling for layout, HUD, overlay, animations, and responsive design
2. **Line 360–399**: HTML structure (canvas, HUD elements, overlay, touch controls)
3. **Lines 400+**: Inline JavaScript

### JavaScript Sections (approximate)
- **Audio System** (~lines 542–650): WebAudio synth for laser, bomb, boss, and music sounds
- **Game State & Constants** (~lines 485–523): Difficulty presets, enemy types, boss data, themes
- **Game Objects** (~lines 700–1200): Player, Enemy, Projectile, Particle classes
- **Game Loop & Collision** (~lines 1200–1500): Main update/render loop, collision detection
- **Event Handlers** (~lines 1500+): Keyboard, mouse, touch, and button events
- **UI Updates** (~lines after 1500): HUD rendering, overlay management, score display

## Common Development Tasks

### Modifying Game Difficulty or Balance
Edit the `DIFFICULTIES` object (~line 485):
```javascript
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, ... },
  // adjust spawnBase/spawnMin for spawn rate, enemySpeed for difficulty
};
```

### Adding a New Enemy Type
Add an entry to `ENEMY_TYPES` (~line 502):
```javascript
const ENEMY_TYPES = {
  newType: { w: 40, h: 40, hp: 2, speedMul: 1.2, color: '#...', dark: '#...', score: 25, minStage: 3 },
  // ...
};
```
Then add rendering logic in the enemy draw method.

### Adding a New Boss
Add entry to `BOSSES` (~line 508) with unique colors and HP multiplier. Boss spawn is triggered when enemy kill count reaches `KILLS_PER_STAGE`.

### Changing Visual Themes
Modify the `THEMES` array (~line 493) to adjust sky gradients, cloud colors, and sun colors. Each theme is randomly selected per game run.

### Adjusting Audio
The audio system uses WebAudio oscillators and filters. Modify `beep()` function parameters (frequency, duration, type, volume, sweep) to change sound effects. Music loops are synthesized in real-time; adjust in `playMusic()`.

### Customizing Touch Controls
Touch control zones are defined in CSS (~lines 291–341). Adjust widths, heights, and button sizing as needed. The joystick base shows on touch input in the move zone.

### Mobile Responsiveness
The entire layout uses `clamp()` for fluid scaling. Canvas and HUD are responsive. Adjust breakpoints or `max-width` constraints in CSS if needed for specific devices.

## Performance & Optimization

### Frame Time Monitoring
A `frameTimes` array tracks recent frame deltas. If average > 1/40 FPS, rendering quality decreases; if < 1/56 FPS (i.e., very smooth), quality increases. Adjust thresholds in the quality loop if needed.

### Rendering Pipeline
- Clear canvas with gradient sky
- Draw enemies, projectiles, and particles
- Draw player
- Apply screen effects (vignette on damage, flash on bomb)
- Render HUD text

Avoid adding expensive DOM manipulation inside the game loop; batch updates outside the loop.

### LocalStorage Usage
The game persists best score, mute preference, difficulty, and reduced-motion setting to `localStorage` using keys defined as constants. Clear storage if you need to reset game state during testing.

## Browser Compatibility

- Requires HTML5 Canvas for rendering
- WebAudio API for audio (gracefully degrades to silence if unavailable)
- Touch Events API for mobile input
- LocalStorage for persistent settings
- CSS Grid/Flexbox for layout
- ES6+ JavaScript (arrow functions, const/let, template literals)

Tested on modern desktop and mobile browsers. Avoid older IE/Edge versions.

## Testing & Debugging

### Quick Play Testing
Open `index.html` in a browser to play. Difficulty can be toggled in the start overlay. Best score and settings persist across sessions.

### Console Debugging
Key game state can be logged:
- `gameState` (enum: start, playing, paused, gameOver)
- `player` object (x, y, hp, ammo, weapon)
- `enemies` array (current active enemies)
- `score` and `mult` (current score and multiplier)

Add `console.log()` statements in the main game loop to monitor state or performance metrics.

### Mute/Audio Toggling
Audio initialization is lazy (on first user interaction). Test mute button and audio mute preference with the UI toggle.

### Reduced Motion Testing
Enable reduced-motion toggle in game and verify that screen shake, vignette pulses, and combo animations are disabled. Test with browser `prefers-reduced-motion` setting to verify it's respected.

### Mobile/Touch Testing
Open on a mobile device or use browser DevTools device emulation. Verify touch controls respond correctly to touch input and that layout is readable on small screens.

## Extending the Game

### Adding New Features
1. **New Weapon**: Add to weapon definitions; implement firing and rendering logic.
2. **Particle Effects**: Extend the `Particle` class and add spawning calls for visual impacts.
3. **Power-ups**: Create a powerup spawning and collision system separate from enemy drops.
4. **Leaderboard**: Extend `localStorage` with a leaderboard array and add a leaderboard view.
5. **Sound Options**: Add volume slider using `masterGain.gain.value`.

### Structural Refactoring
The current single-file approach prioritizes simplicity and no external dependencies. If refactoring:
- Keep the game loop and collision detection performant
- Avoid DOM manipulation inside the loop
- Maintain a clean separation of update/render phases
- Test thoroughly on mobile and slow devices

## Known Limitations & Future Improvements

- Boss AI is scripted pattern-based; more complex pathfinding could add depth
- No persistent leaderboard (only best score)
- Limited to one stage theme per run; could allow theme selection
- No replay/recording feature
- Touch joystick is visual-only (no haptic feedback)

---

**Last Updated:** 2026-09-14  
**Target Model:** Claude 3.5 Sonnet and above  
**Language:** JavaScript (ES6+)
