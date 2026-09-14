# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in a single HTML file. It's a browser-based game with no external dependencies or build process. The game features multiple difficulty levels, stage-based progression with boss encounters, weapon upgrades, and a combo scoring system.

## Architecture

### Single-File Structure
- **index.html** (1669 lines): Contains all HTML, CSS (embedded in `<style>`), and JavaScript (embedded in `<script>`)
- No build tools, package.json, or external dependencies required
- Game runs directly in any modern browser

### Core Game Systems

#### Game Loop & Rendering
- Canvas-based 2D rendering (480×720px base resolution, responsive scaling)
- Fixed time-step game loop with `requestAnimationFrame`
- Render layers: sky gradient → entities → HUD → overlays
- Adaptive performance system that adjusts rendering quality based on frame times

#### Entity Management
- **Player**: Fighter plane at bottom of screen, moves and shoots
- **Enemies**: Three types (grunt, interceptor, tank) with varying HP, speed, and difficulty scaling
- **Bosses**: Six unique bosses (Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan), one per stage
- **Bullets**: Player bullets and enemy bullets tracked separately for collision detection
- **Particles**: Visual effects for explosions, impacts, etc.
- **Power-ups**: Weapons and bombs spawn when enemies die

#### Game Flow
1. **Start Screen**: Difficulty selection, best score display
2. **Stage Loop**: Spawn 18 enemies per stage; killing all triggers boss battle
3. **Boss Battle**: Unique boss with multi-phase pattern; boss health shown in HUD
4. **Stage Clear**: Visual feedback, progression to next stage
5. **Game Over**: Final score display, option to retry

#### Difficulty System
Three presets (easy, normal, hard) control:
- Enemy spawn rates and minimum spawn delay
- Enemy movement speed and shoot accuracy
- Player lives (4 easy, 3 normal/hard)
- Available bombs (3 easy, 2 normal/hard)
- Boss health multiplier

#### Scoring & Progression
- Base score from enemy kills (varies by type)
- Combo multiplier: chain kills quickly for 1.0× → 2.0× multiplier
- Combo resets after 1.5 seconds of no kills
- Best score persisted to localStorage
- Visual combo pop-ups and multiplier display in HUD

#### Player Systems
- **Weapons**: Vulcan (spread bullets) and Laser (precise bursts); collected from defeated enemies
- **Weapon Levels**: Increases fire rate and bullet damage; resets to level 1 on level change
- **Bombs**: Smart bombs that damage all on-screen enemies; limited ammo (2–3 depending on difficulty)
- **Lives/Hearts**: Visual heart display; invulnerability period after hit
- **Movement**: Steered via keyboard, mouse, or touch; constrained to canvas bounds

#### Audio System
- WebAudio API synthesized effects (no audio files)
- Separate SFX for: shooting, explosions (by enemy type), hits, bombs, power-ups, boss attacks, stage clear, game over
- Mutable via settings button; mute state persisted
- Spatial audio: panning based on screen position

#### Visual Themes
Six dynamic themes (Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge) that change sky gradient, cloud colors, and sun tone. Selected randomly per run.

#### Input Handling
- **Keyboard**: Arrow keys or WASD to move, Space to fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Move cursor to steer plane (cursor must be over canvas); click to fire
- **Touch**: Left half drag for movement (joystick), right half buttons for Fire and Bomb
- Priority: Keyboard/joystick > mouse (mouse doesn't override active input)
- Pause button in top-right corner

#### Accessibility & Preferences
- **Reduced Motion**: Toggleable flag that disables screen shake, flash effects, and animation pulses
- **Screen Reader**: Announcements via aria-live region for score, weapon changes, events
- **Persistent Preferences**: Mute status, difficulty, reduced motion setting stored in localStorage
- Color schemes tested for WCAG contrast

### Key Constants & Configuration

#### Canvas & Viewport
- Base resolution: 480×720px
- Responsive scaling up to 1.6× for larger screens
- Safe area insets for notched devices

#### Difficulty Parameters
```javascript
DIFFICULTIES = {
  easy:   { spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul },
  normal: { ... },
  hard:   { ... }
};
```

#### Enemy Types & Boss Definitions
- Enemy types define width, height, HP, speed multiplier, color, score, min stage
- Bosses define hull colors, core color, name, HP multiplier
- Both stored as config objects for easy balance tuning

#### Storage Keys
- `skyfighter.bestScore`: Best score achieved
- `skyfighter.muted`: Mute state (0 or 1)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Reduced motion preference (0 or 1)

## Common Development Tasks

### Running the Game
Simply open `index.html` in any modern browser (no build step needed):
```bash
# Via a simple HTTP server (Python):
python3 -m http.server 8000
# Then open http://localhost:8000/index.html

# Or directly open in browser:
open index.html
```

### Testing a Feature
1. Open `index.html` in browser
2. Test functionality using your browser's developer console for debugging
3. Use browser DevTools to inspect canvas rendering or modify game state

### Tweaking Game Balance
Edit constants at the top of the `<script>` block:
- `DIFFICULTIES`: Adjust spawn rates, speed, shoot chance, lives, bombs, boss HP
- `ENEMY_TYPES`: Modify enemy stats (HP, color, score, speed)
- `BOSSES`: Change boss HP multiplier and appearance
- `KILLS_PER_STAGE`: How many enemy kills trigger a boss
- `THEMES`: Add or modify visual themes

### Debugging
- Open browser DevTools (F12) → Console
- Use console to set `running = true` or modify `player` state to test scenarios
- Log game state: `console.log({ score, stage, lives, enemies, boss })`
- Inspect canvas rendering by adding debug draw calls

### Adding New Features
Due to the single-file structure:
1. Add new HTML elements to the container if needed (ensure z-index layering)
2. Add CSS styles within the `<style>` block, organized by component
3. Add JavaScript in the main IIFE, following existing patterns (use local variables, avoid globals)
4. Update game loop (`animate()` function) or event listeners as needed

### Visual Changes
- Update theme colors in `THEMES` array
- Modify enemy/boss colors in type definitions
- CSS for HUD, overlays, and UI in the `<style>` block
- Canvas drawing happens in `render()` function

## Performance Notes

### Adaptive Quality
The game monitors frame times and automatically reduces rendering quality (quality variable: 0=full, 1=medium, 2=low) if the average frame time exceeds 1/40s. This ensures playability on lower-end devices.

### Optimization Tips
- Keep particle count reasonable (many simultaneous explosions can cause lag)
- Avoid adding large DOM elements; use canvas for visual effects
- Audio effects are synthesized in real-time; muting disables synthesis to save CPU

## Browser Compatibility

- Requires ES6 (arrow functions, const/let, template literals)
- WebAudio API for sound (fallback: muted if unavailable)
- Canvas 2D context
- LocalStorage for settings persistence
- Touch API for mobile controls (gracefully disables if unavailable)
- Tested on modern Firefox, Chrome, Safari, and mobile browsers

## Mobile/Touch Considerations

- Touch controls enabled automatically if `ontouchstart` or `maxTouchPoints > 0` detected
- Joystick UI (left side) for movement, buttons (right side) for actions
- Uses `touch-action: none` to prevent browser gestures from interfering
- Responsive viewport scaling; safe area insets for notched devices
- Tap to start, controls respond immediately (no lag)

## Code Style & Patterns

- Strict mode enabled (`'use strict'`)
- Single module IIFE to encapsulate game state and avoid globals
- Separation of concerns: rendering, input, collision, spawning
- Immutable game constants (DIFFICULTIES, ENEMY_TYPES, BOSSES, THEMES)
- Event-driven for UI (buttons, input handlers)
- Frame-based timing for consistent game logic

## Storage & Persistence

Best score, mute status, difficulty, and reduced motion preference are automatically persisted to browser localStorage and restored on page load. No server communication.
