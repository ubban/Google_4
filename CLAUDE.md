# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file, browser-based Raiden-style vertical shoot-'em-up arcade game. The entire game (HTML, CSS, and JavaScript) is contained in `index.html` (~1670 lines). No build system or dependencies required — just open in a browser.

## Running the Game

- **Play locally**: Open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge)
- **No build step**: The file runs as-is; no npm install, webpack, or transpilation needed
- **No external assets**: All graphics are canvas-drawn, all audio is synthesized with Web Audio API
- **No server required**: Works entirely offline

## Architecture & Core Systems

The game uses a classic arcade game loop architecture. All state and logic live in a single IIFE (Immediately Invoked Function Expression) to avoid global scope pollution.

### Game Loop
- **`loop(now)`**: Main `requestAnimationFrame` callback, called ~60fps
- **`update(dt)`**: Handles all game logic — input, movement, collision, spawning, difficulty progression
- **`draw()`**: Renders everything to canvas without modifying state

The game detects performance (via `trackPerf()`) and dynamically reduces particle/effect quality (`quality` levels 0–2) if frame time drops below 40fps.

### Key State Objects

- **`player`**: Position, weapon, level, health, invulnerability timer, fire cooldown
- **`enemies[]`**: Enemy ships spawned per stage; each has position, type, HP, shoot pattern
- **`bullets[]`**: Player projectiles; pierce value allows lasers to pass through enemies
- **`enemyBullets[]`**: Incoming fire; checked for collision with player
- **`boss`**: Active boss ship; null when not fighting (spawned after 18 enemy kills per stage)
- **`powerUps[]`**: Dropped after enemy deaths — life (+1 HP), bomb (+1 bomb), weapon upgrade
- **`particles[]`**: Explosion debris, respects quality settings to maintain 60fps

### Gameplay Progression

1. **Difficulty**: Three modes (easy/normal/hard) affect enemy spawn rate, movement speed, shoot frequency, health pool, and boss HP multiplier
2. **Stages**: After 18 kills per stage, a boss spawns. Boss victory increments stage and resets kill counter
3. **Weapons**: Player starts with VULCAN (spread shots). Weapon powerups level up the weapon or swap to LASER (piercing, less spread but higher DPS). Level caps at 3, then swaps weapon
4. **Score multiplier**: Killing enemies increments a combo counter; every 5 kills bumps multiplier +1 (capped at 10×). Missing a kill for >2.2 seconds resets it to 1×
5. **Lives**: Varies by difficulty (3–4). Contact with enemy, bullet, or boss costs one life. Game Over when lives reach 0

### Sound System

All SFX are procedurally generated with Web Audio API:
- `beep(freq, dur, type, vol, opts)`: Sine/square/sawtooth oscillators, optional frequency sweep, filter, stereo pan
- `noiseBurst(dur, vol, opts)`: Filtered noise for explosion textures
- Examples: `sfx.shoot()`, `sfx.explodeGrunt()`, `sfx.bomb()`, `sfx.bossDown()`
- Mute state saved to localStorage; reduced-motion mode respects user's OS preference

### Boss Patterns

Each boss (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) fires bullets in different patterns:
- `fireRadial()`: Bullets radiating outward
- `fireAimedSpread()`: Cone aimed at player
- `fireCross()` / `fireColumns()`: Grid patterns
- `fireHomingSeeded()`: Scattered homing missiles
- Patterns stored in `BOSS_PATTERNS` array; boss picks pattern by design index and increments pattern counter each attack

### Input Handling

- **Keyboard**: Arrow keys or WASD for movement, Space to fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Move cursor to steer (glides toward mouse while canvas is active); click to fire
- **Touch**: Left half of screen is joystick (drag to aim), right side has FIRE and BOMB buttons
- Keyboard and joystick take priority over mouse steering to prevent fighting inputs

### Local Storage Keys

- `skyfighter.bestScore`: Best score across sessions
- `skyfighter.muted`: Audio mute state (0/1)
- `skyfighter.difficulty`: Last selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Screen shake / flash effects toggle (0/1)

## Common Tasks

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` object (w, h, hp, speedMul, color, dark, score, minStage, optional weave)
2. Update the spawn logic in `spawnEnemy()` to include the new type in the probability check
3. If adding special visual effects, update `drawPlane()` or add a custom draw function
4. Explosion audio: Add entry to `EXPLODE_SFX` map and create a corresponding `sfx.explodeNewType()` function

### Adding a New Boss

1. Append entry to `BOSSES` array (name, hullTop/Bottom, wing, core colors, hpMul)
2. Create a new pattern array in `BOSS_PATTERNS` with 3+ firing patterns (each a function taking boss object)
3. Boss index is `(stage - 1) % BOSSES.length`, so bosses cycle as you progress
4. If adding special visuals, modify `drawBossShip()` conditionally on `b.design`

### Tweaking Difficulty

Edit `DIFFICULTIES.normal` (or easy/hard) to adjust:
- `spawnBase` / `spawnMin`: Enemy spawn timer (frames between spawns)
- `enemySpeed` / `enemyShootChance`: How fast and how often enemies fire
- `lives` / `bombs` / `bossHpMul`: Player health and bomb count

### Changing Weapon Balance

- Spread/pierce: In `fireWeapon()`, adjust the `spread` array for VULCAN or `pierce` value for LASER
- Fire rate: Change `fireDelay` in the update loop (currently 0.15s for VULCAN, 0.1s for LASER)
- Damage: Bullets deal 1 HP per hit; bosses take 3 HP. Modify `boss.hp -= 3` in the bullet-boss collision handler

### Modifying Themes

Edit `THEMES` array to add sky gradients and cloud colors. Theme is selected by `(stage - 1) % THEMES.length`, so themes cycle.

## Canvas & Rendering

- **Base resolution**: 480×720 (portrait, arcade cabinet ratio)
- **Auto-scale**: Canvas scales to fit window while maintaining aspect ratio, capped at 1.6× to prevent pixelation on large displays
- **Drawing**: All shapes drawn on canvas via `drawPlane()` (player and enemies), `drawBossShip()` (boss), particle explosions, and UI overlays
- **Screen shake**: `screenShake(magnitude, duration)` jitters the canvas by translating the context; reduced-motion mode scales magnitude down and limits duration to 100ms

## UI & Overlay System

The overlay div shows:
- **Start screen**: Difficulty selector, controls hint
- **Pause menu**: Resume button
- **Game Over screen**: Final score, best score, "New Best" message, Play Again button
- **HUD** (always visible during gameplay): Score, best, multiplier, weapon level, hearts, bombs, boss health bar

Reduced-motion preference (OS or toggle) removes animations on card-in, heart pulse, combo pop, screen shake, and flash effects.

## Performance Considerations

- **Quality scaling**: `quality` variable (0=full, 1=medium, 2=low) reduces particle count and disables glow effects at quality 1+
- **Frame tracking**: Last 45 frames tracked; if avg > 25ms, quality increases; if avg < 17.8ms, quality decreases
- **Canvas draw calls**: Minimized by batching (e.g., all enemy bullets drawn in one loop with shadow color applied once)
- **No external image assets**: Entirely canvas primitives and Web Audio, so zero latency from asset loading

## Code Style & Patterns

- **Variables**: PascalCase for objects/constructors, camelCase for primitives and functions
- **Loops**: For enemy/bullet collision, iterating backward (`i--`) and splicing ensures no index skipping when removing items mid-loop
- **Collision**: Simple AABB (axis-aligned bounding box) via `rectsOverlap()` — no spatial hashing
- **Timing**: Durations stored as floats (seconds), decremented by `dt` (deltaTime) each frame; timers compared to 0 to check expiration
- **Audio**: All synthesis happens via Web Audio context created on first user interaction (`ensureAudio()`)

## Notes for Future Work

- The entire game is in one file for simplicity; refactoring into separate modules would improve maintainability but is not necessary for small additions
- Boss telegraph phase (red glow before attack) is 22 frames at 60fps ≈ 367ms; adjust `b.telegraph = 22` to change attack wind-up time
- Combo/multiplier is generous (resets after 2.2 seconds of no kills); adjust `chainTimer` max or reset threshold to make higher multipliers rarer
- Mobile support is solid but untested on all devices; touch-move and tap-fire logic is in `moveZone`, `fireBtn`, and `bombBtn` event handlers

---

Last updated: 2026-09-15
