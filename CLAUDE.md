# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game — a single-file web game built with vanilla HTML, CSS, and JavaScript. The player controls an aircraft, destroys waves of enemies, collects weapon upgrades, and faces off against stage bosses. The game runs entirely in the browser with no external dependencies.

## Running the Game

To test the game, use any local web server or open `index.html` directly in a browser:

```bash
# Option 1: Simple Python server
python3 -m http.server 8000
# Visit http://localhost:8000

# Option 2: Using Node.js http-server (if installed)
npx http-server

# Option 3: Direct browser (less ideal, may have CORS issues with some features)
open index.html
```

The game is fully self-contained — no build step or external dependencies are required.

## Architecture

The entire game logic is in a single `<script>` tag within `index.html`, structured in distinct functional sections:

### Core Game Loop
- **Canvas rendering**: 2D graphics using canvas API
- **Update/draw cycle**: Frame-by-frame simulation with delta-time tracking
- **Adaptive performance**: Monitors frame times and adjusts visual quality if FPS drops

### Key Game Systems

**Player Ship**
- Position and velocity-based movement
- Health system (3-4 lives depending on difficulty)
- Weapon system with multiple types (Vulcan, Laser, Plasma)
- Smart bomb mechanic (limited uses, clears screen)
- Input handling for both keyboard and touch controls

**Enemy AI**
- Wave spawning system with difficulty-based spawn rates
- Multiple enemy types: Grunts, Interceptors (fast), Tanks (durable)
- Individual enemy AI: targeting, movement patterns (some weave)
- Bullet pooling for efficiency

**Boss System**
- 6 unique boss designs with distinct color schemes and HP multipliers
- Boss-specific visual feedback (health bar, danger vignette)
- Boss spawns after defeating 18 enemies per stage
- Progressive difficulty scaling across stages

**Scoring & Progression**
- Score multiplier system (resets on damage, increases on consecutive hits)
- Stage progression tied to enemy kill count (18 kills = stage clear)
- Combo detection for visual pop-up feedback
- Best score persisted to localStorage

**Weapon Upgrades**
- Weapon drops after enemy kills
- Three weapon types: Vulcan (fast bullets), Laser (penetrating), Plasma (slow, high damage)
- Weapon level system (1-5)
- Weapon stat display in HUD

**Difficulty Modes**
- Easy: Slower enemies, lower spawn rate, more lives, weaker bosses
- Normal: Balanced defaults
- Hard: Dense spawns, faster enemies, aggressive AI, stronger bosses
- Player choice persisted to localStorage

### Audio System
- WebAudio API synthesizer (no audio files)
- Procedural sound generation: beeps, noise bursts, chord sounds
- Pan positioning based on screen location
- Master gain with compressor for volume control
- Mute toggle persisted to localStorage

### Input & Controls

**Keyboard & Mouse**
- Arrow keys or WASD for movement
- Mouse movement follows player to cursor
- Space bar or left-click to fire
- X or Shift for bomb
- P or Esc to pause

**Touch**
- Left half of screen (drag zone): pilot aircraft by dragging
- Right half of screen: FIRE and BOMB buttons
- Joystick base appears dynamically when dragging

**Accessibility**
- Reduced motion toggle (honors `prefers-reduced-motion` media query)
- ARIA live region for screen reader announcements
- Semantic button structure with title attributes

### Visual Effects
- Vignette flash on damage (scales with reduced motion setting)
- White flash on bullet hits
- Screen-filling flash on bomb detonation
- Combo pop animation
- Stage banner animations
- Heart beat animation during critical health

### Responsive Design
- Responsive HUD using clamp() for font sizing
- Canvas scales to fill container
- Safe area insets for notched devices
- Touch-friendly button sizing

## Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with properties: `w` (width), `h` (height), `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, and optional `weave`
2. Implement draw logic in the enemy rendering section
3. Update spawn logic if type has special behavior
4. Adjust `KILLS_PER_STAGE` if needed to balance progression

### Adding a New Boss
1. Add entry to `BOSSES` array with `name`, hull/wing/core colors, and `hpMul`
2. Implement draw logic in boss rendering section
3. Boss AI defaults to auto-increasing difficulty (faster shooting at higher stages)

### Adding a New Difficulty Setting
1. Add entry to `DIFFICULTIES` object with spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier
2. Add button to `#diffRow` in HTML
3. New difficulty is automatically persisted to localStorage

### Adding a New Theme/Sky
1. Add entry to `THEMES` array with gradient colors, cloud color, sun color, and display name
2. Theme selection is random per game start

### Modifying Visual Quality Levels
Performance adaptation is in the `trackPerf()` and `quality` variable. Quality levels affect:
- Enemy bullet count
- Particle effects density
- Canvas resolution (via rendering scale)
Adjust constants in the quality-dependent rendering sections.

### Audio Tweaks
All sound effects are synthesized via `beep()` and `noiseBurst()`. Parameters:
- Frequency: oscillator pitch
- Duration: sound length in seconds
- Type: 'sine', 'square', 'sawtooth', 'triangle'
- Volume: 0-1 scale
- Options: sweepTo (frequency ramp), filterType, filterFreq, pan, delay

## File Structure

```
index.html          Single file containing:
├── HTML markup      (container, canvas, HUD, overlay, touch controls)
├── CSS              (styling, animations, responsive layout)
└── JavaScript       (game logic, audio, input handling)
README.md           Minimal project description
```

## Key Variables & State

- `quality`: Adaptive performance level (0 = full, 2 = low)
- `gameState`: 'menu', 'playing', 'paused', 'gameOver'
- `player`: Ship object with x, y, hp, weapon, bombs
- `enemies`, `bosses`: Array pools for active entities
- `score`, `mult`: Current score and combo multiplier
- `stage`, `kills`: Progression tracking
- `localStorage`: Persists best score, mute state, difficulty, reduced motion preference

## Performance Considerations

- Enemy bullet pooling reuses projectiles instead of creating/destroying
- Canvas clipping prevents drawing off-screen entities
- Adaptive quality system degrades effects if frame times exceed 25ms
- Event delegation for UI buttons to minimize listeners
- RAF (requestAnimationFrame) for smooth 60 FPS target

## Testing the Game

1. **Basic gameplay**: Start game, move ship, fire at enemies, verify kills increment stage progress
2. **Difficulty settings**: Verify spawn rates and boss HP differ across Easy/Normal/Hard
3. **Weapon drops**: Confirm weapon upgrades appear after kills and cycle through types
4. **Boss mechanics**: Verify boss appears after 18 kills, health bar shows, and difficulty scaling works
5. **Bomb usage**: Test smart bomb clears screen and depletes bomb count
6. **Touch controls**: On mobile, drag left to move, tap FIRE/BOMB buttons
7. **Persistence**: Close and reopen; verify best score, difficulty, and mute settings persist
8. **Accessibility**: Test pause, check that ARIA announcements work with screen reader, verify reduced motion toggle disables animations
9. **Performance**: On low-end devices, verify quality adapts and game remains playable

## Debugging Tips

- Open browser DevTools console for any JavaScript errors
- Check `localStorage` in DevTools Application tab to inspect saved state
- Modify global variables in console to test (e.g., `quality = 2` to force low quality)
- Use `canvas.width` and `canvas.height` to verify canvas size calculations
- Monitor frame time via `frameTimes` array to debug performance issues
