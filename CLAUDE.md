# Sky Fighter — Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single-file HTML5 canvas application. It features:
- Classic arcade gameplay with enemy waves and boss battles
- Multiple weapons (Vulcan/Laser) with 3 power levels
- Strategic smart-bomb mechanic with limited uses
- Stage-based progression with boss cycling
- Score multiplier system via kill-chain combos
- Mobile-friendly controls (touch joystick, keyboard, mouse)
- Adaptive performance quality tier system
- Accessibility features (reduced-motion toggle, screen reader support)
- Rich synthesized audio with spatial panning

## Architecture

### Single-File Structure
All code (HTML, CSS, JavaScript) lives in `index.html` (~1669 lines) as a self-contained web application. This allows instant play in any modern browser without build steps or dependencies.

### Core Game Loop
The main game loop runs via `requestAnimationFrame` and handles:
1. Input polling (keyboard, touch joystick, mouse)
2. Entity updates (player, enemies, bullets, power-ups, boss)
3. Collision detection (bullets↔enemies, player↔enemies, player↔power-ups)
4. Rendering to canvas

### Game State Management
- **Entities**: player, enemies (array), enemy bullets, player bullets, power-ups, boss (single)
- **Game flow**: title screen → stage loop → boss → stage clear → next stage → game over
- **Persistence**: localStorage saves high score, mute state, difficulty level, and reduced-motion preference
- **Pause & HUD**: Real-time stats display (hearts, bombs, weapon, multiplier) with pause overlay

## Key Technologies

- **Canvas 2D API**: All rendering
- **Web Audio API**: Synthesized sound effects with dynamics compressor, biquad filtering, pitch sweeps
- **localStorage**: Persistent game state across sessions
- **Keyboard, Touch, Mouse APIs**: Player input handling
- **requestAnimationFrame**: Smooth 60fps rendering with frame-time monitoring

## Main Features

### Gameplay
- **3 Difficulty Tiers**: affects enemy spawn rate, bullet speed, damage scaling
- **Weapon System**: Vulcan (rapid-fire hitscan) and Laser (piercing beam), each upgradeable to 3 levels
- **Smart Bombs**: Limited-use screen-clear with boss damage; recharge via power-ups
- **Stage Progression**: 6 stages + infinite loop; bosses from a 3-design roster cycle (Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan)
- **Boss Patterns**: Each boss type has 3+ attack patterns (e.g., Vanguard: radial burst, aimed fan, wide V) with telegraph flash for readability
- **Kill Chain**: Score multiplier builds on consecutive enemy kills, resets on player hit; visual feedback via on-screen combo popup

### Visual Polish
- **Parallax clouds**: Multi-layer background scrolling at different speeds
- **Shaded planes**: Cockpit, flame, tail detail with selective darkening
- **Particle effects**: Bullet glow (quality-adaptive), explosion clouds, power-up glitter
- **Screen shake & hit flash**: On player damage; dampened when reduced-motion is on
- **Vignette pulse**: Red overlay when low-health (< 20%), pulses in sync with screen shake
- **Stage themes**: 6 background color schemes (daylight, sunset, night, etc.) that cycle per stage
- **UI polish**: Pill-shaped HUD badges with glassmorphic backdrop-filter blur, smooth entrance animations

### Accessibility
- **Reduced-Motion**: Disables screen shake, hit flash, vignette pulse animations for photosensitive users; defaults to OS `prefers-reduced-motion` setting
- **Screen Reader Support**: aria-live region announces life loss, boss defeats, and game-over results
- **Control Options**: Keyboard, touch joystick, mouse steering all work independently

### Performance Adaptation
- **Frame-time monitor**: Samples frame duration each render and steps quality tiers automatically
  - **Tier 2 (low)**: Reduce particle counts, drop glow shadows
  - **Tier 1 (mid)**: Normal rendering
  - **Tier 0 (high)**: Full effects
- Steps down when frame-time exceeds 18ms, steps up when it drops below 12ms, staying smooth on slower devices

### Audio System
- **Master gain + dynamics compressor**: Prevents clipping on overlapping explosions
- **Per-sound biquad filtering**: Shapes tone of each SFX
- **Pitch sweeps**: Adds weight and modulation to synth tones
- **Stereo panning**: Events pan left/right based on screen x-position for spatial depth
- **Layered sequences**: Multi-part sounds for explosions, boss attacks, stingers

### Controls
- **Keyboard**: Arrow keys to move, Space to fire, B for bomb, P to pause, M to mute, A for reduced-motion toggle
- **Touch**: Virtual joystick (left side) + fire button (right side)
- **Mouse**: Moving pointer glides plane (capped at 1.7x speed), click to fire, crosshair reticle

## Code Structure (High-Level)

### Game Configuration
- `STAGE_COUNT`, `ENEMY_TYPES`, `BOSS_DESIGNS`: Static tables defining waves, enemy stats, and boss attack patterns
- Quality settings, UI parameters, canvas dimensions, audio sample rates

### Game State Variables
- Player: position, velocity, weapon level, bomb count, health, direction
- Enemies: position, velocity, health, type (grunt/interceptor/tank), color (wave-variant)
- Bullets: position, velocity, width/height, laser flag, owner
- Power-ups: position, velocity, type (weapon/bomb/health)
- Boss: position, bullet list, health, pattern index, attack timer, telegraph flash state
- Global: stage, score, multiplier, combo, paused, game-over, quality tier, etc.

### Key Functions
- `gameLoop()`: Main render/update loop
- `update()`: Step all entities, check collisions, spawn waves/boss
- `render()`: Draw background, entities, HUD, overlays
- `updateQuality()`: Adjust rendering fidelity based on frame-time
- `playSynth()` / `playBomb()` / `playBossAttack()` etc.: Audio synthesis and playback
- `drawPlane()`: Utility to render player and enemy aircraft with shading
- `checkCollisions()`: Broad collision detection for bullets, enemies, power-ups

## Development Notes

### Adding New Features
1. **New Boss Design**: Add entry to `BOSS_DESIGNS` array (colors, attack patterns, health)
2. **New Enemy Type**: Extend `ENEMY_TYPES` with new wave config
3. **New Sound**: Call `playSynth()` with custom osc/filter/gain params
4. **New Power-up Type**: Add case in power-up collision handler

### Testing
- Verified headlessly via Playwright across:
  - All difficulty modes (no crashes)
  - Full stage/boss cycle (all bosses spawn and die correctly)
  - Enemy types (all three archetypes work)
  - Weapon/bomb/combo mechanics
  - Mute, reduced-motion, pause combined

### Performance Considerations
- Canvas rendering is the main bottleneck; quality tiers drop shadow-blur on glow effects and reduce particle counts when frame-time is high
- Audio context is created on first interaction (due to browser autoplay policy)
- Collision detection uses simple AABB (axis-aligned bounding box) checks, O(n²) but acceptable for typical wave sizes

## File Reference
- `index.html`: Complete game (HTML + CSS + JS in one file)
- `README.md`: Minimal project header
- `CLAUDE.md`: This documentation

## Deployment
The game is a static HTML file. Deploy to any web server or serve locally; it will work without backend or build tools.

Open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge) on desktop or mobile.
