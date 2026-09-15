# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 web application. The entire game is contained in `index.html` — a fully-playable web game with no build process or external dependencies. Open the file in any modern browser to run.

### Quick Start

```bash
# Open in browser (macOS)
open index.html

# Open in browser (Linux)
xdg-open index.html

# Or simply open the file directly in your browser
```

The game is immediately playable with no setup required.

## Architecture

### Single-File Design
- **index.html** contains all CSS styling and game logic (1670 lines)
- No external dependencies, CDNs, or build tooling
- Self-contained: can be deployed as a single file

### Rendering & Performance
- **Canvas 2D API** for all game rendering
- Adaptive quality system (`quality` variable: 0=full, 1=medium, 2=low) that dynamically scales particle count and visual effects based on frame time
- No external image/audio assets; all graphics are procedurally drawn with Canvas primitives

### Audio
- **WebAudio API** for procedurally synthesized sound effects
- No audio files; sounds are generated on-the-fly via oscillators, noise buffers, and filters
- `ensureAudio()` initializes audio context on first user interaction
- Mute state persisted in localStorage

### Input Handling
Three input methods coexist (keyboard/joystick always takes priority over mouse):
1. **Keyboard**: Arrow keys or WASD to move, Space to fire, X/Shift to bomb, P/Esc to pause
2. **Mouse**: Move pointer to steer (smooth glide to target), click to fire
3. **Touch**: Left zone drags for movement (virtual joystick), right zone has FIRE and BOMB buttons

## Core Game Systems

### Game State
- `player`: Position, size, weapon type, level, invulnerability timer, fire cooldown
- `enemies[]`: Array of active enemy ships with HP, behavior patterns, and shooting logic
- `bullets[]`: Player projectiles with pierce/laser properties
- `enemyBullets[]`: Enemy projectiles (both aimed and radial patterns)
- `boss`: Current boss ship (null when not active), with attack patterns and telegraph state
- `score`, `lives`, `stage`, `kills`, `chain`, `multiplier`: Game progress metrics
- `running`, `paused`, `gameOver`: State flags

### Difficulty Settings
Three difficulty presets stored in localStorage; each scales:
- `spawnBase` / `spawnMin`: Enemy spawn rate
- `enemySpeed`: How fast enemies move and shoot
- `enemyShootChance`: Probability enemies fire at player
- `lives` / `bombs`: Starting resources
- `bossHpMul`: Boss health multiplier

### Stage Progression
- `KILLS_PER_STAGE` (18) enemy kills trigger boss spawn after 1.2s delay
- After boss defeated, `stage++` and new enemies spawn
- 6 visual themes cycle across stages (sky gradients, clouds, sun colors)
- 6 bosses with unique designs and attack patterns

### Weapon System
Two weapons with 3 levels each:
1. **Vulcan** (default): Spread shot that widens at levels 2+
   - Level 1: 1 center bullet
   - Level 2: 2 side bullets
   - Level 3: 3-way spread
2. **Laser**: Piercing beams that split into multiple lanes at level 2+
   - Level 1: 1 center beam
   - Level 2: 3 beams (left, center, right)
   - Level 3: 4 beams
- Weapon power-ups (6% chance per kill) upgrade level or swap weapon

### Enemy Types
- **Grunt** (minStage 1): Basic red fighter, 34×34, 1 HP, 10 points
- **Interceptor** (minStage 1): Fast orange weaver, 26×26, 1 HP, 15 points; weaves horizontally
- **Tank** (minStage 2): Slow tanky purple unit, 46×46, 4 HP, 35 points; shows health bar

Enemies have two shoot patterns:
- **Spread**: Fires 3 bullets in an arc (fan pattern)
- **Aimed**: Fires at player position (with velocity matching)

### Boss Mechanics
- Each stage has a boss from `BOSSES[]` array with unique color scheme and `hpMul`
- 6 boss types cycle: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Each boss has 3 unique attack patterns in `BOSS_PATTERNS[]`
- Pattern switches happen after telegraph phase (22 frames of visual warning)
- Boss attacks include: radial bursts, aimed spreads, columns, spirals, homing bullets, teleport-and-fire

### Scoring & Combo System
- Base score per enemy type (10–35 points)
- Kill chain: Each enemy killed within 2.2 seconds increments `chain`, raising `multiplier` (1 + floor(chain/5), capped at 10)
- Chain breaks if player takes damage (resets multiplier to 1×)
- Boss kill grants 500 points

### Power-ups
- **Life** (6% spawn chance): Restore 1 HP (capped at max)
- **Bomb** (8% spawn chance): Add 1 smart bomb (capped at max)
- **Weapon** (20% spawn chance): Upgrade weapon level or cycle weapon

### Particle & Visual Effects
- **Explosions**: 3–14 particles (scaled by quality) radiating outward with color-coded trails
- **Screen shake**: Magnitude and duration based on event severity (hit, bomb, boss defeat)
- **Flash effects**: White flash on hit, brighter flash on bomb (reduced or disabled in reduced-motion mode)
- **Vignette pulse**: Red pulsing vignette when at 1 HP
- **Combo pop**: Large animated "X KILLS!" text when reaching kill milestones

## Key Functions & Systems

### Update Loop (`update(dt)`)
Central game logic called ~60 FPS:
1. Process input (keyboard, joystick, mouse steering)
2. Update player position (clamped to canvas)
3. Move bullets and enemy bullets
4. Spawn enemies based on difficulty timer
5. Update enemies: movement (weaving), shooting logic
6. Update boss: entering phase, telegraph, attack patterns
7. Collision detection: bullets→enemies, bullets→boss, enemies→player, bombs→all, power-ups→player
8. Score/kill tracking and chain multiplier

### Render Loop (`draw()`)
Draws in layers (back to front):
1. Sky gradient (3-color based on theme)
2. Sun glow (radial gradient)
3. Cloud layers (2 parallax speeds)
4. Player ship (with flame, laser aura, or invisible during invuln blink)
5. Player bullets (with glow)
6. Enemy bullets (red circles with glow)
7. Enemies (planes + health bars for tanks)
8. Boss ship
9. Power-ups (rotating colored squares)
10. Particles (fading circles)
11. Optional: mouse crosshair guide

Screen shake is applied via `ctx.translate()` with random offsets.

### Canvas Sizing
- Internal resolution: 480×720 (portrait aspect)
- Scales up to fit window (max 1.6× for large displays)
- Responsive to resize and orientation change events

### Audio Functions
- `beep(freq, dur, type, vol, opts)`: Synthesize tone with optional sweep, filter, pan
- `noiseBurst(dur, vol, opts)`: Generate filtered noise
- SFX library: `shoot`, `laser`, `explode*` (type-specific), `hit`, `bomb`, `powerup`, `bossTelegraph`, `bossDown`, `stageClear`, `uiClick`, `gameover`

### Game Flow
1. **Start screen overlay**: Difficulty selection, best score display, controls hint
2. **Game loop**: `resetGame()` → `update()` / `draw()` every frame
3. **Pause overlay**: Same UI, gameplay frozen
4. **Game over overlay**: Final stats (score, stage, new best), "Play Again" button
5. **Stage transitions**: Banner animation on new stage, enemy wave ramps up

## Modifying the Game

### Changing Difficulty Tuning
Edit `DIFFICULTIES` object (lines 485–489) to adjust spawn rates, enemy speed, lives, and boss HP.

### Adding New Enemy Types
1. Add entry to `ENEMY_TYPES` (lines 502–506) with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Update `spawnEnemy()` selection logic (line 1026) if needed
3. Add explosion sound effect to `EXPLODE_SFX` map (line 678)

### Adding New Boss Patterns
1. Create attack pattern function in the pattern library (lines 1324–1370) using firing helpers: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`
2. Add 3-pattern array to `BOSS_PATTERNS` (lines 1372–1409)
3. Add new boss design to `BOSSES` array (lines 508–515) with `name`, hull colors, core color, `hpMul`

### Adjusting Visual Themes
Edit `THEMES` array (lines 493–500) to customize sky gradients, cloud colors, sun colors, and theme names per stage.

### Tweaking Visual Effects
- **Particles**: Adjust spawn count in `spawnExplosion()` (line 1074)
- **Screen shake**: Modify calls to `screenShake(magnitude, duration)`
- **Flash timing**: Change animation durations in CSS `@keyframes`
- **Glow effects**: Toggle via `glow` variable based on quality setting

### Weapon Balancing
- **Spread widths** (line 1127): Adjust angles for vulcan spread
- **Fire rates** (line 1177): Change cooldown between shots
- **Bullet speeds** (lines 1130, 1140): Faster projectiles
- **Laser pierce values** (line 1141): How many enemies one laser passes through

## Storage & Persistence
All data stored in localStorage with keys prefixed `skyfighter.`:
- `bestScore`: Best game score
- `muted`: Audio mute state
- `difficulty`: Last selected difficulty
- `reducedMotion`: Accessibility preference override

## Accessibility Features
- Screen reader announcements via `announce(text)` updates `#srLive` element
- Reduced motion preference via `prefers-reduced-motion` media query
- Toggle reduced motion button to disable animations and screen shake
- Keyboard full support (mouse optional)
- High contrast visuals suitable for various visual impairments

## Testing & Debugging
- Open browser DevTools (F12) → Console for any JavaScript errors
- `quality` variable: Set to 0/1/2 to force full/medium/low graphics
- `DIFFICULTIES` object editable at runtime to test difficulty changes
- Try all three difficulty levels to verify spawn and enemy behavior
- Test boss patterns by playing to stage 2+
- Verify touch controls on mobile device or Chrome DevTools device emulation

## Performance Considerations
- Particle count is adaptive based on frame time; if FPS drops below ~40, particles reduce
- Boss telegraph glow uses `shadowBlur` which is expensive; disabled on low-quality settings
- Canvas scale is capped at 1.6× to keep rendering fast on large displays
- Input is event-driven; movement polling happens only during game update
- No image decoding, no external asset loading — fully CPU/WebAudio based
