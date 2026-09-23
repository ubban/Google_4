# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game (1600+ lines) is embedded in `index.html` with no external dependencies or build system.

## Running the Game

- **Open locally**: Open `index.html` directly in a web browser
- **Development preview**: Use `npx http-server` or similar to serve and test locally
- **Mobile-friendly**: Supports touch controls with full responsive scaling

The game runs in a single `requestAnimationFrame` loop with fixed timestep logic (`dt = Math.min(dt, 0.05)`).

## Code Architecture

The game is organized as a single-file, function-scoped application with these core systems:

### Game State & Configuration
- Global constants: `WIDTH`, `HEIGHT`, `MAX_HEALTH`, `MAX_BOMBS`, damage values, speed configs
- Game state object: `{ score, health, bombs, comboDmg, mults, weapon, ... }`
- Performance tracking (frame time/FPS via `trackPerf()`)

### Core Game Loop
- **`update(dt)`**: Handles all game logic per frame
  - Player input (keyboard + mouse/touch) and movement
  - Enemy spawning and AI (wave system)
  - Bullet collision detection with enemies and obstacles
  - Power-up/dropped item physics and collection
  - Stage progression and difficulty scaling
- **`draw()`**: Renders everything via Canvas 2D context
  - Background/sky gradient
  - Game entities (player, enemies, bullets, particles)
  - HUD overlays (health, bombs, weapon, combo multiplier)
  - Visual effects (screen shake, flash, vignette)

### Player & Combat
- Player aircraft: position, health, bomb count, weapon state
- Weapon system: tracks current weapon index, damage scaling with combo multiplier
- Input handling: `addEventListener('keydown'/'keyup'/'mousemove'/'touchmove')` for cross-platform control
- Smart bomb mechanic: clears enemies and awards score bonus

### Enemy System
- Enemy waves: spawned based on `stageWaves[currentStage]` configuration
- Types: standard fighters with circular/linear/pursuit patterns
- Boss encounters: triggered at stage transitions
- Collision-based damage and destruction

### Visual & Audio
- Canvas context state: shadows, rotations, globalAlpha for effects
- Particles: simple life-based system for explosions/hits
- Screen shake: invoked via `shake()`; fades over `shakeTime`
- Audio: Single audio context for SFX; preloaded/cached note buffers

### UI & State Management
- Game states: running/paused/game-over
- HUD elements: `#hud`, `#hearts` (health), `#bombs`, stats display
- Modal overlays: start screen, pause menu, game-over screen with high scores
- Local storage: persists high scores in browser

## Common Modifications

### Tweaking Difficulty or Balance
- Enemy damage: adjust `dmgFromEnemy` (line ~300s)
- Player max health: modify `MAX_HEALTH` constant
- Wave timing: edit `stageWaves[]` array
- Weapon damage: scale `enemyHealth` or `baseDmg` multipliers

### Adding New Stages/Wave Patterns
- Extend `stageWaves` array with new wave configs
- Each wave: `{ delay, type, pattern, ... }`
- Boss definitions: add boss object with `health`, `pattern`, `moveStyle`

### Adjusting Visual Effects
- Screen shake intensity: change multiplier in `shake()`
- Flash effect: modify `@keyframes flashHit`/`flashBomb` durations
- Vignette color/intensity: edit `#vignette` styles and `vignettePulse` animation

### Mobile & Responsive
- Touch input already mapped via `touchmove` listener
- Canvas scales responsively via `clamp()` CSS functions
- Safe area insets for notches: `env(safe-area-inset-top)` in HUD positioning

## Performance Considerations

- **Timestep capping**: `dt = Math.min(dt, 0.05)` prevents huge jumps on slow frames
- **Frame skipping**: Physics updates decouple from render via fixed `dt`
- **Particle pool**: Reuses array; old particles are replaced (no alloc churn)
- **Canvas context saves**: `ctx.save()`/`ctx.restore()` around rotations/transforms

## File Structure

```
.
├── index.html          # Entire game: HTML + embedded <script>
├── README.md           # Basic project metadata
└── Claude.md           # This file
```

## Testing & Debugging

- **Browser DevTools Console**: Check for JS errors or debug logs
- **FPS counter**: Built-in via `perf` object (printed each frame if needed)
- **Local storage**: Clear with `localStorage.clear()` in console to reset high scores
- **Canvas debugging**: Right-click → Inspect, view canvas rendering in DevTools

## Notes for Future Modifications

- The entire game state and logic lives in one function scope to avoid global namespace pollution
- Canvas 2D rendering is the core; no WebGL or shader complexity
- Audio uses `AudioContext` with pre-computed sine-wave buffers for SFX
- No external libraries; vanilla JS and HTML5 Canvas only
- Combo multiplier system: tie weapon damage to consecutive hits; resets on miss/health loss
