# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML file with embedded CSS and JavaScript. The game features multiple difficulty levels, boss encounters, weapon upgrade systems, and responsive controls for both desktop and mobile.

## Architecture

### Single-File Game Structure

The entire game is in `index.html` with three main sections:

1. **HTML (lines 361-420)**: DOM structure for canvas, UI overlays, HUD, and touch controls
2. **CSS (lines 10-359)**: Styling, animations, responsive design, and accessibility
3. **JavaScript (lines 422-1667)**: Complete game engine and logic

### Core Game Systems

#### Game State Management
- `resetGame()`: Initializes a new game with difficulty-based parameters
- `update(dt)`: Called each frame; handles movement, collision detection, spawning
- `draw()`: Renders all game elements
- `loop(now)`: Main game loop using `requestAnimationFrame`

The game tracks:
- Player object with position, weapon, cooldown, invulnerability
- Arrays for bullets, enemies, enemy bullets, particles, power-ups
- Score, lives, stage, multiplier chain for combos
- Boss state (if active)

#### Difficulty System (lines 485-489)
Three difficulties (`easy`, `normal`, `hard`) with tunable parameters:
- Enemy spawn rate and minimum spawn timer
- Enemy speed, shoot chance
- Starting lives and bombs
- Boss HP multiplier

These are stored in `localStorage` (key: `skyfighter.difficulty`) so they persist.

#### Enemy and Boss Logic

**Enemies** (spawned in `spawnEnemy()`, updated in `update()`):
- Three types: grunt, interceptor, tank — each with different HP, speed, score
- Tank enemies have multi-hit HP displayed as a health bar
- Two firing patterns: "spread" (radiating bullets) or "aimed" (at player)
- Weave movement for interceptors

**Bosses** (spawned when `stageKills >= KILLS_PER_STAGE`):
- Six boss designs with unique visual styling (defined in `BOSSES` array, line 508-515)
- Each boss has 3 attack patterns in `BOSS_PATTERNS` (lines 1372-1409)
- Patterns include radial bursts, aimed spreads, columns, homing, spirals, crosses
- Boss telegraph animation (white flash) before attacks
- Health bar shown during boss fight

#### Weapon and Power-Up System

**Player Weapons** (toggled by power-ups):
- **Vulcan**: Spreads bullets based on level (1-3); fire rate 0.15s
- **Laser**: Piercing projectiles with beam effect; fire rate 0.1s; levels increase lane count

**Power-Up Types** (spawned randomly when enemies die):
- Life: Restores 1 HP
- Bomb: Adds 1 bomb
- Weapon: Levels weapon or switches to laser

#### Input Handling
- **Keyboard**: Arrow keys / WASD for movement, Space for fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Move plane toward cursor, click to fire
- **Touch**: Left zone for movement joystick, right zone for fire/bomb buttons
- Input priorities: Keyboard > Joystick > Mouse (so inputs don't interfere)

#### Audio System (lines 542-676)

Uses WebAudio API for **synthesis only** — no external audio files. All sound effects are procedurally generated:
- `beep()`: Oscillator-based tones with optional frequency sweep, filter, pan
- `noiseBurst()`: White noise with filter envelope
- `sfx` object contains named sounds (shoot, laser, explosions, powerup, etc.)

Muted state persists in `localStorage` (key: `skyfighter.muted`).

#### Visual Effects

- **Particle system**: Small dots spawn on explosions, fade out over time
- **Screen shake**: Magnitude and duration-based; reduced when reduced-motion is enabled
- **Flash overlays**: White flash on player hit, different for bombs
- **Vignette**: Red pulsing vignette when player has 1 life
- **Combat pop**: Floating text showing "X KILLS!" every 10 kills
- **Stage banner**: Animated text on stage transitions

#### Performance Optimization (lines 473-483)

Adaptive quality system (`quality` 0-2) monitors frame time:
- Quality 0 (full): Glow effects, shadow blur, full particle count
- Quality 1 (medium): 60% particle count, reduced shadow blur
- Quality 2 (low): 35% particle count, no shadow effects

Frames are tracked in a rolling 45-frame window; quality adjusts automatically.

#### Combo/Multiplier System

- Chain counter increments each time enemy dies
- Chain timer resets to 2.2s per kill; breaks if timer expires
- Multiplier = 1 + min(9, floor(chain / 5)) — scales up to 10x
- Score multiplied by current multiplier

#### Accessibility Features

- `prefers-reduced-motion` media query respected by default
- Reduced motion toggle button disables animations and screen effects
- Screen reader announcements via `srLive` (aria-live region)
- Semantic HTML with ARIA labels for buttons
- Color contrast maintained for UI text

#### Canvas Sizing (lines 752-765)

- Internal canvas resolution: 480×720 (constant)
- Scales to fill window with CSS transform, up to 1.6× native size
- Maintains aspect ratio; respects max dimensions
- Re-sizes on window resize and orientation change

#### Local Storage Keys

- `skyfighter.bestScore`: Best score ever achieved
- `skyfighter.muted`: "1" if muted, "0" if not
- `skyfighter.difficulty`: "easy", "normal", or "hard"
- `skyfighter.reducedMotion`: "1" if enabled, "0" if not

## Development Tasks

### Running the Game

No build step required. Simply open `index.html` in a web browser:
```bash
# Start a local HTTP server (Python 3)
python -m http.server 8000
# Then open http://localhost:8000/index.html

# Or use any static server (e.g., Node.js http-server)
npx http-server
```

### Testing Gameplay

Key scenarios to verify:
1. **Player movement**: Keyboard/arrow keys, mouse, touch joystick on mobile
2. **Firing**: Space bar, mouse click, touch fire button — cooldown enforcement
3. **Bomb usage**: Clears all enemies and bullets, damages boss
4. **Difficulty modes**: Easy, normal, hard — verify enemy spawn rate and difficulty
5. **Boss patterns**: Each stage spawns correct boss design with unique attacks
6. **Power-ups**: Weapon upgrades level correctly, bomb/life pickups work
7. **Combo multiplier**: Chain counter visible, breaks on gap > 2.2s
8. **Mobile**: Touch controls appear on touch devices, joystick constrains to 50px radius
9. **Pause**: P or Esc toggles pause overlay; resume button works
10. **Audio**: All SFX play (or mute works if disabled)
11. **Accessibility**: Reduced motion toggle, screen reader announcements

### Common Code Patterns

**Collision detection**: Uses `rectsOverlap(a, b)` (line 1069) for all bounding-box checks

**Damage and cleanup**: When entity dies:
1. Call `spawnExplosion(x, y, color, count)` for visual effect
2. Call `maybeSpawnPowerUp(x, y)` for enemies (may drop nothing)
3. Call `addScore(points)` to increment combo chain
4. Splice entity from array
5. Play SFX via `sfx.soundName()`

**Screen effects**: Use `screenShake(magnitude, duration)` and `flashHit()` / `flashBomb()`

**Announcements**: Use `announce(text)` for screen reader updates; text appears in aria-live region

### Extending the Game

#### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502)
2. Add SFX key to `EXPLODE_SFX` mapping (line 678) if unique death sound
3. Add death sound function to `sfx` object (line 624)
4. Update `spawnEnemy()` logic if needed for min stage or spawn chance

#### Adding a New Boss
1. Add to `BOSSES` array (line 508)
2. Create attack pattern array in `BOSS_PATTERNS` (line 1372) — must match index
3. Boss will spawn on stage = (boss index + 1); sprite colors auto-pulled from design

#### Adding a New Theme
1. Add to `THEMES` array (line 493)
2. Includes sky gradient, cloud color, sun color, and display name
3. Theme cycles by stage: `THEMES[(stage - 1) % THEMES.length]`

#### Modifying Weapon Balance
- Adjust `fireDelay` in `update()` (line 1177) for fire rate
- Adjust bullet `speed`, `vx`, `pierce` in `fireWeapon()` (lines 1124-1146)
- Adjust spread angles and lane offsets for weapon levels

## Key Constants

- `BASE_W = 480, BASE_H = 720`: Canvas resolution
- `KILLS_PER_STAGE = 18`: Enemies to defeat before boss
- Quality levels (0–2) based on frame time averages
- Difficulty parameters scale spawn rate, enemy speed, HP, lives

## Files and Modules

This is a single-file project. The entire codebase lives in `index.html`. There are no separate modules, build steps, or dependencies (beyond the Web APIs: Canvas, WebAudio, localStorage).

## Notable Implementation Details

- **No external assets**: All graphics drawn with Canvas 2D API; all audio synthesized with WebAudio
- **Immediate mode rendering**: Canvas cleared and redrawn every frame — no scene graph
- **Frame-based animations**: Effects tied to `frame` counter (increments each loop); no CSS animations for gameplay
- **Soft performance scaling**: Quality degrades gracefully on slow devices without affecting gameplay
- **Bias toward responsiveness**: Reduced motion respected; animations skipped if user prefers
- **Mobile-first input**: Touch controls are first-class; keyboard and mouse are fallbacks

## Debugging Tips

- Open browser DevTools console to see any errors
- Check `quality` variable to see adaptive quality level
- Monitor `frame` counter for animation timing
- Use `console.log()` in `update()` or `draw()` to inspect frame-by-frame state
- Modify `DIFFICULTIES` or `THEMES` to test different configurations instantly
- Use localStorage inspector to verify persistence (`getItem('skyfighter.bestScore')`)
