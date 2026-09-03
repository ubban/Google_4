# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game implemented as a single-file HTML5 application. The game features procedural enemy spawning, multiple weapon systems, boss encounters, and a dynamic difficulty system. All game logic, audio synthesis, and rendering are contained within `index.html` with no external dependencies.

## Quick Start

### Running the Game

1. Open `index.html` in any modern web browser (no build step required)
2. Alternatively, serve via HTTP to enable localStorage and avoid CORS issues:
   ```bash
   python3 -m http.server 8000
   # Then open http://localhost:8000
   ```

### Game Controls

- **Keyboard**: Arrow keys or WASD to move, Space to fire, X/Shift for bombs
- **Touch**: Virtual joystick on left, fire button on right, bomb button available
- **UI**: P or Escape to pause, click difficulty buttons to change before starting

## Architecture & Key Systems

### Game Loop & State Management

The game runs via `requestAnimationFrame` with frame-based timing. Key state variables:
- `running` / `paused` - game state flags
- `player` - player object with position, health, weapons
- `enemies` / `projectiles` / `bombs` - pooled entity arrays
- `stage` / `wave` - progression tracking
- `score` / `multiplier` - scoring system

### Entity System

Entities (player, enemies, projectiles) are stored in arrays and updated/rendered in the main game loop:
- **Player**: manages health, weapons, movement, collision
- **Enemies**: three types (grunt, interceptor, tank) spawn based on stage/wave with adaptive spawning intervals
- **Projectiles**: bullets from player and enemies; support weapon types with different patterns
- **Bombs**: smart bomb system triggered by bomb key or powerups, clears enemies with flash effect

### Configuration Objects

Game behavior is controlled by modular config objects defined near line 485-515:
- `DIFFICULTIES` - spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier
- `THEMES` - 6 visual themes (sky gradients, cloud/sun colors, stage names)
- `ENEMY_TYPES` - grunt, interceptor, tank with stats (HP, speed, score, appearance)
- `BOSSES` - 6 boss patterns (VANGUARD, BEHEMOTH, SPECTER, etc.) with visual colors and HP multipliers

Modifying these objects changes game balance without editing core logic.

### Weapons & Powerup System

Weapon system at line ~1050+:
- Multiple weapon types with different fire patterns (VULCAN, SPREAD, MISSILE, LASER, etc.)
- Weapons upgrade (level 1-3) via powerups
- Fire rate and pattern scale with weapon level
- Multiplier system boosts score for consecutive kills (tracks in `multiplier` variable)

### Audio Synthesis

WebAudio-based sound generation (no external audio files) at lines 542-676:
- `beep()` - tone generation with frequency sweep, filtering, panning
- `noiseBurst()` - noise-based explosion/impact sounds
- Sound effects mapped in `SFX` object: shoot, hit, explode, powerup, boss telegraph, gameover, etc.
- Master gain and compressor for dynamic range control
- Mute toggle persists via localStorage

### Input Handling & Viewport

- Keyboard input (line 769-776) - standard key binding
- Touch/joystick (line 781-820) - virtual analog stick and fire/bomb buttons
- Virtual joystick visualized with base circle and stick element (line 800+)
- Canvas resizing (line 753-765) - scales based on window size with 1.6x max scale, maintains 480x720 base

### Canvas Rendering

- 2D canvas context for all drawing (no WebGL)
- Base canvas size: 480x720 pixels
- Screen effects: vignette on critical health, white flash on hit/bomb, screen shake on damage
- Rendering order: background gradient, stage theme, player, enemies, projectiles, UI HUD, effects

### Persistence & Settings

localStorage keys (line 463-466):
- `skyfighter.bestScore` - high score persistence
- `skyfighter.muted` - audio mute state
- `skyfighter.difficulty` - selected difficulty level
- `skyfighter.reducedMotion` - accessibility setting for reduced visual effects

### Accessibility Features

- Screen reader live region (`srLive`) for game announcements
- Reduced motion toggle respects `prefers-reduced-motion` media query
- High contrast UI with semi-transparent overlays
- Touch controls as fallback to keyboard

## Common Development Tasks

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` object (line 502-506) with properties: w, h, hp, speedMul, color, dark, score, minStage, optional weave
2. Add spawn logic in wave progression system (search "ENEMY_TYPES[")
3. Add explosion sound effect mapping in `EXPLODE_SFX` (line 678)

### Adding a New Weapon

1. Define weapon properties in the weapons configuration area (~line 1050+): fireRate, pattern, projectile count/spread
2. Add weapon name to UI weapon name cycle
3. Update weapon rendering to show visual representation
4. Ensure fire pattern logic handles projectile spawn based on weapon type

### Adding a New Boss

1. Add entry to `BOSSES` array (line 508-515) with name, hull colors, core color, hpMul
2. Create boss AI pattern in boss update logic (search "boss.phase" and "boss.pattern")
3. Add boss telegraph and defeat sound effects
4. Set boss HP based on stage progression: `boss.hp = BOSSES[bossIndex].hpMul * baseBossHp * DIFFICULTIES[difficulty].bossHpMul`

### Adjusting Game Balance

- **Difficulty**: modify `DIFFICULTIES` object spawn rates, enemy speed, enemy shoot chance
- **Enemy/Boss Strength**: scale HP or damage values in `ENEMY_TYPES` and `BOSSES`
- **Score/Multiplier**: modify base scores in `ENEMY_TYPES`, multiplier increment rate
- **Stage Progression**: change `KILLS_PER_STAGE` (line 491) to control stage length

### Testing Visual Changes

Modify CSS in the `<style>` block (line 10-200+) for:
- Canvas gradients and sky backgrounds
- HUD styling and positioning
- Color schemes and animations
- Responsive scaling

## Performance Considerations

- Adaptive quality system (lines 473-483) throttles rendering/effects if FPS drops
- Enemy/projectile pooling reduces GC pressure
- Canvas rendering is immediate (no double-buffering needed)
- Audio synthesis on-demand; avoid excessive simultaneous sounds

## Notes for Contributors

- All code is in a single IIFE (Immediately Invoked Function Expression) to avoid global scope pollution
- No transpilation or bundling; runs natively in browsers
- Test on both desktop (keyboard + mouse) and mobile (touch) inputs
- Verify localStorage access works (may be restricted in certain contexts)
- Keep visual themes cohesive across new enemy/boss additions
