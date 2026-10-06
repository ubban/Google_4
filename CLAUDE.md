# Sky Fighter Codebase Documentation

## Project Overview

**Sky Fighter** is a production-quality Raiden-style vertical shoot-em-up game built as a single-page HTML5 Canvas application. Players control an airplane, switch between weapon types, deploy smart bombs to clear the screen, and battle stage bosses across multiple levels. The game features rich visuals, dynamic audio, mobile controls, and comprehensive accessibility support.

### Core Features
- **Gameplay**: Arcade-style vertical scrolling shooter with wave-based enemy progression and stage bosses
- **Weapons System**: Vulcan/Laser dual-mode with 3 power levels, each with distinct fire rates and damage
- **Smart Bombs**: Limited-use screen-clear attacks that also damage bosses
- **Boss Encounters**: 6 boss designs (Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan) with distinct attack patterns
- **Enemy Variety**: 3 enemy archetypes (grunt, weaving interceptor, multi-hit tank)
- **Power-ups**: Weapon level increases, bomb stock, extra lives with visual drops
- **Difficulty Modes**: Adjustable game difficulty affecting spawn rates and enemy behavior
- **Audio Engine**: Synthesized audio with dynamics compression, per-sound filtering, and stereo panning
- **Mobile Support**: Touch joystick controls with adaptive quality scaling
- **Accessibility**: Reduced-motion toggle (respects prefers-reduced-motion), screen reader announcements, high contrast UI
- **Persistence**: Game state saved to localStorage (score, progress, settings)

## Code Architecture

### Single-Page Structure
The entire game is implemented in a single `index.html` file (~1600 lines) combining HTML markup, CSS styling, and JavaScript game logic in embedded `<script>` tags.

### Key Subsystems

#### 1. **Rendering System** (`drawGame()`, `draw*()` functions)
- Canvas-based 2D rendering with requestAnimationFrame loop
- Parallax cloud layers for visual depth
- Plane rendering with shaded cockpit, flame trail, and damage flash
- Enemy and boss visual states (colors, health indicators)
- HUD overlays: score, lives, weapon level, bomb count, combo popups
- glassmorphic card UI with rounded borders and semi-transparent backgrounds

#### 2. **Game State Management**
- Central `game` object tracking: player, enemies, bosses, bullets, stage, difficulty, score, lives
- `localStorage` persistence for high scores and settings
- State transitions: menu → gameplay → boss → stage clear → next stage / game over

#### 3. **Input Handling**
- **Keyboard**: Arrow keys (movement), Space (fire), B (bomb), P (pause), M (mute), D (difficulty), R (reduced motion toggle)
- **Touch**: Joystick overlay on mobile with dynamic radial drag detection
- **Mouse**: Pointer tracking for steering (priority given to keyboard/touch input)
- All inputs throttled/debounced to prevent accidental double-triggers

#### 4. **Physics & Collision**
- Continuous movement with momentum-based velocity
- Circular hitbox collision detection for bullets → enemies/bosses
- Collision callbacks trigger damage, scoring, power-up drops
- Screen bounds clamping for all moving entities

#### 5. **Enemy & Boss AI**
- **Enemy Spawning**: Wave-based spawning from the top of the screen, positioned by wave pattern
- **Enemy Behavior**: 
  - Grunt: linear downward movement, random fire
  - Interceptor: sinusoidal weaving, aimed fire toward player
  - Tank: slow movement, multiple hit points, aura effect
- **Boss Behavior**: Multi-phase pattern cycling (e.g., Vanguard: radial burst → aimed fan → wide V)
  - Boss telegraph flash 0.4s before each attack pattern
  - Boss health bar rendering; stage clears on death
  - Cyclic boss roster: stage 1→Vanguard, stage 2→Behemoth, stage 3→Specter, then repeat with new designs

#### 6. **Audio Engine**
- **Master Gain + Compressor**: Prevents clipping on overlapping sounds
- **Per-Sound Filters**: Biquad filtering for tone shaping
- **Stereo Panning**: Audio positioned left/right based on on-screen event location
- **Sound Library**: Enemy explosions (grunt/interceptor/tank), hit/bomb/powerup cues, boss telegraph/death, game-over stinger, UI clicks
- **Mute Toggle**: Controlled via UI and M key; persisted to localStorage

#### 7. **Performance Monitoring & Adaptation**
- Rolling frame-time tracker sampling the last 30 frames
- Three quality tiers:
  - **High**: Full particle effects, shadow glow, screen shake
  - **Medium**: Reduced particles, no glow
  - **Low**: Minimal particles, no shake/flash
- Auto-steps down if FPS drops below ~50; auto-recovers if FPS sustains >55

#### 8. **Accessibility Layer**
- **prefers-reduced-motion**: Dampens screen shake (scale 0.3), disables flash/pulse animations
- **aria-live Region**: Announces life loss, boss defeats, game-over results for screen readers
- **High Contrast UI**: White text on dark backgrounds, clear visual feedback
- **Pause/Mute Toggle**: Visual indicators in top bar, keyboard + mouse accessible

## Game Flow & Progression

```
Start Screen (title, instructions)
    ↓
[Player selects difficulty: Easy/Normal/Hard]
    ↓
Stage 1: Enemy waves → Boss encounter (Vanguard/Dreadnought/Wraith)
    ↓
Stage 2: Enemy waves → Boss encounter (Behemoth/Titan/Specter cyclically)
    ↓
Stage N: Continues indefinitely with visual theme rotation
```

### Visual Progression
- **Backgrounds**: 6 stage backdrop themes cycle (daylight, sunset, night, repeat)
- **Boss Designs**: 6 boss designs cycle by stage (each with unique colors and attack patterns)
- **Difficulty Scaling**: Enemies spawn more frequently and move faster at higher difficulties

## Key Data Structures

### `game` Object
```javascript
{
  player: {x, y, width, height, angle, speed, health, fireMode, weaponLevel, bombCount},
  enemies: [{x, y, width, height, type, health, angle, velocityX, velocityY}, ...],
  bosses: [{name, x, y, width, height, health, maxHealth, pattern, phaseTimer, ...}, ...],
  bullets: [{x, y, velocityX, velocityY, source, damage}, ...],
  powerUps: [{x, y, type}, ...],
  explosions: [{x, y, life, radius}, ...],
  particles: [{x, y, vx, vy, life, color}, ...],
  stage: 1,
  wave: 1,
  difficulty: 'normal',
  isGameOver: false,
  score: 0,
  lives: 3,
  combo: 0,
  isPaused: false,
  isMuted: false,
  reduceMotion: false,
  time: 0,
  qualityTier: 'high' | 'medium' | 'low'
}
```

## Important Implementation Details

### Scoring System
- Base kill reward: 100 points (varies by enemy type)
- Combo multiplier: +1x for every 5 consecutive kills; resets on 2-second idle
- Boss kill: 1000 + (current combo) points
- Combo popups displayed on-screen for visual feedback

### Weapon Balance
- **Vulcan (default)**: High fire rate, lower per-shot damage; unlimited ammo
- **Laser**: Lower fire rate, higher per-shot damage; unlimited ammo
- **Power-ups**: Increases `weaponLevel` (1→2→3), improving fire rate and damage per shot

### Boss Health & Balance
- Boss health scales with difficulty
- Multi-phase patterns telegraph 0.4s before firing
- Smart bombs deal fixed damage, allowing player strategies to skip/shorten phases

### Save/Load Mechanism
- `localStorage.setItem()` stores: high score, sound mute state, motion preference
- On load, these are restored to UI and game state
- Per-session: progress (stage, lives, score) is NOT persisted; only statistics

## Development Guidelines

### Adding New Features
1. **New Enemy Type**: Add archetype to `spawnWave()` switch; define movement/AI in `updateEnemies()` and rendering in `drawEnemies()`
2. **New Boss Pattern**: Add pattern name to boss data; define phase logic in `updateBosses()` switch statement for pattern execution
3. **New Audio**: Create synth call in `playSound()` switch; include gain, filter, and panning params
4. **New Power-up**: Define drop type in `dropPowerUp()`; handle pickup in collision detection; apply effect in player logic

### Testing
- **Headless Testing**: Run with Playwright to verify spawning, collisions, weapon pickups, boss patterns, stage advancement without console errors
- **Manual QA**: Test all three difficulties, verify audio, reduced-motion toggle, pause, bomb usage, and boss defeat progression
- **Mobile Testing**: Test touch joystick on tablet/phone; verify performance scaling on older devices

### Debugging Tips
- Check `console.log()` output in browser DevTools; no errors should appear during normal gameplay
- Use `game.isPaused = true` in console to pause and inspect state
- Monitor frame time (logged to console) to trigger quality tier changes manually
- Test audio context creation: some browsers restrict autoplay of audio

## Browser & Platform Support

- **Desktop**: Modern Chrome, Firefox, Safari (2023+)
- **Mobile**: iOS Safari 15+, Android Chrome
- **Audio**: Requires Web Audio API (supported in all modern browsers)
- **Local Storage**: Required for persistence (no server backend)

## File Organization
- `index.html`: Single source file containing all HTML, CSS, and JavaScript

## Known Limitations & Future Enhancements
- No multiplayer or network features
- Boss AI patterns are procedurally generated; tuning is manual via constants at top of code
- Mobile performance may degrade on low-end devices; quality tier auto-scales to compensate
- Audio doesn't load from external files; all sounds are synthesized in real-time

## Attribution
- Game design inspired by classic Raiden series
- Web Audio API synthesis approach derived from modular synth patterns
- Accessibility guidelines follow WCAG 2.1 Level AA
- Canvas rendering patterns follow MDN best practices

---

**Last Updated**: October 6, 2026
**Version**: Production (Single-page HTML5 game)
