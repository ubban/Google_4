# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML/CSS/JavaScript arcade experience. The entire game runs in-browser with procedurally generated audio and responsive canvas rendering.

## Architecture

The codebase is a single `index.html` file (~1670 lines) organized into logical sections:

### Core Game Systems

1. **Input Handling** (lines 767–861)
   - Keyboard: WASD/Arrows for movement, Space/click to fire, X/Shift/P to bomb/pause
   - Mouse: Hover steering with click-to-fire
   - Touch: Left 55% of screen for movement (joystick), right 45% for action buttons (Fire/Bomb)
   - Input priority: keyboard > joystick > mouse

2. **Game State & Loop** (lines 863–914)
   - Main game loop: `loop()` (line 1640) runs at 60 FPS using `requestAnimationFrame`
   - State transitions: menu → playing → paused/game-over → menu
   - Core state variables: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
   - Scoring system with combo multiplier chain (resets on 2.2s idle)

3. **Rendering** (lines 1507–1653)
   - Canvas: 480×720 logical resolution, responsive scaling
   - Procedurally drawn entities: player plane, enemies, boss, bullets, particles
   - Theme system: 6 rotating visual themes with unique sky gradients, clouds, and colors
   - Adaptive performance: quality levels (0=full, 1=medium, 2=low) based on frame time average
   - Visual effects: screen shake, flash animations, reduced-motion support

4. **Physics & Collision** (lines 1069–1301)
   - Simple AABB collision: `rectsOverlap()` checks overlapping rectangles
   - Bullet-enemy, bullet-boss, enemy-player, bomb-area, power-up pickup collisions
   - Invulnerability frames: 1.5s after hit to prevent rapid damage

5. **Enemy System** (lines 502–1041)
   - 3 enemy types: Grunt (basic), Interceptor (fast, weaves), Tank (slow, durable)
   - Spawn difficulty ramps over time (spawnBase decreases)
   - Movement patterns: weaving (sine wave) or straight
   - Behavior: collision, bullet fire (aimed or spread pattern), die → loot
   - Drop power-ups with weighted randomness: 6% life, 8% bomb, 20% weapon, 66% nothing

6. **Boss System** (lines 508–1453)
   - 6 unique bosses with distinct visual designs and attack patterns
   - Boss phases: entering (slides down) → battle → telegraph (1.5s warning) → attack
   - Each boss has 3 unique attack patterns (fireRadial, fireAimedSpread, fireColumns, etc.)
   - Health scales with difficulty (0.8x–1.3x) and boss type multiplier (0.8x–1.7x)
   - Stage progression: kill 18 enemies → boss warns → boss spawns 1.2s later

7. **Player & Weapons** (lines 873–1321)
   - Player: position, speed (260px/s), 40×40 hitbox, health, cooldown
   - Weapons: Vulcan (spread hitscan) and Laser (pierce hitscan), 3 upgrade levels
   - Fire rate: 150ms (Vulcan), 100ms (Laser)
   - Spread and pierce increase per level
   - Powered up by weapon pickups: upgrade level or switch weapon

8. **Audio** (lines 542–676)
   - Web Audio API synthesized sounds (no external files)
   - Osc types: sine, square, sawtooth + noise bursts
   - Volume: compressed and normalized
   - Spatial audio: stereo panning based on entity x-position
   - Sfx library: shoot, laser, hit, bomb, powerup, bossHit, bossTelegraph, bossDown, stageClear, gameover, uiClick
   - Mute toggle persisted to localStorage

9. **UI & Overlay** (lines 468–1504)
   - HTML-based HUD: score, best, multiplier, weapon, hearts, bombs, boss bar
   - Overlay states: start screen, paused, game over
   - Theme buttons: difficulty selector (Easy/Normal/Hard)
   - Control hints and accessibility
   - Toast notifications: stage banners, combo popups, announcements

10. **Difficulty & Balance** (lines 485–489)
    - Easy: spawnBase 75, 4 lives, 3 bombs, 0.8× boss HP
    - Normal: spawnBase 58, 3 lives, 2 bombs, 1.0× boss HP
    - Hard: spawnBase 40, 3 lives, 2 bombs, 1.3× boss HP
    - Enemy shot chance & speed also scale per difficulty

## Key Patterns & Design Decisions

### Adaptive Performance
- `trackPerf()` measures frame times and auto-downgrades quality if avg > 25ms
- Quality affects: particle count, glows/shadows, effect duration
- Prevents frame drop spirals on low-end devices

### Accessibility
- Reduced-motion mode: disables animations for users with vestibular sensitivity
- Screen reader support: `#srLive` live region announces events (boss defeated, lives remaining, etc.)
- Keyboard-navigable UI and full mouse support

### Local Storage
- Keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- Persists across sessions; defaults honored if not set

### Procedural Generation
- Enemies spawn randomly with position, type, speed, and shoot pattern (aimed vs. spread)
- Weave enemies use a seeded sine offset to create distinct flight paths
- Boss attack patterns combine fixed sequences (radial, aimed spread, columns) to create variety

## Modifying the Game

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502)
2. Update `spawnEnemy()` filter logic for minStage
3. Add explosion SFX key to `EXPLODE_SFX` map

### Adding a New Boss Attack Pattern
1. Create a function like `fireRadial()` in the attack library (lines 1324–1370)
2. Add pattern function to the boss's pattern array in `BOSS_PATTERNS[i]` (line 1372)
3. Pattern functions modify `enemyBullets` array and read `boss` state

### Adding a New Visual Theme
1. Add entry to `THEMES` array (line 493) with sky gradient stops, cloud color, sun color, name
2. Theme is indexed by `(stage - 1) % THEMES.length`, so themes cycle

### Adjusting Balance
- Enemy spawn rate: modify `DIFFICULTIES[difficulty].spawnBase` and `spawnMin`
- Enemy speed/aggression: modify `enemySpeed` and `enemyShootChance`
- Boss HP: modify `bossHpMul` in difficulty or boss design's `hpMul`
- Weapon fire rate: change `fireDelay` in weapon fire logic (line 1177)
- Score multiplier chain: adjust `chainTimer` duration (line 1089) and multiplier formula (line 1090)

### Adding UI Elements
- HUD stats: add a stat div inside `#hud` and update it in the update loop
- Overlay content: modify the `#overlayCard` div structure and toggle `#overlay.hidden`

## Common Tasks

### Run the game
Simply open `index.html` in a browser. No build step or server needed.

### Test a specific feature
- **Boss fight**: Set `KILLS_PER_STAGE` to 0 to skip straight to boss on stage 1
- **Weapon upgrades**: Spawn weapon pickups by adding to `powerUps` manually or lowering the `0.34` threshold in `maybeSpawnPowerUp()`
- **Quick difficulty check**: Use localStorage in browser console: `localStorage.setItem('skyfighter.difficulty', 'hard')`

### Debug collision issues
- Add temporary canvas visualization: render bounding boxes with `ctx.strokeRect()` inside `draw()`
- Check `rectsOverlap()` logic; adjust hitbox sizes in entity definitions

### Performance profiling
- Use DevTools Performance tab to identify bottlenecks
- Monitor `quality` variable; if it's 2, rendering is already stripped down
- Particle count is already adaptive based on quality

## Notes on Code Style

- Single-file strategy keeps state close and eliminates module boundaries
- Mutable state objects (player, enemies, etc.) are mutated in-place for speed
- Canvas context saved/restored frequently to manage global state isolation
- Event listeners use arrow functions to preserve `this` context
- No external dependencies; everything is vanilla JS + Web Audio API
