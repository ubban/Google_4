# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 canvas application. No build step, no dependencies—just open index.html in a browser and play.

## Running the Game

To test changes:
- **Local development**: Open `index.html` directly in a browser (double-click or `file://` URL)
- **Server (optional for CORS testing)**: `npx http-server` or `python3 -m http.server 8000`, then visit `http://localhost:8000/index.html`

The game runs at 480×720 canvas resolution, scaled to fit the viewport (max 1.6x). Tested on desktop (mouse/keyboard) and mobile (touch).

## Architecture

The entire game is in a single `<script>` block in index.html, organized in logical sections:

### Core Game Loop (line 1640–1652)
- `requestAnimationFrame(loop)` drives updates at browser refresh rate
- Calls `update(dt)` for game logic, then `draw()` for rendering
- Frame cap at 0.05s per frame to prevent runaway physics on slow refreshes

### Game State (line 863–900)
- `player`: Position, speed, weapon, cooldown, invuln frames
- `bullets`, `enemies`, `enemyBullets`, `powerUps`, `particles`: Pooled arrays
- `boss`: Null until boss fight; contains HP, attack pattern index, telegraph timer
- `score`, `lives`, `stage`, `kills`: Progression tracking
- `chain`, `chainTimer`, `multiplier`: Kill combo system

### Input Handling (line 767–861)
- **Keyboard**: Arrow keys / WASD for movement; Space for fire; X/Shift for bomb; P/Esc for pause
- **Mouse**: Hover to steer; click to fire (fires while button is held)
- **Touch**: Left side drag for joystick movement; FIRE/BOMB buttons on right
- Priority: Keyboard/joystick > mouse steering. Mouse doesn't interfere.

### Audio Synthesis (line 542–676)
- **No audio files**—all sounds generated with Web Audio API (oscillators, filters, noise buffers)
- `beep(freq, dur, type, vol, opts)`: Tone with optional sweep, filter, pan
- `noiseBurst(dur, vol, opts)`: Procedural noise for impacts
- `sfx` object maps game events to sound generators (shoot, explode, powerup, etc.)
- Muted state stored in localStorage; all synth functions check `muted` before playing

### Collision Detection (line 1069–1071)
- `rectsOverlap(a, b)`: AABB using half-widths: `|a.x - b.x| < (a.w + b.w)/2`
- Checked every frame: bullet-enemy, bullet-boss, enemy-player, enemyBullet-player, powerUp-player

### Enemy Spawning (line 1022–1041)
- Spawns on a timer that decreases per frame; restarts when it reaches 0
- Timer basecase scales with difficulty; ramps down over 300 frames to increase spawn rate mid-stage
- Enemy types unlocked by stage (grunts at stage 1, tanks at stage 2)
- Weaving interceptors use sine-wave pattern based on frame count

### Boss System (line 1043–1056, 1372–1439)
- Spawns after 18 kills per stage (`KILLS_PER_STAGE`)
- Boss designs cycle through 6 variants (VANGUARD, BEHEMOTH, etc.); each has unique attack patterns
- Each boss has 3 attack patterns; selects next pattern on attack timer expiry
- `telegraph > 0`: Boss flashes; at 0, pattern fires and cooldown resets
- Boss enters from top of screen; collision damage only after entry completes

### Difficulty System (line 485–489)
- Three modes: Easy, Normal (default), Hard
- Scales spawn rate, enemy speed, enemy shoot chance, lives, bombs, boss HP
- Stored in localStorage; buttons on pause/gameover screen

### Visual Effects
- **Screen shake** (line 726–730): Magnitude/duration; dampened if reduced-motion enabled
- **Flash** (line 698–709): White flash overlay on hit or bomb; CSS animation
- **Combo popup** (line 711–717): "N KILLS!" text with pulsing animation
- **Vignette** (line 75–87): Red pulse when at 1 HP
- **Theme rotation** (line 493–500): 6 sky gradients + cloud colors per stage

### Performance Adaptation (line 473–483)
- Tracks frame times over 45-frame window
- Adjusts `quality` level (0 = full glow, 1 = medium, 2 = low)
- `quality < 2` and avg > 25ms → quality++; `quality > 0` and avg < 17.8ms → quality--
- Disables shadow effects and reduces particle count at lower quality

### Weapon System (line 1124–1146)
- **Vulcan** (default): Spread shots. Level 1 = center; L2 = 2 shots; L3 = 3 shots
- **Laser**: Single lane. Level 1 = 1 shot; L2 = 2 lanes; L3 = 2 lane + center
- Weapon power-ups cycle: Vulcan L1→L2→L3→Laser L1→L2→L3→Vulcan L1, etc.
- Fire cooldown: 0.15s for Vulcan, 0.1s for Laser

### Event Listeners

| Element | Event | Handler |
|---------|-------|---------|
| Window | keydown/keyup | Input tracking; P/Esc pause; X/Shift bomb |
| Canvas | mousemove, mousedown, mouseleave | Mouse steering target, fire state |
| Difficulty buttons | click | Set difficulty (only if not running mid-stage) |
| Mute/Motion/Pause buttons | click | Toggle audio, motion effects, pause state |
| Fire/Bomb buttons (touch) | touchstart, touchend | Mobile control |
| Move zone (touch) | touchstart, touchmove, touchend | Joystick input |

### Data Structures

**Enemy** (lines 1032–1040):
```javascript
{ x, y, w, h, speed, hp, color, dark, type, canShoot, shootTimer, pattern, weave, weaveSeed, baseX, score }
```

**Boss** (lines 1047–1052):
```javascript
{ x, y, w, h, hp, maxHp, entering, telegraph, attackTimer, patternIndex, design, vx }
```

**Bullet** (varies):
- Player bullets: `{ x, y, w, h, speed, vx, pierce }`
- Enemy bullets: `{ x, y, w, h, speed, vx, radial, dx, dy }` (radial mode for boss attacks)

## Key Constants

- `BASE_W / BASE_H = 480 / 720`: Logical game size
- `KILLS_PER_STAGE = 18`: Kills needed before boss warning
- `DIFFICULTIES`: Spawn rates, enemy parameters, lives/bombs per mode
- `ENEMY_TYPES`: Properties (hp, speed, score) per enemy class
- `BOSSES`: 6 boss designs with color schemes and HP multipliers
- `THEMES`: 6 sky gradients, cloud colors for stage rotation
- `BOSS_PATTERNS`: Arrays of firing functions per boss design

## Common Editing Tasks

### Add a new enemy type
1. Add entry to `ENEMY_TYPES` (line 502–506) with w, h, hp, speedMul, score, minStage
2. Update spawn logic in `spawnEnemy()` to include it in the random choices
3. Add explosion sound to `EXPLODE_SFX` if needed (line 678)

### Tweak difficulty
Edit `DIFFICULTIES` object (line 485–489): spawn rates, enemy speed, shoot chance, lives, bombs, boss HP mult.

### Add or modify boss patterns
1. Add firing function to BOSS_PATTERNS section (e.g., `fireCircle`, `fireBounce`)
2. Insert into the appropriate boss's pattern array in `BOSS_PATTERNS` (line 1372–1409)
3. Boss patterns auto-rotate every `attackTimer` frames (line 1429)

### Change weapon behavior
- Spread/lanes: Edit `fireWeapon()` spread arrays (line 1127–1143)
- Fire rate: Adjust `fireDelay` values (line 1177)
- Cooldown post-fire: Edit `player.cooldown = fireDelay` (line 1180)

### Add visual effect
- Define CSS animation in `<style>` (line 10–358)
- Trigger via classList: `element.classList.add('animationName')`
- Example: Combo popup calls `showComboPop()` → adds `.show` class → triggers `@keyframes comboPulse`

### Debug/test
- Open browser DevTools (F12)
- Edit `DIFFICULTIES` in console to see instant effect
- Reduce `KILLS_PER_STAGE` to quickly spawn boss
- Set `player.level = 3` in console to test max weapon

## Browser Compatibility

- Requires: Canvas 2D, requestAnimationFrame, localStorage, Web Audio API
- Tested: Modern Chrome, Firefox, Safari, Edge; iOS Safari and Chrome; Android Chrome
- Mobile: Touch support auto-enabled if `ontouchstart` or `navigator.maxTouchPoints > 0` detected
- No polyfills needed for modern browsers; graceful audio fallback if WebAudio unavailable

## Storage (localStorage)
- `skyfighter.bestScore`: High score (persisted)
- `skyfighter.muted`: Audio state (1/0)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Motion effects state (1/0; respects OS preference)
