# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built with vanilla HTML5 Canvas. It's a single-file, self-contained game with no external dependencies—just open `index.html` in a browser to play.

### Running the Game

- **Development**: Open `index.html` directly in a browser (any modern browser supporting HTML5 Canvas and Web Audio API)
- **No build step required**: The entire game is in one HTML file
- **Live reload**: Edit `index.html` and refresh the browser to see changes

### Testing/QA

- Test on desktop (Chrome, Firefox, Safari, Edge)
- Test on mobile (touch controls, orientation handling, safe area insets)
- Verify audio (mute button, volume levels, no browser console errors)

## Code Architecture

The game is organized as a single IIFE (Immediately Invoked Function Expression) to encapsulate all game state and logic. Key sections:

### Game State & Constants

- **Difficulty settings** (`DIFFICULTIES`): Easy/Normal/Hard with spawn rates, enemy speed, lives, bombs
- **Enemy types** (`ENEMY_TYPES`): grunt, interceptor, tank with different HP, speed, scores
- **Boss designs** (`BOSSES`): 6 unique boss types with color schemes and HP multipliers
- **Visual themes** (`THEMES`): 6 stage themes with sky gradients, clouds, sun colors
- **Boss attack patterns** (`BOSS_PATTERNS`): Pattern arrays indexed by boss type (radial bursts, aimed spreads, spirals, etc.)

### Core Game Loop

```
requestAnimationFrame(loop) → update(dt) → draw() → repeat
```

1. **Input handling**: Keyboard (WASD/Arrows), mouse movement/click, touch joystick + buttons
2. **Update phase** (`update(dt)`):
   - Player movement (keyboard/joystick priority over mouse steering)
   - Weapon firing with cooldown
   - Bullet/enemy/powerup movement
   - Enemy spawning and behavior
   - Collision detection
   - Chain combo tracking
3. **Render phase** (`draw()`):
   - Background (gradient sky with clouds, sun glow)
   - All sprites (player, enemies, boss, bullets, powerups, particles)
   - Screen shake effect (inverted with `reducedMotion`)
   - HUD elements drawn via CSS (score, weapon, lives, bombs)

### Player & Combat

- **Player object**: x, y, w, h, speed (260), weapon (vulcan/laser), level (1-3), cooldown, invulnerability timer
- **Weapons**:
  - **Vulcan**: Rapid spread at 150ms cooldown; 3 projectiles at level 3
  - **Laser**: Slower 100ms cooldown; pierces enemies (1-2 pierce at level 3), higher speed (640 vs 500)
- **Firing**: Space bar or mouse click; cooldown managed per update
- **Weapon pickup**: Advances level or toggles weapon type, resets level to 1

### Enemies

- Spawn with random type, fall from top with varied horizontal speed
- **Grunt** (red): 1 HP, slow, score 10
- **Interceptor** (orange): 1 HP, weaves side-to-side, score 15, appears from stage 1
- **Tank** (purple): 4 HP, slow, shows HP bar, score 35, appears from stage 2
- **Behavior**: Enemies shoot either spread pattern (3 bullets at ±35°) or homing (aimed at player)
- **Death**: Spawn explosion particles, drop random powerups (6% life, 8% bomb, 20% weapon), add score × multiplier, increment combo chain

### Bosses

- Spawn after 18 kills per stage (KILLS_PER_STAGE)
- **Entering phase**: Boss slides down and settles at y=120; invulnerable during entry
- **Attack patterns**: Each boss has 3 unique attack patterns; patterns repeat cyclically
  - Fire patterns include: radial bursts, aimed spreads, spiral bursts, columns, homing missiles, double rings, 8-way cross fire
  - Telegraphs with a visual windup (22 frames) before firing
- **HP scaling**: Base HP = 70 + stage × 35, multiplied by difficulty and boss-specific multiplier
- **Damage**: Player bullets = 3 damage per hit; bombs = 24 damage
- **Death**: Large explosion, score +500, stage advances, show "STAGE X" banner, announce stage theme name

### Scoring & Progression

- **Chain system**: Consecutive kills within 2.2 seconds extend the chain
- **Multiplier**: x = 1 + floor(chain / 5), capped at x10
- **Progression**: Beat boss → stage ++ → enemy difficulty increases, new theme, next boss
- **Best score**: Persisted to localStorage, shown on game-over screen

### Power-ups

- Drop randomly from defeated enemies (weighted probabilities)
- **Types**: Life (refill to max), Bomb (add one), Weapon (upgrade weapon or toggle type)
- **Magnetism**: Player passively collects them as they fall

### Input Handling

**Keyboard**: WASD/Arrows for movement, Space for fire, X/Shift for bomb, P/Esc for pause
**Mouse**: Move cursor to steer (smooth glide toward target); click to fire
**Touch**: Left 55% of screen = joystick; bottom-right = FIRE & BOMB buttons
**Priority**: Keyboard/joystick input overrides mouse steering (no fighting)

### Audio

- **WebAudio synthesis**: No external assets; all sounds generated procedurally via oscillators, noise, filters
- **Key sounds**: shoot (square sweep), laser (sawtooth sweep), explosions (noise + pitch sweep for each enemy type), bomb (complex multi-part burst), powerup (arpeggio), boss telegraph, stage clear, game-over
- **Panning**: Enemy explosions use spatial panning based on screen position
- **Muted**: Toggle via button; persists to localStorage
- **Compression**: Dynamics compressor applied to master output

### Performance Optimization

- **Adaptive quality**: Tracks frame times over 45 frames; reduces particle/glow count if avg frame time > 25ms, restores if < 17.8ms
- **Quality levels**: 0 (full glow/shadows), 1 (medium particles/glow), 2 (minimal effects)
- **Canvas sizing**: Base 480×720; scales to fit window up to 1.6× without blur
- **Collision**: Simple AABB (axis-aligned bounding box) overlap check

### Accessibility

- **Reduced motion**: Toggle button respects `prefers-reduced-motion` media query; disables all animations, screen shake, flash effects
- **Screen reader support**: Live region (`#srLive`) announces score milestones, stage clears, game-over state
- **Color contrast**: HUD text has text-shadow for readability

## Common Development Tasks

### Adding a New Enemy Type

1. Add to `ENEMY_TYPES` object with properties: w, h, hp, speedMul, color, dark, score, minStage, optionally weave
2. Add explosion sound to `sfx` and `EXPLODE_SFX` mapping if unique
3. Edit spawn logic in `spawnEnemy()` if special behavior needed (e.g., weave pattern)

### Adding a New Boss

1. Add design to `BOSSES` array with: name, hullTop, hullBottom, wing, core colors, hpMul
2. Create attack pattern functions (e.g., `fireRadial`, `fireColumns`) and add 3 patterns to `BOSS_PATTERNS` array at matching index
3. Boss rendering in `drawBossShip()` uses design colors automatically

### Tweaking Game Balance

- **Difficulty**: Modify `DIFFICULTIES` object (spawnBase, spawnMin, enemySpeed, enemyShootChance, bossHpMul)
- **Enemy stats**: Adjust ENEMY_TYPES properties
- **Boss HP**: Adjust baseHp formula (line ~1046) or per-boss `hpMul`
- **Scoring**: Modify enemy `.score` properties, base boss score (500), or multiplier formula (`1 + Math.min(9, Math.floor(chain / 5))`)

### Debugging

- **Browser console**: Check for errors
- **Canvas debug**: Add `console.log()` in update/draw loops (beware performance)
- **Entity inspection**: Log `player`, `enemies`, `boss`, `bullets` objects
- **Audio issues**: `audioCtx.state` should be 'running' after first user gesture

## Key Constants to Know

- `BASE_W = 480, BASE_H = 720`: Canvas size
- `KILLS_PER_STAGE = 18`: Enemies to defeat before boss spawns
- `player.speed = 260`: Pixels/sec movement speed
- Boss entry position: y = 120 (settles here)
- Cooldowns: vulcan 150ms, laser 100ms
- Chain timeout: 2.2 seconds
- Invulnerability duration: 1.5 seconds after hit
