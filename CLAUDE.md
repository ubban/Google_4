# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML application with embedded CSS and JavaScript. The game runs entirely in the browser with no external dependencies or build tools.

- **Type**: Browser-based canvas game
- **Technology**: HTML5, Canvas 2D, Web Audio API
- **Entry Point**: `index.html`
- **Deployment**: Serve the HTML file directly

## Quick Start

To play or test the game:
1. Open `index.html` in a web browser
2. Select difficulty (Easy/Normal/Hard)
3. Click "Start Game"

No build process or installation required.

## Architecture Overview

The game code is organized into logical sections within the `<script>` tag:

### Core Game State (line ~864)
- `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss` objects
- `score`, `lives`, `stage`, `chain`, `multiplier`, `frame`
- Single game loop via `requestAnimationFrame`

### Major Systems

**1. Input Handling (line ~767)**
- Keyboard: Arrow keys/WASD for movement, Space for fire, X/Shift for bomb, P/Esc to pause
- Mouse: Movement steers the player, click to fire
- Touch: Joystick movement on left half, Fire/Bomb buttons on right half
- Input priority: Keyboard/Joystick > Mouse (mouse used as fallback only)

**2. Audio System (line ~542)**
- Web Audio synth (no external audio files)
- SFX for all game events: weapons, explosions, power-ups, boss attacks
- Master gain and compressor for dynamic range control
- Pan effect for spatial audio based on bullet x-position
- Mute toggle persisted to localStorage

**3. Difficulty System (line ~485)**
Three presets affecting spawn rate, enemy speed, shooting probability, boss health:
- `easy`: Slower enemies, fewer bullets, 4 lives
- `normal`: Baseline difficulty (default), 3 lives
- `hard`: Fast enemies, aggressive shooting, 3 lives with reduced bomb count
- Selection persisted to localStorage

**4. Enemy & Boss Spawning (line ~1022, ~1043)**
- **Enemy Types**: grunt (weak), interceptor (fast/weaving), tank (tanky/multi-hit)
- Enemy types unlock progressively with stage progression
- Spawn rate accelerates as frame count increases
- **Boss Patterns**: 6 boss designs (VANGUARD→TITAN), each with 3 unique attack patterns
- Boss telegraph mechanic with 22-frame warning before attack
- Bosses enter from top before engaging in combat

**5. Collision Detection (line ~1069)**
- Rectangle overlap (`rectsOverlap`) for all collision types
- Player-enemy, player-bullet, player-boss, bullet-enemy, bullet-boss

**6. Scoring & Chain System (line ~1087)**
- Base score per kill multiplied by chain multiplier (1-10x)
- Chain increases with each kill, resets after 2.2s inactivity
- Multiplier capped at 10x (1 + floor(chain/5))
- Boss kills: flat 500 points (not multiplied)

**7. Weapon System (line ~1124)**
- **Vulcan**: Horizontal spread at level 3, higher fire rate
- **Laser**: Pierces enemies (limited), narrower projectiles, slower fire rate
- Weapon upgrade drops: level up weapon or switch to laser at level 3
- Bullet pierce mechanics prevent one shot from clearing entire screen

**8. Power-ups (line ~1058)**
- `life`: Restores one heart (max=3)
- `bomb`: Restores one bomb (max varies by difficulty)
- `weapon`: Advances weapon level or switches weapon type
- Spawn chance tied to enemy kill (6% life, 8% bomb, 20% weapon)

**9. Boss Attack Patterns (line ~1372)**
Each of 6 bosses has 3 distinct attack patterns:
- Radial bursts, aimed spreads, homing projectiles, telegraphing, teleport attacks
- Pattern selection cycles: pattern index increments after each telegraph
- Projectiles use `radial` property to distinguish from player bullets

**10. Performance Adaptation (line ~473)**
- Tracks frame time over last 45 frames
- Reduces quality (particle count, glow effects) if avg frame time > 25ms
- Increases quality if avg frame time < ~17.9ms
- Quality levels: 0 (full), 1 (medium), 2 (low)

**11. Rendering (line ~1520)**
- Screen shake applied to canvas context translate
- Sky gradient + sun glow + animated clouds per theme
- Player plane rendering with engine flame
- Boss ships with animated parts and telegraph glow
- Particle effects for explosions
- Health bars for tanks and bosses

**12. Theme System (line ~493)**
Six alternating themes tied to stage progression:
- Each theme: sky gradient colors, cloud color, sun color, theme name
- Cycles every 6 stages

### UI & Accessibility

**HUD Elements (line ~367)**
- Score, best score, combo multiplier, weapon level
- Heart display (shows lost hearts as faded)
- Bomb count display
- Boss health bar (appears when boss active)
- Combo pop-up animation ("X KILLS!")
- Stage banners

**Accessibility Features**
- Screen reader live region (`#srLive`) for game events
- Reduced motion toggle: disables animations/shakes/flashes when enabled
- Respects system `prefers-reduced-motion` setting
- Touch-accessible buttons and controls

**Settings Persisted to localStorage**
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Reduced motion toggle

### Responsive Design

- Base resolution: 480×720px
- Canvas scales proportionally to fit window (max 1.6x)
- Responsive UI (clamp font sizes, safe area insets for notch devices)
- Touch zone layout: 55% left half for movement, 45% right half for actions

## Key Concepts for Modification

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Add spawn logic to `spawnEnemy()` filtering by `minStage` and probability
3. Add explosion sound effect name to `EXPLODE_SFX` if needed

### Adding a New Boss
1. Add entry to `BOSSES` array with design colors and hpMul
2. Add attack pattern array to `BOSS_PATTERNS` with 3 attack functions
3. Boss index in `BOSS_PATTERNS` must match `BOSSES` array index
4. Attack functions use helper functions like `fireRadial()`, `fireAimedSpread()`, etc.

### Adjusting Difficulty
Edit `DIFFICULTIES` object:
- `spawnBase` / `spawnMin`: Control enemy spawn rate (lower = more enemies)
- `enemySpeed`: Multiplier for all enemy movement
- `enemyShootChance`: Probability enemy can shoot (0-1)
- `lives`, `bombs`: Starting values
- `bossHpMul`: Multiplier for boss health

### Adding New Audio
1. Add function to `sfx` object (line ~624) using `beep()` or `noiseBurst()` helpers
2. Call via `sfx.functionName()` in game logic
3. Web Audio synth parameters: freq, duration, type, volume, sweepTo, filter, pan

### Visual Customization
- Theme colors in `THEMES` array (sky gradients, clouds, sun)
- Enemy colors in `ENEMY_TYPES` (color + dark variant)
- Boss hull/core colors in `BOSSES`
- Particle colors in explosion calls (`spawnExplosion()`)

## Common Modification Patterns

**Weapon Balancing**
- Fire rate: adjust `fireDelay` in update() (~line 1177)
- Spread angle: modify spread arrays in `fireWeapon()`
- Pierce count: change `pierce` value when creating bullets

**Enemy Tuning**
- Movement: adjust `speed` calculation in `spawnEnemy()`
- Shooting: change `shootTimer` threshold (~60-120 frame range)
- Weave pattern: modify sine amplitude in enemy update loop (~line 1213)

**Scoring Tweaks**
- Base points per enemy: modify `score` field in `ENEMY_TYPES`
- Multiplier formula: adjust line 1090 (`1 + Math.min(9, Math.floor(chain / 5))`)
- Chain timeout: modify `chainTimer` duration (currently 2.2s at line 1089)

**Boss Difficulty**
- Pattern frequency: change `attackTimer` values in `BOSS_PATTERNS` functions
- Projectile speed: modify speed parameters in firing functions
- Pattern complexity: add/remove bullets in pattern functions

## Testing Notes

- Game is fully self-contained; no server needed
- Test all difficulties to verify balance changes
- Check mobile layout by resizing browser to phone dimensions
- Verify audio on multiple browsers (some older browsers may not support Web Audio)
- Performance degrades gracefully: particle effects and glow reduce automatically
- Reduced motion toggle should disable all animations without breaking gameplay
