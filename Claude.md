# Sky Fighter — Development Guide

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-page HTML5 canvas application. This document guides Claude Code agents in understanding the codebase and making effective improvements.

## Project Overview

- **Type**: Single-file HTML5 game
- **Architecture**: Monolithic canvas-based game loop in `index.html` (1,669 lines)
- **Core Language**: JavaScript with HTML/CSS
- **No external dependencies** — pure vanilla JavaScript for maximum portability
- **Playable**: Open `index.html` in any modern browser

## Game Mechanics

### Core Systems

1. **Player Aircraft**: Center-controlled vertical movement with weapon systems
   - Keyboard: Arrow keys to move, Space to fire
   - Touch: On-screen joystick and fire button
   - Mouse: Pointer steering (capped at 1.7x speed) with click to fire

2. **Weapon System**: Progressive upgrade path
   - Vulcan gun (rapid fire, spread pattern at higher levels)
   - Laser (concentrated beam, piercing at higher levels)
   - Three power levels per weapon with visual and mechanical differences
   - Weapon power-ups drop from defeated enemies

3. **Bombs**: Limited-use screen-clear with boss damage
   - Spawned as power-ups
   - Counter displayed in HUD
   - Clears all on-screen bullets and damages boss

4. **Enemy Variety**: Three distinct archetypes
   - **Grunt**: Basic enemy, quick kill
   - **Weaving Interceptor**: Patterns evasive movement
   - **Tank**: Multi-hit health bar, bullets on death

5. **Boss Encounters**: Six rotating boss designs
   - **Vanguard**: Radial burst, aimed fan, wide V patterns
   - **Behemoth**: Heavy columns, homing spread, bullet wall
   - **Specter**: Teleport dash, spiral, twin aimed shots
   - **Dreadnought, Wraith, Titan**: Additional designs with unique patterns
   - Telegraph flash (0.2s) before each attack for fair difficulty
   - Boss health bar displayed during encounter
   - Distinct defeat stinger and stage-advance fanfare

6. **Scoring System**: Kill-chain multiplier
   - Combo counter increments on hits
   - Resets on player death
   - Multiplies base enemy score

### Game Flow

- **Stages**: Progressive difficulty with visual themes (daylight → sunset → night → cycle)
- **Wave Structure**: Enemy waves culminate in boss battle
- **Difficulty Modes**: Easy, Normal, Hard with adjusted enemy count/speed/boss health
- **Game Over**: On player death (3 lives by default)
- **Persistence**: High score and game state saved to localStorage

## UI & Accessibility

### HUD (Head-Up Display)
- Pill-style layout: centered at top
- Shows: Current wave, lives (heart icons), weapon/bomb count, score, combo
- Responsive sizing (clamps 12–16px with viewport scaling)

### Controls Panel (Top-right)
- Mute toggle (speaker icon)
- Reduced-motion toggle (accessibility)
- Pause button
- Fixed position, z-index layering ensures clickability

### Accessibility Features
- Reduced-motion toggle (disables screen shake, flash, pulse animations)
- Respects `prefers-reduced-motion` OS setting by default
- ARIA live region announces life loss, boss defeats, and game over results
- All colors meet WCAG contrast standards in dark theme

### Visual Effects
- Parallax cloud layers (background depth)
- Sun glow with directional shading
- Low-health vignette pulse (gradual intensity increase on damage)
- Hit flash on bullet collision (dampened if reduced-motion enabled)
- Screen shake on impact (strength tied to damage)
- Particle effects for explosions and power-ups
- Glassmorphic overlay card for start/pause screens

## Performance & Optimization

### Adaptive Quality System
- Rolling frame-time monitor tracks recent FPS
- Three quality tiers auto-adjust based on device performance:
  - **Tier 1 (High)**: All effects (particles, shadows, blur glow)
  - **Tier 2 (Medium)**: Reduced particle counts
  - **Tier 3 (Low)**: Shadows and blur glow disabled
- Automatically steps down if device can't sustain 60 FPS
- Steps back up once recovery threshold is reached
- Logged to console (searchable with `performance-tier`)

### Audio Engine
- Custom synth-based audio (no audio files)
- Master gain with dynamics compressor (prevents clipping)
- Per-sound biquad filtering for tone shaping
- Pitch sweeps for sonic weight
- Stereo panning driven by on-screen position
- Mute toggle persists to localStorage
- Distinct sounds:
  - Enemy explosions (vary by type: grunt/interceptor/tank)
  - Bullet hits, bomb detonation, power-up pickup cues
  - Boss telegraph (pre-attack warning)
  - Boss defeat stinger
  - Stage-clear fanfare
  - Game-over result announcement

## Code Structure (High-Level)

Within `index.html`:

1. **HTML Head** (meta, styles)
2. **CSS** (full responsive design, dark theme, animations)
3. **Canvas & HUD Container** (game display area)
4. **JavaScript** (game logic, organized by responsibility):
   - Utility functions (distance, collision, interpolation)
   - Input handling (keyboard, touch joystick, mouse steering)
   - Audio engine (synth, playback scheduling)
   - Player aircraft and weapon logic
   - Enemy spawning and behavior
   - Boss design patterns and health tracking
   - Particle effects and visual state
   - Game state machine (menu → wave → boss → game over)
   - LocalStorage persistence
   - Main game loop (update, render, requestAnimationFrame)

## Key Integration Points

### Adding New Features
- **New Boss**: Define in `bosses` array (name, colors, pattern list, health)
- **New Enemy**: Add to `enemies` array with position/velocity/type
- **New Weapon**: Extend weapon upgrade logic in `advanceWeapon()`
- **New Visual Effect**: Create in `particles` system or extend HUD rendering
- **Audio**: Use `playSound()` with synth definition (frequency, duration, envelope)

### Testing the Game
1. Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge)
2. Use the keyboard, touch (on mobile), or mouse to play
3. Check browser console for performance logs (`[perf]` prefix)
4. Verify localStorage persists high score and mute/motion settings

### Known Constraints
- **Single file**: All code in `index.html` for easy distribution
- **Canvas-based**: No DOM manipulation for game objects (performance)
- **No external assets**: All audio is synthesized; all graphics are procedural
- **Mobile-first**: Touch joystick is primary mobile control; keyboard fallback

## Development Best Practices

1. **Preserve the monolithic structure** for portability unless a major refactor is explicitly requested
2. **Test on multiple devices** (desktop, tablet, phone) to verify touch and performance
3. **Use localStorage sparingly** — only for settings and high score
4. **Respect accessibility defaults** — reduced-motion toggle should work without user action
5. **Keep the game loop locked at 60 FPS** — profile with the frame-time monitor before adding effects
6. **Audio is optional** — mute toggle should disable all sound without breaking game logic
7. **Document complex patterns** (e.g., boss attack sequences) with comments
8. **Test cross-browser** — especially Safari (WebKit quirks with audio timing and touch events)

## Branch Strategy

- **Default branch**: `main` (production-ready game)
- **Feature branch**: `claude/inspiring-planck-pqcfy3` (current development)
- All changes committed and pushed to the designated branch

---

Generated by Claude Code on 2026-10-01
