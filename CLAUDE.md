# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML5 file. It features multiple enemy types, dynamic boss encounters, procedural sound synthesis, and responsive input handling for desktop and mobile devices. The entire game runs client-side with no external dependencies.

## File Structure

- **index.html** — The complete game implementation (single file)
  - Embedded CSS for styling and animations
  - Embedded JavaScript (WebAudio synthesis, canvas rendering, game logic)
  - 1,670 lines total

- **README.md** — Minimal documentation (timestamp only)

## Development & Running

The game is a standalone HTML file—no build step or server is required.

### To Play
- Open `index.html` in a modern web browser
- Desktop: Mouse to steer, click to fire, Space to fire, X/Shift to bomb, P/Esc to pause
- Mobile: Drag left zone to move, tap FIRE button, tap BOMB button

### Web Development (if running locally)
```bash
# Simple HTTP server to serve the file
python3 -m http.server 8000
# or
npx http-server
```

Then visit `http://localhost:8000` in your browser.

## Architecture & Key Systems

### Game Loop & Rendering

The game uses `requestAnimationFrame` for a 60 FPS target with adaptive performance scaling:

```
loop() → trackPerf() → update(dt) → draw() → next frame
```

- **trackPerf()** — Monitors frame times over 45 frames; adjusts quality level (0=full, 1=medium, 2=low) based on performance
- **update(dt)** — All game logic: player movement, enemy AI, collision detection, scoring
- **draw()** — Canvas rendering: sky gradient, clouds, entities, particles, HUD

Key state is in global variables (not class-based). Canvas is 480×720 internally, scaled to fit viewport.

### Game States

1. **Menu/Overlay** — Before game starts, when paused, or after game over
2. **Running** — Active gameplay with update and draw
3. **Paused** — Running=true, paused=true; overlay shown
4. **Game Over** — running=false; final stats displayed

Transition via `resetGame()`, `togglePause()`, `endGame()`.

### Player System

Player object holds:
- Position (x, y), size (w=40, h=40)
- Weapon type ('vulcan' or 'laser') with level (1–3)
- Cooldown timer for fire rate
- Invulnerability timer after being hit
- Movement speed (260 px/s) with dual input priority:
  1. Keyboard/joystick (highest priority)
  2. Mouse steering (only if no keyboard input)

Firing managed by `fireWeapon()`:
- **Vulcan** — Spread shot at level 3 (3 angles), hitscan
- **Laser** — Piercing shot, wider shot count at level 3
- Fire delay: 0.15s (vulcan) or 0.1s (laser)

### Enemy System

Enemies stored in `enemies[]` array. Each enemy has:
- Type key (grunt, interceptor, tank) mapping to `ENEMY_TYPES` config
- Position, velocity, health, shooting state
- Weaving behavior (applies sine wave offset to movement)
- Two firing patterns: 'spread' (three-angle burst) or 'aimed' (toward player)

Key configs in `ENEMY_TYPES`:
- **grunt** — Weak, common, score 10
- **interceptor** — Fast, weave pattern, score 15
- **tank** — Slow, tough (4 HP), score 35, only from stage 2+

Spawn logic: `spawnEnemy()` picks type weighted by stage and random roll. Spawn rate decreases as frame count increases (difficulty ramp).

### Boss System

When `stageKills >= KILLS_PER_STAGE` (18 kills), boss spawns after 1.2s delay. Each stage cycles through 6 unique bosses (BOSSES array) with:
- Unique visual design (hull color, core color, wing color)
- HP multiplier (`hpMul`)
- Attack pattern set (3 patterns per boss in `BOSS_PATTERNS`)

Boss behavior in `updateBoss(dt)`:
1. **Entering phase** — Slides down from top
2. **Active phase** — Moves side-to-side, rotates through attack patterns
3. Telegraph warning (22 frames) before each attack, then attack fires
4. On death: large explosion, 500 points, advance stage, reset stage kills

Attack patterns are stored as functions taking the boss object and firing enemy bullets in various patterns (radial, aimed spreads, homing, spirals, etc.).

### Scoring & Multiplier

- Base score from kills: `ENEMY_TYPES[type].score`
- **Chain system** — Each kill increments chain; chain decays after 2.2s of no kills
- **Multiplier** — 1 + floor(chain / 5), up to 10x
- Breaking chain: on player hit, reset to 1x
- Boss kill: always 500 points (applies current multiplier)

### Difficulty System

Three modes (stored in localStorage, applied at game start):

| | Easy | Normal | Hard |
|---|---|---|---|
| Spawn rate (base/min) | 75/40 | 58/26 | 40/18 |
| Enemy speed | 1.0× | 1.4× | 1.9× |
| Enemy shoot chance | 45% | 70% | 95% |
| Lives/Bombs | 4/3 | 3/2 | 3/2 |
| Boss HP | 0.8× | 1.0× | 1.3× |

### Input Handling

- **Keyboard** — Arrow keys or WASD for movement; Space to fire; X/Shift for bomb; P/Esc to pause
- **Mouse** — Move cursor over canvas to steer player toward it; click to fire
- **Touch** — Left 55% of screen is move zone (joystick); right 45% has FIRE/BOMB buttons

Input priority: keyboard/joystick > mouse steering.

### Sound System

All sound is synthesized via Web Audio API (no external audio files):

- **beep()** — Oscillator with optional frequency sweep, filter, and panning
- **noiseBurst()** — White noise filtered through lowpass (for explosions)
- Individual sounds in `sfx` object (shoot, laser, explode variants, hit, bomb, powerup, etc.)

All audio is muted if user clicks mute button (🔊/🔇, stored in localStorage).

### Visual Systems

**Screen Shake** — `screenShake(mag, dur)` sets shake magnitude and duration; applied each frame as random offset. Reduced by 85% if reduced motion is enabled.

**Flash Effects** — Quick white flash overlay on hit (0.25s) or bomb (0.5s).

**Particles** — Explosions spawn particles with velocity, life timer, and fading alpha. Particle count scales with quality level.

**Themes** — Six environment themes cycle per stage: sky gradient, sun glow, cloud colors. Each has a name (Daylight Front, Sunset Corridor, etc.).

### Power-Ups

Drop on enemy death with weighted probability:
- Life (+1 health, 6% chance)
- Bomb (+1 bomb, 8% chance)
- Weapon (upgrade or swap, 20% chance)

Weapon progression:
- Level 1 → 2 → 3 (more bullets/spread)
- Level 3 to next weapon → resets to level 1

## Key Constants & Configuration

**Canvas & Scaling**
```javascript
BASE_W = 480, BASE_H = 720  // Internal resolution
scale = min(window.width/480, window.height/720, 1.6)  // Max scale 1.6x
```

**Game Balance**
```javascript
KILLS_PER_STAGE = 18  // Kills needed to trigger boss
fireDelay = 0.15s (vulcan) / 0.1s (laser)
playerSpeed = 260 px/s
```

**Difficulty Ramp** — Spawn rate decreases by 4 every 300 frames (5 seconds).

## Common Editing Tasks

### Adding an Enemy Type
1. Add entry to `ENEMY_TYPES` with (w, h, hp, speedMul, color, dark, score, minStage, weave)
2. Update `EXPLODE_SFX` if you want a custom explosion sound
3. Optionally add sound to `sfx` object if needed
4. Weights in `spawnEnemy()` probability roll

### Adding a Boss Pattern
1. Create a function `function fireX(b) { ... }` that calls `enemyBullets.push(...)` with new bullets
2. Add to appropriate slot in `BOSS_PATTERNS[bossIndex]` array (each boss has 3 patterns)
3. Patterns cycle in `updateBoss()` via `b.patternIndex % patterns.length`

### Tweaking Game Feel
- **Enemy spawn:** Adjust `DIFFICULTIES` spawn rates or change multiplier in `spawnTimer` calc
- **Boss HP:** Modify `baseHp = 70 + stage * 35` formula in `spawnBoss()`
- **Screen shake:** Increase/decrease `mag` param in `screenShake()` calls
- **Weapon balance:** Adjust fire delay in `fireWeapon()` or spread angles

### Adding a New Theme
1. Add object to `THEMES` array with (sky, cloud, sun, name)
   - `sky` — array of 3 colors for gradient stops
   - `cloud` — RGB string (e.g., "255,255,255")
   - `sun` — RGB string
   - `name` — stage display name
2. Themes cycle per stage: `THEMES[(stage - 1) % THEMES.length]`

## Performance Considerations

- **Quality scaling** — Adaptive quality (0/1/2) reduces particle count, shadow blur, and glow effects on low-end devices
- **Collision detection** — Simple rect overlap checks; boss/enemy/bullet/player checked each frame
- **Canvas rendering** — Uses canvas context save/restore and globalAlpha for efficiency
- **Audio** — Web Audio nodes created on demand (first beep/burst triggers audioCtx init)

## Accessibility Features

- **Screen reader support** — `announce()` function updates aria-live region with game events
- **Reduced motion** — `prefers-reduced-motion` CSS media query respected; stored in localStorage; disables animations, shake, flash
- **High contrast UI** — Status displays with dark backgrounds and colored text
- **Touch controls** — Mobile-friendly button layout with 100px touch targets

## Testing the Game

1. **Gameplay loop** — Start game, fly around, shoot enemies, collect power-ups, reach boss
2. **Difficulty modes** — Verify enemy spawns and HP scale correctly per difficulty
3. **Input** — Test keyboard, mouse, and touch input paths
4. **Mobile** — Test on phone/tablet; verify touch controls and responsive scaling
5. **Audio** — Mute/unmute toggle; verify sound effects play
6. **Performance** — Open DevTools Performance tab; check 60 FPS baseline, quality scaling on slower devices
7. **Accessibility** — Tab through UI; test with reduced motion enabled; screen reader

## Debugging Tips

- `console.log()` in update loop runs every frame; use sparingly
- Check DevTools Performance panel for frame drops (quality scaling should engage around 25ms/frame)
- Network tab: verify no external requests (game is 100% self-contained)
- Application tab: check localStorage keys (skyfighter.* keys)
- Use DevTools debugger breakpoints on `useBomb()`, `addScore()`, `killBoss()` to trace game flow

