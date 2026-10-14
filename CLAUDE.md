# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single-file web application. The game runs entirely in the browser with procedurally generated audio, canvas-based rendering, and responsive touch/keyboard/mouse controls.

## Architecture & Game Design

### Single-File Structure
The entire game is contained in `index.html` (~1,670 lines), organized as:
1. **HTML structure** (lines 1–365): Game canvas, HUD elements, overlay screens, touch controls UI
2. **CSS styling** (lines 10–350): Responsive design, animations (keyframes), dark theme, accessibility
3. **JavaScript game code** (lines 423–1,669): Wrapped in IIFE for closure; all game logic, state, and rendering

### Game Loop & State
- **Main loop**: `requestAnimationFrame(loop)` at ~60 FPS
- **Core update/draw pattern**: 
  - `update(dt)` handles input, collision, physics, enemy/boss spawning
  - `draw()` renders all game elements to canvas
- **Game state** (line ~865):
  - `score`, `lives`, `maxLives`, `running`, `paused`, `gameOver`, `stage`, `frame`
  - Player, bullets, enemies, enemy bullets, particles, power-ups, boss
  - Weapon state, multiplier chain, combo timer

### Key Game Objects & Systems

#### Player
- Position, velocity, collision bounds (w, h)
- Health/lives; invulnerability timer post-hit
- Current weapon (`vulcan`, `laser`, `spread`), weapon level
- Firing cooldown and momentum-based steering (keyboard > joystick > mouse)

#### Weapon System
Three weapon types (fireWeapon function, ~line 1124):
- **Vulcan**: Rapid-fire straight bullets
- **Laser**: Slower, wider bullets; pierces enemies
- **Spread**: Multi-directional cone; high spread count

Weapon upgrades via power-ups; fire rate and bullet count scale with level.

#### Enemy Types
Defined in `ENEMY_TYPES` constant (~line 502):
- **grunt**: Basic, low HP, low score
- **interceptor**: Fast, weaving path, decent HP
- **tank**: Slow, high HP, heavy fire, high score

Each has:
- Speed, HP, collision size (w, h), color, score value
- Shoot pattern (aimed vs. spread)
- Stage unlock requirement (minStage)

#### Boss System
Six boss designs in `BOSSES` array (~line 508). Per-stage boss spawns when kill count reaches `KILLS_PER_STAGE` (18). Bosses have:
- Named attack patterns (e.g., VANGUARD: radial burst + aimed fan + wide V-fan)
- Telegraph/wind-up phase (0.8s before attack)
- Scaling HP based on stage and difficulty
- Unique visual design (custom draw function)

#### Scoring & Multiplier Chain
- Kill multiplier starts at 1x, increases per consecutive kill
- Chain breaks if >2 seconds without a kill
- Combo pop-ups display current chain count
- Best score persisted to localStorage

#### Difficulty System
`DIFFICULTIES` constant defines per-level tuning:
- `enemySpeed`, `enemyShootChance`, `bossHpMul`, `damageScale`
- Normal is baseline; Easy/Hard adjust enemy and boss parameters

### Input Handling

#### Keyboard & Gamepad (Joystick)
- Arrow keys / WASD for movement
- Space for fire
- Joystick input combined with keyboard (vector sum, normalized if > 1)

#### Mouse
- Pointer position drives "glide steering": plane smoothly moves toward mouse cursor when no keyboard/joystick input
- Secondary control method with lower priority

#### Touch Controls
- Dynamically enabled on touch devices
- Left side (55%): movement zone
- Right side (45%): fire/bomb buttons
- Joystick metaphor (visual base + stick) appears on first touch in move zone

### Audio System
WebAudio API synthesis (no external audio files):
- `ensureAudio()` initializes AudioContext on first user interaction
- `beep(freq, dur, type, vol, opts)`: sine/square/sawtooth with optional filter, pan, delay
- `noiseBurst(dur, vol, opts)`: white noise burst (filtered)
- `sfx` object maps game events to sound generators (kill, pickup, bomb, hit, level-up, etc.)
- Mute state persisted in localStorage

### Visual Effects & Feedback
- **Screen shake**: `screenShake(mag, dur)` triggers on hit/bomb
- **Flash overlay**: `#flash` element pulses on hit (`flashHit`) or bomb (`flashBomb`)
- **Vignette**: Red pulsing edge effect during critical health (5% of max HP)
- **Combo pop-up**: Floats from kill point, fades over 0.6s
- **Stage banner**: Slides in at boss spawn

### Accessibility & Motion Preferences
- Respects `prefers-reduced-motion` media query; disables shake/flash/vignette if set
- Motion toggle button (`#motionBtn`) allows opt-in/opt-out of effects
- Screen reader live region (`#srLive`) announces game state changes
- Color contrast meets WCAG AA for HUD stats

### Performance Adaptation
- Tracks frame times; reduces rendering quality on sustained frame drops
- Quality levels: 0 (full—shadows, particles), 1 (medium—fewer shadows), 2 (low—no shadows, reduced particles)
- Enemy particle count and shadow effects scale down at lower quality tiers

## Development Commands

### Local Testing
Open `index.html` in a modern web browser (Chrome, Firefox, Safari, Edge):
```bash
# Development server (any local HTTP server)
python3 -m http.server 8000
# Then navigate to http://localhost:8000/index.html
```

Or open directly (for Chrome/Firefox):
```bash
open index.html
# or
firefox index.html
```

### Making Changes
1. Edit `index.html` directly in your editor
2. Save and refresh the browser (Cmd+R or Ctrl+R)
3. Test in multiple browsers and at different viewport sizes to check responsive behavior

### Testing Focus Areas
- **Input methods**: keyboard (arrows + space), mouse hover, touch joystick/buttons on mobile
- **Difficulty levels**: Swap difficulty via buttons on start screen; verify enemy speed/HP/shoot rate change
- **Weapon system**: Kill enemies to pick up weapon upgrades; verify spread and laser feel different
- **Boss patterns**: Reach stage 2+ and verify boss attacks match their named patterns (telegraph phase, attack timing)
- **Audio**: Toggle mute; verify sound effects fire on kill, pickup, bomb, hit, level-up
- **Accessibility**: Toggle reduced motion; verify screen reader announces score/lives changes
- **Responsive layout**: Test at small (mobile 375px), medium (tablet 768px), and large (desktop 1920px) viewports

### No Build Step Required
The game is ready to play as-is; no build tools, transpilation, or asset bundling needed. All CSS and JavaScript are inline; fonts are system fonts.

## Code Style & Patterns

- **Variables & scope**: Global DOM refs (canvas, UI elements) at top; game state below; functions organized by feature (audio, input, game loop, drawing)
- **Delta time**: Deltatime (`dt`) passed to update(); all motion and timers are dt-scaled for frame-rate independence
- **Canvas transforms**: `ctx.save()` / `ctx.restore()` used around rotations, translations (e.g., drawing rotated boss ships)
- **Collision**: Simple AABB (axis-aligned bounding box) for bullets vs. enemies and enemy bullets vs. player; particles use point-in-rect
- **Particle lifecycle**: Particles pushed to `particles` array with ttl (time to live); decremented each frame; filtered out when <= 0

## Key Constants & Tuning

| Constant | Line | Purpose |
|----------|------|---------|
| `BASE_W`, `BASE_H` | ~753 | Reference canvas size (480×720); scales responsively |
| `DIFFICULTIES` | ~485 | Per-level enemy/boss tuning multipliers |
| `KILLS_PER_STAGE` | ~491 | Enemy kills before boss spawn (18) |
| `THEMES` | ~493 | Background gradient colors per stage |
| `ENEMY_TYPES` | ~502 | Enemy stats (hp, speed, score, shape) |
| `BOSSES` | ~508 | Boss names, attack patterns, HP multipliers |
| `STORAGE_*` | ~463–466 | localStorage keys for persistence |

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with hp, speed, speedMul, w, h, color, dark, score, minStage
2. Update `spawnEnemy()` filter on line ~1026 to include new type when stage unlocks it
3. Add drawing logic to `draw()` function (mirror existing enemy draw)
4. Test spawning by advancing to stage where minStage is unlocked

### Adding a New Boss Attack Pattern
1. Add comment describing pattern in `BOSSES` array entry (e.g., "CUSTOM: description")
2. Find the boss's pattern generation in `updateBoss()` around line 1411; add new `else if` block keyed to patternIndex
3. Use existing bullet-spawn helpers: `spawnBullets(x, y, count, angleStart, angleSpan)` or manual push to `enemyBullets`
4. Test by reaching that stage and triggering the phase

### Tweaking Difficulty
Edit the `DIFFICULTIES` object; fields are self-documenting:
- `enemySpeed`: Higher = faster enemies
- `enemyShootChance`: Probability (0–1) of enemy firing per cycle
- `bossHpMul`: Multiplier on base boss HP
- `damageScale`: Multiplier on all weapon damage (affects kill rate)

### Modifying Audio
Edit the `sfx` object function calls:
- `beep(freq, dur, type, vol, opts)`: Adjust `freq` for pitch, `dur` for length
- `noiseBurst(dur, vol, opts)`: Use for effects like power-up whoosh
- Add new SFX entries to sfx object; call from relevant game events

### Adjusting Visual Effects
- **Screen shake**: Change `screenShake(mag, dur)` arguments (mag = pixels, dur = seconds)
- **Flash**: Tweak `flashHit()` and `flashBomb()` animation durations (currently 0.25s and 0.5s)
- **Vignette**: Modify the `#vignette.on` opacity animation or its appearance in CSS (~lines 75–87)

## Testing & Verification Checklist

- [ ] Game starts and displays HUD (score, lives, hearts/bombs display)
- [ ] Difficulty selection buttons work and affect spawn rate
- [ ] Player can move (keyboard + mouse OR touch joystick)
- [ ] Firing works and bullets appear
- [ ] Enemies spawn, move, shoot, and die with correct HP
- [ ] Killing enemies increases score and multiplier
- [ ] Multiplier resets after 2 seconds without a kill
- [ ] Weapon power-ups spawn and upgrade weapon type/level
- [ ] Boss spawns after 18 kills; telegraphs before each attack
- [ ] All 6 boss patterns attack correctly
- [ ] Game Over triggers at 0 lives; overlay shows final score
- [ ] New Best Score highlighting works (localStorage persists best)
- [ ] Mute button works; SFX disable when muted
- [ ] Reduced-motion preference disables shake/flash/vignette
- [ ] Touch controls hidden on desktop; visible on mobile
- [ ] Canvas resizes smoothly when window is resized

## Files & Deployment

- **Single file**: `index.html` (all-in-one)
- **Deployment**: Copy `index.html` to any web server; no build or dependencies needed
- **Browser support**: Modern browsers (ES6+, Canvas 2D, Web Audio API, localStorage)
  - Chrome/Edge 60+, Firefox 55+, Safari 12+

## Debugging Tips

- Check browser console (F12) for errors; WebAudio failures often silent
- Use browser DevTools performance tab if game feels sluggish; watch quality level in code
- To inspect game state in console: `player`, `score`, `stage`, `enemies.length`, etc., are global within IIFE
- Pause/unpause with #pauseBtn to freeze frame-by-frame debugging
