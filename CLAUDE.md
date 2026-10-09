# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely with HTML5 Canvas and vanilla JavaScript. The entire game lives in a single `index.html` file with embedded CSS and JavaScript (no build step required).

## Running the Game

The game runs directly in a web browser — no build tools needed:

```bash
# Option 1: Open directly in a browser
open index.html

# Option 2: Start a local web server (recommended for development)
python -m http.server 8000
# Then visit http://localhost:8000
```

The game is fully playable on both desktop (mouse/keyboard) and mobile (touch controls).

## Codebase Structure (Single-File Architecture)

All code is in `index.html`, organized into logical sections:

### HTML Structure (Lines 340-420)
- Game canvas container
- HUD elements (score, lives, weapons, bombs)
- Control buttons (mute, motion toggle, pause)
- Screen reader announcements
- Menu/difficulty selection

### CSS Styling (Lines 10-360)
- Base styling for dark theme
- Canvas and container layouts
- HUD component styles (stats, hearts, bombs)
- Animation keyframes (heartBeat, vignettePulse, screen effects)
- Touch control overlays
- Responsive design using clamp() and viewport units

### JavaScript Game Logic (Lines 422-end)
The main game script is an IIFE with these key subsystems:

#### Game Loop & Timing (Lines 476-530)
- `trackPerf(dt)` — handles frame timing and performance monitoring
- Main animation loop using `requestAnimationFrame`
- Tracks deltaTime for frame-rate-independent updates

#### Audio System (Lines 545-676)
- `beep(freq, dur, type, vol, opts)` — generates tones using Web Audio API
- `noiseBurst(dur, vol, opts)` — generates explosion/impact sounds
- `ensureAudio()` — initializes audio context on first user interaction (mobile requirement)
- Stereo panning for spatial audio feedback
- Mute toggle support

#### Input Handling (Lines 785-880)
- **Keyboard**: Arrow keys, Space (shoot), B (bomb), P (pause), M (mute)
- **Touch**: Virtual joystick (two-stick arcade controls)
  - Left stick: move player
  - Right stick: aim/shoot
- **Mouse**: Crosshair aiming on desktop
- `joystickStart/Move/End()` — touch input tracking

#### Player Entity (Lines 990-1050)
- Position, velocity, health/lives system
- Weapon selection and power levels
- Bomb count and activation
- Weapon switching with scroll wheel or buttons
- Collision detection with enemies/projectiles

#### Enemies & Bosses (Lines 1055-1150+)
- Enemy spawning based on stage/difficulty
- Three enemy types: basic/medium/strong with different patterns
- Boss enemies with multiple health and special attack patterns
- Circular bullet patterns and spray attacks
- Difficulty scaling on Easy/Normal/Hard

#### Combat Systems (Lines 1200-1300)
- Player bullets with weapon types (Vulcan, Cannon, Laser, Spreader)
- Different fire rates, damage, and visual patterns per weapon
- Enemy projectiles with tracking/circular patterns
- Bomb explosions with area damage
- Hit detection and screen flash feedback

#### Scoring & Multiplier (Lines 1050+)
- Kill streak tracking
- Score multiplier (1x → 8x with consecutive kills)
- Difficulty bonus (Easy: 0.8x, Normal: 1x, Hard: 1.5x)
- Best score persistence with localStorage

#### Visual Effects (Lines 698-770)
- Screen shake on damage/bombs
- Hit flash (white fade)
- Bomb flash (larger white fade)
- Vignette pulse when health critical
- Combo pop-up notifications
- Stage banner notifications

#### Rendering (Lines 1500+)
- Canvas-based drawing for all entities
- Particle effects for explosions and impacts
- HUD updates (hearts, bombs, score, multiplier)
- Touch control UI rendering

#### Difficulty & Balance
- Three difficulty levels affecting:
  - Enemy spawn rates and patterns
  - Enemy bullet density
  - Score multipliers
  - Boss behavior
- Adjustable in the menu or with `setDifficulty(name)`

## Development Workflow

### Editing Game Mechanics

1. **Modify player behavior** → Search for `player` object, update velocity/position in game loop
2. **Adjust enemy spawning** → Look for `enemySpawner` or stage-based spawning logic
3. **Tweak weapon balance** → Find weapon definitions with fire rate, damage, and spread values
4. **Change difficulty scaling** → Find `difficultyMult` or per-difficulty adjustments
5. **Add visual effects** → Use `screenShake()`, `flashHit()`, `flashBomb()`, or add canvas drawing calls

### Adding Sounds

Call `beep(frequency, duration, waveType, volume, options)` for tones or `noiseBurst(duration, volume, options)` for noise-based effects. Frequency in Hz, duration in seconds.

### Testing Changes

1. Save the file
2. Reload the browser tab (Cmd/Ctrl+R)
3. Test the specific feature (e.g., fire a weapon, hit enemies, trigger bombs)
4. Check console for any errors (F12 → Console tab)

### Performance Monitoring

The game logs frame timing data to console (`trackPerf`). Check if frame rate is stable near 60 FPS. Use the motion toggle button (🎆) to disable animations on lower-end devices.

### Accessibility Considerations

- Screen reader announcements via `announce(text)` to `#srLive` element
- Keyboard-only playable (arrows + space)
- High contrast colors for HUD
- Reduced motion support (respects `prefers-reduced-motion` and toggle button)
- Touch controls for mobile

## Common Tasks

**Add a new weapon type:**
- Define stats (fire rate, spread, damage) in weapon data
- Add firing logic in the game loop
- Add visual rendering in the draw phase

**Adjust enemy difficulty:**
- Modify spawn frequency in `enemySpawner`
- Change bullet patterns in enemy update logic
- Adjust health/damage values

**Change visual theme:**
- Update colors in CSS (lines 10-360)
- Modify canvas gradient in `#gameCanvas` style
- Adjust HUD colors and effects

## Git Workflow

Work on feature branches (e.g., `claude/improving-<feature>`) and commit with clear messages. Keep commits focused on single features or fixes.
