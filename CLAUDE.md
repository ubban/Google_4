# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. It features:
- Progressive difficulty with multiple stages and boss encounters
- Weapon progression system (Vulcan, Spread, Laser)
- Bomb mechanics with smart detonation
- Adaptive performance scaling
- Procedural audio synthesis (no external audio files)
- Touch and mouse/keyboard controls
- Multiple visual themes and difficulty levels
- Accessibility features (reduced motion support, screen reader announcements)

## Codebase Architecture

All game code is contained in `index.html` (~1,669 lines) as a self-contained, single-file application with no external dependencies.

### Code Structure

The file is organized as follows:

1. **HTML & CSS** (lines 1–421)
   - Game canvas container and HUD (heads-up display)
   - Visual elements: score, weapon info, health, bombs, boss health bar
   - Touch control overlay (joystick + action buttons for mobile)
   - Overlay menu for difficulty selection and game-over state
   - CSS animations for visual effects (screen shake, combo pop-ups, boss health vignette)

2. **JavaScript Game Engine** (line 422 onwards)
   - **Game State**: Canvas context, DOM elements, game variables (score, lives, stage, etc.)
   - **Configuration**: Difficulty presets, enemy types, boss definitions, visual themes
   - **Audio System**: WebAudio API-based procedural sound generation (no external audio files)
   - **Input Handling**: Mouse, keyboard, and touch controls
   - **Game Loop**: Core update/render cycle with `requestAnimationFrame`
   - **Game Logic**: Enemy spawning, collision detection, weapon behavior, boss AI
   - **Rendering**: Canvas 2D drawing for player, enemies, bullets, effects
   - **Persistence**: LocalStorage for best score, mute state, difficulty, reduced-motion preference

### Key Gameplay Systems

- **Weapon System**: Three weapon types with leveling (Vulcan, Spread, Laser)
- **Bomb Mechanic**: Limited bombs with strategic detonation affecting surrounding enemies
- **Boss Encounters**: Six unique boss types with distinct visual styles, health scaling, and attack patterns
- **Difficulty Levels**: Easy (4 lives, 3 bombs), Normal (3 lives, 2 bombs), Hard (3 lives, 2 bombs)
- **Scoring**: Base score for kills + multiplier based on no-damage streaks and weapon type
- **Stage Progression**: Advance one stage per 18 enemy kills; visual theme changes per stage
- **Performance Adaptation**: Automatic quality scaling (glow effects, shadows) based on frame-time tracking

### Procedural Audio

The audio system uses the WebAudio API to synthesize sounds on-the-fly:
- Square/sine waves with frequency sweeps for weapon fire
- Noise bursts for explosions and impact
- Context-aware panning based on screen position
- Master gain with dynamic compression

## Development Notes

### No Build Step

The game requires **no build process** — simply open `index.html` in a web browser. All logic is inline JavaScript with no imports or external dependencies.

### Testing the Game

To run the game locally:
```bash
# Option 1: Open directly in a browser
open index.html

# Option 2: Serve via a simple HTTP server (to avoid CORS issues with some systems)
python3 -m http.server 8000
# Then visit http://localhost:8000 in a browser
```

### Key Browser APIs Used

- **Canvas 2D**: `canvas.getContext('2d')` for all rendering
- **WebAudio**: `AudioContext` for procedural audio (requires user interaction to resume)
- **LocalStorage**: Persisting high score, mute state, settings
- **RequestAnimationFrame**: Game loop timing
- **Pointer Events & Touch Events**: Input handling across desktop and mobile

### Performance Considerations

- **Adaptive Quality**: Frame times are tracked and quality is adjusted (quality variable ranges 0–2)
  - Quality 0: Full glow effects and shadows
  - Quality 1–2: Reduced visual effects to maintain 60 FPS
- **Canvas Size**: Responsive to window size; max-width/height constraints in CSS prevent distortion on ultra-wide displays
- **Mobile Optimization**: Touch controls automatically show on mobile; joystick opacity and button sizes scale with viewport

### State Management

Game state is scattered across JavaScript variables. Key state variables:
- `score`, `bestScore`, `lives`, `mult` (multiplier), `stage`, `difficulty`
- `player` object: position, size, weapon level, health
- `enemies`, `bullets`, `enemyBullets`, `powerUps` arrays
- `boss` object: active boss or null
- `muted`, `reducedMotion`, `paused` boolean flags

LocalStorage keys:
- `skyfighter.bestScore` — High score
- `skyfighter.muted` — Mute toggle
- `skyfighter.difficulty` — Preferred difficulty
- `skyfighter.reducedMotion` — Motion reduction preference

### Adding New Features

Common modifications:
- **New enemy type**: Add entry to `ENEMY_TYPES` object, update enemy rendering in the render section
- **New weapon**: Add to weapon definitions (Vulcan/Spread/Laser logic), update weapon upgrade system
- **New boss**: Add to `BOSSES` array, implement boss behavior in the main game update loop
- **New visual theme**: Add to `THEMES` array with sky gradient colors and lighting
- **Difficulty tuning**: Adjust `DIFFICULTIES` object parameters (`spawnBase`, `enemySpeed`, etc.)

### Audio Synthesis Functions

- `beep(freq, dur, type, vol, opts)`: Oscillator-based tone with optional frequency sweep, filter, panning
- `noiseBurst(dur, vol, opts)`: Filtered white noise for explosive sounds
- `ensureAudio()`: Initializes AudioContext on first use (respects mute state)

### Accessibility Features

- **Reduced Motion**: Respects `prefers-reduced-motion` system preference; removes animations when enabled
- **Screen Reader Support**: Announcements for game events via `announce()` function and `#srLive` element
- **Keyboard Controls**: Full game playable without mouse/touch
- **Audio Toggle**: Mute button accessible from any game state
