# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML/Canvas application. The entire game (HTML, CSS, JavaScript) is in `index.html`.

### Features
- **Player Aircraft**: Controllable with keyboard, mouse, or touch (mobile joystick)
- **Weapon System**: Two weapons (Vulcan spreads, Laser penetrating) with 3 upgrade levels each
- **Enemy Types**: Grunts, Interceptors, Tanks with distinct visual/behavioral patterns
- **Boss Battles**: 6 unique boss designs with complex attack patterns per stage
- **Power-ups**: Weapon upgrades, extra lives, bombs
- **Difficulty Levels**: Easy/Normal/Hard with tuned spawn rates and enemy parameters
- **Stage System**: Progressive stages with escalating difficulty; boss every 18 enemy kills
- **Audio**: Fully synthesized WebAudio effects, no external sound files
- **Theme System**: 6 dynamic sky themes, each with unique color palettes

## Architecture

The game operates as a single `requestAnimationFrame` loop:

```
Input Processing
    ↓
Game State Update (if running, not paused)
    ├─ Player movement & weapon firing
    ├─ Enemy spawning & behavior
    ├─ Bullet movement (player & enemy)
    ├─ Collision detection (entities vs bullets, vs player, vs boss)
    ├─ Power-up collection
    ├─ Particle effects
    └─ Boss attack patterns (if boss active)
    ↓
Canvas Rendering
    ├─ Sky gradient & sun glow
    ├─ Cloud parallax layers
    ├─ All entities (player, enemies, boss, bullets, particles, power-ups)
    └─ HUD overlays (score, weapon, health, bomb count)
```

### Core Data Structures

**Player Object**
- `x, y`: Position on canvas
- `w, h`: Collision bounds
- `weapon`: 'vulcan' or 'laser'
- `level`: 1-3 (affects bullet spread/pierce)
- `cooldown`, `rapid`: Fire rate control
- `invuln`: Invincibility timer after being hit

**Enemy Object**
- `type`: 'grunt' | 'interceptor' | 'tank' (each has different HP, speed, scoring)
- `x, y, w, h`: Position & bounds
- `speed`: Vertical movement rate
- `pattern`: 'spread' (multi-angle burst) or 'aimed' (seeks player)
- `canShoot`: Boolean (some enemies don't fire)
- `weave`: Boolean & `weaveSeed` (sinusoidal horizontal motion)
- `hp`: Health (1 for most, 4 for tanks)

**Boss Object**
- `design`: Reference to BOSSES array element (name, colors, hpMul)
- `x, y, w, h`: Position & bounds
- `hp, maxHp`: Health tracking
- `entering`: Animation state (scrolls up onto screen)
- `telegraph`: Counter for warning before attack (22 frames)
- `patternIndex`: Cycles through BOSS_PATTERNS
- `vx`: Horizontal velocity for left-right pacing

**Bullet Types**
- **Player bullets**: `{x, y, w, h, speed, vx, pierce, laser}`
  - Non-laser bullets go straight up; laser can spread horizontally
  - `pierce` > 0 lets lasers pass through multiple enemies
- **Enemy bullets**: `{x, y, w, h, speed, vx}` or `{..., radial: true, dx, dy}`
  - Radial bullets use `dx/dy` velocity instead of `speed/vx`

**Particle Object**
- `{x, y, vx, vy, life, maxLife, color}`
- Simple point particles that fade with life

## Input System

The game handles three input methods, with keyboard/joystick taking priority over mouse:

1. **Keyboard**: Arrow keys or WASD to move, Space/X/Shift to fire/bomb, P/Esc to pause
2. **Mouse**: Pointer position steers the plane; click/drag to fire (desktop)
3. **Touch** (mobile): Left side is joystick movement zone; right side has FIRE & BOMB buttons

The `mouseActive` flag tracks whether keyboard is in use — keyboard always wins input contention, preventing the mouse cursor from interfering.

## Game State

**Running States**
- `running`: Boolean (false = game over or menu)
- `paused`: Boolean (overlay shown, no update/draw)
- `gameOver`: Boolean (final stats shown)

**Score & Progression**
- `score`: Current run's score
- `bestScore`: Persisted in localStorage (key: `skyfighter.bestScore`)
- `stage`: Current stage (1+); stages cycle through THEMES and BOSS_PATTERNS
- `kills`: Total kills this run
- `stageKills`: Kills toward next boss (resets at stage transition)
- `chain`: Consecutive kills without missing (decays if no kill for 2.2 seconds)
- `multiplier`: Score multiplier based on chain (1 + floor(chain/5), capped at 10)

**Difficulty Settings**
- Stored in localStorage; applied via `DIFFICULTIES` object with keys: `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`

**Audio & Visual**
- `muted`: Boolean; stored in localStorage
- `reducedMotion`: Boolean; respects prefers-reduced-motion media query, stored in localStorage
- `quality`: Adaptive performance level (0=full, 1=medium, 2=low); affects particle count, shadow blur, etc.

## Key Systems & Functions

### Enemy Spawning
- Enemies spawn above the screen, move downward
- Spawn rate decreases as time progresses (difficulty ramp every 300 frames)
- Tank enemies only appear from stage 2+
- After 18 kills, boss is queued to spawn ~1.2 seconds later

### Collision Detection
- `rectsOverlap(a, b)`: Rectangle center-distance check; used for all collision tests
- Player bullets check vs enemies (damage/destroy)
- Player bullets check vs boss (special damage 3 per bullet)
- Enemy bullets check vs player (instant hit)
- Enemies check vs player (instant hit, removes enemy)
- Boss check vs player (instant hit)
- Power-ups check vs player (instant pickup)

### Weapon System
- **Vulcan**: 1-3 bullets per shot depending on level; fixed upward trajectory
- **Laser**: Level determines lane count & pierce (can pass through 1-2 enemies)
- Fire delay: Laser 0.1s, Vulcan 0.15s
- Weapon switches unlock after level 3, resetting to level 1 on switch

### Boss Patterns
- Each boss has 3 attack patterns (functions in BOSS_PATTERNS array)
- Pattern cycles: telegraph (22 frames), attack fires, next cycle queues after 85 frames
- Patterns include: radial bursts, aimed spreads, columns, homing, teleport-dash, double rings

### Sound System
- `ensureAudio()`: Lazy-initialize AudioContext (required for user gesture)
- `beep(freq, dur, type, vol, opts)`: Tone synthesis with optional frequency sweep, filtering, panning
- `noiseBurst(dur, vol, opts)`: White noise with envelope & filtering
- All sounds are generated in real-time; no external assets

### Canvas Rendering
- Base resolution: 480×720 (maintains aspect, scales to fit window up to 1.6× max)
- Shake effect: Random offset each frame if `shakeTime > 0`
- Theme system: Sky gradients, sun glow, cloud layers (parallax at different speeds)
- Glow effects: Shadow blur on bullets/particles when quality = 0

### Persistence & Settings
- localStorage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- Settings are synced to UI buttons and applied on startup

## Development Workflow

### Running the Game
Since this is a single HTML file, just open it in a web browser:
```bash
# Start a simple HTTP server
python -m http.server 8000
# Then visit http://localhost:8000/index.html
```

Or use the `run` skill from Claude Code to start a local server automatically.

### Testing & Debugging

**Testing game mechanics:**
- Open DevTools (F12), keep the console visible while playing
- Audio is initialized on first user gesture; check console for errors if silent
- `localStorage` persists between sessions; clear it to reset high score/settings

**Debugging patterns:**
- Modify `DIFFICULTIES`, `ENEMY_TYPES`, or `BOSSES` constants to test balance
- Add temporary logging in `update()` or `draw()` functions to track state
- Adjust `quality` variable (0/1/2) to test performance scaling
- Test `reducedMotion` mode to verify animations disable correctly

**Common edits:**
- Difficulty tuning: Edit `DIFFICULTIES[difficulty]` values
- Enemy behavior: Change `spawnEnemy()`, enemy speed multipliers, or shoot chance
- Boss patterns: Add new functions to `BOSS_PATTERNS` array
- Visual effects: Tweak colors, particle counts, shadow blur in `draw()` and explosion spawns

### Performance Considerations
- **Particle count scales with quality level**: Full quality generates more particles; on slower devices, quality drops to reduce overhead
- **Shadow blur disabled on low quality**: Expensive on some browsers
- **Frame time is capped at 50ms**: `dt = Math.min(dt, 0.05)` prevents frame skips from breaking physics
- **Enemy & bullet limits**: No explicit cap, but typical runs have 20-30 enemies and 50-100 bullets on screen

## Modding & Extension Ideas

**Easy modifications:**
- Add new themes: Push to `THEMES` array with sky colors, cloud color, sun color, and name
- Add new boss designs: Push to `BOSSES` array with hull/wing/core colors and hpMul multiplier
- Tweak difficulty: Edit `DIFFICULTIES` object values
- Change start lives/bombs: Edit `DIFFICULTIES[difficulty].lives` and `.bombs`

**Moderate modifications:**
- New weapon type: Add key to `ENEMY_TYPES` or duplicate an existing type with new parameters
- New enemy behavior: Create a new `spawnEnemy()` variant or add weave patterns
- New boss attack pattern: Create a new function (e.g., `fireWave()`) and push to a boss's pattern array

**Complex modifications:**
- New screen state (e.g., shop, level select): Add state flags, rework the overlay display logic
- Multiplayer: Requires network sync; would need major refactoring
- Mobile app (Electron/React Native): Extract game loop into a module, port input handling

## File Structure

This project is a single file:
- **index.html**: All HTML markup, CSS styling, and JavaScript game code
  - Lines 1-361: HTML & CSS (structure, styling, animations)
  - Lines 422-1667: JavaScript (constants, state, functions, main loop)

## Useful Constants & Defaults

- `BASE_W: 480, BASE_H: 720`: Canvas resolution (fixed, scales with CSS)
- `KILLS_PER_STAGE: 18`: Enemy kills before boss spawns
- `BOSS_PATTERNS`: 6 arrays, each with 3 attack patterns
- `THEMES`: 6 environment themes, cycling per stage
- `STORAGE_*`: localStorage keys for persistence (best score, muted, difficulty, reduced motion)

---

### Tips for Efficient Development

1. **Use the difficulty/theme constants**: Most game balance lives in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES`
2. **Test on both desktop & mobile**: Touch controls are separate; don't assume keyboard parity
3. **Audio requires user gesture**: Always call `ensureAudio()` after input events
4. **Keep the main loop tight**: Performance scales down gracefully; don't add unbounded spawning
5. **Collision detection is O(n²) worst-case**: Typical counts (30 enemies, 100 bullets) are fine; watch for explosions at high stage counts
