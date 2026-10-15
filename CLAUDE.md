# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game implemented as a single-file HTML5 application with embedded CSS and JavaScript (1669 lines total). The game features:

- Progressive stage-based gameplay with boss encounters
- Multiple weapon systems (Vulcan rapid-fire and Laser)
- Smart bomb mechanic with limited uses
- Difficulty levels (Easy, Normal, Hard)
- Score multiplier system via kill chains
- Responsive controls (keyboard, mouse, mobile touch)
- Procedural audio effects generated via Web Audio API
- Adaptive performance quality adjustment

## Running the Game

**No build process required.** Simply open `index.html` in a web browser. The game:
- Runs entirely in the browser with no dependencies
- Stores best score and settings in localStorage
- Auto-detects mobile/touch devices and displays appropriate controls
- Scales responsively to any screen size (base resolution: 480×720)

## Architecture

The entire game lives in a single IIFE (Immediately Invoked Function Expression) within `<script>` tags. The structure is:

### Rendering & DOM
- **Canvas-based 2D rendering** using CanvasRenderContext2D
- **HUD overlay** with score, lives, weapon stats, boss health bar
- **Canvas resizing** with `resize()` function that maintains 480×720 aspect ratio and scales to fit window

### Game Loop
- **`loop(now)`** – requestAnimationFrame-driven main loop that calls `update(dt)` then `draw()`
- **Frame-based time management** – deltaTime capped at 50ms to prevent spiral-of-death physics
- **Performance tracking** – `trackPerf(dt)` monitors FPS and auto-adjusts `quality` (0=full glow effects, 2=minimum effects)

### Core Game Objects
All stored in module-scoped arrays (not classes):

- **`player`** – `{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }` – position, dimensions, weapon state, invulnerability timer
- **`enemies`** – array of `{ x, y, w, h, hp, type, color, dark, vx, vy, shootTimer, shootDelay }`
  - Types: `'grunt'`, `'interceptor'`, `'tank'` (each with different stats in `ENEMY_TYPES`)
- **`boss`** – single enemy with special mechanics; `null` when not active
- **`bullets`** – player projectiles: `{ x, y, vx, vy, w, h, laser }`
- **`enemyBullets`** – enemy projectiles: `{ x, y, vx, vy, w }`
- **`powerUps`** – `{ x, y, w, h, type, vx, vy }` – types: `'life'`, `'bomb'`, `'weapon'`
- **`particles`** – visual effects: `{ x, y, vx, vy, life, maxLife, color }`

### Input Systems

1. **Keyboard** – Stored in `keys` object (e.g., `keys['ArrowUp']`, `keys['KeyW']`)
2. **Mouse** – Canvas coords mapped to game coords via `canvasPointFromEvent()`; steering uses `mouseTarget` which player glides toward
3. **Touch/Mobile** – Joystick in lower-left, Fire/Bomb buttons lower-right; `joyVec` and `firing` flags drive input

Priority: keyboard/joystick > mouse (mouse never overrides active keys)

### Physics & Collision

- **Movement** – Simple velocity-based (no acceleration curves)
- **Collision detection** – AABB (axis-aligned bounding box) via `dist()` for circle-like behavior
- **Boundary wrapping** – Off-screen bullets/enemies removed; screen bounds: `0..canvas.width`, `0..canvas.height`

### Game State

- **Difficulty** – Stored in `DIFFICULTIES` object; affects enemy spawn rate, speed, shoot chance, lives, bombs
- **Stage progression** – `stage` increments after `KILLS_PER_STAGE` (18) kills; themes cycle from `THEMES` array
- **Boss encounter** – Triggered when stage conditions met; boss has unique bullet patterns and health bar
- **Score multiplier** – `chain` / `chainTimer` system: killing enemies quickly within ~0.8s increases `multiplier`

### Rendering Pipeline

1. Clear canvas
2. Apply screen shake if active
3. Draw sky gradient (varies per stage theme)
4. Draw sun/clouds (parallax scrolling)
5. Draw player, enemies, bullets, particles
6. HUD updates (score, weapon, hearts, bombs) happen via DOM manipulation, not canvas

### Audio System

- **Web Audio API** – Procedural synthesis; no audio files
- **SFX objects** in `sfx` object: shoot, laser, explosions (per enemy type), bomb, power-up, boss sounds, UI click, game over
- **Master gain** controls mute state; localStorage persists mute preference
- **Audio context** lazy-initialized on first user interaction (`ensureAudio()`)

## Key Configuration Constants

Located near start of script:

- **`DIFFICULTIES`** – object with easy/normal/hard spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier
- **`KILLS_PER_STAGE`** – 18; enemies killed per stage before boss
- **`THEMES`** – Array of 6 sky/cloud/sun color palettes; cycles per stage
- **`ENEMY_TYPES`** – Object defining grunt, interceptor, tank with hp, points, shoot delay
- **`BASE_W`, `BASE_H`** – 480×720; canvas resolution (not display size)
- **Storage keys** – `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`

## Common Development Tasks

### Adding a new enemy type
1. Add entry to `ENEMY_TYPES` object with `hp`, `color`, `dark`, `points` properties
2. Update spawn logic in `update()` to create enemies with that type
3. Add drawing logic in `draw()` if custom appearance needed
4. Add SFX in `EXPLODE_SFX` mapping and `sfx` object

### Adjusting difficulty/balance
- Modify `DIFFICULTIES` object for spawn rates, enemy speed, player lives/bombs, boss scaling
- Adjust `KILLS_PER_STAGE` for stage length
- Modify `ENEMY_TYPES` stats for enemy power

### Tweaking weapon behavior
- Player weapon state: `player.weapon` ('vulcan' or 'laser')
- Shooting logic in `update()` under player cooldown section
- Modify bullet creation (velocity, appearance) to change weapon feel
- Weapon power-ups drop when player kills with score multiplier

### Screen effects
- `screenShake(magnitude, duration)` – triggers camera shake (respects reduced motion)
- `flashHit()`, `flashBomb()` – white flash overlay for impact feedback
- `vignetteEl` – red pulse on critical health (1 life remaining)

## File Location Reference

- **HTML/CSS** – lines 1–359: document structure, responsive styling, animations
- **Script initialization** – lines 422–470: IIFE setup, DOM element caching
- **Performance tracking** – lines 473–483
- **Game constants** – lines 485–500+
- **Audio system** – lines 550–676
- **Input handling** – lines 752–862
- **Game state/reset** – lines 863–900+
- **Main update loop** – search for `function update(dt)`
- **Main render loop** – search for `function draw()`
- **Main game loop** – search for `function loop(now)`

## Performance Considerations

- **Quality levels** – Glow effects (shadow, blur) disabled at `quality >= 1` to maintain 40 FPS
- **Particle limits** – Particles created on explosion; check `particles` array length if performance degrades
- **Enemy spawning** – Tied to difficulty and stage; higher difficulties spawn more frequently
- **Canvas scaling** – Rendered at fixed 480×720, then CSS-scaled; no GPU-intensive 3D or complex filters

## Known Limitations & Future Improvements

- Single file limits code organization; consider modularizing if game grows significantly
- Touch controls are contextual UI buttons; no analog stick simulation on desktop
- Boss patterns are hard-coded; procedural generation could add variety
- No pause-menu options; only mute/motion-reduction toggles
- Multiplayer/networking not supported
- Audio synthesis can glitch on low-end devices with heavy CPU usage

## Browser Compatibility

- Requires: Canvas 2D, requestAnimationFrame, Web Audio API, localStorage, TouchEvent API (mobile)
- Tested: Modern Chrome, Firefox, Safari, Edge
- Graceful degradation: Game runs even if audio fails (muted)
