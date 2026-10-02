# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** — A Raiden-style vertical shoot-'em-up game built with pure HTML5, Canvas API, and Web Audio synthesis. No external dependencies or build tools. Single file delivery (index.html).

## Architecture & Structure

The entire game is in `index.html` (~1669 lines), organized in sections:

### 1. **HTML Structure** (lines 1-420)
- `<canvas>` for game rendering
- HUD display (score, multiplier, weapon level, hearts, bombs, boss health bar)
- Overlay UI (menu, pause screen, game over screen)
- Touch control zones (mobile joystick, fire/bomb buttons)
- Screen effects: vignette, flash, combo popup, stage banner
- Screen reader live region for accessibility

### 2. **Inline CSS** (lines 10-358)
- Dark-themed styling with glassmorphism UI elements
- Responsive canvas scaling (480×720 base, scales with viewport)
- Touch-safe controls: large buttons, proper spacing
- Accessibility: reduced motion support, screen reader integration
- Animations: card transitions, pulse effects, screen shake, flash
- Color-coded HUD stats (yellow score, green weapon, orange multiplier)

### 3. **JavaScript Game Engine** (lines 422-1666)
All game logic runs in a single IIFE. Key systems:

#### State Management
- `player` object: position, velocity, weapon, level, cooldowns, invulnerability
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps` arrays
- Game flags: `running`, `paused`, `gameOver`
- Score tracking: `score`, `bestScore`, `chain`, `multiplier`
- Stage system: progressive difficulty, boss encounters

#### Input Handling (lines 767-861)
- **Keyboard**: Arrow keys / WASD (movement), Space (fire), X / Shift (bomb), P / Esc (pause)
- **Mouse**: Move to aim, click to fire (auto-center when idle)
- **Touch**: Drag left zone to move (joystick), tap right zone buttons to fire/bomb
- All three input methods coexist (keyboard/joystick takes priority over mouse steering)

#### Audio System (lines 542-676)
- Web Audio synth: no external audio files
- `beep()`: oscillators with frequency sweeps, filters, panning
- `noiseBurst()`: procedural explosion sounds
- 15+ sound effects (shoot, laser, explosions, powerup, boss patterns, UI clicks)
- Audio context only initialized on first user interaction
- Global mute toggle with localStorage persistence

#### Game Loop (lines 1640-1665)
- `requestAnimationFrame(loop)`: 60fps target
- `update(dt)`: physics, spawning, collision detection
- `draw()`: canvas rendering with adaptive quality
- Performance tracking: frame times → adaptive quality scaling (0 = full, 2 = low)

#### Difficulty System (lines 485-489)
- 3 presets: Easy (forgiving spawning, weak enemies, 4 lives), Normal (balanced), Hard (intense)
- Per-difficulty: `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`
- Persisted in localStorage; changeable from menu before starting

#### Stage Progression (lines 491, 1204-1207, 1441-1454)
- Kill 18 enemies → boss spawns with 1.2s warning
- 6 themed sky backgrounds (Daylight, Sunset, Night, Storm, Arctic, Volcanic) cycling per stage
- 6 boss types with unique firing patterns and HP multipliers
- Boss defeat triggers next stage with more difficult spawning

#### Enemy System (lines 502-506, 1022-1041)
- 3 enemy types with stage-gated progression:
  - `grunt`: slow, low HP (1), common, score 10
  - `interceptor`: fast, weaving, score 15 (stage 1+)
  - `tank`: slow, high HP (4), heavily armored with health bar, score 35 (stage 2+)
- AI patterns: `spread` (5-shot fan) or `aimed` (tracking the player)
- Weaving interceptors oscillate on a sine curve
- Bullet behavior: fire every 60-90 frames when canShoot flag set

#### Weapon System (lines 871-873, 1124-1146, 1304-1322)
- **Vulcan** (default): rapid-fire bullets, spray pattern at levels 2-3
- **Laser**: slower fire rate, piercing shots (can hit multiple enemies), glow aura around player
- **Power-ups**: weapon (advances level or switches weapon type), life (restore HP), bomb (add bomb)
- Player weapon level caps at 3 before switching to next weapon type

#### Boss Patterns (lines 1372-1409, 1411-1439)
Each boss has 3 attack patterns (radial, aimed, column-based variants). Pattern index cycles, telegraph phase gives 22 frames warning with visual/audio cues. Boss behaviors:
- **Vanguard**: radial bursts, aimed fan, wide V-patterns
- **Behemoth**: heavy columns, slow homing, dense radial walls
- **Specter**: teleport dashes, tight spirals, double-aimed shots
- **Dreadnought**: 8-way crosses, homing missiles, radial walls
- **Wraith**: teleport barrage, expanding double rings, cross-spreads
- **Titan**: heavy columns, full rings, relentless homing

#### Scoring & Chain (lines 1087-1101)
- Kill enemies in succession → `chain` counter increments
- Chain multiplier: 1x (0-4 kills) → 2x (5-9) → 3x (10-14) ... max 10x
- Chain timer: 2.2s decay; missing an enemy breaks the chain
- Every 10 kills shows "X KILLS!" popup

#### Collision & Damage (lines 1232-1302)
- Rectangle overlap detection (`rectsOverlap`)
- Player bullets deal 1 HP to enemies, 3 HP to bosses
- Laser bullets pierce (consume `pierce` count), regular bullets remove on hit
- Enemy contact or bullet hit: lose life, invulnerability for 1.5s, chain breaks
- Bombs clear all enemy bullets, destroy all enemies (5 pts each), damage boss (24 HP)

#### Particle System (lines 1073-1085, 1296-1297)
- Explosions spawn particles with velocity, gravity, alpha fade
- Quality-dependent particle count (high detail: 14-22 particles per explosion, low: 4-8)
- Colors vary by explosion type (enemies, player, boss, powerups)

#### Visual Effects (lines 726-730, 698-724)
- Screen shake: magnitude and duration, reduced in reduced-motion mode
- Flash overlays: hit (white 0.25s) and bomb (white 0.5s)
- Vignette pulse: red radial gradient when player at 1 HP (critical state)
- Combo popup: yellow bounce animation at screen center

#### Responsive Design (lines 752-765)
- Base canvas: 480×720 (portrait)
- Scales to fit viewport while maintaining aspect ratio (max 1.6× scaling)
- Touch controls enabled on devices with `ontouchstart` or `maxTouchPoints > 0`
- Safe area insets for notches/safe zones on mobile

#### Accessibility (lines 468-471, 549-550)
- Screen reader live region announces game events (stage clear, lives remaining, new best score)
- Reduced motion support: localStorage toggle, respects `prefers-reduced-motion` media query
- Removes animations/shakes when reduced motion enabled
- Keyboard-only playable (no mouse required)
- High contrast colors with text shadows

#### Persistence (lines 463-527)
- localStorage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- Best score persists across sessions
- User preferences (mute, difficulty, motion) remembered

## Common Development Tasks

### Run the Game
Simply open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge). No server or build step needed. Works offline.

### Making Gameplay Changes

**Difficulty rebalancing** (line 485-489):
```javascript
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  // Adjust spawnBase (enemy spawn delay in frames), enemyShootChance (0-1), lives, bombs, bossHpMul
```

**Enemy properties** (line 502-506):
```javascript
const ENEMY_TYPES = {
  grunt:       { w: 34, h: 34, hp: 1, speedMul: 1, color: '#e53935', score: 10, minStage: 1 },
  // Adjust w/h (sprite size), hp, speedMul, color, score, minStage (when enemy appears)
```

**Boss properties & patterns** (line 508-515, 1372-1409):
- Boss design colors, HP multiplier, and attack pattern functions are defined in `BOSSES` and `BOSS_PATTERNS`
- Add new boss: append to both arrays with matching index order

**Weapon balancing** (line 1124-1146):
- Adjust `fireDelay` (line 1177)
- Modify spread pattern angles or bullet velocity
- Change pierce value for lasers

**Player movement speed** (line 877):
```javascript
player = { x: ..., y: ..., speed: 260, ... };  // pixels/second
```

### Adding Visual Content

**Theme colors** (line 493-500):
- Add new theme object to `THEMES` array: sky gradient stops, cloud color, sun color
- Themes cycle per stage: `THEMES[(stage - 1) % THEMES.length]`

**Sound effects** (line 624-676):
- New SFX: add function to `sfx` object using `beep()` or `noiseBurst()`
- Call with `sfx.customSound()` where needed
- No external audio required; all procedural

**Particle effects** (line 1073-1085):
- `spawnExplosion(x, y, color, count)`: creates particles
- Adjust count or velocity spread for intensity

### Debugging

- **Console logs**: Safe to add anywhere in update/draw loop (check performance)
- **Visual debugging**: Draw debug rectangles/circles with `ctx.strokeRect()`, `ctx.arc()`
- **FPS/performance**: Frame time tracking happens automatically (quality variable shows current adaptive level)
- **Input state**: `keys` object holds keyboard state; `joyVec` holds joystick input; `firing` flag tracks fire state
- **Game state**: `running`, `paused`, `gameOver`, `player`, `score`, `stage` are all global within IIFE

### Code Organization Notes

- **No external dependencies**: All game code is self-contained
- **Canvas API**: 2D context only; no WebGL
- **Browser APIs used**: Canvas 2D, Web Audio API, localStorage, requestAnimationFrame, touch/mouse/keyboard events
- **Mobile-first**: Designed to work on phones; tested with reduced motion and touch controls
- **Accessibility first**: ARIA live regions, color contrast, keyboard navigation

## Scope for Future Changes

The game is complete and playable as-is. Typical enhancements would be:
- New enemy types or boss patterns
- Additional weapons or power-up types
- Leaderboard/score sharing (via external backend)
- Sound toggle UI refinement
- Controller/gamepad input
- Procedural level generation
- Multiplayer (would require architecture change to network code)
