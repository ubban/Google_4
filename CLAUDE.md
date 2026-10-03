# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML application. The entire game engine, rendering, input handling, and audio synthesis runs in a single `index.html` file with no external dependencies.

### Key Characteristics

- **Single-file architecture** — all game code, styles, and HTML in `index.html`
- **Canvas 2D rendering** — no WebGL or graphics libraries
- **Procedural audio** — WebAudio API for sound synthesis, no audio files
- **Responsive design** — supports desktop (mouse/keyboard) and mobile (touch) controls
- **Difficulty settings** — Easy, Normal, Hard with different enemy spawn rates and parameters
- **Progressive stages** — boss fights after 18 enemy kills per stage
- **Six unique bosses** — each with distinct attack patterns

## Game Architecture

### Core Systems

The game operates as a classic game loop managing these key subsystems:

#### 1. **Player & Input System** (lines 768-861)
- **Input modes**: Keyboard (WASD/Arrows), Mouse (aim + click), Touch (drag to move, buttons to fire/bomb)
- **Player object**: Position, velocity, weapon type/level, cooldown, invulnerability timer
- **Weapon types**: VULCAN (rapid-fire bullets) and LASER (piercing shots, limited spray)
- **Mouse steering** — plane glides toward pointer; keyboard input always takes priority
- **Touch joystick** — appears on left side for movement; buttons on right for fire/bomb

#### 2. **Enemy System** (lines 502-506, 1200-1228)
- **Three enemy types**: grunt (weak), interceptor (fast + weaving), tank (heavy)
- **Difficulty-based spawning** — base spawn timer decreases as stage progresses
- **Behavior**: Moving downscreen, shooting at player, weaving patterns on some types
- **Collision**: Can damage player on contact; killed by player bullets
- **Spread pattern** — Tank can shoot in 3-bullet spreads; others aim directly at player

#### 3. **Boss System** (lines 508-514, 1372-1434)
- **Six bosses** with themed designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- **Trigger**: Spawns after 18 enemies killed per stage
- **Telegraph system** — 22-frame delay before each attack pattern fires
- **Boss attacks** — 3 distinct patterns per boss (radial bursts, aimed spreads, columns, homing, teleport combos)
- **Entry animation** — boss slides in from top
- **Health bar** — displayed at top during boss fight

#### 4. **Collision & Physics** (lines 1232-1294)
- **Axis-aligned bounding boxes** — `rectsOverlap(obj1, obj2)` checks rectangles
- **Bullet-enemy** — bullets pierce if laser weapon, otherwise consumed; enemies lose 1 HP
- **Bullet-boss** — boss takes 3 damage per hit
- **Enemy-player** — direct contact damages player and enemy dies
- **Enemy bullet-player** — hits can damage player if not invuln
- **Power-up pickup** — collects weapons, bombs, lives

#### 5. **Scoring & Progression** (lines 863-868)
- **Score multiplier** — starts at 1x, increases with kill chains
- **Kill chain** — consecutive kills within timer window; breaks on damage
- **Stage progression** — beat 18 enemies → boss fight → next stage (up to 6 theme variations)
- **Best score** — saved to localStorage, displayed on game-over

#### 6. **Audio System** (lines 542-676)
- **WebAudio synthesis** — oscillators (square, sine, sawtooth, triangle), noise bursts, filters
- **Procedural SFX** — unique sounds for shoot, hit, bomb, power-up, boss effects
- **Panning** — bullets and explosions use stereo panning based on screen position
- **Mute toggle** — persisted in localStorage

### Game Loop & Update Flow

**Main loop** (lines ~1180-1302):
1. Handle input and update player position/firing
2. Update enemy spawning (timer-based, difficulty-scaled)
3. Update all moving entities (bullets, enemies, boss)
4. **Collision detection** — bullets vs enemies/boss, enemies/bullets vs player, power-ups vs player
5. **Physics updates** — particles decay, screen shake applies
6. **Rendering** — draw background, enemies, bullets, particles, UI

**Timing**: Uses `requestAnimationFrame` with delta-time (`dt`) for frame-rate independent updates.

### Constants & Configuration

**Difficulty settings** (lines 485-489):
```javascript
DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
}
```

**Canvas size** (lines 753): 480×720 base resolution (scales to fit window)

**Theme colors** (lines 493-500): Six sky themes with different gradients and lighting

**Enemy types** (lines 502-506): hp, speed, score values, appearance conditions

**Boss patterns** (lines 1372-1409): Attack sequences per boss (radial bursts, aimed spreads, teleport + barrage, etc.)

## Development Guide

### Running the Game

**Development mode**: Open `index.html` in a browser (any HTTP or `file://` URL).
- No build step, no dev server needed
- Changes to code take effect on page reload
- Use browser DevTools console for debugging

**Testing locally**:
```bash
# Option 1: Simple HTTP server (Python 3)
python -m http.server 8000
# Visit http://localhost:8000/index.html

# Option 2: Using Node (if available)
npx http-server

# Option 3: Open directly in browser
open file:///path/to/Google_4/index.html
```

### Code Organization (within index.html)

- **Lines 1-359**: CSS styling (canvas, UI overlays, animations, responsive layout)
- **Lines 422-676**: Game configuration, audio system, helper functions
- **Lines 768-861**: Input handling (keyboard, mouse, touch)
- **Lines 863-899**: Game state initialization and reset
- **Lines 1100+**: Main game loop (`gameFrame()`), entity updates, collision detection
- **Lines 1500-1669**: Rendering functions, drawing enemies/bullets/particles/UI

### Common Development Tasks

#### Adding a New Enemy Type
1. Add object definition to `ENEMY_TYPES` (lines 502-506) with hp, speed, score, appearance conditions
2. Add spawn logic to `spawnEnemy()` based on `stage` and `Math.random()`
3. Add matching SFX to `sfx` object with key in `EXPLODE_SFX` map
4. Update collision detection if new behavior needed

#### Modifying Boss Patterns
- Each boss has 3 attack patterns in `BOSS_PATTERNS` array (lines 1372-1409)
- Pattern functions use helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.
- To change: Edit the pattern array at index corresponding to boss order in `BOSSES`
- Boss pattern index cycles through 3 patterns, then repeats

#### Adjusting Difficulty
- Edit `DIFFICULTIES` object (lines 485-489) to change spawn rates, enemy speed, player lives
- `spawnBase` — initial spawn timer (larger = slower spawning)
- `spawnMin` — minimum spawn timer as game progresses (harder difficulty = lower min)
- `enemySpeed` — bullet speed multiplier
- `lives` / `bombs` / `bossHpMul` — per-difficulty resource parameters

#### Adding Visual Effects
- **Particles**: `spawnExplosion(x, y, color, count)` — creates burst of colored dots
- **Screen shake**: `screenShake(magnitude, duration)` — applies jitter to canvas
- **Flash**: `flashHit()` or `flashBomb()` — white flash overlay
- **Combo text**: `showComboPop(text)` — floating text at center
- **Stage banner**: `showStageBanner(text)` — large text announcement

#### Playing Sounds
- Call `sfx.soundName()` from `sfx` object (lines 624-676)
- Procedural sounds use `beep(freq, duration, type, volume, opts)` and `noiseBurst()`
- Options: `sweepTo`, `filterFreq`, `filterType`, `pan`, `delay`

### Testing Checklist

**Gameplay**:
- [ ] Enemies spawn at expected rate per difficulty
- [ ] Player can steer and fire with mouse, keyboard, and touch
- [ ] Bullets hit enemies and bosses; enemies are destroyed or lose HP
- [ ] Boss spawns after 18 kills and uses correct attack pattern
- [ ] Power-ups appear after random enemy kills and upgrade player
- [ ] Bombs clear bullets on-screen and can kill weak enemies
- [ ] Lives and game-over state work correctly
- [ ] Score multiplier increases with kill chains, resets on damage
- [ ] Stages progress and theme changes

**UI & Settings**:
- [ ] Mute button toggles sound (persists on reload)
- [ ] Reduced motion option disables animations
- [ ] Difficulty buttons work (can't change mid-game)
- [ ] Best score displays and persists
- [ ] Touch controls appear on mobile, hidden on desktop

**Edge Cases**:
- [ ] Multiple bosses don't spawn at once
- [ ] Player invulnerability prevents rapid re-hits
- [ ] Boss health bar updates smoothly
- [ ] Screen shake disabled in reduced-motion mode
- [ ] Canvas scales correctly on resize/orientation change

### Performance Considerations

- **Adaptive quality** (lines 473-483) — if frame time exceeds 1/40th second consistently, quality setting degrades
- **Particle limits** — explosion particles are capped to avoid memory buildup
- **Enemy limits** — off-screen enemies are culled (`enemies.filter(en => en.y <= canvas.height + 30)`)
- **Canvas size** — fixed at 480×720 to keep fill rate predictable

### Debugging Tips

In browser console:
```javascript
// Spawn a boss immediately (skip 18 kills)
stageKills = 18;

// Skip to next stage
stage++;
spawnBoss();

// Modify player lives
lives = 5; renderHearts(5, maxLives);

// Disable enemy spawning
spawnTimer = Infinity;

// Instant weapon upgrade
player.level = 3; player.weapon = 'laser';
```

## Browser Compatibility

- **Requires**: Canvas 2D, WebAudio API, localStorage
- **Tested**: Chrome, Firefox, Safari (desktop & mobile)
- **Mobile**: Touch input works on iOS and Android; audio context requires user gesture (click to start)
- **Fallback**: Game continues if WebAudio fails; audio just won't play

## Storage

Settings stored in localStorage under these keys:
- `skyfighter.bestScore` — high score
- `skyfighter.muted` — audio mute state
- `skyfighter.difficulty` — selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion` — accessibility flag (0 or 1)

## Future Enhancement Ideas

- Leaderboard integration
- Ship customization (appearance skins)
- Additional weapon types or power-ups
- Persistent upgrades between plays
- Sound settings (volume slider)
- Tutorial/help overlay on first play
