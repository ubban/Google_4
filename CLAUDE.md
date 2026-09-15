# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The game runs entirely in the browser with vanilla HTML, CSS, and JavaScript — no build process, no dependencies, no external assets. Open `index.html` in a browser to play.

## Architecture

The codebase is monolithic in a single file but organized into logical sections:

### Game Loop
- `requestAnimationFrame(loop)` drives all updates and rendering at 60 FPS
- `update(dt)` handles input, state changes, collision detection
- `draw()` renders the scene using Canvas 2D
- `trackPerf(dt)` measures frame times and adapts rendering quality down on slower devices

### Core Systems

**Input & Control**: Handles keyboard (WASD/Arrows for movement, Space/Shift for fire/bomb), mouse (steering), touch (joystick on bottom-left, action buttons on bottom-right). Mouse input uses a smooth easing system where the plane glides toward the pointer.

**Difficulty & Settings**: Four persistent settings stored in localStorage:
- `skyfighter.difficulty` (easy/normal/hard) affects spawn rates, enemy speed, player lives, bomb count, boss HP
- `skyfighter.muted` (audio on/off)
- `skyfighter.reducedMotion` (disables screen shake, flash effects, animations)
- `skyfighter.bestScore` (high score tracking)

**Audio**: Completely synthesized using WebAudio API (no external files). The `sfx` object contains 11 different sound effects built from oscillators, filters, and noise bursts. Spatial panning is computed from world position.

**Game State Machine**: Controlled by `running`, `paused`, and `gameOver` flags. Overlay UI shows at start, pause, and game-over; hidden during gameplay.

**Entity Management**: Arrays for `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`. Collision detection uses axis-aligned AABB (`rectsOverlap`). Entities self-destruct by falling off-screen or dying.

### Progression & Difficulty Tuning

**Stage System**: Each stage spawns 18 enemies (`KILLS_PER_STAGE`), then a boss. Defeating the boss advances to the next stage. Themes cycle through 6 sky colors/cloud sets. Bosses cycle through 6 designs with unique attack patterns.

**Difficulty Scaling**: Enemy spawn rate decreases (harder to hit enemies, tighter windows) and their speed/fire rate increase as you stay alive. Boss HP scales by stage and difficulty multiplier.

**Scoring**: Base points per enemy type. Kill chains reset after 2.2 seconds of no kills; multiplier grows every 5 kills (1x to 10x). Bosses award 500 points flat.

**Weapon Progression**: Three weapon levels via pickups. Vulcan (spread shots) vs. Laser (piercing lines) can switch at level 3. Higher levels unlock wider spreads or more laser lanes.

### Rendering & Performance

**Canvas-Based Drawing**: All graphics hand-drawn with paths and fills—no image assets. Player, enemies, and bosses are stylized vector-like shapes. Particle system uses transparency decay for explosion effects.

**Sky Backgrounds**: Each stage has a 3-color gradient sky. Parallax clouds scroll at two speeds for depth. Sun glow fades behind clouds.

**Visual Effects**:
- Screen shake (intensity/duration configurable per event)
- Flash (white or flash overlay for hits/bombs)
- Vignette (red pulsing fade when critical health)
- Combo pop (animated text for milestone kills)
- Stage banner (intro text on stage transition)

**Quality Adaptation**: `quality` variable (0=full, 1=medium, 2=low) adjusts particle counts and shadow rendering if avg frame time drifts. Users can also toggle reduced-motion preference to disable shakes/flashes/animations entirely.

### Boss Patterns

Six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) each have 3 distinct attack patterns. Patterns fire in sequence with a telegraph phase (20-22 frames) before each volley. Patterns include:
- Radial bursts (all directions at once)
- Aimed spreads (fan toward player)
- Columns (straight down from multiple x-positions)
- Homing seeded volleys (tracking with noise)
- Teleport + attack combos

Radial bullets track `dx/dy` velocity rather than speed + direction, and update position in `updateBoss`.

## Development

### Running the Game

Simply open `index.html` in any modern browser. No server, build step, or setup required.

To test across devices:
- **Local**: Open file:// directly, or use `python -m http.server` (Python 3) in the directory
- **Mobile**: Serve over HTTP and visit from phone/tablet to test touch controls

### Testing Input Methods

- **Keyboard**: WASD/Arrows to move, Space or click to fire, X/Shift to bomb, P/Esc to pause
- **Mouse**: Move cursor over canvas to steer, click to fire
- **Touch**: Drag left side for movement joystick, tap FIRE/BOMB buttons on right side

To test difficulty scaling, toggle the difficulty buttons on the start screen and observe:
- Easy: slower spawns, slower enemies, 4 lives, 3 bombs
- Normal: baseline
- Hard: fast spawns, 1.9x enemy speed, 1.3x boss HP, reduced lives/bombs

### Tweaking Constants

Most tuning lives in global constants:

- `DIFFICULTIES` object: spawn rates, enemy speed/AI, starting lives/bombs, boss HP scaling
- `THEMES` array: sky colors and cloud tints (6 themes, cycled per stage)
- `ENEMY_TYPES`: sprite dimensions, HP, speed multiplier, color, score value
- `BOSSES` array: design colors, HP multiplier for each boss
- `BOSS_PATTERNS` array: attack pattern definitions for each boss
- `KILLS_PER_STAGE`: enemies before boss spawns (18 = ~2 min per stage at normal)
- `BASE_W`, `BASE_H`: game canvas size (480×720)

Weapon fire rates and spread angles are hardcoded in `fireWeapon()` per weapon level.

### Adding New Content

**New Enemy Type**: Add entry to `ENEMY_TYPES` with `w/h/hp/speedMul/color/dark/score/minStage`. The spawner picks enemies whose `minStage <= current stage`.

**New Boss**: Add to `BOSSES` array with design colors and HP multiplier. The index determines which pattern set it uses. Add a corresponding pattern array to `BOSS_PATTERNS` (3 attack functions).

**New Attack Pattern**: Write a function that calls one of `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, or `fireDoubleRing` to push to `enemyBullets`. Add it to the relevant boss's pattern list in `BOSS_PATTERNS`.

**New Theme**: Add to `THEMES` array with `sky` (3-color gradient stops), `cloud` (RGB string), `sun` (RGB string), and a descriptive `name`.

**New Sound**: Add function to `sfx` object using `beep(freq, duration, type, volume, opts)` or `noiseBurst(duration, volume, opts)`. Frequencies in Hz; durations in seconds. Filter types: 'lowpass', 'highpass', 'bandpass'. See existing sfx for patterns (attack + release envelopes, optional frequency sweep).

### Debugging & Performance

**FPS Issues**: Open DevTools (F12) and check `trackPerf()` logic. On slower devices, `quality` auto-reduces particle counts and glow effects. Users can also enable reduced-motion.

**Collision Problems**: `rectsOverlap(a, b)` uses simple AABB. If behavior seems off, verify entity `x`, `y`, `w`, `h` values. Debug by logging hitbox positions or drawing boxes around entities in `draw()`.

**Audio Not Working**: WebAudio requires a user gesture to start (click/touch to resume). `ensureAudio()` is called on first input to initialize the audio context. Check console for any WebAudio errors (e.g., on older browsers).

**Input Lag**: Mouse steering uses smooth interpolation (`mouseTarget` easing). If unresponsive, check that canvas rect is being calculated correctly in `canvasPointFromEvent()`.

## Key Data Structures

**Player**:
```
{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }
```

**Enemy**:
```
{ x, y, w, h, speed, shootTimer, canShoot, pattern, type, hp, color, dark, score, weave, weaveSeed, baseX }
```

**Bullet** (player):
```
{ x, y, w, h, speed, vx, pierce, laser? }
```

**Enemy Bullet**:
```
{ x, y, w, h, speed, vx?, radial?, dx?, dy? }
```

**Boss**:
```
{ x, y, w, h, hp, maxHp, entering, phaseTimer, attackTimer, patternIndex, vx, design, telegraph }
```

**Power-Up**:
```
{ x, y, w, h, speed, type: 'life'|'bomb'|'weapon' }
```

**Particle**:
```
{ x, y, vx, vy, life, maxLife, color }
```

## Browser Support

Requires:
- Canvas 2D context (`getContext('2d')`)
- RequestAnimationFrame
- WebAudio API (for sound; game runs silent if unavailable)
- localStorage (for persistence; graceful fallback to defaults if unavailable)
- Touch Events API (optional; touch controls only on touch devices)
- CSS Grid & Flexbox

Modern browsers (Chrome, Firefox, Safari, Edge from 2018+) are fully supported. IE11 will not work.

## Accessibility

- Screen-reader announcements for game events (`announce()` updates `#srLive` aria-live region)
- Keyboard-only controls (no mouse required for play, only menu navigation)
- Reduced-motion preference respected (disables animations on `prefers-reduced-motion: reduce`)
- Toggle for reduced-motion and audio in the UI

## Future Enhancements

Candidates for expansion without architectural change:

- More enemy types (e.g., shielded units, spawners)
- More bosses and attack patterns (simply extend arrays)
- Leaderboard (use localStorage or IndexedDB)
- Combo visual/audio feedback (extend `showComboPop`)
- Gameplay modifiers (e.g., slow-mo, bullet-time, invincible power-up)
- Custom controls (rebind keys/buttons)
- Mobile-specific optimizations (e.g., auto-quality for low-end devices)
