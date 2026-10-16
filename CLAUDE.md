# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game, implemented as a single-file HTML5 game. The entire game (HTML, CSS, and JavaScript) is contained in `index.html`.

## Architecture

The game uses a classic game loop pattern driven by `requestAnimationFrame`:

1. **Game State Management** (lines 863-869): Maintains player position/stats, enemy/bullet arrays, score, lives, bombs, current stage, boss state, and chain combo state
2. **Update Loop** (lines 1148-1302): Handles input, enemy spawning, collision detection, entity updates, and score/multiplier logic
3. **Rendering** (lines 1520-1638): Draws background with parallax clouds, entities, and HUD overlays
4. **Input Handling** (lines 768-862): Supports keyboard (arrow keys/WASD), mouse steering, and mobile touch controls with virtual joystick

### Key Game Systems

**Enemy System:**
- Three enemy types defined in `ENEMY_TYPES` (line 502-506): Grunt, Interceptor, Tank
- Each stage increases spawning difficulty and enemy variety
- Enemies follow predictable patterns; some weave or aim at the player
- Killing 18 enemies (`KILLS_PER_STAGE`) triggers boss spawn

**Boss System:**
- Six distinct boss designs in `BOSSES` (lines 508-515) with unique visual styles and HP multipliers
- Each boss has 3 attack patterns in `BOSS_PATTERNS` (lines 1372-1409)
- Attack patterns include radial bursts, aimed spreads, spirals, and homing projectiles
- Boss telegraph warning (22 frames) before each attack pattern fires

**Weapon System:**
- Two weapons: Vulcan (spread projectiles) and Laser (piercing beam)
- Three weapon levels (level 1-3 increases projectile count)
- Weapon progression tracked in `player.weapon` and `player.level`

**Scoring & Combo:**
- Kill chain: each enemy killed within 2.2 seconds increments the chain counter
- Multiplier: `1 + floor(chain / 5)`, capped at 10x
- Breaking the chain (time out or taking damage) resets multiplier to 1x
- Boss kill awards 500 base points

**Difficulty Settings** (line 485-489):
- Easy, Normal, Hard modifying: spawn rates, enemy speed/accuracy, lives, bombs, boss HP
- All three difficulties available at start screen
- Saved to localStorage

### Visual & Audio Design

**Themes** (lines 493-500):
- Six parallax sky backgrounds with color schemes (Daylight, Sunset, Night, Storm, Arctic, Volcanic)
- Theme cycles with each stage cleared

**Audio** (lines 542-676):
- Pure WebAudio synthesizer (no external audio files)
- Separate SFX functions for weapons, explosions, power-ups, boss actions, UI clicks
- Master gain with dynamics compressor (tracks lines 549-557)
- Supports stereo panning based on projectile/explosion X position

**Performance Tuning** (lines 473-483):
- Adaptive quality based on frame time: 0 (full), 1 (medium, 60% particles), 2 (low, 35% particles)
- Quality downsteps when avg frame time > 1/40s; upsteps when < 1/56s
- Affects particle count and glow effects (shadow blur)

### Input & Controls

**Keyboard:** Arrow keys or WASD for movement, Space to fire, X/Shift for bomb, P/Esc to pause  
**Mouse:** Move to steer, click to fire  
**Touch:** Drag left half to move (with visual joystick), tap buttons on right side to fire/bomb  
**Pause:** P key, Esc key, or pause button → pauses and shows overlay  

### Game Loop Flow

1. `resetGame()`: Initialize player, clear entities, read difficulty settings
2. `loop(now)`: Main frame loop
   - If `running && !paused`: call `update(dt)` then `draw()`
   - Otherwise: only call `draw()`
3. `update(dt)`: 
   - Read input, move player
   - Update bullets (move, filter out-of-bounds)
   - Spawn enemies on timer (decelerating difficulty ramp)
   - Update enemies (move, shoot)
   - Check boss spawn condition (18 kills)
   - Collision detection: bullets vs enemies/boss, enemy bullets vs player, enemies vs player, power-ups vs player
   - Apply score/multiplier on kills
4. `draw()`: 
   - Apply screen shake (if not reduced-motion)
   - Draw sky gradient + sun glow + parallax clouds
   - Draw player plane, bullets, enemies, boss, power-ups, particles
   - HUD elements managed separately (DOM, not canvas)

## Development Commands

Since this is a static HTML file with no build step:

- **Run locally**: Open `index.html` in a browser (e.g., `python -m http.server` and navigate to `localhost:8000`)
- **Test on mobile**: Host locally, access via phone on same network (use phone IP address)
- **No linting/testing infrastructure**: Code style and correctness depend on review only

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502) with stats: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Update `spawnEnemy()` (line 1022) to select new type in random pool with probability
3. Add explosion SFX to `EXPLODE_SFX` mapping (line 678) if needed
4. Add draw logic in `draw()` function if the plane sprite needs custom rendering

### Adding a New Boss Pattern
1. Create pattern function in `BOSS_PATTERNS` (line 1372), e.g., `b => fireRadial(b, 14, 150)`
2. Pattern receives boss object `b` and should call enemy bullet helper: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`
3. Patterns automatically cycle through the array per boss based on `b.patternIndex`

### Adding a New Difficulty
1. Add entry to `DIFFICULTIES` object (line 485) with spawn/enemy/life/bomb/boss HP multiplier values
2. Difficulty buttons auto-populate from CSS class `.diffBtn` with `data-diff` attribute (line 412-414)
3. Difficulty reads/writes to localStorage via `STORAGE_DIFF` key

### Adjusting Screen Shake & Particle Effects
- `screenShake(mag, dur)` (line 727): Sets `shakeMag` (amount) and `shakeTime` (duration). Reduced by 85% if reduced motion enabled.
- `spawnExplosion(x, y, color, count)` (line 1073): Spawns `count` particles (scaled by quality). Called on enemy/boss kill and bomb use.

### Audio & Muting
- Sound effects are synthesized in `sfx` object (line 624)
- Each SFX uses `beep()` (sine/square/sawtooth) or `noiseBurst()` with optional filters and panning
- Master volume controlled by `masterGain` (line 555)
- Mute state reads/writes to localStorage via `STORAGE_MUTE` key

## Code Style Notes

- Single self-executing function wraps all code (line 423) to avoid global namespace pollution
- Inline CSS and no external assets (fully self-contained)
- Accessibility: screen reader live region (`srLive`), reduced-motion preference detection, semantic button roles
- Canvas coordinates: origin at top-left; Y increases downward
- All numeric constants (speeds, sizes, timers) hardcoded; consider exposing difficult-to-tune ones as parameters

## State Persistence (localStorage)

- `skyfighter.bestScore`: Best score achieved
- `skyfighter.muted`: Mute state (1 or 0)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Reduced motion preference (1 or 0)

## Known Limitations

- Single-file format makes large refactors difficult; split into modules if game grows significantly
- No external asset pipeline; all visuals must be canvas-drawn or data-URIs
- Fixed canvas size (480×720 internal), scaled on window resize; mobile aspect ratio varies
