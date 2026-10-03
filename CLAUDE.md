# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in vanilla JavaScript with Canvas 2D rendering. It's a single-page application (index.html) with no external dependencies, featuring procedurally-generated boss patterns, multiple difficulty levels, and Web Audio API-based sound effects.

## Running the Game

- Open `index.html` directly in any modern browser (Chrome, Firefox, Safari, Edge)
- No build step, no server, no dependencies required
- Responsive design supports both desktop (mouse/keyboard) and mobile (touch controls)
- Game state and settings persist in localStorage

## Architecture Overview

The entire game runs in a single ~1650-line vanilla JavaScript IIFE (Immediately Invoked Function Expression) inside `index.html`. The architecture follows a classic game loop pattern:

### Core Game Loop
- **Frame-based update**: `update(dt)` modifies game state based on elapsed time
- **Rendering**: `draw()` renders all game objects to canvas
- **requestAnimationFrame**: Runs at ~60fps, with delta-time clamping at 0.05s to prevent huge jumps
- Located at the bottom of the script: `loop(now)` calls update/draw repeatedly

### Game State Management
All state is stored in module-level variables (not in objects):
- **Player state**: `player` object with position, speed, weapon, level, health
- **Entity collections**: `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- **Game flags**: `running`, `paused`, `gameOver`
- **Stage/progression**: `stage`, `kills`, `stageKills`, `score`, `multiplier`, `chain`
- **Input state**: `keys` object for keyboard, `firing` flag, `mouseActive`, joystick vectors

Key reset function: `resetGame()` initializes all state when the game starts.

### System Breakdown

**Input System**
- Keyboard: Arrow keys, WASD for movement; Space for fire; X/Shift for bombs; P/Esc to pause
- Mouse: Hover to steer, click to fire (on canvas only)
- Touch: Left side drag zone for movement, right side has FIRE and BOMB buttons
- Joystick visualization appears on mobile when touching left area
- All input integrates into movement vector `(mvx, mvy)` and firing/bomb flags

**Collision Detection**
- Simple AABB (axis-aligned bounding box): `rectsOverlap(a, b)` checks center distance
- Applied to: enemy/bullet hits, player/enemy hits, player/bullets, player/boss, powerups
- Boss only collides after `b.entering` animation completes

**Entity Management**
- Enemies spawn with randomized patterns: "spread" or "aimed" firing
- Three enemy types (grunt/interceptor/tank) with different stats, unlock based on stage
- Enemies despawn when off-screen; bullets/bombs handled via filtering
- Tank enemies have visible HP bars drawn above them

**Boss System**
- Boss spawns after `KILLS_PER_STAGE` (18) regular kills, with animation entering from top
- 6 boss designs in `BOSSES` array, cycling per stage with unique aesthetics
- 6 attack pattern sets in `BOSS_PATTERNS`, each boss has 3 patterns that repeat
- Patterns include: radial bursts, aimed spreads, columns, homing, spirals, etc.
- Telegraph mechanic: brief visual warning before each attack pattern fires
- Boss HP scaled by difficulty multiplier and design-specific HP multiplier

**Difficulty System**
- Three difficulties (easy/normal/hard) in `DIFFICULTIES` object
- Affects: spawn rates, enemy speeds, shoot frequency, starting lives/bombs, boss HP
- Selected via difficulty buttons before game start
- Stored in localStorage; persists across sessions

**Weapon System**
- Two weapons: Vulcan (multi-spread bullets) and Laser (piercing, multi-lane)
- Each has 3 levels; leveling up increases spread/lanes
- Weapon powerup cycles: level 1→2→3, then switches weapon and resets to level 1
- Fire rate: 0.15s for Vulcan, 0.1s for Laser (via `player.cooldown`)

**Scoring & Chain Multiplier**
- Base score depends on enemy type (grunt=10, interceptor=15, tank=35, boss=500)
- Kill combo: every kill within 2.2s increases `chain`, multiplier caps at 10x
- Chain display updates in HUD; breaks on taking damage or timeout
- Bomb kills also add to chain

**Audio System**
- Uses Web Audio API (no external files) with synth-generated sound effects
- `ensureAudio()` initializes AudioContext on first interaction
- `beep()` for pitched sounds (oscillator + optional filter/pan/sweep)
- `noiseBurst()` for noise (white noise buffer with ADSR envelope)
- Effects: shoot, laser, explosions (grunt/interceptor/tank variants), hit, bomb, powerup, boss telegraph/down, stage clear, UI click, game over
- Master gain + compressor for audio polish
- Mute state persists in localStorage

**Rendering & Themes**
- Canvas: 480×720 base resolution, scaled to fit window (max 1.6× zoom)
- 6 rotating themes per stage with different sky gradients and cloud colors
- Theme data: `THEMES` array with sky gradient stops, cloud color, sun color
- Screen shake applied via canvas translation when taking damage or bomb detonates
- Particles for explosions: created with velocity/gravity simulation, fade to transparent
- Flash effects for hits/bombs via CSS animation triggers
- Performance adaptive: `quality` variable (0/1/2) reduces particle count and disables glow at lower quality

**UI & Overlays**
- HUD shows: Score, Best Score, Multiplier, Weapon type/level, Hearts (lives), Bombs
- Boss health bar appears when boss is active
- Stage banners animate in at stage transitions and boss warnings
- Combo popups show kill count milestones
- Main overlay handles: title screen (difficulty selection), pause menu, game over screen
- Accessibility: screen reader live region (`srLive`) announces key events

**Mobile Responsiveness**
- Touch-enabled by default if device detected (`ontouchstart` or `maxTouchPoints > 0`)
- Safe area insets for notches/home indicators via CSS env() variables
- Reduced motion preference respected via `prefers-reduced-motion` media query
- Can toggle in-game via motion button (🎆/🚫)
- CSS grid/flexbox layout adapts to viewport

## Key Code Patterns

### Difficulty & Enemy Spawning
```javascript
const d = DIFFICULTIES[difficulty];
spawnEnemy(); // Picks enemy type based on stage, spawn timer decreases each frame
```
Spawn timer ramped up over time: `Math.max(d.spawnMin, d.spawnBase - difficultyRamp)`

### Collision Loop (example: bullet vs enemy)
```javascript
for (let i = enemies.length - 1; i >= 0; i--) {
  for (let j = bullets.length - 1; j >= 0; j--) {
    if (rectsOverlap(enemies[i], bullets[j])) {
      // Handle hit: reduce HP, create particles, remove bullet/enemy if dead
    }
  }
}
```
Always iterate backwards when removing items to avoid index shifts.

### Boss Attack Patterns
Patterns are functions that fire bullets into `enemyBullets` array with radial or aimed properties:
- `fireRadial(b, count, speed)` - ring of bullets around boss
- `fireAimedSpread(b, count, speed, angle)` - fan aimed at player
- `fireColumns(b, speed)` - vertical columns
- etc.

### Frame-Based Animation
Use `frame` counter (incremented each update) for time-based effects:
```javascript
const t = frame * 0.15; // Scaling factor for smooth cycles
ctx.rotate(Math.sin(t + i) * 0.15); // Smooth rotation
```

## Common Development Tasks

### Add a New Enemy Type
1. Define in `ENEMY_TYPES` object with stats: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Optionally add a `weave` property for sinusoidal movement
3. Add explosion sound variant to `sfx` object if needed, update `EXPLODE_SFX` mapping
4. Enemy spawning already filters by `minStage`, so it auto-unlocks at the right stage

### Add a New Boss Pattern
1. Create a function in scope that fires bullets (e.g., `fireAimedSpread`, `fireRadial`)
2. Add 3 attack functions to a new entry in `BOSS_PATTERNS` array (index corresponds to boss index)
3. Pattern functions receive the boss object `b` and fire into `enemyBullets`
4. Telegraph delay happens automatically via boss state machine

### Adjust Game Balance
- **Difficulty**: Modify `DIFFICULTIES` object (spawn rate, enemy speed, shoot chance, lives, bombs, boss HP)
- **Enemy stats**: Edit `ENEMY_TYPES` values
- **Boss HP**: Base formula in `spawnBoss()` uses `baseHp = 70 + stage * 35` × difficulty multiplier × design multiplier
- **Weapon balance**: Fire rates in `update()` function, spread/lanes in `fireWeapon()`
- **Spawn rate**: `spawnBase` and `spawnMin` in `DIFFICULTIES`

### Modify Visual Theme
- **Sky gradient**: Edit `THEMES` array, change `sky` array colors
- **Enemy colors**: Change `color` and `dark` fields in `ENEMY_TYPES`
- **Boss colors**: Edit `BOSSES` array color fields
- **Particle effects**: Modify `spawnExplosion()` parameters or the particle color passed to it
- **Player appearance**: Edit `drawPlane()` function to change shape/colors

### Add Persistent Settings
1. Add a localStorage key constant at top: `const STORAGE_XXX = 'skyfighter.xxx'`
2. Read with: `localStorage.getItem(STORAGE_XXX)`
3. Write with: `localStorage.setItem(STORAGE_XXX, value)`
4. Current settings: bestScore, muted, difficulty, reducedMotion

## Performance Notes

- **Adaptive quality**: `quality` variable automatically degrades particle count and glow effects if FPS drops
- **Frame time tracking**: Last 45 frames averaged; if avg > 1/40 fps, quality increases; if < 1/56 fps, quality decreases
- **Delta-time clamping**: `dt = Math.min(dt, 0.05)` prevents massive jumps on lag spikes
- **Collision: O(n²)** in worst case, but typically small entity counts keep it fast
- **Canvas rendering**: No off-screen canvas optimization; single draw pass per frame
- **Sound**: Audio context initialized only once and reused; suspended until first user interaction

## Git Development Notes

- Work on branch `claude/inspiring-planck-rdfir9` for feature development
- Single-file project means all changes are in `index.html`
- Large IIFE means code is self-contained; no separate module files to coordinate
- Test changes by opening `index.html` in a browser and playing through scenarios
- Commit messages should reference what mechanic changed (e.g., "Adjust boss attack frequency" or "Add new enemy type")

## Testing Strategy

No automated test framework; validation is manual:
1. Open `index.html` in browser
2. Test all three difficulties via difficulty buttons
3. Play through multiple stages to verify progression
4. Test mobile via Chrome DevTools device emulation (both portrait and landscape)
5. Verify audio by toggling mute button
6. Test pause/resume functionality
7. Verify localStorage persistence by refreshing and checking best score/difficulty persistence
8. Check console for errors (Ctrl+Shift+K or F12 → Console)

## Important Implementation Details

- **No external dependencies**: Game must remain browser-runnable without build steps
- **Accessibility**: Screen reader announcements via `announce()` function; support reduced motion preference
- **Responsive**: Scaling factor applied to canvas based on viewport; safe area insets for mobile
- **Deterministic RNG**: Uses Math.random(); no seed control, so boss spawn positions vary each run
- **localStorage is the only persistent state**: Best score, settings stored there; nothing in memory survives page reload
