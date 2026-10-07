# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file HTML5 Raiden-style vertical shoot-'em-up game. It runs entirely in the browser with no external dependencies — all game logic, rendering, UI, and audio synthesis are embedded in `index.html`.

## Architecture

### Game Structure
- **Single-file design** (`index.html`): ~1670 lines containing HTML structure, CSS styling, and JavaScript game engine
- **Canvas-based rendering**: 480×720px logical canvas, responsive scaling for various screen sizes
- **Game loop**: `requestAnimationFrame` at 60fps with frame-time tracking and adaptive quality scaling
- **State machine**: menu → gameplay → pause/game over → restart

### Core Systems

#### Game Loop & Timing (lines 1640–1666)
- Entry point: `loop()` function called via `requestAnimationFrame`
- Delta time calculation with 50ms cap to handle frame rate drops
- Calls `update(dt)` for logic, then `draw()` for rendering
- Performance tracking via `trackPerf()` for adaptive quality (3 levels: full, medium, low)

#### Input Handling (lines 768–861)
- **Keyboard**: Arrow keys + WASD for movement, Space for fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Direct steering (glides player toward pointer) + click to fire
- **Touch**: Left zone for joystick movement, right zone for fire/bomb buttons
- Input priority: keyboard/joystick > mouse (prevents fighting over player position)

#### Game State (lines 863–911)
- Core variables: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Score/progression: `score`, `stage`, `kills`, `stageKills` (18 kills per stage triggers boss)
- Progression: `chain` counter (increments on kills) → `multiplier` (1–10x based on chain)
- `resetGame()` initializes all state; called on start or restart

#### Update Loop (lines 1148–1302)
1. **Player movement** (1152–1172): Keyboard/joystick priority, fallback to mouse steering, clamped to canvas
2. **Weapon cooldown & firing** (1176–1181): Two rates (laser 0.1s, vulcan 0.15s)
3. **Bullet updates** (1188–1189): Move all player bullets, cull off-screen
4. **Enemy/boss spawn & update** (1197–1230): Spawn timer counts down; at 18 kills, queue boss spawn
5. **Collision detection** (1232–1294): player-enemy, bullet-enemy, bullet-boss, player-powerup
6. **Particle updates** (1296–1297): Decay life, cull dead

#### Rendering (lines 1520–1638)
- Screen shake applied via canvas translate (1523–1527)
- Sky background: gradient + sun glow + animated cloud layers (1529–1544)
- Renders all game entities: player, enemies, bullets, boss, particles, power-ups
- Glow effects conditional on `quality` level (shadow/blur disabled on low quality)

#### Audio (lines 542–676)
- **WebAudio synthesis** (no external files): oscillators for tones, noise buffers for impacts
- Lazy-initialized `audioCtx` on first user interaction
- `beep()` creates tonal sfx (sweep, filter, pan); `noiseBurst()` for explosive sounds
- `sfx` object maps events → audio functions (shoot, explode, powerup, boss hit, etc.)
- Pan calculated from bullet/hit x-position for stereo effect

### Game Mechanics

#### Weapons (lines 871–1146)
- **Vulcan** (spread shot): lv1 single, lv2 dual, lv3 triple spread
- **Laser** (piercing): lv1 single, lv2 dual, lv3 triple lanes + pierce up to 2 enemies
- Weapon cycle: upgrade level 3 → swap weapon, reset to level 1
- Power-ups randomly spawn from defeated enemies (6% life, 8% bomb, 20% weapon)

#### Enemies (lines 502–506, 1022–1041)
- **Types**: Grunt (simple), Interceptor (fast, weaving), Tank (slow, 4 HP, more score)
- Stage progression: Tanks appear at stage 2+
- Movement: Basic vertical + optional weave pattern (sine wave)
- Shooting: 2 patterns — spread (3-way) or aimed at player; rate 60–150ms intervals
- Difficulty scaling: enemy speed, shoot chance, spawn rate increase with difficulty

#### Bosses (lines 508–515, 1043–1056, 1411–1439, 1441–1454)
- **Six designs** (cycle each stage): Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan
- Each has unique HP multiplier (0.8–1.7×)
- Spawn after 18 stage kills; slow entrance, then patrol horizontally
- **Attack patterns** (6 per boss): radial bursts, aimed spreads, columns, homing, teleport, double rings
- Telegraph phase (22 frames) before each attack; visual core highlight + sound cue
- Destruction triggers: adds 500 points, advances stage, clears all enemy bullets

#### Progression (lines 485–500, 1026, 1087–1101)
- **Difficulty**: Easy (4 lives), Normal (3), Hard (3) — affects spawn rate, enemy stats, boss HP
- **Chain system**: Kills without taking damage increment chain; multiplier = 1 + floor(chain/5), capped at 10×
- Chain breaks on hit; timer resets on each kill (2.2 seconds idle to lose it)
- **Themes** (6 visual sets): cycling sky gradient, clouds, sun color per stage

#### Collision & Damage (lines 1069–1287)
- Simple AABB overlap: `rectsOverlap()` checks horizontal & vertical distance
- Player bullet → enemy: 1 HP per hit (laser can pierce 1–2 targets)
- Player bullet → boss: 3 HP per hit
- Enemy/boss bullet → player: 1 life lost, 1.5s invuln, chain breaks
- Bomb: destroys all on-screen enemies (adds 5 pts each) + deals 24 HP to boss

### UI & Effects

#### HUD (lines 34–69)
- Score, best score, multiplier, weapon name/level, hearts (lives), bombs
- Boss health bar (appears when boss spawned, tracks HP/max HP)
- Stage banner (appears for 2s on stage start/boss approach)
- Combo pop-up (shows at 10/20/30/etc kills for encouragement)

#### Visual Effects
- **Screen shake** (727–730): `screenShake(magnitude, duration)` applied to canvas context
- **Flash** (698–709): Player hit (white 0.25s flash), bomb (bright 0.5s flash)
- **Vignette** (75–87): Red pulse effect on critical health (1 life left)
- **Reduced motion**: Respects system preference; animations & effects disabled when enabled

#### Accessibility
- ARIA live region (`#srLive`) for screen reader announcements: score, boss names, stage transitions
- `announce()` function (468–471) queues text for accessibility
- Reduced motion toggle in UI; prefers-reduced-motion media query on startup
- Color contrast: white text on dark backgrounds, glowing colors for visual feedback

### State Management & Storage

**LocalStorage keys**:
- `skyfighter.bestScore`: High score persistence
- `skyfighter.muted`: Audio mute state (1/0)
- `skyfighter.difficulty`: Last selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Motion preference override (1/0)

### Responsive Design (lines 752–765)
- Base canvas: 480×720px (portrait aspect ratio)
- Scales to fit viewport, max 1.6× to avoid huge displays
- Touch controls: left 55% for joystick, right 45% for buttons
- Safe areas considered for notched/bezel devices (env constants)

## Key Code Patterns

### Particle Spawning
`spawnExplosion(x, y, color, count)` creates burst of colored particles at position; count scales with quality level. Used for enemy/boss destruction, player hit, bomb radius.

### Pattern Firing (Boss Attacks)
Boss patterns are functions that push bullet objects to `enemyBullets` array. Functions like `fireRadial()`, `fireAimedSpread()`, `fireCross()` handle geometry. Patterns stored in `BOSS_PATTERNS` array, indexed by boss design.

### Adaptive Quality
Frame times tracked in rolling 45-frame window. If average > 1/40s (dropping below 40fps), quality level increases (reducing particles, shadows, blur). If < 1/56s (well above 56fps), quality decreases to restore effects.

### Invulnerability & Knockback
`player.invuln` is a timer. While > 0, player blinks (draw skipped every other frame) and damage is ignored. Set to 1.5s on hit; allows brief safe period for player recovery.

### Enemy Weaving
Interceptor-type enemies: `weave` flag true; movement calculated as `baseX + sin(frame * 0.05 + seed) * 60`. Creates smooth sine-wave flight without storing velocity.

## Common Development Tasks

### Adding a New Enemy Type
1. Define in `ENEMY_TYPES` (line 502): w, h, hp, color, score, minStage, optional weave/speedMul
2. Add spawn condition in `spawnEnemy()` (line 1029): adjust probability thresholds
3. Add explosion sfx reference in `EXPLODE_SFX` (line 678)
4. Create or map sfx function in `sfx` object (line 624+)

### Adding a New Boss
1. Add design object to `BOSSES` array (line 508): name, colors, hpMul
2. Create attack pattern function in `BOSS_PATTERNS` array (line 1372); fires using helper functions (fireRadial, fireAimedSpread, etc.)
3. Boss cycles automatically; new bosses appear starting from next available stage

### Adding a New Weapon
1. Add weapon case in `applyPowerUp()` (1311+) and `weaponLabel()` (871)
2. Define firing logic in `fireWeapon()` (1124+): bullet spread, speed, pierce value
3. Add sound effect in `sfx` object
4. Update `fireDelay` in weapon check (1177)

### Tuning Difficulty
- Edit `DIFFICULTIES` object (line 485): spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul
- Spawn timer: `Math.max(spawnMin, spawnBase - difficultyRamp)` (line 1202)

### Adding Visual Polish
- Screen shake: call `screenShake(magnitude, duration)` in event handlers
- Flash: call `flashHit()` or `flashBomb()` on impacts
- Particles: call `spawnExplosion(x, y, color, count)`
- Sounds: call corresponding `sfx.*()` function

## Performance Considerations

- **Particle count** limited by quality level to maintain 60fps on low-end devices
- **Shadows/blur** disabled on quality level 1–2 (medium/low)
- **Canvas size fixed** (480×720) and scaled via CSS; avoids high-res rendering costs
- **Object pooling** not used; relies on array filtering (small object counts keep this fast)
- **Audio context** lazy-initialized only on first user interaction to save memory

## Testing & Running

- Open `index.html` directly in a modern browser (Chrome, Firefox, Safari, Edge)
- No build step or server required
- Test on mobile by navigating to file URL or serving via any HTTP server
- Monitor performance via browser DevTools; frame rate info in console if needed

## Notes

- All random number generation uses `Math.random()` without seeding; game is not deterministic
- Boss patterns are hardcoded per design; all bosses within a pattern family share the same attacks
- Sound is entirely synthesized; no audio files involved
- Game does not use external CDNs or dependencies
