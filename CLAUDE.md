# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The game features:
- Multiple enemy types with different behaviors
- Boss encounters at the end of each stage
- Weapon switching system (Vulcan + Laser)
- Score multiplier chain mechanics
- Touch and keyboard controls
- WebAudio-based sound synthesis (no external audio files)
- Adaptive rendering quality based on frame rate performance
- Three difficulty levels (Easy/Normal/Hard)
- Accessibility features (reduced motion support, screen reader friendly)

## Development Commands

Since this is a single-file HTML game with no build process:

- **Run the game**: Open `index.html` in a web browser
- **Local server** (if needed): `python3 -m http.server 8000` then open `http://localhost:8000`
- **Testing**: Use the browser's DevTools console to check for errors
- **Performance profiling**: Use Chrome DevTools > Performance tab to analyze frame timing

## Architecture Overview

### Code Organization (Single File)

The `index.html` file contains three main sections:

1. **HTML & DOM Elements** (lines 361–420): Game canvas, HUD displays, overlay screens, touch controls, accessibility elements
2. **CSS Styling** (lines 10–359): Responsive design with clamp() for scaling, dark theme, animations (pulse, flash, combo pop), touch button layout
3. **JavaScript Game Logic** (lines 422–1669): All game mechanics

### JavaScript Subsystems

#### 1. **Initialization & State** (lines 423–523)
- Global constants: `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`
- Local storage keys for persisted settings (best score, mute state, difficulty, reduced motion preference)
- Canvas sizing and responsive behavior (base 480×720, scales to fit window)

#### 2. **Audio System** (lines 542–678)
- **WebAudio Synthesis**: No external audio files; all sounds generated with oscillators and noise bursts
- `beep(freq, dur, type, vol, opts)`: Synthesize tonal sounds with optional frequency sweep, filtering, panning
- `noiseBurst(dur, vol, opts)`: Generate explosive/impact sounds
- `sfx` object: Pre-composed sound effects for shooting, explosions, boss hits, power-ups, etc.
- Audio context lazy-initialized on first user interaction (unlocks AudioContext)
- Master gain with dynamics compression for consistent loudness

#### 3. **Input Handling** (lines 767–861)
- **Keyboard**: Arrow keys / WASD for movement, Space/Left Shift/X for actions, P/Esc for pause
- **Mouse**: Click to fire, move mouse to steer player (when hovering canvas)
- **Touch**: Joystick in left half of screen for movement, Fire/Bomb buttons in right half
- Input priority: keyboard/joystick > mouse (mouse is secondary steering aid)
- Mobile detection via `ontouchstart` or `navigator.maxTouchPoints`

#### 4. **Game State & Loop** (lines 863–1100+)
- **Core state**: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- **Game variables**: `score`, `lives`, `stage`, `stageKills`, `multiplier`, `chain`, `paused`, `running`
- **Rendering**: All drawing via Canvas 2D API; no external graphics
- **Game loop**: Likely in `update()` and `draw()` functions (RAF-based)

#### 5. **Entity Systems**
- **Player** (line 877): Position, speed (260 px/s), weapon (vulcan/laser), level, invulnerability timer
- **Enemies** (line 1022–1041): Spawn with type (grunt/interceptor/tank), shooting behavior, weaving patterns
- **Bosses** (line 1043–1056): Per-stage design, entering phase, attack patterns, telegraph effects
- **Power-ups** (line 1058–1067): Life, bomb, weapon upgrades (6% life, 8% bomb, 20% weapon drop rates)
- **Particles**: Explosion debris with velocity and life timers (quality-scaled)

#### 6. **Gameplay Mechanics**
- **Chain Multiplier** (line 1087–1094): Killing enemies within 2.2s increases chain count; multiplier = 1 + floor(chain/5), max ×10
- **Stage Progression** (line 491): Kill 18 enemies per stage to trigger boss; boss defeat advances stage
- **Difficulty Scaling**: Affects spawn rates, enemy speed/aggression, boss HP, starting lives/bombs
- **Weapon System**: Vulcan fires rapid shots; Laser fires slower, more powerful shots; level affects damage/spread
- **Screen Effects**: Shake (mag, duration), flash (hit/bomb animations), vignette pulse (critical health)

#### 7. **Rendering & Performance** (lines 472–483)
- **Adaptive Quality**: Tracks frame times over 45 frames; adjusts `quality` level (0=full, 1=medium, 2=low)
- `quality` affects: particle count, boss glow/shadow blur, canvas detail
- All drawing is procedural (no textures); colors defined in constants and applied via gradients
- **Canvas coordinates**: 480×720 base; scales via CSS for responsive design

## Key Data Structures

### Difficulty Profiles
```javascript
DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
}
```

### Enemy Types
```javascript
ENEMY_TYPES = {
  grunt:       { w: 34, h: 34, hp: 1, speedMul: 1,    color: '#e53935', score: 10, minStage: 1 },
  interceptor: { w: 26, h: 26, hp: 1, speedMul: 1.7,  color: '#ff7043', score: 15, minStage: 1, weave: true },
  tank:        { w: 46, h: 46, hp: 4, speedMul: 0.55, color: '#5e35b1', score: 35, minStage: 2 }
}
```

### Boss Design (6 bosses, cycling per stage)
```javascript
BOSSES = [
  { name: 'VANGUARD',    hullTop: '#8e24aa', hullBottom: '#4a148c', core: '#ff6ec7', hpMul: 1.0 },
  { name: 'BEHEMOTH',    hullTop: '#ff8f00', hullBottom: '#bf360c', core: '#ffd54f', hpMul: 1.35 },
  // ... 4 more
]
```

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502–506) with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Update `spawnEnemy()` (line 1026) to include new type in spawn probability
3. Add explosion SFX to `EXPLODE_SFX` lookup (line 678) if desired

### Adding a New Boss
1. Add entry to `BOSSES` array (line 508–515) with design colors and `hpMul`
2. Boss graphics are rendered in `drawBossShip()` (line 970–1020); procedurally drawn from hull/wing/core colors
3. Boss behavior is in main update loop (search for `boss.` to find boss logic)

### Adjusting Weapon Balance
- Vulcan: Fast-firing, lower damage per shot (search `weapon === 'vulcan'` in update loop)
- Laser: Slower rate, higher damage (search `weapon === 'laser'`)
- Modify spawn rate of weapon power-ups in `maybeSpawnPowerUp()` (line 1062–1063)

### Tweaking Difficulty
- Edit `DIFFICULTIES` constants (line 485–489)
- `spawnBase`/`spawnMin`: Enemy spawn timing (lower = more frequent)
- `enemySpeed`: Multiplier on enemy velocity
- `enemyShootChance`: Probability enemy will shoot each frame
- `lives`/`bombs`: Starting resources
- `bossHpMul`: Scales base boss HP

### Adding a New Stage Theme
- Add to `THEMES` array (line 493–500) with `sky` gradient stops, `cloud` color, `sun` color, `name`
- Themes cycle through stages and are applied in the render loop (search `THEMES[` in draw code)

## Performance Considerations

- **Quality Scaling**: Monitor `quality` variable; particle effects and shadow blur automatically reduce on low-end devices
- **Canvas Size**: Base is 480×720; scales via CSS. Avoid increasing resolution if performance is a concern
- **Audio**: WebAudio synth is lightweight, but avoid stacking many sounds simultaneously (handled by individual `beep()` calls with short durations)
- **Collision Detection**: Uses simple AABB rectangles via `rectsOverlap()` (line 1069–1071)

## Testing & Debugging

- **Console logging**: Add `console.log()` around critical updates (e.g., enemy spawn, collision detection)
- **DevTools**: Check Network tab for missing assets (there shouldn't be any external files), Performance tab for frame drops
- **Screen reader testing**: Check `srLive` element (line 390) for accessible announcements
- **Mobile testing**: Use Chrome DevTools device emulation or physical mobile device to test touch controls and performance

## Local Storage Keys

Game settings persisted across sessions:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio muted (0/1)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Accessibility setting (0/1)

## Accessibility

- **Keyboard**: All controls have keyboard shortcuts (no mouse-only actions)
- **Screen Reader**: Game status announced via `srLive` element and `announce()` function
- **Reduced Motion**: Respects OS preference; users can toggle via button; disables shake, flash, animations
- **Color Contrast**: Dark theme with bright text and UI elements for visibility
