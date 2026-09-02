# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 Canvas application. The entire game logic, styling, and markup are contained in `index.html` (1669 lines).

**Key Technologies:**
- Canvas 2D API for rendering
- Web Audio API for sound effects and music
- LocalStorage for persistence (high scores, difficulty, settings)
- Touch Events + Mouse/Keyboard input handling
- Responsive design with mobile support

## Architecture

### Single-File Design Philosophy

Everything lives in `index.html`. The structure is:
1. **HTML Head** (lines 1-360): Semantic markup + CSS styles + favicon
2. **HTML Body** (lines 361-420): Game canvas, HUD elements, overlay UI, touch controls
3. **JavaScript** (lines 422-1669): Game loop, entity systems, input handling, rendering

### Entity-Based System

Game entities are plain objects managed in arrays:
- `player`: { x, y, w, h, speed, cooldown, rapid, weapon, level, invuln, ... }
- `enemies`: Array of enemy objects with position, health, AI behavior
- `bullets`: Array of player projectiles
- `enemyBullets`: Array of enemy projectiles
- `powerUps`: Array of pickups (life, bomb, weapon)
- `particles`: Array of visual effects
- `boss`: Single boss object during stage bosses (null when not fighting)

No classes or prototypes; all objects are data containers with functions operating on them procedurally.

### Core Systems

**Difficulty Levels** (line 485-489):
```
const DIFFICULTIES = { easy, normal, hard }
```
Each difficulty adjusts: spawn rate, enemy speed, shoot chance, lives, bomb count, boss HP multiplier.

**Enemy Types** (line 502-527):
- `grunt`: Basic enemy, spawns from stage 1
- `interceptor`: Faster, spawns from stage 2
- `tank`: Heavily armored, spawns from stage 3

**Boss Designs** (searched in code): Each stage has a unique boss with health scaling, attack patterns, and visual design.

**Weapon System**:
- Vulcan (default): Spread pattern, rapid fire
- Laser: Straight beam, higher damage
- Levels 1-3: Spread increases with levels, picked up as power-ups

**Visual Themes** (line 493-499):
Six different sky color schemes rotate through stages (Daylight, Sunset, Night, Storm, Arctic, Volcanic).

### Game State & Flow

**Main Variables** (line 863-869):
- `running`, `paused`, `gameOver`: Game state flags
- `lives`, `maxLives`: Player health (1-4 depending on difficulty)
- `score`, `bestScore`: Scoring and high score tracking
- `chain`, `chainTimer`, `multiplier`: Kill-chain combo system (x1 to x10 multiplier)
- `stage`, `stageKills`: Progress tracking (18 kills → boss → next stage)
- `bomb`, `maxBombs`: Smart bomb resource management

**Game Loop** (found in update/render cycle):
1. Input processing (keyboard, mouse, touch, joystick)
2. Entity updates (movement, collision, state changes)
3. Spawn management (enemies, powerUps, effects)
4. Canvas rendering (background, entities, HUD)
5. Performance monitoring and adaptive quality

**Quality Levels** (line 473-483):
- Level 0: Full detail (particle count, shadows, glow effects)
- Level 1: Medium (60% of particles)
- Level 2: Low (35% of particles)

Auto-adjusts based on frame time to maintain 40+ FPS.

### Input Handling

**Keyboard** (searched in code):
- Arrows or WASD: Move
- Space: Fire weapon
- X or Shift: Drop bomb
- P or Esc: Pause
- Q/M: Mute toggle
- Period/Motion icon: Toggle reduced motion

**Mouse**:
- Move to steer (plane glides toward pointer)
- Click to fire

**Touch**:
- Left side (55% of screen): Joystick-style movement (drag)
- Right side action zone: FIRE and BOMB buttons
- Joystick base appears dynamically

**Priority**: Keyboard/joystick input takes priority over mouse to prevent fighting.

## Common Development Tasks

### Running the Game

Just open `index.html` in any modern browser:
```bash
# Quick test
open index.html  # macOS
xdg-open index.html  # Linux
start index.html  # Windows

# Or serve locally if needed
python -m http.server 8000
# Then visit http://localhost:8000/index.html
```

### Testing Features

1. **Difficulty Balance**: Edit `DIFFICULTIES` object (line 485-489)
2. **Enemy Spawn Rates**: Adjust `spawnBase`, `spawnMin` in difficulties
3. **Boss Difficulty**: Change `bossHpMul`, modify boss attack patterns
4. **Weapon Behavior**: Search for weapon-specific code (Vulcan vs Laser firing patterns)
5. **Visual Themes**: Add/modify themes in `THEMES` array (line 493-499)

### Adding New Features

**New Enemy Type**:
1. Add entry to `ENEMY_TYPES` (line 502-527)
2. Set `minStage` to control when it appears
3. Define `w`, `h`, `speedMul`, `hp`, `color`, `dark`, `score`, `weave`
4. Add rendering code in `drawPlane()` or create new draw function
5. Set explosion sound in `EXPLODE_SFX` (line 678)

**New Boss**:
1. Add to `BOSSES` array (found after line 1050)
2. Define `name`, `wing`, `hullTop`, `hullBottom`, `core`, `hpMul`
3. Implement attack patterns in the boss update logic

**New Power-Up Type**:
1. Add condition to `maybeSpawnPowerUp()` (line 1058-1067)
2. Add collision handling in the main update loop
3. Add rendering code for the new type

### Audio System

**Sound Effects** (line 624-676):
- Uses Web Audio API (context created on first user interaction via `ensureAudio()`)
- Synthesized sounds: beep, noiseBurst, etc.
- All effects stored in `const sfx = {...}` object
- Mute state persists in localStorage

**Playing SFX**:
```
sfx.hit();  // Plays a hit sound
```

Add new sounds by creating beep/noise definitions in the sfx object.

### Persistence

**localStorage Keys** (line 463-466):
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Selected difficulty
- `skyfighter.reducedMotion`: Accessibility preference

Modify `STORAGE_*` constants to add new persistent settings.

## Performance Considerations

1. **Particle Budget**: Quality level auto-adjusts particle spawn counts
2. **Shadow/Glow Effects**: Disabled at low quality for performance
3. **Frame Tracking**: `trackPerf()` monitors average frame time over 45 frames
4. **Resolution**: Canvas scales responsively but maintains aspect ratio

Target FPS: 60 (quality adjusts if avg frame time > 1/40s)

## Accessibility Features

- **Reduced Motion**: Respects `prefers-reduced-motion` media query, disables animations
- **Screen Reader Support**: HUD updates announced to `#srLive` with `aria-live="polite"`
- **Semantic HTML**: Proper heading hierarchy, button roles, alt descriptions
- **High Contrast**: Colors chosen for sufficient WCAG AA contrast
- **Mobile Safe Area**: Uses `env(safe-area-inset-*)` for notches/rounded corners

## Testing Checklist

When making changes, verify:
- [ ] Game runs in desktop browsers (Chrome, Firefox, Safari, Edge)
- [ ] Mobile touch controls work (drag left to move, tap FIRE/BOMB)
- [ ] Score multiplier increases correctly on kill chains
- [ ] Boss health bar updates during boss fights
- [ ] All three weapon types fire and cycle correctly
- [ ] Bombs clear screen and apply invulnerability
- [ ] Lives/hearts display updates on damage
- [ ] High score saves and persists on page reload
- [ ] Mute/motion toggle buttons work and persist
- [ ] Difficulty selection reflects in game speed/spawns
- [ ] Screen reader announcements work (combo pop, stage banner)
- [ ] Reduced motion preference disables animations

## File Organization Tip

For large edits, search by line number or use distinctive function/variable names:
- `resize()`: Canvas sizing (line 754)
- `trackPerf()`: Performance monitoring (line 476)
- `updateMotionBtn()`: Reduced motion toggle (line 529)
- `resetGame()`: Game initialization (line 875)
- `spawnEnemy()`: Enemy spawning (line 1022)
- `spawnBoss()`: Boss spawning (line 1043)
- `drawPlane()`: Player/enemy rendering (line 913)
- `drawBossShip()`: Boss rendering (line 970)

Search these names when navigating the code.
