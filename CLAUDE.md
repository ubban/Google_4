# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a vertical shoot-'em-up game built as a single-file HTML5 canvas application. It features Raiden-style gameplay with weapon progression, smart bombs, multiple difficulty levels, and procedural bosses. The entire game logic, rendering, and audio synthesis are implemented in a single `index.html` file (~1669 lines).

## Development & Running

**There is no build system or external dependencies.** The game runs directly in any modern browser by opening `index.html`.

### Running the Game
- Open `index.html` in a web browser
- Alternatively, run a simple HTTP server: `python -m http.server 8000` then visit `http://localhost:8000`

### Development
Since this is a single-file application with no build steps, testing, or linting:
- Edit `index.html` directly
- Use browser DevTools (F12) to debug
- Test changes by reloading the page in the browser
- Check browser console for any errors

## Architecture Overview

The game uses a **single IIFE (Immediately Invoked Function Expression)** containing all game logic. Key systems:

### 1. **Game Loop & Rendering**
- Canvas-based 2D rendering using `requestAnimationFrame`
- Main game loop handles: input → update → render
- Adaptive performance tracking (scales rendering quality based on frame time)
- Separate HUD layer with stats, weapons, lives, bombs display

### 2. **Core Game Systems**

#### Game State
- `state` object tracks: current wave, stage, lives, bombs, score, multiplier, enemy list, player state
- Difficulty system (easy/normal/hard) affects spawn rates, enemy speed, enemy accuracy, starting lives/bombs, boss HP

#### Player/Aircraft
- Positioned at canvas bottom, moves left/right within bounds
- Health system (hearts/lives)
- Weapon system with 3 types: stream, laser, spread
- Weapon levels (0-3) with visual/sound feedback
- Smart bombs with area damage effect

#### Enemies
- **Types**: grunt (red, 1 HP), interceptor (fast, 1 HP), tank (slow, 4 HP)
- Spawn rate decreases as waves progress, spawning becomes more aggressive
- Each enemy type has unique visual, speed, and shooting behavior
- Enemies drop loot: weapon upgrades or bomb refills

#### Bosses
- 6 boss types with unique visual designs and color schemes
- Boss progression through stages
- Boss telegraph mechanic (brief warning before attack)
- Bosses cycle through attack patterns and visual damage states

### 3. **Audio System**
- **WebAudio API synth** - all sounds generated procedurally (no external audio files)
- `beep()` function: generates tones with pitch sweeps, filters, and panning
- `noiseBurst()` function: generates noise for explosions
- SFX library (`sfx` object) with sounds for: shooting, explosions, powerups, boss hits, stage clear, etc.
- Audio context is lazy-initialized on first user interaction (browser requirement)
- Master gain with dynamics compressor for audio balance

### 4. **Input Handling**
- **Keyboard**: Arrow keys + Space (fire) + X/Shift (bomb) + P/Esc (pause)
- **Mouse**: Move to aim, click to fire
- **Touch**: Custom joystick in left area, FIRE and BOMB buttons on right
- Mobile joystick uses touch events and visual feedback

### 5. **Accessibility & Preferences**
- Screen reader live region for announcements
- Reduced motion preference respect (via `prefers-reduced-motion` media query)
- Mute toggle persisted in localStorage
- Difficulty and best score saved in localStorage

### 6. **Visual Systems**
- **Gradient sky backgrounds**: 6 themes with different color schemes
- **Particles**: enemy explosions, coin pickups, score popups
- **Visual effects**: vignette on low health (pulse animation), flash on hit/bomb, screen shake
- **HUD**: score, best score, multiplier, weapon info, health hearts, bomb count, boss health bar

## Key Variables & Constants

### Difficulty Settings (`DIFFICULTIES`)
```
easy/normal/hard: spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul
```

### Enemy Types (`ENEMY_TYPES`)
- Each type defines: width, height, HP, speed multiplier, color, score value, minimum stage to spawn

### Bosses (`BOSSES`)
- Each boss has: name, hull colors (top/bottom), wing color, core color, HP multiplier

### Storage Keys
- `skyfighter.bestScore`: best score (number)
- `skyfighter.muted`: audio mute state (0/1)
- `skyfighter.difficulty`: selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: reduced motion preference (0/1)

## Common Modifications

### Adjust Game Difficulty
- Modify `DIFFICULTIES` object (spawn rates, enemy speed, player starting lives/bombs)
- Adjust `KILLS_PER_STAGE` for stage progression speed

### Add New Enemy Type
1. Add entry to `ENEMY_TYPES` with color, HP, speed multiplier, score
2. Add corresponding explosion SFX to `sfx` and `EXPLODE_SFX` mapping
3. Add spawn logic in the enemy spawning section

### Modify Boss Behavior
- Boss stats are in `BOSSES` array
- Boss attack patterns are in the boss state machine (look for `boss.phase` and boss update logic)

### Adjust Visual Themes
- Edit `THEMES` array: change sky gradient colors, cloud colors, sun colors

### Add New Sound Effect
- Create function in `sfx` object using `beep()` or `noiseBurst()` helpers
- Map to gameplay events in the respective event handlers

## File Structure

- **index.html**: Single file containing:
  - HTML structure (canvas, HUD, overlay UI, mobile touch controls)
  - CSS (layout, animations, responsive design, dark theme)
  - JavaScript (entire game engine, rendering, input, audio, state management)

## Browser Compatibility

- Requires: HTML5 Canvas, WebAudio API, modern ES6+ JavaScript
- Touch support for mobile/tablet
- LocalStorage for persistence
- Tested on modern browsers (Chrome, Firefox, Safari, Edge)

## Performance Notes

- Adaptive quality system reduces particle/effect complexity under frame rate stress
- Touch joystick uses requestAnimationFrame for smooth updates
- Canvas rendering optimized with `clearRect` and minimal redraws
- Audio context uses dynamics compressor to prevent clipping
