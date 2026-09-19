# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built entirely in a single HTML5 file. The game runs in any modern browser with no build process or external dependencies. It features:

- Two weapon systems (Vulcan rapid-fire and Laser piercing shots)
- Progressive difficulty across stages with boss encounters
- Combo/chain multiplier system for scoring
- Smart bomb power-ups for clearing enemies
- Multiple difficulty levels (Easy, Normal, Hard)
- WebAudio synth sound effects (no external audio files)
- Full mobile support with touch controls
- Accessibility features (reduced motion, screen reader announcements)
- Persistent high score and settings via localStorage

## Architecture

The entire game logic is contained in a single `<script>` block within `index.html`, organized as an Immediately Invoked Function Expression (IIFE) to encapsulate state. The codebase follows this high-level structure:

### Game Loop & Rendering (lines 1640-1665)
- `loop(now)` - Main game loop using `requestAnimationFrame`
- Calls `update(dt)` for logic when running
- Calls `draw()` for rendering
- Tracks deltaTime (`dt`) for frame-rate independent updates

### State Management (lines 863-911)
Game state is stored in variables at function scope:
- `player` - Player position, weapons, cooldowns, invunerability
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps` - Entity arrays
- `score`, `lives`, `bombCount`, `stage`, `multiplier` - Game metrics
- Difficulty settings loaded from localStorage on startup

### Update Logic (lines 1148-1302)
`update(dt)` handles:
1. Player movement (keyboard/joystick/mouse)
2. Firing and cooldown
3. Enemy spawning and behavior
4. Collision detection (enemies, bullets, power-ups, damage)
5. Score and chain multiplier updates
6. Boss behavior updates
7. Particle and projectile cleanup

### Rendering (lines 1520-1638)
`draw()` renders in this order:
1. Sky gradient with clouds and sun (parallax effect)
2. Player plane
3. Player bullets
4. Enemy bullets
5. Enemies
6. Boss
7. Power-ups
8. Particle effects

### Key Systems

#### Weapon System (lines 871-1146)
- **Vulcan** (default): Rapid spread shots that increase in spread at levels 2-3
- **Laser**: Piercing shots with fewer bullets initially, parallelizes at level 2
- Weapons switch automatically when collected at max level
- Level system: collect weapon power-ups to upgrade, switching at level 4

#### Enemy System (lines 1022-1041, 1210-1228)
Three enemy types with stage-gating:
- `grunt` - Basic red enemy (stage 1+)
- `interceptor` - Fast orange enemy with weaving pattern (stage 1+)
- `tank` - Slow purple tank with 4 HP (stage 2+)

Enemy spawning uses adaptive difficulty ramping based on frame count. Enemies use two shooting patterns:
- `spread` - Radial spread shot
- `aimed` - Aimed directly at player

#### Boss System (lines 1043-1056, 1372-1439)
Six unique bosses cycle per stage:
1. VANGUARD - Radial bursts and spread fans
2. BEHEMOTH - Heavy columns and homing spreads
3. SPECTER - Teleport dashes and spiral patterns
4. DREADNOUGHT - 8-way crosses and homing volleys
5. WRAITH - Teleport barrages and expanding rings
6. TITAN - Heavy columns with relentless homing

Each boss has custom colors and HP multipliers. Boss patterns are indexed and cycle through the stage.

#### Audio (lines 542-676)
WebAudio synth generates all sound via oscillators and noise:
- `beep()` - Tonal sounds (frequency sweep, filters, panning)
- `noiseBurst()` - Noise-based sounds (explosions, hits)
- `sfx` object contains named sound effects
- Audio context is lazily initialized on first user interaction
- Master gain and compressor for volume control

#### Scoring & Chain Multiplier (lines 1087-1101, 1304-1322)
- Each kill increments `chain`
- Multiplier = 1 + floor(chain / 5), capped at 10x
- Chain timer (2.2s) resets on missed shots or taking damage
- Chain milestone messages at 10, 20, 30+ kills

#### Accessibility (lines 468-540)
- Reduced motion preference detection from `prefers-reduced-motion` media query
- Screen reader announcements via `srLive` element
- Reduced animation durations when motion is reduced
- Motion toggle button for manual override

## How to Run & Test

1. **Direct file open**: Open `index.html` in any modern browser (no server needed)
2. **Local server** (optional): `python3 -m http.server` or similar
3. **Testing**:
   - Test each difficulty level (Easy, Normal, Hard)
   - Verify enemy spawning patterns and boss encounters
   - Check weapon pickups and level progression
   - Confirm bomb mechanics and screen shake
   - Test both keyboard (WASD/Arrows/Space/X) and mouse/touch controls
   - Verify audio works (or mute toggle if issues)
   - Check localStorage persistence of high score and settings

## Common Tasks

### Adding a new enemy type
1. Add entry to `ENEMY_TYPES` (line 502-506)
2. Set `minStage` to gate its appearance
3. Add explosion SFX variant in `EXPLODE_SFX` (line 678)
4. Ensure it appears in `spawnEnemy()` difficulty logic

### Modifying boss patterns
Edit `BOSS_PATTERNS` array (line 1372+). Each boss has an array of pattern functions. Pattern functions like `fireRadial()`, `fireAimedSpread()`, etc. are helpers starting at line 1324.

### Adjusting difficulty parameters
Modify `DIFFICULTIES` object (line 485-489):
- `spawnBase`/`spawnMin` - Enemy spawn rates
- `enemySpeed` - How fast enemies move
- `enemyShootChance` - Probability enemies fire
- `lives`/`bombs` - Starting quantities
- `bossHpMul` - Boss health scaling

### Tweaking visual effects or themes
- Visual themes in `THEMES` (line 493-500): sky gradients, cloud colors, sun colors
- Screen shake in `screenShake()` (line 727-730)
- Flash effects triggered by `flashHit()` / `flashBomb()` (line 698-709)
- Particle system in `spawnExplosion()` (line 1073-1085)

### Adding sound effects
1. Add function to `sfx` object using `beep()` or `noiseBurst()`
2. Call `ensureAudio()` before attempting audio
3. Respect `muted` flag to allow user mute control
4. Use `panFor()` helper for stereo panning based on world position

## Important Patterns & Gotchas

- **No external dependencies**: All rendering, audio, and logic is vanilla JavaScript. Keep it that way.
- **Single-file constraint**: Everything must stay in `index.html`. If refactoring becomes complex, consider module organization within the IIFE.
- **Frame-based rendering**: Some effects use `frame` counter (incremented each update). This couples some logic to frame rate.
- **LocalStorage keys**: Use `STORAGE_*` constants (line 463-466) to avoid conflicts. Keys are prefixed with `'skyfighter.'`
- **Canvas coordinate system**: Origin is top-left. Y increases downward. Planes face upward (negative Y direction).
- **Mobile-first input**: Priority is keyboard > joystick > mouse. Mouse doesn't interfere when keyboard is active.
- **Collision detection**: Uses simple rectangle overlap with `rectsOverlap()` (line 1069-1071). No physics engine.
- **Performance adaptation**: `quality` variable (0=full, 1=medium, 2=low) adjusts particle counts and shadow effects based on frame rate.
- **Accessibility**: Screen reader announcements via `announce()` helper. Respect `reducedMotion` flag for animations.

## File Structure

```
index.html
├── <head>
│   ├── Metadata (charset, viewport, theme color, favicon)
│   └── <style> - All CSS (typography, HUD, UI, canvas, animations)
└── <body>
    ├── Game canvas and overlays (HTML structure)
    └── <script> - Full game logic (all JavaScript)
```

No separate files. All assets are inline (SVG favicon, Web Audio).
