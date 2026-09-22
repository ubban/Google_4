# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a vertical shoot-'em-up game (Raiden-style) built as a single-file HTML5 application. The entire game—rendering, game logic, UI, and audio synthesis—is contained in a single `index.html` file (~1670 lines). No build tools or external dependencies are required.

## Running the Game

Simply open `index.html` in a web browser. The game will start with a title screen showing difficulty options (Easy, Normal, Hard).

- **Controls (Desktop)**: Mouse to move, click to fire, Space/X/Shift for bombs, Arrows/WASD to move, P/Esc to pause
- **Controls (Mobile)**: Drag left side to move, tap FIRE and BOMB buttons

## Game Architecture

The game is structured as a single IIFE (Immediately Invoked Function Expression) containing multiple functional modules:

### Core Systems

1. **Game Loop** (`requestAnimationFrame` loop at the bottom)
   - Handles delta time, updates, and rendering each frame
   - Implements adaptive frame-rate quality detection (`trackPerf`)

2. **Player State & Controls**
   - Player object: position, velocity, health (lives), bombs, current weapon
   - Keyboard/mouse event listeners for desktop input
   - Touch zone handlers for mobile (joystick on left, action buttons on right)
   - Pause state management

3. **Entity Management**
   - Enemies (grunts, interceptors, tanks) spawned based on difficulty and stage
   - Bullets (player and enemy) stored in arrays with position, velocity, damage
   - Explosions (particles) for visual feedback
   - Boss entities (one per stage, after killing `KILLS_PER_STAGE` enemies)

4. **Weapons & Upgrades**
   - Three weapon types: Vulcan (rapid fire), Cannon (slower, higher damage), Laser (piercing)
   - Weapon level increases from pickups dropped by enemies
   - Weapon state: fire rate, damage, spread pattern

5. **Bomb System**
   - Limited smart bombs that clear screen and damage all enemies
   - Bomb count resets per stage
   - Visual bomb indicator in HUD

6. **Difficulty System**
   - Three presets (easy/normal/hard) with different spawn rates, enemy behavior, lives, bombs, boss HP
   - Difficulty preference saved to `localStorage`

7. **Visual Theme System**
   - 6 themes cycling through stages (Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge)
   - Each theme has sky gradient, cloud color, sun color, and custom styling

8. **Audio System**
   - WebAudio API synth (no external audio files)
   - Generates beeps and noise bursts for: weapons, hits, explosions, bombs
   - Master gain control and mute toggle (saved to `localStorage`)
   - Panning based on screen position

9. **HUD & UI**
   - Score, best score, multiplier, weapon info, health/bombs
   - Boss health bar (appears when boss is active)
   - Combo pop-ups on consecutive hits
   - Stage banners on stage transitions
   - Flash and vignette effects for screen hit feedback
   - Title overlay with difficulty selection

10. **Persistence**
    - Best score, muted state, difficulty, reduced motion preference stored in `localStorage`

## Key Data Structures

- **DIFFICULTIES**: Object defining spawn rates, enemy speeds, player lives, bomb count per difficulty
- **THEMES**: Array of visual themes with color palettes and names
- **ENEMY_TYPES**: Object defining grunt, interceptor, tank properties (size, HP, speed, color, spawn stage)
- **BOSSES**: Array of 6 boss configurations with unique visual styling and HP multipliers
- **KILLS_PER_STAGE**: Constant (18) defining enemy kills needed before boss spawns

## Code Organization in index.html

- **Lines 1–359**: CSS (styles, animations, responsive layout, theme colors)
- **Lines 361–421**: HTML structure (canvas, HUD, overlay, touch controls)
- **Lines 422–end**: JavaScript game logic
  - Setup and constants
  - Audio system (synth, beep, noise functions)
  - Rendering functions (background, player, enemies, bullets, effects)
  - Update functions (movement, collision, entity updates)
  - Event handlers (input, pause, UI buttons, difficulty selection)
  - Game loop and initialization

## Modifying the Game

### Adding Enemy Types
Define in `ENEMY_TYPES` object with: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, and optional `weave` property for evasive movement.

### Adding Bosses
Extend `BOSSES` array with new objects containing: `name`, `hullTop`, `hullBottom`, `wing`, `core` (colors), and `hpMul` (health multiplier).

### Adding Visual Themes
Extend `THEMES` array with new theme objects: `sky` (gradient stops), `cloud`, `sun` (color strings), and `name`.

### Tweaking Difficulty
Modify `DIFFICULTIES` object values: `spawnBase`/`spawnMin` (enemy spawn timings), `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`.

### Audio Adjustments
Modify `beep()` and `noiseBurst()` calls throughout the code; adjust `masterGain.gain.value` for volume.

## Performance Considerations

- Adaptive quality system (`quality` variable) reduces particle effects and rendering detail if frame rate drops
- Entity culling: off-screen enemies/bullets are cleaned up automatically
- LocalStorage is used only for user preferences (no large data persistence)
- Canvas rendering is 2D context with basic shapes (circles, rectangles, arcs) for performance

## Accessibility Features

- Screen reader announcements via `announce()` function (updates `#srLive` aria-live region)
- Reduced motion preference respect (disables animations for users with prefers-reduced-motion)
- High contrast UI with text shadows for readability
- Touch controls clearly labeled
- Keyboard-only playable (arrows, space, X, P for pause)

## Testing / Verification

Manual testing in browser:
1. Start game, play through Easy/Normal/Hard modes
2. Verify weapon upgrades work (test all 3 weapon types)
3. Check bomb mechanics (fire smart bombs, verify screen clear)
4. Test pause (P key or button)
5. Test mobile controls (drag left, tap buttons)
6. Verify mute toggle and reduced motion toggle persist
7. Check best score saves and displays correctly
8. Test all 6 visual themes appear across 6+ stages
9. Verify audio synth plays without errors

## Common Tasks

- **Change spawn rates**: Modify `DIFFICULTIES` object values
- **Adjust weapon damage**: Edit weapon fire rate in player update section
- **Add visual feedback**: Create new animation in CSS or call `flashEl.classList.add('hit')` style effects
- **Adjust difficulty balance**: Tweak `bossHpMul`, `enemySpeed`, `enemyShootChance`
- **Test on mobile**: Use browser DevTools device emulation, or open on actual phone
