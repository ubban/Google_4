# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The entire game (code, styling, and assets) is contained in `index.html` (~1670 lines).

## Architecture

### Single-File Structure
The game is a complete, self-contained implementation with three layers:
1. **HTML**: DOM structure for canvas, HUD, overlays, and touch controls
2. **CSS**: Responsive styling, animations, and visual effects
3. **JavaScript**: Game engine, rendering, and state management

No external dependencies or assets are required—audio is synthesized via Web Audio API.

### Core Game Engine
The game loop (`loop()` function, line 1640) runs at 60 FPS and orchestrates:
1. **Update phase** (`update()`, line 1148): Player input, collision detection, entity updates
2. **Render phase** (`draw()`, line 1520): Canvas drawing with background, entities, particles
3. **Performance tracking** (`trackPerf()`, line 476): Adaptive quality degradation on low-end devices

### Key Systems

**Game State** (lines 863-870): `running`, `paused`, `gameOver`, `score`, `lives`, `stage`, `multiplier`

**Player System** (lines 875-901):
- Movement: Keyboard (arrows/WASD), joystick (touch), or mouse-based steering
- Weapons: Vulcan (spread shot) or Laser (piercing beam), with 3 upgrade levels
- Invulnerability: 1.5s after taking damage

**Enemy System** (lines 1022-1041, 1210-1227):
- Three types: Grunt, Interceptor, Tank
- Behaviors: Weaving movement, aimed/spread shooting patterns
- Progressive difficulty: Tank enemies appear in stage 2+

**Boss System** (lines 1043-1056, 1372-1439):
- Six unique boss designs with individual stats and attack patterns
- Telegraph system: 22-frame warning before attack
- Pattern-based attacks: radial bursts, aimed spreads, columns, spirals, etc.
- Dynamic health scaling based on difficulty and stage

**Collision & Scoring** (lines 1069-1071, 1087-1094):
- AABB rectangle overlap detection
- Combo multiplier: up to 10x based on kill chain (decays after 2.2 seconds)
- Boss kills grant 500 bonus points

**Power-Up System** (lines 1058-1067, 1304-1322):
- Life restore (6% spawn chance)
- Bomb recovery (8% spawn chance)
- Weapon upgrade: +1 level up to 3, then toggle Vulcan ↔ Laser (20% spawn chance)

**Bomb Mechanic** (lines 1103-1122):
- Clears all enemy bullets and enemies
- Damages boss by 24 HP
- Costs 1 bomb, max 2-3 depending on difficulty

### Difficulty Scaling
Three presets (lines 485-489) control: spawn rate, enemy speed, enemy accuracy, health, bombs
- Easy: 75 frame base spawn, 1.0x enemy speed, 0.8x boss HP
- Normal: 58 frame base spawn, 1.4x enemy speed, 1.0x boss HP
- Hard: 40 frame base spawn, 1.9x enemy speed, 1.3x boss HP

### Visual Features
- **Themes** (lines 493-500): Six atmospheric sky gradients + cloud colors that rotate per stage
- **Screen Effects**: Screen shake (magnitude/duration), flash on damage, vignette on critical health
- **Particles**: Explosion effects with velocity and lifetime
- **HUD**: Score, high score, multiplier, weapon level, hearts, bombs
- **Boss bar**: Tracks boss HP as gradient bar

### Audio System
**Web Audio Synth** (lines 542-676):
- No external assets—all sounds generated algorithmically
- Oscillators: square, sine, sawtooth (tonal)
- Noise bursts with filters for impact
- Panning based on screen position (stereo effects)
- Master gain with dynamics compressor
- 15 distinct SFX: shoot, laser, explosions (grunt/interceptor/tank), hit, bomb, powerup, boss patterns, stage clear, game over

### Input Handling
**Keyboard** (lines 769-776):
- Arrow keys or WASD: movement
- Space: fire
- X or Shift: bomb
- P or Esc: pause

**Mouse** (lines 847-861):
- Move to steer (smooth glide toward pointer)
- Click to fire

**Touch** (lines 814-834):
- Left 55% of screen: joystick (virtual stick appears on touch)
- Right 45% of screen: Fire/Bomb buttons
- Tap indicators on-screen

## Customization Points

### Adding a New Enemy Type
Add entry to `ENEMY_TYPES` (line 502) with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optionally `weave`. Shooting behavior is hardcoded in `update()` (lines 1217-1226).

### Adding Boss Attack Patterns
- Extend `BOSSES` array (line 508) with new design
- Add pattern functions to `BOSS_PATTERNS` (line 1372) matching boss index
- Available pattern helpers: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`

### Adding New Themes
Insert into `THEMES` array (line 493) with `sky` (3-stop gradient), `cloud` color, `sun` color, `name`.

### Adjusting Game Feel
- Player speed: `player.speed` (line 877, default 260 px/s)
- Bullet speeds: hardcoded in `fireWeapon()` (lines 1126-1145) and boss patterns
- Screen shake parameters: `screenShake()` calls throughout
- Difficulty curve: spawn timer logic (line 1201) and stage-based enemy selection (lines 1026-1030)

## Running & Testing

**Play**: Open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge).

**Defaults**:
- Difficulty: Normal (remembered in localStorage)
- Mute: Off (toggle with 🔊 button)
- Reduced Motion: Auto-detects OS preference, toggle with 🎆 button

**Local Storage Keys**:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio on/off
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Motion preferences

## Performance Considerations

**Adaptive Quality** (lines 474-483):
- Tracks last 45 frame times
- Degrades quality if avg > 25ms (quality 0→1) or improves if < ~18ms
- Three levels: 0 (full glow + shadow), 1 (50% particle count), 2 (35% particles, no glow)

**Canvas Sizing** (lines 753-765):
- Fixed logical size: 480×720 px
- Scales to fit screen (max 1.6x) with device pixel ratio handling
- Respects safe area insets on notched devices

## Accessibility

- **Screen reader announcements** via `announce()` (line 468) for: pause state, difficulty, reduced motion toggle, boss defeat
- **Reduced motion support** (line 349): disables animations if OS preference set
- **Keyboard-navigable UI**: all buttons accessible
- **High contrast HUD**: text shadows, opacity values chosen for readability

## Development Workflow

1. **Edit**: Modify `index.html` directly
2. **Test**: Reload browser; use browser dev tools for performance profiling
3. **Debug**: Check console for any errors; use Network tab if future versions add assets
4. **Commit**: Changes to this single file track all updates

No build step or package manager required.
