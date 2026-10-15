# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game. The entire game is implemented as a single-file HTML application with embedded CSS and JavaScript, enabling instant play in any modern browser.

**Repository**: [github.com/ubban/Google_4](https://github.com/ubban/Google_4)

## Running the Game

Simply open `index.html` in a web browser. The game runs at 60 FPS and is fully playable on desktop (keyboard/mouse) and mobile (touch controls).

```bash
# No build step required — open locally or serve via a web server
open index.html
# or
python3 -m http.server 8000  # then visit http://localhost:8000
```

## Architecture

The game follows a classic game loop pattern within a single 1669-line HTML file:

### Core Systems

1. **Canvas & Rendering** (lines ~10–200)
   - 480×720 game canvas with responsive scaling
   - Draws planes (player + enemies), bullets, particles, and UI
   - Screen shake and flash effects for visual feedback
   - Dark space background with gradient

2. **Audio System** (lines ~200–450)
   - Web Audio API for sound effects (SFX)
   - Mute toggle stored in `localStorage`
   - Sound effects: shoot, explode, powerup, boss hit, stage transition, etc.
   - Contextual audio playback (direction-aware for some effects)

3. **Input Handling** (lines ~450–900)
   - **Keyboard**: Arrow keys for movement, Space/X/Shift for bomb, P/Esc to pause
   - **Mouse**: Movement toward pointer; click to fire
   - **Touch**: Virtual joystick on left side, fire/bomb buttons on right
   - Movement and firing states integrated into main game update loop

4. **Game State & Loop** (lines ~900–1300)
   - Central update function called every frame (~60 FPS)
   - Tracks: player position/health, bullets, enemies, boss, particles, power-ups
   - Collision detection (rectangles for simplicity)
   - Score chain multiplier that increases on consecutive kills within time window
   - Difficulty settings (easy/normal/hard) that affect spawn rates, enemy stats, and starting lives

5. **Entities**
   - **Player**: Single ship with two weapon types (Vulcan/Laser), 3 levels per weapon, bomb count
   - **Enemies**: Multiple types (fighter, tank) with different behaviors (weaving, shot patterns)
   - **Boss**: One boss per stage, complex attack patterns, health bar UI
   - **Bullets & Effects**: Player bullets (pierce for lasers), enemy bullets (straight/homing/radial), explosions, particles

6. **Boss Mechanics** (lines ~1300–1500)
   - Five boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH) with distinct attack patterns
   - Each boss has ~3 attack phases that cycle
   - Attack patterns include: radial bursts, aimed spreads, spirals, teleportation, homing projectiles
   - Boss defeats trigger stage transition and score multiplier

### Game Flow

1. **Initialization**: `resetGame()` sets up player, spawns initial enemies
2. **Per-Frame Update**: Enemy spawning (rate increases with difficulty), collision checks, entity updates
3. **Stage Progression**: After 8 enemy kills, boss warning appears → boss spawns after 1.2s → defeating boss advances stage
4. **Scoring**: Base points for kills, multiplied by chain multiplier (resets after 2s of no kills)
5. **Game Over**: Lose all lives → show game over screen with high score

## Key Game Constants & Configuration

```javascript
const KILLS_PER_STAGE = 8;      // Enemy kills before boss spawns
const DIFFICULTIES = {...};     // Define spawn rates, lives, bombs per difficulty
const BOSSES = [...];           // Boss designs and stats
const ENEMY_TYPES = {...};      // Enemy definitions (fighter, tank)
const BOSS_PATTERNS = [...];    // Attack pattern functions for each boss
```

## Customization Points

- **Difficulty balance**: Modify `DIFFICULTIES` object to adjust spawn timings, enemy stats, or starting resources
- **Enemy behaviors**: Add new enemy types or adjust weaving/shooting patterns in the update loop
- **Boss attacks**: Add new attack functions and phases in `BOSS_PATTERNS`
- **Visual tweaks**: Adjust colors, sizes, or animations in the drawing functions (`drawPlane`, `drawBossShip`, etc.)
- **Sound design**: Replace SFX with new Web Audio synthesis or pre-recorded samples

## Development Tips

- **Testing a specific boss**: Modify `resetGame()` to skip to a higher stage, or trigger boss spawn immediately
- **Tuning difficulty**: Adjust `spawnTimer` calculations and enemy HP in the update loop
- **Performance**: Monitor particle count and enemy count; cull off-screen entities if frame rate drops
- **Mobile debugging**: Use browser DevTools remotely; touch input logs to console for troubleshooting

## Browser Compatibility

Requires:
- ES6 (arrow functions, const/let, template literals)
- Canvas 2D context
- Web Audio API
- localStorage for persistence
- Touch Events API (for mobile)

Tested on modern versions of Chrome, Firefox, Safari, Edge.
