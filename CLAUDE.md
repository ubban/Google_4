# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely with HTML5 Canvas and Web Audio API. The entire game is contained in a single `index.html` file with no external dependencies or build process.

**Key Features:**
- Desktop (keyboard/mouse) and mobile (touch) controls
- Three difficulty levels: Easy, Normal, Hard
- Multiple enemy types with progressive difficulty scaling
- Six boss designs with unique attack patterns
- Weapon upgrades (Vulcan spread shots → Laser pierce shots)
- Power-ups: health, bombs, weapons
- Combo multiplier system for score
- Adaptive performance quality scaling (quality 0=full, 1=medium, 2=low)
- Web Audio synthesis for all sound effects
- Accessibility: reduced-motion support, ARIA live regions
- Local storage persistence: high scores, settings

## Architecture

The game is structured as a single IIFE (Immediately Invoked Function Expression) containing:

### Game State (lines 863-869)
- `player`: position, speed, weapon, health, invulnerability timer
- `bullets`: player projectiles (supporting pierce for lasers)
- `enemies`: spawned enemy ships
- `enemyBullets`: enemy projectiles with various patterns
- `particles`: explosion particles and visual effects
- `powerUps`: health/bomb/weapon pickups
- `boss`: optional boss object during stage boss phase
- Global counters: `score`, `lives`, `running`, `paused`, `chain`, `multiplier`, `stage`

### Core Systems

**Enemy System (lines 1022-1228)**
- Three enemy types: grunt, interceptor, tank (lines 502-506)
- Procedural spawning with difficulty-based spawn rates
- Two firing patterns: spread (radial) and aimed (homing)
- Weaving behavior for interceptors (sine wave movement)
- HP system for tanks (multi-hit enemies)

**Boss System (lines 507-515, 1043-1056, 1372-1434)**
- Six boss designs with unique visual appearance (colors, glow effects)
- Each boss has 3 attack patterns (fireRadial, fireAimedSpread, fireColumns, etc.)
- Telegraph phase (22 frames) before attack to allow player reaction
- HP scales with stage and difficulty multiplier
- Boss enters from top of screen gradually

**Weapon System (lines 1124-1146, 1304-1322)**
- Two weapons: Vulcan (spread shots) and Laser (piercing shots)
- Each weapon has 3 levels of upgrade
- Vulcan: wider spread at higher levels
- Laser: more lanes + higher pierce count at higher levels
- Fire rate differs: 150ms (Vulcan) vs 100ms (Laser)

**Scoring & Combo (lines 1087-1101)**
- Base score per kill depends on enemy type (10-35)
- Kill chain increases multiplier (1→10x over 50+ kills)
- 2.2 second window to break chain if player takes damage
- Combo pop animation shows kill milestones (10, 20, etc.)

**Audio System (lines 542-676)**
- Web Audio API synth-based effects (no external audio files)
- Dynamic panning based on event position
- Frequencies, sweep, filters, and noise bursts for variety
- Master gain with compressor
- Mute state persists in localStorage

**Input System (lines 767-862)**
- Keyboard: Arrow keys + WASD (movement), Space (fire), X/Shift (bomb), P/Esc (pause)
- Mouse: position drives player steering (smooth glide, keyboard has priority)
- Touch: left zone = movement joystick, right zone = fire/bomb buttons
- Canvas point calculations handle CSS scaling

**Canvas & Rendering (lines 752-1637)**
- Base resolution: 480×720 pixels
- Scaling allows up to 1.6x for larger screens
- Viewport fills canvas with linear gradient sky background
- Procedural cloud animation at two parallax speeds
- Quality-based shadow/glow effects for performance

**Theme System (lines 493-500)**
- Six procedurally themed skies (Daylight, Sunset, Night, Storm, Arctic, Volcanic)
- Themes rotate by stage, each with sky gradients, cloud colors, sun glow

## Development Notes

### Running the Game
Simply open `index.html` in a web browser. No server or build step required. For testing on mobile, serve via HTTP (browsers require HTTPS or localhost for Web Audio API).

### Modifying Gameplay

**Difficulty Tweaking**: Edit `DIFFICULTIES` object (lines 485-489):
- `spawnBase`/`spawnMin`: controls enemy frequency (lower = faster)
- `enemySpeed`: multiplier for how fast enemies move
- `enemyShootChance`: probability enemies fire (0-1)
- `bossHpMul`: scales boss health

**Enemy Types**: Add or modify entries in `ENEMY_TYPES` (lines 502-506):
- `w`/`h`: collision box size
- `hp`: hits to kill
- `speedMul`: relative speed vs base
- `score`: points awarded
- `minStage`: earliest stage this enemy appears

**Boss Attacks**: Edit `BOSS_PATTERNS` (lines 1372-1409):
- Array of 6 patterns (one per boss)
- Each pattern is an array of 3 attack functions
- Functions receive boss object `b`, fire bullets via `enemyBullets.push()`
- Use helper functions: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, etc.

**Visual Tweaks**:
- Colors: search for hex values in CSS (lines 10-358) or in drawing functions
- Player ship design: `drawPlane()` function (lines 913-968)
- Boss ship design: `drawBossShip()` function (lines 970-1020)

### Performance Optimization

The game includes adaptive quality scaling (lines 473-483):
- Tracks frame times over 45 frames
- If average >25ms: reduce quality (less shadow/glow)
- If average <17ms: increase quality
- This keeps the game running smoothly across devices

Quality levels affect:
- Shadow blur on bullets and boss core
- Which particle counts are used in explosions

### Persistence

Data stored in localStorage:
- `skyfighter.bestScore`: high score (number)
- `skyfighter.muted`: audio mute state ('0' or '1')
- `skyfighter.difficulty`: selected difficulty ('easy'/'normal'/'hard')
- `skyfighter.reducedMotion`: accessibility preference ('0' or '1')

## Common Modifications

### Add a new weapon
1. Update weapon spawn in `maybeSpawnPowerUp()` to include new weapon type
2. Add weapon logic in `fireWeapon()` function
3. Add weapon label in `weaponLabel()` function
4. Update HUD display logic

### Add a new enemy type
1. Add entry to `ENEMY_TYPES` object with stats
2. Update spawn probability in `spawnEnemy()` function
3. Add explosion sound effect in `EXPLODE_SFX` mapping
4. Create custom drawing or reuse `drawPlane()`

### Add visual effects
- Particles: spawn via `spawnExplosion()` with color and count
- Screen shake: call `screenShake(magnitude, duration)`
- Flash: call `flashHit()` or `flashBomb()`
- Pop text: call `showComboPop()` or `showStageBanner()`

## Testing

There is no test suite. Validation happens through:
- Visual inspection in-browser
- Manual playtesting across difficulty settings
- Browser console for any errors (mute/motion buttons must work, audio must initialize on first interaction)

To verify a change works:
1. Open index.html in browser
2. Click a difficulty button
3. Click "Start Game"
4. Play through at least one boss fight
5. Verify: shooting, bombs, collisions, score updates, boss HP bar, stage progression

