# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Sky Fighter: Raiden-Style Vertical Shoot-'em-Up

A single-file browser game built with vanilla JavaScript, Canvas, and WebAudio. No build step or external dependencies required.

## Quick Start

- **Play the game**: Open `index.html` in a web browser
- **Deploy**: Serve `index.html` on any HTTP server; no build process needed
- **Edit**: All code is embedded in `index.html` — HTML (DOM structure), CSS (styles), and JavaScript (game logic) in one file

## Game Architecture

The entire game runs in a self-contained IIFE (Immediately Invoked Function Expression) starting at line 423. Key systems:

### Core Game Loop (lines 1640–1653)
- `loop()` runs on `requestAnimationFrame`, handling updates and rendering
- `update(dt)` processes game state each frame
- `draw()` renders everything to the canvas
- Performance tracking (`trackPerf()`, `quality` variable) adapts rendering detail to frame rate

### Player & Movement (lines 752–861)
- Canvas is fixed at 480×720 logical pixels, scaled to fit viewport
- Player position (`player.x`, `player.y`) controlled by keyboard (arrows/WASD), mouse (aim), joystick (touch), or arrow keys
- Keyboard input stored in `keys` object; mouse/touch input updates `mouseTarget` and `joyVec`
- Mouse always takes priority when active; keyboard/joystick can override

### Weapons & Firing (lines 1124–1146)
- Two weapon types: `vulcan` (spread shot) and `laser` (piercing beam)
- Weapon upgrades via power-ups: levels 1–3 improve spread/lanes, switching weapon at level 3 resets to level 1
- Fire rate differs by weapon: vulcan 0.15s, laser 0.1s
- `fireWeapon()` adds bullets to array; bullets move upward (negative Y)

### Enemies (lines 1022–1227)
- Three types: `grunt` (1 HP), `interceptor` (fast, 1 HP), `tank` (4 HP, slow)
- Spawn rate increases with difficulty and stage progression
- Two shoot patterns: `spread` (static spray) or `aimed` (at player)
- Interceptors weave left-right; killed enemies spawn power-ups randomly
- Stage progresses after 18 kills; boss warning at kill 18 with 1.2s delay

### Boss System (lines 1043–1453)
- One boss per stage from the `BOSSES` array (6 designs, cycles)
- Each boss has unique attack pattern set in `BOSS_PATTERNS` (6 patterns, one per boss design)
- Boss enters from top, moves horizontally with bounce, telegraphs attack (22 frames), then fires
- Attack patterns vary: radial bursts, aimed spreads, columns, spirals, crosses, homing shots, double rings
- Boss defeated at 0 HP; triggers stage clear sound, score bonus, stage increment

### Difficulty Settings (lines 485–489)
- Three presets: Easy, Normal, Hard — stored in localStorage
- Affects spawn rates, enemy speed, shooting chance, lives, bombs, boss HP multiplier
- Difficulty can be changed before starting, persists across sessions

### Scoring & Multiplier Chain (lines 1087–1101)
- Base score for each kill; multiplied by chain counter (starts at 1, caps at 10)
- Each consecutive kill within 2.2s extends chain; chain breaks after timeout or player takes damage
- Multiplier increases every 5 kills: 1x → 2x → 3x ... up to 10x

### Audio (lines 542–676)
- WebAudio synth, no external audio files
- `beep()` generates tonal sounds (oscillator + optional filter + optional panning)
- `noiseBurst()` generates noise (white noise with decay envelope)
- Each game action has unique SFX: shoot, laser, explosions, hit, bomb, powerup, boss hit, stage clear, gameover
- Master gain at 0.9; muted state persists in localStorage

### Touch & Mobile (lines 778–834)
- Auto-enables on touch devices: virtual joystick (left side) and fire/bomb buttons (right side)
- Joystick shows circle base when active, constrained to 50px radius
- Fire button: touchstart/touchend toggle; bomb button immediate on touch
- All input prioritizes keyboard/joystick over mouse aim

### Collision Detection (line 1069)
- `rectsOverlap(a, b)` uses AABB: checks distance between centers vs. summed half-widths
- Tests: bullets vs. enemies/boss, enemies/bullets vs. player, boss vs. player, powerups vs. player

### Visual State & Feedback
- **Screen shake**: Triggered by player hit (mag 10), bomb (14), boss death (18) — reduced by 85% if reduced-motion is on
- **Vignette flash**: Red pulsing overlay when player at 1 life
- **Flash effect**: White flash on player hit or bomb detonation
- **Combo pop**: Animated text at kills 10, 20, 30... (every 10th kill)
- **Stage banner**: Text animation at stage start and boss approach
- **Boss health bar**: Only visible when boss active; hides after death
- **Hearts & bombs HUD**: Emoji-based with lost/active states

### Performance Scaling (lines 473–483)
- `quality` levels: 0 (full), 1 (medium), 2 (low)
- Adjusts particle count (explosion size), shadow/glow effects, and screen shake intensity
- Auto-adjusts based on 45-frame running average of frame times

### Theme Cycling (lines 493–500)
- Six sky themes with different gradients and clouds
- Cycles every stage: `THEMES[(stage - 1) % THEMES.length]`
- Each has sky colors, cloud color, sun glow, and name for accessibility announcements

## Constants & Enums

### Difficulty Presets (`DIFFICULTIES`)
```javascript
{
  spawnBase: base spawn interval (frames),
  spawnMin: minimum spawn interval,
  enemySpeed: multiplier for all enemy speed,
  enemyShootChance: probability enemy can shoot,
  lives: starting lives,
  bombs: starting bombs,
  bossHpMul: multiplier for boss HP
}
```

### Enemy Types (`ENEMY_TYPES`)
```javascript
{
  grunt, interceptor, tank
  Properties: w, h, hp, speedMul, color, dark, score, minStage, weave?
}
```

### Bosses (`BOSSES`)
```javascript
Array of 6 designs: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
Properties: name, hullTop/hullBottom/wing/core (colors), hpMul
```

### Boss Attack Patterns (`BOSS_PATTERNS`)
Each boss has 3 patterns, cycling through. Patterns include:
- `fireRadial(b, count, speed, offset?)`: bullets in circle
- `fireAimedSpread(b, count, speed, spreadStep)`: bullets aimed at player with spread
- `fireColumns(b, speed)`: 5 straight columns downward
- `fireSpiralBurst(b, speed)`: 8 bullets in rotating spiral
- `fireCross(b, speed)`: 8-way cross
- `fireHomingSeeded(b, count, speed)`: homing bullets with scatter
- `fireDoubleRing(b, speedA, speedB)`: two concentric rings

### Power-Up Types
- `'life'`: +1 life (capped at max)
- `'bomb'`: +1 bomb (capped at max)
- `'weapon'`: upgrade weapon level or switch weapon (cycles vulcan ↔ laser)

## Storage Keys
- `skyfighter.bestScore`: Best score achieved (number)
- `skyfighter.muted`: Mute state (0 or 1)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Reduced motion enabled (0 or 1)

## Accessibility
- Screen reader announcements via `announce()` function, posted to `#srLive` ARIA live region
- Reduced-motion mode disables animations (screen shake, flash, vignette pulse, combo pop, etc.)
- Respects system `prefers-reduced-motion` media query; user can override in settings
- Keyboard-only control support (arrows + space + X/Shift)
- High contrast dark UI with yellow text accents

## Making Changes

**Adding a new enemy type**: Update `ENEMY_TYPES`, add spawn logic in `spawnEnemy()`, add explosion SFX in `sfx` object

**Adding a new boss pattern**: Add function to boss pattern library (`fireRadial` style), add 3 patterns to new `BOSS_PATTERNS[index]` entry

**Tweaking difficulty**: Adjust `DIFFICULTIES` values; Higher spawnBase = longer waits, lower enemySpeed = slower enemies, etc.

**Adding a theme**: Push new object to `THEMES` array with sky gradient stops, cloud color, sun color, and name

**Changing game speed**: Adjust `player.speed` (260), enemy speed multiplier in `DIFFICULTIES`, or bullet speeds in weapon fire functions

**Adjusting collision**: Modify `rectsOverlap()` or individual entity dimensions (`.w`, `.h`)
