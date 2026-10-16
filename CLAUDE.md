# Sky Fighter — Game Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. Players control a fighter jet, switch between weapon types, drop bombs, and defeat stage bosses.

**Repository**: `ubban/google_4`  
**Main File**: `index.html` (1669 lines)  
**Tech Stack**: HTML5, CSS3, Canvas 2D, Vanilla JavaScript

## Quick Start

1. Clone/open the repository
2. Open `index.html` in a web browser
3. Use keyboard/mouse to play:
   - **Arrow Keys** or **Mouse**: Move player
   - **Spacebar** or **Click**: Shoot
   - **E Key**: Switch weapon
   - **Q Key** or **W Key**: Drop bomb (smart bombs)

## Architecture

### Game Structure (Single File Design)

The entire game is contained in `index.html` with three main sections:

#### 1. **HTML Structure** (lines 1–200+)
- `<canvas id="gameCanvas">` – Main game rendering surface
- `#hud` – Heads-up display (health, bombs, weapon, multiplier)
- `#bossBarWrap` – Boss health bar
- `#vignette` – Red flash effect on damage
- `#flash` – Screen flash animations
- `#comboPop` – Combo multiplier popup
- `#stageBanner` – Stage intro banner
- `#topBar` – Control buttons (mute, settings, fullscreen)
- `#overlay` – Game over/pause screen

#### 2. **CSS Styling** (embedded `<style>`)
- Dark space theme with gradient sky background
- Responsive design using `clamp()` for scaling
- Keyframe animations for UI effects (heartbeat, vignette pulse, combo animation)
- Fullscreen and mobile-friendly layout with `env(safe-area-inset-*)`

#### 3. **JavaScript Game Engine** (lines ~800–1669)

Core systems:

- **Canvas 2D Rendering**: All graphics drawn to canvas with sprite-like rendering
- **Game State Manager**: Tracks player health, score, stage, wave number
- **Entity System**: Player ship, enemies, bullets, bombs
- **Physics**: Basic collision detection between bullets/enemies and player/enemies
- **Wave System**: Enemies spawn in waves; bosses appear after certain waves
- **Weapon System**: Multiple weapon types (default gun, spread shot, etc.) with switching
- **Bomb System**: Limited smart bombs that clear enemies when detonated
- **Audio**: Sound effects for shooting, hits, explosions (via Web Audio API)

### Key Game Mechanics

1. **Player Control**: 
   - Moves with arrow keys or mouse
   - Always fires upward
   - Limited to screen boundaries

2. **Enemies**:
   - Spawn in waves with increasing difficulty
   - Move in patterns and shoot at player
   - Drop items on defeat (health, ammo, multiplier bonuses)

3. **Bosses**:
   - Appear after wave 3, 6, 9, etc.
   - Display unique visual design and attack patterns
   - Defeat triggers stage completion

4. **Weapons**:
   - Standard gun (rapid fire)
   - Spread shot (cone pattern)
   - Switchable with 'E' key
   - Damage scales with multiplier

5. **Health System**:
   - Player starts with 3 hearts
   - Enemy bullets deal 1 damage each
   - Lose game when health reaches 0
   - Health pickups restore 1 heart

6. **Bombs**:
   - Player starts with 3 bombs
   - Deal massive damage to all enemies on screen
   - Triggered with 'Q' or 'W' key
   - Cooldown timer before reuse

7. **Scoring & Multiplier**:
   - Defeating enemies increases combo multiplier
   - Multiplier resets on player damage
   - Combo pickups extend multiplier duration

## Development Notes

### Code Style & Organization

The JavaScript code is organized in logical sections within the `<script>` tag:

1. **Initialization**: Canvas setup, variable declarations
2. **Entity Classes**: Player, Enemy, Bullet, Bomb classes
3. **Game Loop**: Main update and render functions
4. **Input Handlers**: Keyboard/mouse event listeners
5. **Utility Functions**: Collision detection, random values, visual effects

### Important Implementation Details

- **Canvas Rendering**: Uses simple shape rendering (circles, rectangles, paths) rather than sprite sheets
- **Collision Detection**: Uses bounding box (AABB) collision tests
- **Particle Effects**: Temporary visual effects for explosions/hits
- **Wave Progression**: Difficulty increases per wave; boss waves trigger at predetermined points
- **Performance**: Single canvas redraw per frame (~60fps target)

### Common Modifications

**To add a new weapon type:**
1. Add weapon stats in the weapons configuration object
2. Implement firing pattern in the player's shoot method
3. Add switching logic in the weapon swap handler

**To add new enemy types:**
1. Create a new Enemy subclass or variant
2. Define movement pattern and firing behavior
3. Add to enemy spawning system with appropriate wave

**To adjust difficulty:**
1. Modify enemy spawn rates in wave definitions
2. Adjust boss health and attack patterns
3. Change damage values for bullets and bombs

**To add new stages/bosses:**
1. Define stage layout and wave progression
2. Create boss entity with unique sprite and attack pattern
3. Configure boss health, phase changes, and rewards

## Testing & Debugging

- Open browser DevTools (F12) to see console for errors
- Use browser's Performance tab to monitor frame rate
- Test on multiple browsers and devices for compatibility
- Mobile testing: Use responsive design mode in DevTools

## Build & Deployment

The game requires no build process—simply host `index.html` on any web server:

```bash
# Local testing
python3 -m http.server 8000
# Visit http://localhost:8000
```

## Performance Considerations

- Canvas rendering is optimized for ~1500–2000 entities max
- Bullet limit of 200 per player to prevent lag
- Enemy limit per wave: ~50 active enemies
- Consider reducing particle effects on low-end devices

## Future Enhancement Ideas

- Add sound effects and background music
- Implement difficulty levels (Easy/Normal/Hard)
- Add more boss varieties and stages
- Implement local score persistence
- Add mobile touch controls
- Create power-up system (shields, temporary fire rate boost)
- Implement leaderboard/save system

## File Checklist for Contributors

When working on this project:
- ✓ Test gameplay on desktop and mobile
- ✓ Verify no console errors
- ✓ Check performance (maintain 60fps)
- ✓ Update this CLAUDE.md if architecture changes
- ✓ Commit with clear messages describing changes

## Attribution

**Last Updated**: 2026-10-16  
**Maintained by**: Claude Code agent

---

For questions or updates to this documentation, refer to the git log for recent changes and commits.
