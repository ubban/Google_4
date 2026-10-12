# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 canvas application. The entire game—including UI, game logic, audio synthesis, and rendering—is contained in `index.html` (1,670 lines). It's a fully playable browser game with no build process or external dependencies.

## Quick Start

- **Run the game**: Open `index.html` in any modern web browser
- **No build step needed**: The game runs directly as a static HTML file
- **Test locally**: `python -m http.server 8000` (or any simple HTTP server), then open `http://localhost:8000`

## Architecture Overview

The game is organized as a single IIFE (Immediately Invoked Function Expression) containing several subsystems:

### Core Game Loop
- **Main loop** (`loop()` function, line 1640): Runs via `requestAnimationFrame`, handles delta-time updates and rendering
- **Update cycle** (`update()` function): Processes input, entity movement, collisions, and state changes
- **Render cycle** (`draw()` function, line 1520): Canvas 2D drawing of all game elements with gradient backgrounds

### Key Game Systems

**Input System** (lines 768-850)
- Keyboard input (arrow keys, WASD, Space, X/Shift for bomb, P/Esc for pause)
- Mouse targeting and click-to-fire
- Touch controls with on-screen joystick (virtual stick + fire/bomb buttons)
- Mobile joystick implementation tracks touch origin and constrains movement radius

**Entity System**
- **Player** (`player` object): Position, velocity, health, weapon state, invulnerability frames
- **Enemies** (`enemies` array): Spawned based on difficulty, pursue downward with random horizontal movement, shoot occasionally
- **Bosses** (`boss` object): Appear after `KILLS_PER_STAGE` (18) enemy kills; more complex with telegraphed attack patterns
- **Bullets** (`bullets` for player fire, `enemyBullets` for enemy/boss projectiles): Differ in visual style and collision handling
- **Power-ups** (`powerUps` array): Weapon upgrades (cycling: spread → laser → dual), extra lives, and bombs (smart damage areas)
- **Particles** (`particles` array): Visual feedback for explosions and hits

**Audio System** (lines 542-676, WebAudio API)
- No external audio files—all sounds generated via WebAudio synthesis
- Two function families: `beep()` for tonal sounds (with frequency sweep, filters, panning) and `noiseBurst()` for percussive sounds
- Sound effects object (`sfx`) pre-defines all game sounds: shoot, laser, explosions (by enemy type), bombs, powerups, bosses, UI clicks
- Spatial audio via panning based on x-position for immersion
- Master gain and compressor for controlled output

**Difficulty System** (line 485)
- Three tiers: easy, normal, hard
- Each adjusts spawn rate, enemy speed, shoot chance, starting lives/bombs, and boss HP
- Persisted to localStorage; can be changed pre-game

**Rendering System**
- Canvas dimensions: 480×720 (BASE_W, BASE_H) with responsive scaling up to 1.6× (line 757)
- Multi-layer rendering: sky gradients → clouds → player → enemies → boss → projectiles → particles
- Sprites are procedurally drawn: `drawPlane()` for ship silhouettes, `drawBossShip()` for larger craft
- Glow effects and shadows conditionally applied based on adaptive quality setting
- Screen shake and flash effects for impact feedback (respects reduced-motion preference)

**Adaptive Quality** (lines 474-483)
- Tracks frame times over 45 frames and dynamically adjusts quality level (0=full, 1=medium, 2=low)
- High performance target: 40+ FPS average; if achieved, quality increases; 56+ FPS to decrease
- Quality affects glow/shadow rendering and animation intensity

**Progression System**
- Stages advance after defeating a boss; enemy spawning becomes more aggressive each stage
- Six stage themes with unique color palettes (Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge)
- Score multiplier increases with consecutive enemy kills (combo chain); breaks on player hit
- High score and difficulty preference saved to localStorage

### Boss Design (lines 1350-1440)
- Six boss variants (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with different visual colors and HP multipliers
- Each boss has 3 distinct attack patterns (e.g., columns + ring + homing), cycled sequentially
- Telegraph phase (visual delay) before each attack cue improves readability
- Attack patterns use radial fire, aimed spread, dual rings, and homing projectiles

## Code Patterns & Conventions

**Update Functions**: Most subsystems have an `update<System>(dt)` pattern (e.g., `updatePlayer()`, `updateBoss()`, `updateEnemies()`)—called each frame with delta-time in seconds.

**Object Pools**: Bullets, enemies, and particles are managed as simple arrays (push on spawn, filter on cleanup), avoiding heap churn.

**Constants at Top**: Game-tuning constants (DIFFICULTIES, THEMES, ENEMY_TYPES, KILLS_PER_STAGE, etc.) are declared early for easy tweaking.

**Canvas State Management**: Heavy use of `ctx.save()`/`ctx.restore()` to isolate transform and fill/stroke style changes.

**Accessibility Features**:
- Screen reader announcements via `announce()` (line 468)
- Reduced motion support via `prefers-reduced-motion` media query (line 520), toggleable via UI
- High contrast UI overlays

## Performance Considerations

- Single canvas 2D context; no WebGL
- Entity counts are capped naturally (spawning rate, max bomb radius), preventing runaway iteration
- Collision detection is axis-aligned (simpler than pixel-perfect) and early-exit on first hit per entity
- Frame-rate adaptive quality tuning prevents lag on lower-end devices
- No memory leaks from lingering event listeners (all attached to document or canvas, not dynamic DOM)

## Testing & Development

**Manual testing**: Open in browser, play through multiple stages at different difficulties. Watch for:
- Smooth frame rate (adaptive quality should maintain 40+ FPS)
- Collision accuracy (especially boss hit detection)
- Audio balance (no clipping, spatial panning audible)
- Mobile responsiveness (joystick sensitivity, touch fire/bomb reliability)
- Accessibility (reduced motion, keyboard-only play)

**Breakpoints for debugging**:
- Add `debugger;` in the game loop or specific update functions to pause execution
- Use browser DevTools to inspect entity arrays (`player`, `enemies`, `boss`, `bullets`)
- Monitor frame time via console (frameTimes array tracks recent delta times)

## Making Changes

**To add a feature** (e.g., new weapon, enemy type, visual effect):
1. Define constants (e.g., new weapon pattern in an array at the top)
2. Add update logic in the relevant subsystem (e.g., weapon fire in `firePlayerBullets()`)
3. Add rendering in `draw()` if needed
4. Add audio cue in `sfx` object if appropriate
5. Test across difficulty levels and stages

**To tweak difficulty**: Adjust the DIFFICULTIES object (spawn rates, speeds, shoot chances, HP multipliers) and test enemy/boss responsiveness.

**To add a stage theme**: Append a new entry to THEMES (sky color stops, cloud color, sun color, display name) and watch it cycle in Stage 7+.

**Mobile/touch fixes**: Joystick logic is in `joystickStart()`, `joystickMove()`, `joystickEnd()`; touch event listeners are on the `moveZone` element.

## Key Files & Line Ranges

- **HTML structure & CSS**: Lines 1–421 (viewport setup, canvas container, overlay, HUD, touch controls, styling)
- **Game constants**: Lines 485–525 (DIFFICULTIES, THEMES, ENEMY_TYPES, BOSSES)
- **Audio subsystem**: Lines 542–676 (WebAudio init, beep/noiseBurst functions, sfx object)
- **UI & rendering helpers**: Lines 680–764 (hearts, bombs, flash, stage banner, screen shake)
- **Input handling**: Lines 767–850 (keyboard, mouse, touch/joystick)
- **Game state & player**: Lines 850–1050 (player object, movement, firing, collisions)
- **Enemy spawning & updates**: Lines 1100–1200
- **Boss logic**: Lines 1350–1440 (boss update, attack patterns, death)
- **Main game loop**: Lines 1640–1665 (loop, event listeners, requestAnimationFrame start)

## Storage Keys

- `skyfighter.bestScore`: High score (number)
- `skyfighter.muted`: Audio mute state (0 or 1)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Reduced motion preference (0 or 1)

## Common Edits

- **Adjust difficulty curve**: Modify DIFFICULTIES object, particularly `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, and `bossHpMul`
- **Change player speed or fire rate**: Edit `player.speed` or `player.fireRate` in `resetGame()` and firing logic
- **Modify weapon behavior**: Adjust projectile size, speed, or fire pattern in weapon-specific fire functions
- **Add screen juice**: Enhance `screenShake()` magnitudes or durations, or add new visual effects (color overlays, post-process filters)
- **Balance boss difficulty**: Tweak `BOSS_PATTERNS` attack frequencies, bullet counts, or damage per hit
