# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a vertical shoot-'em-up game (Raiden-style) implemented as a single self-contained HTML file. It runs entirely in the browser with no external dependencies:
- **2D Canvas rendering** for graphics
- **WebAudio synthesis** for sound effects (procedurally generated, no audio files)
- **Touch + keyboard + mouse** input support
- **LocalStorage** for persistence (scores, settings)

## Running the Game

Simply open `index.html` in a web browser. The game runs immediately with no build step required.

For local development, use a simple HTTP server to avoid CORS issues:
```bash
python3 -m http.server 8000
# or: npx http-server
```

Then navigate to `http://localhost:8000` in your browser.

## Architecture Overview

The entire game is a single `<script>` IIFE (Immediately Invoked Function Expression) that:
1. Initializes Canvas and DOM elements (lines 422–461)
2. Sets up game constants and system initialization
3. Implements the main game loop with `requestAnimationFrame`
4. Manages all game state in closure variables

### Key State Variables

- **Player**: Position, weapon (vulcan/laser), level, health, invulnerability timer
- **Game flow**: `running`, `paused`, `gameOver`, `stage`, `kills`
- **Score system**: `score`, `chain`, `multiplier`, `bestScore` (localStorage)
- **Entities**: `enemies`, `bullets`, `enemyBullets`, `powerUps`, `boss`, `particles`

### Main Systems

**1. Input Handling (lines 767–861)**
- Keyboard: arrow keys, WASD for movement; Space/click to fire; X/Shift/P to bomb/pause
- Touch: left zone for joystick movement, right zone for fire/bomb buttons
- Mouse: steering via pointer position, click to fire
- All input sources coexist—keyboard/joystick take priority over mouse

**2. Game Loop (lines 1640–1665)**
- Calls `update(dt)` for logic (line 1648)
- Calls `draw()` for rendering (line 1650)
- Adaptive performance tracking (line 1646) reduces quality at 40 FPS or lower
- Screen shake decays each frame (line 1649)

**3. Update Logic (lines 1148–1302)**
- Player movement and firing
- Enemy spawning and behavior (weaving, shooting patterns)
- Bullet/enemy collision detection
- Power-up collection
- Boss state machine
- Particle and projectile decay

**4. Rendering (lines 1520–1637)**
- Sky gradient with parallax clouds
- Player ship with weapon aura (laser mode)
- Enemies with health bars (tanks)
- Boss with core glow and telegraph state
- Bullets with glow (quality-dependent)
- Power-ups with rotation
- Screen shake offset applied to all transforms
- Reduced motion support (CSS class + parameter checks)

**5. Audio System (lines 542–676)**
- Single `AudioContext` (deferred until first input via `ensureAudio()`)
- SFX created via synthesized beep/noise functions, no audio files
- `beep()` creates tones with optional frequency sweep, filter, panning
- `noiseBurst()` creates filtered white noise for explosions
- Mute toggle stored in localStorage

**6. Boss System (lines 1043–1409)**
- Six unique boss types with different attack patterns
- Boss lifecycle: entering → combat → defeated → stage progression
- Each boss has 3 attack patterns, selected in sequence
- Patterns include: radial bursts, aimed spreads, homing missiles, teleport attacks
- Telegraph phase (visual warning before attack) at line 1421
- Boss health bar updates in real-time (line 1300)

**7. Weapon System (lines 1124–1146)**
- **Vulcan**: Spread fire (3 lanes at level 3), medium rate
- **Laser**: Piercing beam (up to 3 lanes at level 3), faster rate
- Weapon upgrade pickups cycle: level 1→2→3, then swap to other weapon at level 1
- Laser has visual aura around player

**8. Difficulty Scaling (lines 485–489)**
- Easy/Normal/Hard presets affecting spawn rates, enemy speed, shoot chance, lives, bomb count, boss HP
- Adaptive spawn rate (line 1201) increases difficulty over time via `difficultyRamp`
- Player can change difficulty before starting (line 748)

**9. Scoring & Combo System (lines 1087–1101)**
- Kill combo tracked in `chain` variable
- Multiplier (1–10x) based on chain length
- Chain timer resets if no kills for 2.2 seconds
- Boss kill gives flat 500 points

**10. Particle System (lines 1073–1085, 1296–1297)**
- Explosions spawn particles with random velocity, gravity disabled
- Life-based fade (alpha interpolation on line 1629)
- Quality setting scales particle count (line 1074)

## Important Patterns

**Collision Detection**: `rectsOverlap(a, b)` (line 1069) uses AABB with half-widths. No advanced physics.

**Performance Scaling**: `quality` variable (0=full, 1=medium, 2=low) based on frame time. Effects adjusted:
- Particle counts reduced
- Shadow/glow effects disabled at lower quality
- Boss telegraph glow reduced

**Persistence**: Three localStorage keys:
- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

**Screen Shake**: Simple translation offset applied in `draw()` if `shakeTime > 0`. Magnitude decays linearly.

**Reduced Motion**: Respects browser `prefers-reduced-motion` media query (line 520). All animations/shakes/flashes are conditional on `!reducedMotion`.

**Canvas Scaling**: Base resolution is 480×720 (line 753). Scales to fit window while maintaining aspect ratio and max 1.6× size (line 757).

## Common Edits

**To add a new enemy type:**
1. Add entry to `ENEMY_TYPES` object (line 502)
2. Add key to choices filter in `spawnEnemy()` (line 1026)
3. Add explosion SFX mapping in `EXPLODE_SFX` (line 678) if custom sound

**To add a new boss pattern:**
1. Create pattern function using helper functions (`fireRadial`, `fireAimedSpread`, etc.)
2. Add 3 patterns to a new array in `BOSS_PATTERNS` (line 1372)
3. Add boss design to `BOSSES` array (line 508) with unique colors

**To add a new theme:**
Add entry to `THEMES` array (line 493) with sky gradient stops, cloud/sun colors, and name. Themes cycle by stage.

**To tune difficulty:**
Adjust `DIFFICULTIES` object (line 485) or modify the `difficultyRamp` calculation in the spawn timer logic (line 1201).

**To add a new power-up type:**
1. Add spawn condition in `maybeSpawnPowerUp()` (line 1058)
2. Add visual rendering in `draw()` around line 1617
3. Add effect in `applyPowerUp()` (line 1304)

## Gotchas

- **Canvas coordinates**: Origin (0,0) is top-left. Y increases downward. All drawing uses `ctx.translate()` + local coordinates.
- **Frame-based vs time-based**: Most game logic uses `dt` (delta time in seconds), but enemy spawning uses frame count (`spawnTimer`, `frame`). Mix carefully.
- **Closure state**: All game variables are in the IIFE closure—no global state. This is intentional for isolation.
- **Audio context delay**: Audio Context requires user interaction to start (line 774). First input calls `ensureAudio()`.
- **TouchEvent handling**: `preventDefault()` on touch events (lines 816, 821, 831, 834) to prevent default scrolling.
- **Radial bullets**: Enemy bullets from boss attacks use `radial: true` flag and `dx/dy` velocity (not `vx/speed` like player bullets).
- **Boss telegraph timing**: The telegraph phase lasts exactly 22 frames (line 1421). Audio plays at start, attack fires at end.
- **Invulnerability flashing**: Player blinks at invuln (line 1546–1548) via modulo frame count, but respects reduced motion.

## Testing Suggestions

- **Mobile:** Test touch joystick and button layout on small screens. Check safe-area insets for notches.
- **Performance:** Open DevTools, throttle CPU to low-end device, verify quality scaling kicks in.
- **Accessibility:** Enable reduced-motion in OS settings, verify no animations play and sound plays instead.
- **Boss patterns:** Play through all 6 bosses (stage 1–6), verify attack patterns match implementation, no visual glitches.
- **Edge cases:** Bomb during boss telegraph, pause mid-attack, rapid weapon pickup spam.
