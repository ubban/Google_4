# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The game features:

- Canvas-based rendering with procedural graphics (no external image assets)
- Web Audio API synthesized sound effects (no audio files)
- Three difficulty levels with progressive stage-based gameplay
- Boss encounters with screen-clearing bomb mechanics
- Weapon upgrade system and score multiplier mechanics
- Full touch/mobile support with joystick controls and accessibility features

## Architecture

### File Structure

The entire game is contained in **index.html** (~1670 lines), structured as follows:

```
index.html
├── HTML Structure (lines 360-420)
│   └── Canvas container, HUD elements, overlay UI
├── CSS Styling (lines 10-358)
│   ├── Game canvas and background
│   ├── HUD stats display
│   ├── Overlay (menu/game over screen)
│   ├── Boss health bar
│   ├── Touch control elements
│   └── Animations (combo pop, stage banner, etc.)
└── JavaScript (lines 422-1669)
    ├── Configuration & Constants
    ├── Audio System (Web Audio API)
    ├── Input Handling (keyboard/mouse/touch)
    ├── Game State & Reset
    ├── Rendering Functions (drawPlane, drawBossShip, etc.)
    ├── Update Loop (physics, collision, spawning)
    ├── Main Loop (requestAnimationFrame)
    └── Event Listeners
```

### Core Game Logic Flow

1. **Game State** (lines 863-869): Player, enemies, bullets, particles, power-ups, boss
2. **Reset/Initialize** (`resetGame()`, line 875): Sets up player, arrays, and initial values
3. **Main Loop** (`loop()`, line 1640): Orchestrates update → draw cycle
4. **Update** (`update()`, line 1148): Handles:
   - Player movement and firing
   - Enemy spawning and AI
   - Collision detection (player/bullets/enemies)
   - Particle updates
   - Stage progression and boss spawning
5. **Draw** (`draw()`, line 1520): Renders:
   - Background clouds
   - HUD and game elements
   - Player, enemies, bullets, particles, power-ups
   - Boss and UI overlays

### Key Constants & Configuration

**Difficulty Settings** (line 485): Control spawn rates, enemy speed, lives, bombs, boss HP multiplier

**Enemy Types** (line 502): grunt, interceptor, tank — define speed, HP, score, colors

**Bosses** (line 508): Six named bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with color schemes and HP multipliers

**Themes** (line 493): Six sky/background color themes applied per stage

### Audio System

The game uses **Web Audio API** (lines 543-637) to generate all sound effects synthetically:
- `beep()`: Tone generation with optional frequency sweep, filtering, panning
- `noiseBurst()`: Noise-based effects
- Sound effects triggered on: weapon fire, enemy hit, bomb, stage clear, game over

No external audio files required; all synthesis is real-time.

### Input System

**Keyboard** (lines 769-776):
- Arrow keys / WASD: Player movement
- Space: Fire
- X / Shift: Drop bomb
- P / Esc: Pause

**Mouse** (lines 841-861):
- Movement: Aim (player position determined by mouse on canvas)
- Click: Fire

**Touch** (lines 778-837):
- Left zone drag: Joystick movement
- FIRE button (bottom right)
- BOMB button (top right)

### Physics & Collision

- **Player speed**: 260px/s (constant across difficulties)
- **Bullet speed**: Fixed 400px/s
- **Enemy bullets**: Variable by type
- **Collision detection**: Rectangle-based (AABB) for all entities
- **Invulnerability frames**: After taking damage or using bomb

### Game Progression

- **Stages**: Progress after killing `KILLS_PER_STAGE` (18) enemies
- **Boss Encounters**: Spawn at specific stages; must be defeated to progress
- **Difficulty Scaling**: Enemy spawn rate, speed, shoot chance increase per difficulty
- **Score Multiplier**: Built from kill chain; resets if hit or too much time passes

## Development Tasks

### Running/Testing

The game runs directly in any browser:
```bash
# Serve locally (Python 3)
python -m http.server 8000
# Then open http://localhost:8000

# Or use any static server
```

### Making Changes

1. **Edit index.html** directly
2. **Test in browser** after each change:
   - Verify gameplay mechanics (spawning, collision, firing)
   - Test across difficulty levels
   - Check mobile responsiveness (toggle device emulation)
   - Verify audio (unmute button) if changing Web Audio code

### Key Areas for Common Modifications

**Difficulty/Balance** (line 485-489):
- `spawnBase` / `spawnMin`: Enemy spawn frequency (lower = more spawns)
- `enemySpeed`: Multiplier on base enemy speeds
- `enemyShootChance`: Probability enemy fires per frame
- `lives`, `bombs`, `bossHpMul`: Starting resources

**Enemy Types** (line 502-506):
- Add new type: Define in `ENEMY_TYPES`, then update spawn logic in `update()` to use it
- Modify color/stats: Update the corresponding entry

**Boss Design** (line 508-515):
- Colors and HP scaling; add new entries for new boss stages

**Weapons/Upgrades** (weapon system):
- `player.weapon` switches between 'vulcan' and 'laser'
- Firing code: lines 1200+; modify bullet patterns here
- `player.level`: Increases damage/fire rate; scale by this in firing logic

**Visual/Animations**:
- CSS animations: Edit keyframes in `<style>` (lines 71-127)
- Canvas drawing: Edit `draw()` and related functions for rendering changes
- Particle effects: Modify particle spawn in `update()` and render in `draw()`

### Performance Considerations

- **Adaptive Quality** (lines 473-482): Game auto-adjusts rendering quality based on frame time
- **Canvas Size**: Base 480×720, scaled to fit window (line 753)
- **Particle Limits**: Monitor particle array size; excessive particles degrade performance
- **Draw Order**: Layered (clouds → enemies → bullets → particles → UI) to avoid overdraw

## Storage & Persistence

Game uses localStorage for:
- `skyfighter.bestScore`: Best score (number)
- `skyfighter.muted`: Audio mute state (0/1)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Accessibility preference (0/1)

## Accessibility Features

- **Screen Reader Support**: `aria-live` region for announcements (line 451)
- **Reduced Motion**: Respects `prefers-reduced-motion` media query; disables shake/flash
- **Keyboard Controls**: Full game playable with keyboard alone
- **Touch Controls**: Mobile-optimized UI with large touch targets

## Browser Compatibility

- Requires: HTML5 Canvas, Web Audio API, ES6+ JavaScript
- Tested: Modern browsers (Chrome, Firefox, Safari, Edge)
- Mobile: Responsive design; touch controls on iOS/Android

## Common Debugging

**Frame rate issues**: Check adaptive quality system and particle count
**Audio not playing**: Verify Web Audio context is initialized (browser autoplay policy may block audio initially)
**Touch controls not working**: Ensure `isTouchDevice` detection is correct; check touch event listeners
**Collision bugs**: Debug AABB collision rectangles; verify entity x/y/w/h values
