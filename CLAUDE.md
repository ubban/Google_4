# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with HTML5 Canvas. The entire game is implemented as a single-file application (`index.html` — 1669 lines) containing markup, CSS, and JavaScript.

The game features:
- Player-controlled aircraft shooting enemies and bosses across multiple stages
- Three difficulty levels (Easy/Normal/Hard) affecting spawn rates and enemy behavior
- Weapon upgrade system (Vulcan: spread shots; Laser: piercing shots)
- Boss encounters with distinct attack patterns after clearing each stage
- Power-ups for health, bombs, and weapons
- Score multiplier system based on enemy kill chains
- Particle effects and screen shake
- Synthesized sound effects (no external audio assets)
- Full mobile touch controls and keyboard controls
- Accessibility features (reduced motion, screen reader support)

## Running the Game

Since this is a client-side HTML5 game with no build step:

```bash
# Open in a browser (development)
open index.html

# Or serve locally (recommended for testing)
python3 -m http.server 8000
# Then visit http://localhost:8000
```

No dependencies, no build tools, no external assets. The game runs entirely in-browser.

## Code Architecture

All game logic lives in the `<script>` section of `index.html`. The code is organized by concern:

### 1. **State Management**
- Game canvas and contexts (`canvas`, `ctx`)
- Player object: `{ x, y, w, h, weapon, level, speed, invuln, cooldown }`
- Enemies array: each with position, type, hp, shooting behavior
- Bullets and enemy bullets: collision-tested against enemies/boss/player
- Power-ups, particles, and boss object
- Game flow: `running`, `paused`, `gameOver`, `stage`, `lives`, `score`
- Difficulty settings and visual themes

### 2. **Difficulty System** (lines ~485-491)
Configurable per-difficulty:
- `spawnBase` / `spawnMin`: Enemy spawn rate (higher = slower)
- `enemySpeed`: Multiplier on enemy velocity
- `enemyShootChance`: Probability enemies fire
- `lives` / `bombs`: Starting count
- `bossHpMul`: Scales all boss health

The difficulty affects gameplay feel significantly; changes here ripple through balance.

### 3. **Enemy Spawning and Types** (lines ~502-506, 1022-1041)
- **Enemy types** (ENEMY_TYPES): `grunt`, `interceptor`, `tank` — each with distinct color, size, speed, and hp
- **Spawning logic**: Type is randomly selected based on stage availability and spawn probabilities
- **Patterns**: Each enemy either fires spread shots or aims at the player; some weave side-to-side
- Enemies are cleaned up when they leave the bottom of the screen

### 4. **Weapon System** (lines ~1124-1146)
Two weapon types with progression:
- **Vulcan** (spread): Shots fan at higher levels; good crowd control
- **Laser** (straight): Piercing shots; levels increase lanes fired from; better DPS
- Switch weapons by collecting weapon power-ups; level resets to 1 on switch
- Fire rate: Vulcan 0.15s, Laser 0.1s (line 1177)

### 5. **Boss System** (lines ~508-515, 1043-1056, 1372-1410+)
- **Boss designs** (BOSSES): 6 named bosses, each with unique color scheme and hp multiplier
- **Boss patterns** (BOSS_PATTERNS): 3 attack patterns per boss, cycled during fight
- Boss enters from top, moves side-to-side, telegraphs attacks before firing
- Patterns include: radial bursts, aimed spreads, teleport dashes, homing missiles, 8-way crosses
- Bosses appear after killing 18 enemies (KILLS_PER_STAGE); each stage uses the next boss in the list

### 6. **Collision and Damage**
- `rectsOverlap()`: AABB collision check (line 1069)
- Player invulnerability frames after being hit (line 1174)
- Bullets test against all entities; piercing shots (laser) reduce pierce count instead of deleting
- Damage: regular bullet = 1 enemy hp, 3 boss hp; bomb = kills all enemies and damages boss (lines 1115-1121)

### 7. **Particle and Visual Effects**
- **Particles**: Spawned on explosions with random velocity; fade out over time
- **Screen shake**: Called with intensity and duration; reduces adaptively at lower quality
- **Flash effects**: White flash on hit, bomb flash on bomb use (CSS animations)
- **Vignette**: Red edge vignette when player takes damage (animated)
- **Quality adaptation**: Monitors frame times and reduces particle counts and effects if FPS drops (lines 474-483)

### 8. **Audio (Web Audio Synth)**
- No external audio files; all sounds synthesized with oscillators, filters, and envelope shaping
- `beep()`: Parametric tone generator with optional frequency sweep, filter, panning
- `noiseBurst()`: Noise-based sound with envelope
- Sound pool is unlimited; synthesized on-the-fly with no pre-caching
- Master gain with compressor for level control
- Respects mute toggle and audio context suspension

### 9. **HUD and Overlay**
- Score display with multiplier
- Best score (persisted to localStorage)
- Current weapon name and level
- Lives and bombs remaining (heart/bomb icons)
- Boss health bar (shown when boss is active)
- Stage banners and combo milestone popups (animated)
- Start screen overlay with difficulty selection and controls legend
- Game over screen with final stats and new best indicator
- Screen reader live region for accessibility

### 10. **Input Handling**
- **Keyboard**: Arrow keys or WASD to move, Space to fire, X or Shift for bomb, P or Esc to pause
- **Mouse**: Move to steer, click to fire
- **Touch**: Drag left side to move, tap right side buttons for fire/bomb; visual joystick base appears when moving
- All input is polled during update, not event-driven, for frame-perfect response

### 11. **Persistence**
- Best score, mute state, difficulty, and reduced motion preference saved to localStorage
- Keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`

## Game Loop

The main game loop (lines ~1530+) runs at requestAnimationFrame:
1. **Update** (line 1148): Physics, input, spawning, collision detection
2. **Render** (line 1650+): Clear canvas, draw sky gradient + clouds, draw all entities, HUD, effects
3. **Performance tracking**: Measure frame time and adapt quality settings

Key timing constants:
- Player speed: 280 units/s (line 879)
- Bullet speeds: Vulcan 500 u/s, Laser 640 u/s (lines 1130, 1140)
- Enemy speeds: 90–180 u/s depending on type and difficulty
- Invulnerability: 2.0s per hit (line 885)
- Combo chain timeout: 2.2s (line 1089)

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line ~502) with visual and behavioral properties
2. Reference `minStage` to control when it appears
3. Spawning logic already handles random selection; update spawn probabilities if desired (line 1029)
4. Consider adding explosion sound via `EXPLODE_SFX` mapping (line 1245) if unique

### Adding a New Boss Pattern
1. Add function to BOSS_PATTERNS array (line ~1372)
2. Patterns receive the boss object and fire `enemyBullets` via helper functions (`fireRadial`, `fireAimedSpread`, etc.)
3. Use `b.patternIndex` and `frame` for timing/animation if needed
4. Test at all difficulty levels; boss hp scales, but pattern speed is fixed

### Tweaking Balance
- **Difficulty**: Edit DIFFICULTIES object (line 485)
- **Spawn rate**: `spawnBase` and `spawnMin` in difficulty (lower = faster spawns)
- **Enemy speed**: `enemySpeed` multiplier or individual type `speedMul`
- **Boss hp**: Tweak `hpMul` per boss in BOSSES, or base hp formula at line 1046
- **Weapon damage**: Player bullets = 1 hp per hit to enemies; change line 1237 to adjust
- **Bomb damage**: Boss damage at line 1118

### Adding Visual Themes
1. Add entry to `THEMES` array (line ~493) with sky gradient colors and cloud/sun tone
2. Theme applies automatically as stages progress
3. Colors are in format `'#rrggbb'` or `'r,g,b'` for CSS rgba

### Performance Optimization
- Quality levels 0 (full), 1 (medium), 2 (low) control particle counts and shadow blur
- Adaptation triggers if avg frame time exceeds 1/40s or falls below 1/56s (lines 481-482)
- Disable shadow rendering for bosses and explosions at lower quality (line 1002)
- Particle counts scale: `Math.max(3, Math.round(count * quality_factor))` (line 1074)

## Key Implementation Details

### Collision vs. Performance
Collision detection is O(n²) for bullets vs. entities (nested loops at line 1232+). With typical counts (<20 enemies, <50 bullets), this is negligible. If gameplay adds many more projectiles, consider spatial partitioning.

### Invulnerability Frames
Player.invuln is decremented each frame; loseLife() sets it to 2.0s. During invulnerability, the player is drawn semi-transparent (not collision-tested), providing visual feedback and a brief safe period.

### Chain Multiplier
Chain increments on every kill and decays if no kill occurs within 2.2s. Multiplier caps at 1 + min(9, floor(chain/5)), so max 10x at 45+ kills without break. This encourages aggressive play.

### Boss Telegraph
Boss displays white flashing core + glow before firing; controlled via `boss.telegraph` countdown. Visual cue gives player time to react.

### Touch Control UX
On touch, the joystick base element appears on first move input and follows the drag; buttons remain visible and tap-able. No hidden regions or overlapping hit areas ensure mobile playability.

## Debugging & Testing

- **Console logging**: Game state can be inspected via console (e.g., `player`, `enemies.length`, `boss.hp`)
- **Quick restart**: Click "Start Game" button in overlay; no page reload needed
- **Difficulty switching**: Select difficulty before starting; changes persist to next session
- **Reduced motion**: Toggle via button; useful for testing accessibility
- **Mute**: Toggle via button; no need to reload

## External Resources

- **Colors**: Theme palette in THEMES array; customize `sky`, `cloud`, `sun` RGB values
- **Sound effects**: All procedurally generated; see `beep()`, `noiseBurst()`, and `sfx` object methods
- **Controls hint**: Update at line 417 if input scheme changes
- **Boss names & designs**: Edit BOSSES array for new boss visuals or rebalancing
