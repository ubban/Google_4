# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a browser-based Raiden-style vertical shoot-'em-up game. The entire game is implemented in a single HTML5 file with no external dependencies. It features:

- Canvas 2D rendering with real-time game loop
- Procedurally-generated enemy spawning and attack patterns
- Multiple difficulty levels (Easy, Normal, Hard)
- Boss encounters with unique attack patterns
- Power-up system for weapons and health
- Touch controls for mobile and keyboard/mouse controls for desktop
- Synthesized sound effects via Web Audio API
- Local storage for high scores and preferences
- Adaptive performance based on frame time

## Quick Start

To run the game, simply open `index.html` in a web browser. No build process, server, or dependencies are required.

```bash
# No npm install needed - it's a single-file HTML game
# Just open in a browser:
open index.html
```

## Architecture Overview

The entire game logic is contained in a single `<script>` block within `index.html`. The code is organized into functional sections:

### Core Game Loop (`loop` function, line 1640)
- `requestAnimationFrame` drives the main loop at ~60fps
- `update(dt)` handles game state and physics
- `draw()` renders the frame
- Adaptive performance system adjusts quality (0=full, 1=medium, 2=low) based on frame times

### Game State (line 863)
Key global variables:
- `player` — player ship position, weapon, health
- `bullets`, `enemies`, `enemyBullets`, `particles` — entity arrays
- `boss` — current boss or null
- `stage`, `kills`, `chain`, `multiplier` — progression and scoring
- `running`, `paused`, `gameOver` — game state flags

### Input Handling (lines 767–861)
- Keyboard events map to directional movement and firing
- Mouse movement steers player while canvas is hovered
- Touch events handle mobile joystick (left side) and action buttons (right side, fire/bomb)
- Single-flag model prevents conflicts between input methods

### Collision Detection (`rectsOverlap` function, line 1069)
Simple axis-aligned bounding box collision for bullets, enemies, powerups, and boss.

### Boss System (lines 1043–1439)
- 6 boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) cycle per stage
- Each boss has unique attack patterns defined in `BOSS_PATTERNS` array (line 1372)
- Boss enters from top, moves side-to-side, telegraphs attacks, and spawns bullets
- Attack patterns include radial bursts, aimed spreads, columns, spirals, and homing projectiles

### Difficulty System (line 485)
Three presets (easy/normal/hard) control:
- Enemy spawn frequency and minimum spawn interval
- Enemy speed and shooting frequency
- Player lives and bomb count
- Boss HP multiplier

### Audio (lines 542–676)
All sound effects are synthesized with Web Audio API oscillators and noise bursts:
- `beep()` — sine/square/sawtooth waves with frequency sweeps
- `noiseBurst()` — filtered noise for explosions and impacts
- Individual SFX functions map to game events (shoot, explode, bomb, powerup, etc.)
- Panning based on x position for spatial audio
- Mute toggle stored in localStorage

### Rendering (lines 913–1638)
- `drawPlane()` — draws player, enemy, and boss ships
- `drawBossShip()` — renders boss with animated wings and core
- `drawClouds()` — parallax scrolling cloud layers
- Quality flag determines whether to render glow effects (shadow/blur)
- Screen shake effect applied to canvas context transform
- Particle system for explosions

### Theme System (line 493)
Six themes cycle per stage, each with sky gradient colors, cloud color, and sun color. Affects visual appearance only, not gameplay.

### Player Weapon System (line 871)
Two weapons toggle via powerups:
- **Vulcan** — spread shot that improves with levels (1, 2, or 3 bullets)
- **Laser** — fewer but more damaging bullets with pierce capability

### Scoring & Multiplier (lines 1087–1101)
- Kills increment a chain counter
- Chain decays over 2.2 seconds of inactivity
- Multiplier increases every 5 kills (1x base, up to 10x)
- Score multiplied by chain multiplier on each kill

## Key Functions by Responsibility

### Game Initialization & Reset
- `resetGame()` (line 875) — set up new game with selected difficulty
- `setDifficulty()` (line 741) — change difficulty and persist to localStorage

### Update Loop
- `update(dt)` (line 1148) — player movement, firing, collision detection, enemy/boss updates
- `updateBoss(dt)` (line 1411) — boss position, telegraph timer, pattern execution
- `fireWeapon()` (line 1124) — spawn player bullets based on weapon type
- `useBomb()` (line 1103) — clear all enemy bullets, deal damage to enemies and boss

### Rendering
- `draw()` (line 1520) — clear canvas, draw sky/theme, clouds, entities, effects
- `drawPlane()` (line 913) — triangular ship shape with flame effect when firing
- `drawBossShip()` (line 970) — complex multi-layered boss design with wings and glowing core

### Spawning & Effects
- `spawnEnemy()` (line 1022) — choose enemy type based on stage and difficulty, create with random pattern
- `spawnBoss()` (line 1043) — instantiate boss for next stage
- `spawnExplosion()` (line 1073) — particle emitter for impact/death effects
- `maybeSpawnPowerUp()` (line 1058) — drop powerup (life/bomb/weapon) with random chance

### Boss Attack Patterns
Functions fire different bullet patterns used by bosses:
- `fireRadial()` (line 1324) — even spread in all directions
- `fireAimedSpread()` (line 1330) — cone of bullets aimed at player
- `fireColumns()` (line 1342) — vertical lines
- `fireSpiralBurst()` (line 1347) — rotating spiral
- `fireDoubleRing()` (line 1367) — concentric rings at different speeds

### UI & Feedback
- `renderHearts()` (line 680) — display health as emoji hearts
- `renderBombs()` (line 690) — display bomb count as emoji
- `showComboPop()` (line 712) — animate kill counter popup
- `showStageBanner()` (line 719) — animate stage entry banner
- `screenShake()` (line 727) — apply temporary screen shake (respects reduced motion)
- `flashHit()`, `flashBomb()` (line 698–709) — white flash overlay animation

### Game End States
- `loseLife()` (line 1456) — lose one heart, check for game over
- `killBoss()` (line 1441) — advance stage, spawn particles, add score bonus
- `endGame()` (line 1469) — finalize score, show game over screen, check for new best

## Common Development Tasks

### Adjusting Difficulty
Modify the `DIFFICULTIES` object (line 485). Properties:
- `spawnBase`, `spawnMin` — control enemy spawn rate (frames)
- `enemySpeed`, `enemyShootChance` — enemy behavior
- `lives`, `bombs` — starting player resources
- `bossHpMul` — multiplier on boss health formula

### Adding a New Boss
1. Add entry to `BOSSES` array (line 508) with name, colors, and hpMul
2. Add pattern array to `BOSS_PATTERNS` (line 1372) with attack functions
3. Boss index automatically cycles per stage in `spawnBoss()`

### Tuning Enemy Types
Edit `ENEMY_TYPES` object (line 502):
- `w`, `h` — collision box size
- `hp` — bullets needed to kill
- `speedMul` — relative to difficulty speed
- `score` — points awarded
- `minStage` — when type first appears
- `weave` — optional zigzag movement

### Adding New Sound Effects
1. Define a function in `sfx` object (line 624) using `beep()` or `noiseBurst()`
2. Call `ensureAudio()` first to initialize audio context
3. Respect the `muted` flag
4. Use `panFor()` to pan based on source x position (line 562)

### Modifying Visual Themes
Edit `THEMES` array (line 493). Each theme has:
- `sky` — three-stop gradient colors for sky
- `cloud`, `sun` — RGB color strings for foreground effects
- `name` — displayed when entering stage

## Storage & Persistence

The game uses `localStorage` with these keys:
- `skyfighter.bestScore` — highest score
- `skyfighter.muted` — audio mute state (0 or 1)
- `skyfighter.difficulty` — last selected difficulty
- `skyfighter.reducedMotion` — accessibility preference

Read at startup (line 517), persisted when changed.

## Performance Considerations

- **Adaptive quality system** (line 474) monitors frame times and reduces shadow/blur effects if needed
- **Particle limits** — explosion count scaled by quality setting (line 1074)
- **Canvas resolution** — fixed at 480×720 game pixels, scaled to window size (line 753)
- **Entity culling** — bullets and enemies removed when off-screen
- **requestAnimationFrame** — single RAF call; no setTimeout or setInterval in the main loop

## Accessibility Features

- Screen reader live region (`#srLive`) announces game events
- Reduced motion toggle disables animations (CSS: `body.reduced-motion`)
- Keyboard-only playable (arrows/WASD + Space + X)
- Touch controls on mobile
- Emoji-based UI (hearts for health, bombs for powerups, plane for title)

## Testing the Game

No automated test suite exists. Manual testing should cover:
1. **Difficulty modes** — spawn rates, enemy behavior, boss HP
2. **Weapons** — vulcan spread levels, laser pierce, damage
3. **Boss patterns** — verify pattern execution timing and bullet positions per boss
4. **Mobile** — touch joystick movement, fire/bomb button responsiveness
5. **Audio** — SFX for each event, panning across screen
6. **Score** — chain multiplier, power-up scoring, stage bonuses
7. **Accessibility** — reduced motion disables animations, screen reader announces events
