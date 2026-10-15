# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML/JavaScript application. The entire game logic, rendering, audio synthesis, UI, and controls are contained in `index.html` (~1,700 lines).

### Key Characteristics
- **Single-file architecture**: No build process, no external dependencies, no package manager
- **Canvas-based rendering**: Uses 2D canvas for all game graphics
- **Web Audio API synth**: All sound effects are procedurally generated using oscillators and filters
- **Responsive design**: Supports both desktop (mouse/keyboard) and mobile (touch controls) input
- **Accessibility features**: Reduced motion support, screen reader announcements, dark theme

## How to Develop

### Local Testing
The game runs directly in any modern browser. No build, lint, or test commands are required.

**To test locally:**
```bash
# Simply open index.html in a web browser (file:// protocol works fine)
open index.html

# Or serve via HTTP if you need it for testing (local-only):
python3 -m http.server 8000
# Then visit http://localhost:8000
```

**To test on mobile/tablet:**
- Serve the file via HTTP from your machine: `python3 -m http.server 8000`
- Access from mobile device on the same network at `http://<your-ip>:8000`
- Test both portrait and landscape orientations
- Test touch controls: joystick zone on left, FIRE/BOMB buttons on right

### Code Organization

The `<script>` block in `index.html` (lines 422+) is organized as an IIFE (Immediately Invoked Function Expression) to maintain local scope. Key sections within the script:

| Section | Description |
|---------|-------------|
| **DOM & Storage** | Canvas, overlay, HUD element references; localStorage keys for best score, mute state, difficulty, reduced-motion preference |
| **Performance** | Adaptive quality system (`quality` variable) that reduces particle effects/draw calls if frame times degrade |
| **Game Config** | `DIFFICULTIES` (Easy/Normal/Hard parameters), `THEMES` (6 visual themes), `ENEMY_TYPES` (grunt/interceptor/tank), `BOSSES` (6 boss definitions) |
| **Audio System** | Web Audio synth functions (`beep()`, `noiseBurst()`) for procedural sound; no external audio files |
| **Game State** | Player, enemies, bullets, particles, bombs; difficulty and lives management; score and multiplier tracking |
| **Rendering** | Canvas draw functions for player, enemies, bullets, particles, UI overlays, HUD |
| **Input Handling** | Keyboard (WASD, arrows, space, X, P), mouse (move to steer, click to fire), touch (joystick + buttons) |
| **Game Loop** | `update()` and `draw()` functions; adaptive frame timing; `requestAnimationFrame` main loop |
| **Stage & Difficulty** | Enemy spawn logic; stage progression (18 kills = next stage); boss encounters |

## Game Mechanics

### Player Controls
- **Movement**: Mouse position / Arrow keys / WASD / Touch joystick
- **Fire**: Mouse click / Space / Touch FIRE button
- **Bomb**: X key / Shift / Touch BOMB button
- **Pause**: P key / Esc / Pause button in HUD
- **Mute/Motion**: Icon buttons in top-right corner

### Difficulty Settings
Three presets affect spawn rate, enemy behavior, player lives, bomb count, and boss health:
- **Easy**: Fewer enemies, slower, less frequent shots, 4 lives, 3 bombs
- **Normal**: Balanced (default), 3 lives, 2 bombs
- **Hard**: Frequent spawns, fast, high shot rate, 3 lives, 2 bombs, tougher bosses

### Stage Progression
- Each stage requires 18 enemy kills
- Stages 1–6 have unique themes (6-stage cycle; repeats after stage 6)
- After each stage, one of 6 bosses spawns
- Boss defeat advances to next stage

### Enemy Types
| Type | HP | Speed | Score | First Stage | Special |
|------|----|----|-------|------------|---------|
| Grunt | 1 | 1.0x | 10 | 1 | Basic enemy |
| Interceptor | 1 | 1.7x | 15 | 1 | Weaves side-to-side |
| Tank | 4 | 0.55x | 35 | 2 | High health, slow |

### Weapons & Power-ups
- **Weapon leveling**: Collected weapon pickups increase level (max ~5), increasing spread and rate
- **Smart bombs**: Clear all enemies and bullets on screen; damage bosses
- **Score multiplier**: Increases with consecutive hits; resets on player hit; boosts score by multiplier factor

## Important Constants & Variables

### Stage & Difficulty
```javascript
const KILLS_PER_STAGE = 18;              // Enemies to defeat to advance
const DIFFICULTIES = { easy, normal, hard }; // Difficulty configs
```

### Visual Themes
```javascript
const THEMES = [                         // 6 stage themes
  { sky: [...], cloud: '...', sun: '...', name: '...' },
  // ...
];
```

### Game Objects
- `player`: position, velocity, health, weapon level, bomb count, invincibility timer
- `enemies`: array of active enemies; spawn controlled by difficulty
- `bullets`: projectiles from player and enemies
- `particles`: visual effects (explosions, impacts); affected by adaptive quality
- `boss`: null or current boss object (only one active)

### Storage Keys
```javascript
const STORAGE_BEST = 'skyfighter.bestScore';     // User's high score
const STORAGE_MUTE = 'skyfighter.muted';         // Audio mute state
const STORAGE_DIFF = 'skyfighter.difficulty';    // Last selected difficulty
const STORAGE_REDUCED = 'skyfighter.reducedMotion'; // Accessibility pref
```

## Making Changes

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (copy an existing type, adjust stats)
2. Ensure `minStage` is set (first stage where this enemy appears)
3. Spawn logic already iterates all types; no other changes needed

### Modifying Difficulty
Edit the corresponding property in `DIFFICULTIES`:
- `spawnBase`, `spawnMin`: Lower = more frequent spawns
- `enemySpeed`: Multiplier on enemy speed
- `enemyShootChance`: 0–1 probability per frame
- `lives`, `bombs`: Starting values
- `bossHpMul`: Multiplier on boss max health

### Changing Visual Theme
Edit `THEMES` array. Each theme has:
- `sky`: 3-color gradient (stops at 0%, 45%, 100% vertically)
- `cloud`: RGB string for cloud color
- `sun`: RGB string for sun glow color
- `name`: Display name (shown in stage banner)

### Adjusting Audio
Modify `beep()` and `noiseBurst()` parameters:
- `freq`: Oscillator frequency (Hz)
- `dur`: Duration (seconds)
- `vol`: Gain level
- `opts.sweepTo`: Frequency sweep target
- `opts.filterFreq`: Low-pass filter cutoff
- `opts.pan`: Stereo pan (-1 to 1)

### Enabling Particle Effects in Reduced-Motion Mode
The game respects `prefers-reduced-motion` media query and `reducedMotion` flag. To exclude a particle effect from reduction, check the flag before creating it or use CSS-only animations instead.

### Testing Touch/Mobile
- Use Chrome DevTools mobile device emulation (Device Toolbar)
- Test both portrait and landscape (canvas scales responsively)
- Test on actual mobile devices via HTTP server
- Verify joystick and button responsiveness

## File Structure

```
/home/user/Google_4/
├── index.html          # Single-file game (1,669 lines of HTML/CSS/JS)
├── README.md           # Project metadata
└── CLAUDE.md           # This file
```

## Performance & Optimization

### Adaptive Quality System
The game monitors frame times and automatically reduces visual complexity:
- `quality = 0`: Full detail (particles, smooth animations)
- `quality = 1`: Medium (fewer particles)
- `quality = 2`: Low (minimal particles, simplified effects)

When FPS drops below 40, quality decreases. When FPS exceeds 56, quality increases. Monitor `frameTimes` array (max 45 frames) for tuning thresholds.

### Canvas Rendering
- Full-screen canvas at device pixel ratio
- Clears and redraws every frame (no dirty-rect optimization)
- Particle rendering is quality-dependent
- Consider using `ctx.globalAlpha` for fade effects instead of per-pixel alpha

### Input Handling
- Mouse: No debouncing; raw `mousemove` and click events
- Keyboard: Key states tracked in object for frame-perfect input
- Touch: Custom joystick implementation with zone detection

## Deployment

The game is a single static HTML file. Deploy by:
1. Committing `index.html` to the repository
2. Hosting on any web server (GitHub Pages, Netlify, Firebase Hosting, etc.)
3. No build step or server-side logic required

## Testing Checklist

When making gameplay changes, verify:
- [ ] Game starts without errors (console clean)
- [ ] Difficulty selection and persistence work (localStorage)
- [ ] All enemy types spawn correctly for their `minStage`
- [ ] Bosses appear and defeat advances stage
- [ ] Weapon pickup increases level visibly
- [ ] Score multiplier resets on player hit
- [ ] Audio plays (or respects mute) without console errors
- [ ] Reduced-motion preference is respected (no shake/flash)
- [ ] Touch controls work on mobile (joystick, FIRE, BOMB buttons)
- [ ] Keyboard controls work on desktop (WASD, arrows, space, X, P)
- [ ] Game pauses and resumes correctly
- [ ] High score persists across browser sessions
