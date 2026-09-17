# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a browser-based Raiden-style vertical shooter game. It's a single-file HTML application (index.html) with no external dependencies — all logic, rendering, and sound synthesis are self-contained.

## Architecture

### Single-File Structure
- **index.html** contains HTML, CSS, and JavaScript in one file
- No build process or external libraries required
- Runs directly in browser with ~1670 lines of code

### Core Systems

#### Game Loop & Rendering
- Uses `requestAnimationFrame` for 60fps target
- Delta-time based updates (`dt`) for consistent movement across frame rates
- Canvas 2D context (`ctx`) for all rendering
- Adaptive quality system that reduces particle count on slower devices

#### Player & Input
- **Keyboard**: Arrow keys or WASD for movement, Space to fire, X/Shift for bomb
- **Mouse**: Move to steer, click to fire (takes priority over keyboard when active)
- **Touch**: Left side joystick for movement, FIRE/BOMB buttons on right (mobile layout)
- Input state tracked in `keys` object; joystick has continuous velocity (`joyVec`)

#### Collision Detection
- Uses axis-aligned bounding boxes with `rectsOverlap(a, b)` helper
- Checks player vs enemies, player vs bullets, bullets vs enemies, bullets vs boss
- Boss collision is disabled during entrance animation

#### Weapon System
- Two weapons: **VULCAN** (spreads at level 3) and **LASER** (multi-lane at level 2+)
- Weapon upgrade powerups cycle through level 1→2→3, then switch weapon and reset level
- Laser shots have pierce mechanic (pass through multiple enemies)

#### Bomb System
- Limited-use smart bomb: clears all enemy bullets and damages/kills all on-screen enemies
- Bomb usage adds score and resets active combo chain
- Starts with 2–3 bombs depending on difficulty

#### Scoring & Combo
- Score multiplier increases with consecutive kills (every 5 kills = +1 multiplier, max 10x)
- Multiplier resets after 2.2 seconds without a kill
- Milestone popups at 10, 20, 30 kills

### Game Progression

#### Stages & Difficulty
- Game advances through infinite stages; each stage requires 18 enemy kills to trigger boss
- Three difficulties (Easy/Normal/Hard) with different spawn rates, enemy speed, and HP multipliers
- Settings persist in localStorage

#### Enemy Types
- **Grunt** (red, square): 1 HP, standard enemy available from stage 1
- **Interceptor** (orange, fast): 1 HP, weaves side-to-side, available from stage 1
- **Tank** (purple, slow): 4 HP, heavy hitter, available from stage 2+

#### Boss Pattern System
- 6 unique boss designs with distinct visuals and attack patterns
- Boss cycles through 3 pattern variants before repeating
- Pattern types include: radial bursts, aimed spreads, homing missiles, columns, spirals
- Boss telegraph phase (red glow) warns before attack; attack phase triggers bullet pattern

### Audio
- **No external audio files** — uses WebAudio API to synthesize all sounds
- `beep()` function creates tones via oscillator + filter + gain envelope
- `noiseBurst()` creates percussive sounds from noise buffer
- Panning by screen position (left enemies → left speaker, etc.)
- Master gain and compressor for dynamic range control
- Mute state persists in localStorage

### Visual Theme System
- 6 rotating sky themes (Daylight, Sunset, Night, Storm, Arctic, Volcanic)
- Each theme defines sky gradient, cloud color, and sun glow
- Theme advances per stage automatically
- Parallax clouds at two speeds for depth

### UI & HUD
- Persistent top HUD shows Score, Best, Multiplier, Weapon, Hearts, Bombs
- Boss health bar appears when boss is active
- Combo popups and stage banners with animations
- Reduced motion support (accessibility): toggleable screen effects
- Pause overlay with difficulty selection

### Performance Optimization
- Adaptive quality based on frame time tracking (3 levels: full → medium → low)
- Lower quality reduces particle count and disables shadow/glow effects
- Filters on canvas CSS scale smoothly for different viewport sizes

## Code Organization

### Key Constants
- `DIFFICULTIES`: Parameters for each difficulty (spawn rates, enemy speed, boss HP multiplier)
- `ENEMY_TYPES`: Enemy definitions (size, HP, speed, score value, availability stage)
- `BOSSES`: 6 boss designs with visual properties and HP multiplier
- `THEMES`: 6 sky themes with color gradients
- `BOSS_PATTERNS`: Attack patterns for each boss type

### Game State Variables
- `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- `score`, `lives`, `running`, `paused`, `gameOver`, `stage`, `stageKills`, `chain`, `multiplier`
- `frame` counter (increments every frame, used for timing and animations)

### Main Update Loop
1. `update(dt)` — game logic (input, collision, spawn, state changes)
2. `draw()` — canvas rendering (background, entities, effects)
3. `trackPerf(dt)` — monitors frame time for adaptive quality

## Common Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` with visual/gameplay properties
2. Set `minStage` to control when it appears
3. If custom explosion sound needed, add function to `sfx` object and reference in `EXPLODE_SFX`

### Adding a New Boss
1. Add design to `BOSSES` array (hull colors, core color, name, hpMul)
2. Create attack patterns array in `BOSS_PATTERNS` (matched by index)
3. Patterns use helper functions: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`

### Balancing Difficulty
- Adjust `DIFFICULTIES` object: `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`
- Weapon fire rate: `fireDelay` in `update()`
- Enemy HP values in `ENEMY_TYPES`
- Boss base HP calculation in `spawnBoss()` (line 1046)

### Audio Tweaks
- Beep frequency, duration, volume in `sfx` object (e.g., `sfx.shoot = () => beep(760, 0.07, ...)`)
- Filter type and frequency parameters control tone character
- Sweep parameters (`sweepTo`) create pitch slides

## Key Data Structures

### Player
```js
{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }
```

### Bullet (player)
```js
{ x, y, w, h, speed, vx, pierce, laser? }
```

### Enemy
```js
{ x, y, w, h, speed, shootTimer, canShoot, pattern, type, hp, color, dark, score, weave, weaveSeed, baseX }
```

### Boss
```js
{ x, y, w, h, hp, maxHp, entering, phaseTimer, attackTimer, patternIndex, vx, design, telegraph }
```

### Particle (explosion)
```js
{ x, y, vx, vy, life, maxLife, color }
```

## Storage Keys
- `skyfighter.bestScore`: High score (number)
- `skyfighter.muted`: Audio mute state (0/1)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Motion effects toggle (0/1)

## Browser APIs Used
- **Canvas 2D**: All graphics rendering
- **WebAudio**: Sound synthesis (oscillators, filters, gain, noise)
- **localStorage**: Persistent settings/scores
- **Touch Events**: Mobile input (touches, identifiers)
- **requestAnimationFrame**: Game loop
- **matchMedia**: Reduced motion preference detection

## Testing & Verification
When making changes:
1. Test on desktop (keyboard/mouse) and mobile (touch) if possible
2. Check all three difficulty levels for balance
3. Verify boss patterns execute correctly (no stuck states)
4. Listen for audio regression (mute toggle, all SFX types)
5. Confirm localStorage persists after page reload
