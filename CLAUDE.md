# Sky Fighter — Raiden-Style Vertical Shoot-Em-Up

## Project Overview

**Sky Fighter** is a browser-based vertical shoot-em-up arcade game inspired by classic Raiden-style gameplay. The entire game is contained in a single HTML5 file (`index.html`) with no external dependencies, featuring:

- **Canvas-based rendering** with procedural graphics (no image assets)
- **WebAudio API synthesis** for all sound effects (no audio files)
- **Responsive design** optimized for desktop and mobile touch controls
- **Progressive difficulty** across 6 distinct boss encounters with themed stages
- **Adaptive performance** that scales visual quality based on frame rate

## Architecture

### Single-File Design
All code (HTML, CSS, JavaScript) lives in `index.html`. The architecture follows a clean procedural pattern:

1. **HTML structure** — Minimal semantic markup with canvas, overlay UI, and touch control zones
2. **CSS styling** — Responsive flexbox layout with animations, safe-area support for notched devices
3. **Vanilla JavaScript** — No frameworks; pure ES5+ with a self-executing IIFE wrapping all game logic

### Game Loop
- **Update phase** — Player input, entity physics, collision detection, state transitions
- **Draw phase** — Canvas rendering with adaptive quality (full/medium/low detail)
- Uses `requestAnimationFrame` for smooth 60 FPS targeting
- Delta-time tracking prevents physics bugs during frame drops

### Performance Optimization
```javascript
let quality = 0; // 0 = full, 1 = medium, 2 = low
// Adaptive system: if avg frame time > 1/40ms, reduce quality; if < 1/56ms, increase
```
- Particle counts scale by quality setting
- Shadow effects disabled on lower quality
- Glow effects skipped on medium/low

## Core Game Mechanics

### Player Ship
- **Position** — Center-bottom of screen, constrained to canvas bounds
- **Weapons** — Two interchangeable systems:
  - **Vulcan** — Rapid spread fire (3 lanes at level 3, 2 lanes at level 2, single at level 1)
  - **Laser** — Piercing fire with fewer bullets, faster shoot rate (pierce: 1-2 projectiles)
- **Invulnerability** — 1.5s after taking damage, visual blink effect
- **Health** — 3-4 hearts (Easy=4, Normal/Hard=3)
- **Movement** — Keyboard (WASD/Arrows), mouse steering (follows cursor), or touch joystick

### Enemy Types
Three types with stage-based progression:

| Type | HP | Speed | Score | Appearance | Behavior |
|------|--|---------|----|-----|----------|
| **Grunt** | 1 | 1.0x | 10 | Red (#e53935) | Straight down, spawns stage 1 |
| **Interceptor** | 1 | 1.7x | 15 | Orange (#ff7043) | Weaves side-to-side, spawns stage 1 |
| **Tank** | 4 | 0.55x | 35 | Purple (#5e35b1) | Slow but durable, spawns stage 2 |

Enemies spawn at the top with randomized horizontal positions. Shoot timers trigger spread or aimed fire patterns. Waves intensify based on elapsed time (difficulty ramp every 5 seconds).

### Boss Encounters
Six unique bosses, one per stage, with distinct attack patterns:

1. **VANGUARD** — Radial burst, aimed fan, V-fan spreads
2. **BEHEMOTH** — Heavy columns, homing spreads, dense radial walls
3. **SPECTER** — Teleport dashes, spiral bursts, dual-aimed shots
4. **DREADNOUGHT** — 8-way cross fire, homing missiles, radial walls
5. **WRAITH** — Teleport barrage, double-ring fire, cross-spreads
6. **TITAN** — Relentless columns, full rings, heavy homing

Each boss:
- Enters screen from top, pauses in center for battle
- Cycles attack patterns with 22-frame telegraph (flashing core)
- Has stage-based HP: `(70 + stage * 35) × difficulty × boss.hpMul`
- Moves horizontally, bouncing off edges
- Spawns 40-particle explosion when defeated

### Scoring System
- **Base points** — Grunts=10, Interceptors=15, Tanks=35, Boss=500
- **Combo multiplier** — `1 + floor(chain / 5)`, capped at 10x
- **Chain mechanic** — Kill resets 2.2s timer; miss breaks chain to 1x
- **Kill milestones** — Displays "10 KILLS!" pop-up every 10 kills
- **Stage progression** — After 18 kills, boss warning appears; boss spawns after 1.2s

### Power-Ups
Drop randomly from enemy deaths (6% life, 8% bomb, 20% weapon):

| Type | Effect | Color |
|------|--------|-------|
| **Life** | +1 HP (max capped) | Red (#ff5252) |
| **Bomb** | +1 smart bomb (max capped) | Gold (#ffc107) |
| **Weapon** | Level up weapon; at level 3, switch to alternate weapon & reset to level 1 | Lime (#7CFC00) |

### Smart Bombs
- **Count** — 2-3 depending on difficulty
- **Effect** — Clears all enemy bullets instantly + kills all on-screen enemies (adds 5 points each) + damages boss (24 HP)
- **Audio** — Distinctive multi-tone explosion sound
- **Screen effect** — Full-screen white flash

### Difficulty Modes
Three presets affecting spawn rates, enemy aggressiveness, lives, and boss HP:

| Mode | Spawn Rate | Enemy Speed | Shoot Chance | Lives | Bombs | Boss HP |
|------|-----------|-------------|--------------|-------|-------|---------|
| **Easy** | 75-40 frames | 1.0x | 45% | 4 | 3 | 0.8x |
| **Normal** | 58-26 frames | 1.4x | 70% | 3 | 2 | 1.0x |
| **Hard** | 40-18 frames | 1.9x | 95% | 3 | 2 | 1.3x |

### Visuals

#### Themes
Six procedurally-generated sky themes (one per stage, looping):
1. **Daylight Front** — Blue gradient, white clouds
2. **Sunset Corridor** — Purple-orange gradient, warm tones
3. **Night Approach** — Dark blue, cool highlights
4. **Storm Front** — Green-gray, dramatic weather
5. **Arctic Wastes** — Cyan gradient, frost effects
6. **Volcanic Ridge** — Red-orange gradient, intense heat

Each theme has custom sky colors, cloud tints, and sun gradients that shift during play.

#### Visual Effects
- **Screen shake** — Intensity and duration based on events (bomb=14px for 0.4s)
- **Flash effects** — Hit (white fade 0.25s) and bomb (white pulse 0.5s)
- **Vignette** — Red pulsing border when health = 1 (critical state)
- **Combo pop** — Gold text with drop-shadow, animates at screen center
- **Stage banner** — Large text announcing stage transitions

### Input Handling
**Desktop:**
- Arrow keys or WASD — Move ship
- Mouse — Steer toward cursor (lower priority than keyboard)
- Space or Mouse Click — Fire
- X or Shift — Drop bomb
- P or Esc — Pause

**Mobile:**
- Left 55% of screen — Virtual joystick (drag to move)
- Right 45%, bottom — Two buttons (FIRE, BOMB)
- Tap fire/bomb for one-shot, hold fire for continuous

## UI & UX

### HUD (Always visible)
- **Score** — Current session score
- **Best** — High score (persisted in `localStorage`)
- **Multiplier (x)** — Current combo level
- **Weapon stat** — Name (VULCAN/LASER) and level (1-3)
- **Hearts** — Visual health indicator (filled/empty)
- **Bombs** — Visual bomb counter (filled/dimmed)

### Overlays
- **Title screen** — Game intro, difficulty selector, controls hint
- **Pause screen** — Resume button (P/Esc to toggle)
- **Game Over** — Final score, stage reached, new best score badge
- **Boss bar** — Displays boss name and health (appears when boss spawns)

### Accessibility
- **Screen reader announcements** — Score changes, boss defeated, lives remaining
- **Reduced motion mode** — Toggle via button; disables animations for users preferring `prefers-reduced-motion`
- **Mute toggle** — Silences all audio without affecting game logic
- **Safe area support** — Respects notches/safe zones on modern phones

## Technical Details

### WebAudio Synthesis
No external audio files. All sounds generated via oscillators and noise bursts:

- **Beep synth** — Frequency sweep with envelope (attack/decay/release)
- **Noise burst** — Procedural white noise with filter (lowpass/highpass)
- **Spatial audio** — Pan effects based on bullet x-position for stereo separation
- **Master gain** — Compressor + master volume (0.9) to prevent clipping

Example:
```javascript
beep(760, 0.07, 'square', 0.045, { sweepTo: 480, filterType: 'lowpass', filterFreq: 2600 })
// 760Hz square wave for 70ms, sweep down to 480Hz, lowpass filter at 2.6kHz
```

### Canvas Scaling
- **Internal resolution** — 480×720 (portrait, 2:3 aspect ratio)
- **Max scale** — 1.6x of internal size
- **Responsive** — Scales to fit window while maintaining aspect ratio
- **Touch-action** — Disabled to prevent browser gesture interference

### Persistence
Game state persists via `localStorage`:
- `skyfighter.bestScore` — High score
- `skyfighter.muted` — Audio mute toggle
- `skyfighter.difficulty` — Selected difficulty mode
- `skyfighter.reducedMotion` — Motion preference override

### Entity Management
**Arrays of entity objects:**
```javascript
const player = { x, y, w, h, speed, weapon, level, invuln, cooldown, rapid };
const bullets = [{ x, y, w, h, speed, vx, pierce, laser }];
const enemies = [{ x, y, type, hp, speed, shootTimer, canShoot, pattern, ... }];
const enemyBullets = [{ x, y, speed, vx, radial, dx, dy }];
const boss = { x, y, w, h, hp, maxHp, design, entering, phaseTimer, attackTimer, patternIndex, vx, telegraph };
const powerUps = [{ x, y, type, speed, w, h }];
const particles = [{ x, y, vx, vy, life, maxLife, color }];
```

## Development & Extending

### Adding a New Boss
1. Add entry to `BOSSES` array with colors and HP multiplier
2. Create attack pattern array in `BOSS_PATTERNS` (functions that call `fireRadial`, `fireAimedSpread`, etc.)
3. Patterns are auto-assigned by boss stage index

Example:
```javascript
const BOSSES = [
  // ... existing bosses
  { name: 'PHOENIX', hullTop: '#ff6600', hullBottom: '#ff3300', wing: '#ff3300', core: '#ffaa00', hpMul: 1.4 }
];

const BOSS_PATTERNS = [
  // ... existing patterns
  [ // PHOENIX patterns
    b => fireRadial(b, 16, 160),
    b => { fireColumns(b, 240); fireAimedSpread(b, 3, 280, 0.15); },
    b => fireDoubleRing(b, 150, 240)
  ]
];
```

### Adding a New Theme
```javascript
const THEMES = [
  // ... existing themes
  { sky: ['#0a0a2e', '#16213e', '#0f3460'], cloud: '220,230,240', sun: '200,210,255', name: 'Deep Blue' }
];
```

### Tweaking Difficulty
Edit `DIFFICULTIES` object. Spawn timer decreases over time based on difficulty ramp:
```javascript
const difficultyRamp = Math.floor(frame / 300) * 4; // -4 frames per 5s
spawnTimer = Math.max(d.spawnMin, d.spawnBase - difficultyRamp);
```

### Adding Sound Effects
```javascript
const sfx = {
  newSound: () => {
    beep(440, 0.2, 'sine', 0.1, { sweepTo: 220, filterFreq: 3000 });
    noiseBurst(0.1, 0.08, { filterType: 'highpass', filterFreq: 5000 });
  }
};
```

## Performance Notes

- **Particle limits** — Capped at 3, 6, or 14 count per explosion based on quality
- **Shadow rendering** — Only at quality level 0 (full detail)
- **Glow effects** — Disabled at quality 1+ (medium/low)
- **Frame rate target** — 60 FPS; adaptive scaling kicks in if sustained <40 FPS or >56 FPS

## Known Limitations

1. **Single-file constraint** — No module system; all code in IIFE namespace
2. **No asset pipeline** — Graphics and audio generated on-the-fly (suitable for small scope)
3. **Touch controls** — Simple directional joystick; no multi-touch gestures
4. **Boss HP display** — Bar updates every frame but not sub-pixel smooth at very high DPI

## Future Enhancement Ideas

- Multi-stage campaign with checkpoint saves
- Power-up shop between stages (spending points for ship upgrades)
- Daily challenge mode with fixed seeds
- Leaderboard integration (localStorage or external API)
- Custom game speed slider (0.5x-1.5x)
- Ship skins and weapon appearance customization
- Tutorial stage with difficulty ramp-up

---

**Last Updated:** October 2026
**Author:** Sky Fighter Team
