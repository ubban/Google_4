# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file HTML5 game — a Raiden-style vertical shoot-'em-up with boss stages. The entire game (HTML, CSS, and JavaScript) is contained in `index.html`.

### Key Features
- Canvas-based 2D rendering
- Three difficulty levels (Easy, Normal, Hard) with dynamic spawn rates and enemy stats
- Weapon system: Vulcan (spread shots) and Laser (piercing, multi-lane)
- Boss battles with procedural attack patterns
- Power-ups: extra lives, bombs, weapon upgrades
- Adaptive performance: automatic quality downgrade if frame rate drops
- WebAudio API synth (no external audio assets)
- Touch, mouse, and keyboard controls
- Accessibility: screen reader announcements, reduced-motion preference detection
- Persistent storage: best score, difficulty, audio mute state, motion preferences

## Architecture

### Code Organization (all in index.html)

1. **HTML (lines 361–420)**: DOM structure with canvas, HUD, overlay, buttons
2. **CSS (lines 10–359)**: Responsive design with animations, flex layouts, dark theme
3. **JavaScript (lines 422–1667)**: Single IIFE with all game logic

### JavaScript Sections

- **Setup & State** (lines 423–533): Constants, difficulty presets, theme palettes, boss designs, enemy types; persistent storage for best score, settings
- **Audio** (lines 542–676): WebAudio context, synth sound generators (`beep`, `noiseBurst`), SFX effects object
- **Rendering Helpers** (lines 680–730): UI updates (hearts, bombs, combo pop, stage banner), screen shake, flash effects
- **Canvas & Input** (lines 752–862): Canvas sizing, keyboard/mouse/touch input handling, joystick logic for mobile
- **Game State & Core Loop** (lines 863–1666): Main `update()` and `draw()` functions, spawn logic, collision detection, boss AI, game state transitions

### Key Data Structures

**Player:**
```javascript
{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }
```

**Enemy:**
```javascript
{ x, y, w, h, type, hp, speed, color, dark, score, pattern, shootTimer, weave, weaveSeed, baseX }
```

**Boss:**
```javascript
{ x, y, w, h, hp, maxHp, entering, phaseTimer, attackTimer, patternIndex, vx, design, telegraph }
```

**Power-up:**
```javascript
{ x, y, w, h, speed, type: 'life' | 'bomb' | 'weapon' }
```

## Game Constants & Configuration

| Constant | Purpose | Location |
|----------|---------|----------|
| `DIFFICULTIES` | Spawn rates, enemy stats per difficulty | Line 485 |
| `KILLS_PER_STAGE` | Enemy kills needed to trigger boss (18) | Line 491 |
| `THEMES` | 6 sky/cloud color schemes | Line 493 |
| `ENEMY_TYPES` | Grunt, Interceptor, Tank stats | Line 502 |
| `BOSSES` | 6 boss designs with HP multipliers | Line 508 |
| `BOSS_PATTERNS` | Attack patterns for each boss type | Line 1372 |
| `BASE_W`, `BASE_H` | Canvas dimensions (480×720) | Line 753 |

## Development & Testing

### Running the Game
Open `index.html` directly in a browser. No build step, no server required.

### Testing Difficulty Settings
- Click difficulty buttons before starting (Easy/Normal/Hard)
- Settings persist via localStorage
- Each difficulty affects spawn rate, enemy speed, shoot chance, boss HP

### Testing Features
- **Weapons**: Collect weapon power-ups to cycle between Vulcan (spread) and Laser (pierce)
- **Bombs**: Press X, Shift, or tap BOMB; destroys all enemies on screen and damages boss
- **Collision**: Test by running into enemies or their bullets
- **Boss patterns**: Each of 6 bosses has 3 attack patterns; they cycle and become more dangerous
- **Touch controls**: On mobile, left zone for movement (joystick), right zone for fire/bomb

### Performance Testing
- Frame times are tracked in `frameTimes[]` (target: 45+ samples)
- `quality` auto-scales: 0 = full effects, 1 = medium, 2 = low (fewer particles, no glow)
- Average frame time > 1/40 frame triggers quality downgrade
- View quality by checking shadows (`ctx.shadowBlur`) and particle count

### Accessibility Testing
- Reduced motion button (🎆 icon): removes animations, screen shake, flashes
- Screen reader: status announcements in `#srLive` (line 390)
- Test with `prefers-reduced-motion: reduce` media query

## Common Modifications

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502) with `{ w, h, hp, speedMul, color, dark, score, minStage }`
2. Update `spawnEnemy()` (line 1022) to include new type in spawn choices
3. Add explosion SFX to `EXPLODE_SFX` (line 678)

### Adding a New Boss Attack Pattern
1. Create a fire function at line 1324+: `fireRadial`, `fireAimedSpread`, `fireColumns`, etc.
2. Add pattern array to `BOSS_PATTERNS` (line 1372) — pattern 3 attacks per boss
3. Boss design colors are in `BOSSES` (line 508); update `drawBossShip()` (line 970) if visual changes needed

### Tuning Difficulty
Edit `DIFFICULTIES` (line 485):
- `spawnBase`: milliseconds between enemy spawns (lower = faster)
- `spawnMin`: spawn cap as difficulty ramps
- `enemySpeed`: movement multiplier
- `enemyShootChance`: 0–1, probability enemy fires
- `lives`, `bombs`: starting counts
- `bossHpMul`: boss health scaling

### Changing Colors or Theme
- HUD colors: CSS variables or inline styles (lines 10–359)
- Sky themes: `THEMES` array (line 493) has gradients and cloud/sun colors
- Enemy colors: `ENEMY_TYPES` color/dark fields (line 502)
- Boss colors: `BOSSES` design colors (line 508)

### Adjusting Canvas Size
Change `BASE_W` and `BASE_H` (line 753). Game scales to fit screen while maintaining aspect ratio via `resize()` (line 754).

## Collision & Physics

- **Bounding boxes**: `rectsOverlap()` (line 1069) checks axis-aligned rectangles
- **Movement**: Pixel-based with delta-time scaling (`dt`)
- **Keyboard input**: Processed each frame to allow smooth diagonal movement (lines 1152–1170)
- **Mouse steering**: Smoothly glides player toward cursor when no keyboard input active (line 1164)
- **Joystick (touch)**: Left zone spawns visual joystick, constrains movement to 50px radius

## Audio

All sound is synthesized via WebAudio API; no audio files needed.

- `beep(freq, dur, type, vol, opts)`: Tone with optional sweep/filter/pan/delay
- `noiseBurst(dur, vol, opts)`: Filtered noise for explosions
- `sfx.*`: Named effects (shoot, laser, hit, bomb, powerup, bossHit, etc.)
- Mute state stored in localStorage; button toggles `muted` flag

Adjust volume by changing `gain.gain.value` in `beep()` and `noiseBurst()` (lines 588–620).

## Player State Management

- `player.invuln`: Invincibility timer after hit (1.5s default)
- `player.cooldown`: Fire rate limiter (0.15s for Vulcan, 0.1s for Laser)
- `player.level`: Weapon level 1–3 (affects spread/lanes)
- `chain`/`chainTimer`/`multiplier`: Combo scoring; resets if no kills for 2.2s

## Stage & Boss Progression

- Each stage requires 18 enemy kills to trigger boss spawn
- Boss enters with 1.2s delay after kill threshold
- Boss victory spawns 500 points and advances stage
- Themes and bosses cycle: `(stage - 1) % 6`
- Stages are unlimited; difficulty increases naturally via spawn rates and enemy variety

## Performance Optimization Notes

- Particles and effects are culled aggressively based on `quality` level
- Shadow blur reduced or disabled on lower quality
- Enemy count and bullet density depend on frame rate
- Canvas clears each frame; no dirty-rect optimization (canvas is 480×720, acceptable)
- No asset loading; all rendering is procedural

## Storage Keys

- `skyfighter.bestScore`: Best score (number)
- `skyfighter.muted`: Audio mute state ('0' or '1')
- `skyfighter.difficulty`: Last selected difficulty ('easy', 'normal', 'hard')
- `skyfighter.reducedMotion`: Reduced motion preference ('0' or '1')
