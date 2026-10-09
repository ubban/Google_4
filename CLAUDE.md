# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single HTML5 application. The entire game (styles, logic, and rendering) is contained in `index.html` (~1670 lines).

### Key Characteristics
- Pure HTML5 Canvas rendering (no external graphics libraries)
- Synthesized WebAudio sounds (no external audio files)
- Responsive touch + keyboard + mouse controls
- Adaptive performance scaling (quality levels 0-2)
- 6 randomly-selected sky themes, 6 boss designs
- Progressive difficulty through stages (no hard stage limit)
- Accessibility: reduced motion toggle, screen reader announcements

## Development

### Building and Running
The game runs immediately by opening `index.html` in a browser. No build process, dependencies, or transpilation required.

**Testing locally:** Open `index.html` in any modern browser that supports:
- HTML5 Canvas API
- Web Audio API (for sound)
- ES6+ JavaScript (arrow functions, `const`/`let`, template literals)
- Touch Events API

### Code Editing
All code lives in `index.html`. The structure is:
1. **HTML structure** (lines ~360–410): Canvas, UI overlays, touch control zones
2. **Embedded CSS** (lines ~10–359): Responsive design with dark theme
3. **Game script** (lines 422–end): All game logic in a single IIFE (Immediately Invoked Function Expression)

### Linting and Validation
No linters or formatters are configured. The codebase uses:
- `'use strict'` mode
- Single quotes for strings (not enforced, but consistent)
- Consistent 2-space indentation

## Game Architecture

### Core Game Loop
- **Main loop:** `update()` (dt in seconds) → `draw()` → requestAnimationFrame
- **Frame rate:** Target 60 FPS; adaptive quality degrades rendering if frame times exceed 25ms average
- **Time tracking:** `lastTime` stores previous frame timestamp for delta calculation

### Key State Objects

**Player**
```javascript
player = { 
  x, y,           // position (canvas coords)
  w, h,           // width/height (40x40)
  speed: 260,     // pixels/second
  cooldown,       // weapon fire cooldown timer
  rapid,          // rapid-fire timer (weapon level bonus)
  weapon,         // 'vulcan' or 'laser'
  level,          // 1–5 (unlocks weapon upgrades)
  invuln          // invulnerability duration after hit
}
```

**Entities:** `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss` — all are arrays except `boss` (single object or null)

**Game state:**
- `score`, `lives`, `multiplier`, `chain` (combo counter for scoring)
- `stage`, `stageKills` (kills remaining to trigger boss)
- `difficulty` ('easy', 'normal', 'hard') — from localStorage
- `running`, `paused`, `gameOver` — state flags

### Entity Lifecycle
1. **Spawn:** Added to array (e.g., `enemies.push(...)`)
2. **Update:** Position, timers, collision checks each frame
3. **Draw:** Rendered on canvas
4. **Death:** Removed from array; may spawn particles, effects, or drop power-ups

### Collision Detection
- **Player vs. enemy bullets:** `checkCircleCollision(player, enemyBullet)` — circles around x,y with radius `w/2`
- **Player shots vs. enemies:** `checkRectCollision(bullet, enemy)` — bullet is rect, enemy is rect
- **Boss HP:** Decremented directly by bullet hits; kill when `boss.hp <= 0`

### Weapon System
- **Vulcan:** Fast spread shots, 5 shots per volley, upgrades reduce cooldown
- **Laser:** Slower, higher damage, piercing beam with area-of-effect indicator
- **Power-ups:** Drop from killed enemies; cycle weapon type or boost level (max 5)
- **Weapon fire:** Triggered by keyboard (spacebar/mouse), touch (fireBtn), or auto-held (mouse while moving)

### Difficulty Levels
Defined in `DIFFICULTIES` object; affects spawn rate, enemy speed, shoot chance, boss HP scaling:
```javascript
easy:   spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8
normal: spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0
hard:   spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3
```

### Stage / Boss System
- **Progression:** Kill 18 enemies (`KILLS_PER_STAGE`) per stage → triggers boss fight
- **Boss entrance:** Boss spawns at top, moves to y=120, then patrols horizontally
- **Boss attacks:** Telegraph → execute attack pattern (2–3s cycle); 6 boss designs, each with unique pattern set
- **Stage themes:** 6 sky gradients cycle (`THEMES` array); matched to stage modulo 6

### Scoring & Multipliers
- **Base enemy score:** Varies by type (grunt: 10, interceptor: 15, tank: 35)
- **Combo chain:** Hit without missing (time window); increases multiplier by 0.1 per hit, resets on miss
- **Chain timeout:** 5 seconds; breaks if player takes damage
- **Boss kill bonus:** Flat +500 points

### Sound (WebAudio Synthesis)
- **No external files:** All sounds generated by oscillators, filters, and envelope curves
- **Key sounds:** gunfire, laser, enemy hit, bomb, screen shake pulse, boss telegraph, stage clear, game over
- **Muted state:** Stored in localStorage; respects Web Audio suspended state on user interaction

### Visual Effects
- **Screen shake:** `screenShake(magnitude, duration)` — applies random translation to canvas each frame
- **Flash vignette:** Red pulse on critical health
- **Combo popup:** Animated text ("x18 COMBO") at screen center
- **Stage banner:** Large "STAGE N" announcement with fade-in/out
- **Particle explosions:** Colored dots scattered on kill; velocity-based physics
- **Blinking sprite:** Player blinks during invulnerability (every 4 frames)

### Input Handling

**Keyboard:**
- Arrow keys / WASD: Move player
- Spacebar / Mouse down: Fire weapon
- P or Pause btn: Toggle pause
- Mute / Motion buttons: Toggle settings

**Touch:**
- Left zone (55% width): Movement joystick (appears on first touch)
- Right zone (45% width): Fire button (held) and Bomb button
- Full screen support with safe-area insets for notched devices

**Mouse:**
- Move cursor to steer player (glides toward pointer)
- Click and drag to fire
- Keyboard/joystick input overrides mouse

### Local Storage Keys
```javascript
'skyfighter.bestScore'   // High score (number)
'skyfighter.muted'       // Mute state (0|1)
'skyfighter.difficulty'  // Difficulty ('easy'|'normal'|'hard')
'skyfighter.reducedMotion' // Reduced motion toggle (0|1)
```

## Common Edits and Entry Points

### Adding an Enemy Type
1. Extend `ENEMY_TYPES` object with new properties (w, h, hp, speedMul, color, score, minStage, weave)
2. Update `spawnEnemies()` to include new type in pool; respect `minStage` constraint
3. Adjust `KILLS_PER_STAGE` or difficulty if spawn rates change significantly

### Adding a Boss Pattern
1. Define pattern function in `BOSS_PATTERNS` array (index maps to boss in `BOSSES`)
2. Function receives boss object; call `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.
3. Boss cycles patterns; add telegraph delay between attacks (currently ~20 frames)

### Modifying Difficulty
Edit `DIFFICULTIES` object; all scaling is multiplicative or directly mapped.

### Changing Themes
Add entries to `THEMES` array: `{ sky: [top, mid, bottom], cloud: "R,G,B", sun: "R,G,B", name: "..." }`

### Adjusting Scoring
- Enemy base scores: `ENEMY_TYPES[type].score`
- Multiplier gain per chain hit: `multiplier += 0.1` (in `addScore()`)
- Chain timeout: `5.0` (in `chainTimer` max)
- Boss bonus: `500` (in `killBoss()`)

## Important Implementation Notes

### Performance Scaling
The `quality` variable (0=full, 1=medium, 2=low) tracks adaptive performance:
- Quality 0: Full glow effects, shadow blur, smooth particle rendering
- Quality 1–2: Reduced shadow blur, no glow on bullets, fewer particle updates

Set by `trackPerf()` based on 45-frame rolling average; adjust thresholds if frame times spike.

### Canvas Scaling
Canvas size matches viewport but uses DPI-aware scaling via `devicePixelRatio` (see top of script). Resize events are not handled; full-window canvas refresh is done on load.

### Browser Compatibility
Uses standard APIs (Canvas 2D, Web Audio, Touch Events). Tested on modern Chrome, Firefox, Safari, Edge. Older IE or no-WebAudio browsers will fail silently (game won't run, no error thrown — handle gracefully).

### Entity Pooling
No object pooling is implemented. Entities are created on-demand and garbage-collected when removed. For large entity counts (>500 simultaneous), consider implementing pool reuse.

## Testing Checklist (Manual)

- [ ] Game loads and runs in browser
- [ ] All three difficulties selectable and spawn rates noticeably different
- [ ] Player movement smooth and responsive (keyboard + touch)
- [ ] Weapons fire correctly and collisions register
- [ ] Boss spawns after 18 kills and uses correct patterns
- [ ] Score, multiplier, and stage progression update correctly
- [ ] Bomb explosion clears screen and resets combo properly
- [ ] Sound effects play (or mute toggle silences them)
- [ ] Reduced motion toggle removes animations
- [ ] Game over screen shows final score and high score
- [ ] High score persists across page reloads
- [ ] Screen reader announcements (game state changes) work on pause/stage/game-over
- [ ] Touch controls visible and responsive on mobile devices
- [ ] Mouse cursor acts as targeting reticle while hovering canvas
