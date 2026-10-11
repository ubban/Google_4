# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely as a single HTML file. Players control a fighter jet, dodge enemy fire, collect weapon power-ups, and defeat boss squadrons across multiple stages. The game features:

- Multiple difficulty levels (Easy, Normal, Hard)
- Two weapon types: Vulcan (rapid-fire) and Laser (piercing)
- Smart bombs for area damage
- Combo/chain system with score multipliers
- 6 unique visual themes with different sky gradients
- Boss encounters with unique attack patterns
- Responsive design for desktop and mobile touch controls
- Web Audio API for procedurally generated sound effects

## Architecture

The entire game is implemented in `/index.html` as a single ~1669-line file organized in the following logical sections:

### Code Organization (by order of appearance in index.html)

1. **DOM Elements & Constants (lines 422-520)**
   - Canvas and UI element references
   - Game configuration constants: difficulties, enemy types, bosses, themes
   - Storage keys for persistence (best score, mute state, difficulty, reduced motion preference)

2. **Audio System (lines 545-678)**
   - Web Audio API context initialization (`ensureAudio()`)
   - Sound effect generation: `beep()`, `noiseBurst()` functions
   - SFX objects: all procedurally generated (shoot, laser, explosions, powerups, boss hits, etc.)
   - Dynamic stereo panning based on screen position

3. **UI Rendering Functions (lines 680-762)**
   - HUD updates: hearts, bombs, score, multiplier display
   - Screen shake and visual feedback (flash animations)
   - Combo pop notifications and stage banners
   - Canvas responsive sizing

4. **Input Handling (lines 768-862)**
   - Keyboard input: arrow keys, WASD, Space, X, Shift for bombs, P/Esc for pause
   - Touch controls: joystick for movement (left side), fire/bomb buttons (right side)
   - Mouse input: player glides toward cursor while maintaining keyboard/joystick priority
   - Touch and mouse event listeners with preventDefault for mobile support

5. **Game State & Core Game Loop (lines 863-900 initialization, then main update)**
   - Global state: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
   - Game flags: `running`, `paused`, `gameOver`
   - Spawn management: `spawnTimer`, `frame` counter, `kills`, `stage`, `stageKills`
   - Chain/combo system: `chain`, `chainTimer`, `multiplier`

6. **Collision & Spawn Functions (varies)**
   - `rectsOverlap()` - AABB collision detection
   - `spawnEnemy()` - creates enemies from ENEMY_TYPES based on difficulty
   - `spawnBoss()` - transitions to boss encounter after KILLS_PER_STAGE (18) enemies killed
   - `maybeSpawnPowerUp()` - random weapon/rapid-fire powerups from defeated enemies
   - `fireWeapon()` - shoots bullets based on player.weapon and player.level
   - `useBomb()` - area-damage attack, applies knockback to enemies

7. **Update Function (lines 1148-1410)**
   - Player movement with boundary clamping
   - Input polling and firing logic
   - Bullet physics and removal
   - Enemy spawning with difficulty scaling
   - Enemy movement (straight or weaving) and AI shooting
   - Collision detection between player bullets ↔ enemies and enemy bullets ↔ player
   - Damage, score, and chain/combo calculation
   - Invulnerability frames after being hit

8. **Boss System (lines 1411-1650)**
   - `updateBoss()` - boss-specific AI, unique attack patterns per boss
   - Boss attack patterns: straight beams, spreads, homing, weaving
   - Phase-based mechanics (e.g., tanks spawn more enemies when destroyed)
   - Boss defeat triggers stage completion and next stage spawn

9. **Render Function (lines 1650+)**
   - Canvas clear and background gradient (theme-based)
   - Particle rendering
   - Player/enemy/boss sprite drawing with rotation/scaling
   - Bullet rendering
   - HUD updates
   - Screen shake offset application

10. **Game Loop (lines 1652-1665)**
    - `requestAnimationFrame(loop)` - 60 FPS delta-time based update
    - `deltaTime` calculation from `lastTime`
    - Adaptive performance tracking (quality variable adjusts rendering based on FPS)
    - Pause state handling

### Key Game Systems

**Difficulty Scaling**
- `DIFFICULTIES` object defines: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier
- Spawn timer decreases over time (ramp calculation every 300 frames adds 4 difficulty points)

**Weapon System**
- Vulcan: rapid-fire, 0.15s cooldown, limited pierce (2-3 shots)
- Laser: longer range, 0.1s cooldown, high pierce, can pass through multiple enemies
- Powerups upgrade weapon level (1-4), applying damage multiplier to base bullet

**Chain/Combo System**
- Destroying enemies without being hit extends chain (chainTimer reset)
- Chain counter = combo multiplier applied to enemy scores
- Chain breaks after 3 seconds of no kills
- Displayed via combo pop animation in center

**Boss System**
- 6 unique bosses, each with different sprites and attack patterns
- Boss HP scales with selected difficulty
- Boss telegraph warning (visual flash + sound) before attacks
- Defeat displays stage clear banner and transitions to next stage
- Bosses appear after 18 enemy kills per stage

## How to Run Locally

Open `index.html` in a modern web browser:

```bash
# Using a simple HTTP server (Python 3)
python3 -m http.server 8000

# Using Node's http-server
npx http-server

# Or just open the file directly in a browser:
file:///path/to/Google_4/index.html
```

The game automatically detects touch input and loads the appropriate controls.

## Development Workflow

### Making Changes

1. Edit `/index.html` directly
2. Refresh the browser to see changes immediately (no build step required)
3. Use browser DevTools (F12) to debug:
   - Canvas rendering in the Elements panel
   - JavaScript errors in the Console
   - Performance profiling with Ctrl+Shift+P → "Show Rendering"

### Common Tasks

**Adding a New Sound Effect**
- Define function in the `sfx` object (around line 624)
- Use `beep(freq, duration, waveType, volume, options)` or `noiseBurst()` primitives
- Call `ensureAudio()` to initialize context if needed
- Options: `{ delay, sweepTo, filterType, filterFreq, pan }`

**Creating a New Enemy Type**
- Add entry to `ENEMY_TYPES` object (around line 502) with properties: `w`, `h`, `hp`, `score`, `sprite`, optional `pattern`, `weave`, `canShoot`
- Add sprite rendering logic in render function (search for `case 'grunt'` around line 1580+)
- Enemy sprites are drawn with `ctx.fillRect()` and colored fills

**Adding a New Boss**
- Add to `BOSSES` array (around line 508) with: `name`, `hp`, `sprite`
- Implement attack pattern in `updateBoss()` function (around line 1411)
- Example: use `bossShootTimer`, `bossAttackPhase` to trigger pattern changes
- Add render case in the render function

**Modifying Difficulty Parameters**
- Edit `DIFFICULTIES` object (around line 485)
- Properties control enemy spawn timing, speed, shoot frequency, player lives/bombs, boss scaling
- All difficulty values are relative; easy = easier, hard = harder

**Adjusting Visual Themes**
- Edit `THEMES` array (around line 493)
- Each theme defines: `sky` (3-stop gradient array), `cloud`, `sun` RGB colors, `name`
- Theme is selected randomly per game; modify `theme = THEMES[stage - 1]` in `resetGame()` to make it deterministic

### Testing

The game has no automated tests. Manual testing checklist:

- **Gameplay**: Play through Easy, Normal, Hard difficulties; verify difficulty scaling works
- **Input**: Test keyboard (arrows, WASD, Space), mouse (hover and click), mobile (touch joystick, buttons)
- **Bosses**: Reach stage 2+ to test boss spawning and defeat mechanics
- **Audio**: Verify SFX play correctly; test mute button; test reduced-motion setting
- **Pause**: Press P or Esc during gameplay; verify game state is frozen
- **Persistence**: Play, close tab, reopen; check high score, difficulty, mute state persist
- **Performance**: Check FPS in DevTools; verify quality adapts on low-end devices

### Performance Considerations

- **Adaptive Quality**: Frame rate is monitored; if FPS drops below 56, rendering quality downgrades (fewer particles, simpler effects)
- **Object Pooling**: Bullets, enemies, and particles are created on-demand and removed when off-screen
- **Reduced Motion**: Players with `prefers-reduced-motion` media query enabled get minimal animations; controlled by toggle button
- **Canvas Size**: Base render resolution is 480×720; CSS scaling ensures responsive display without changing game logic

## Technical Notes

### Browser APIs Used
- **Canvas 2D Context** (`ctx.fillRect()`, `ctx.fillText()`, `ctx.clearRect()`)
- **Web Audio API** (OscillatorNode, NoiseBuffer, Gains, Panners, Compressor)
- **Touch Events** (touchstart, touchmove, touchend, passive: false)
- **LocalStorage** (best score, settings persistence)
- **requestAnimationFrame** (game loop)
- **matchMedia** (prefers-reduced-motion detection)

### Key Algorithms

**Difficulty Ramp** (line 1201)
```
difficultyRamp = Math.floor(frame / 300) * 4
```
Increases spawn rate difficulty every 300 frames throughout the stage.

**Chain Timer Reset** (line 1185)
```
if (chainTimer <= 0) breakChain()
```
If no enemies killed for 3+ seconds, combo resets to 1x multiplier.

**Bullet Piercing** (line 1239)
```
if (!b.laser || b.pierce <= 0) { bullets.splice(j, 1) }
```
Laser bullets reduce pierce count instead of removing; vulcan removes on hit.

**Enemy AI Targeting** (lines 1221-1223)
```
const dx = player.x - en.x, dy = player.y - en.y
const dist = Math.hypot(dx, dy) || 1
vx = (dx / dist) * 150  // normalized direction × speed
```
Enemies aim directly at player position.

### Common Bugs & Pitfalls

- **Collision Detection**: `rectsOverlap()` uses AABB; rotated sprites may cause false negatives
- **Audio Context State**: Web Audio requires user gesture to unlock; `ensureAudio()` handles this
- **Touch Event Handling**: Must call `preventDefault()` on touchstart/touchmove; passive listeners break movement
- **Screen Shake**: Uses global `shakeTime` and `shakeMag`; applies offset to canvas transform
- **Boss Phase Transitions**: Check exact HP thresholds; off-by-one errors cause boss to get stuck in a phase

## Accessibility

- **Screen Reader**: Uses `#srLive` with `aria-live="polite"` for announcements
- **Reduced Motion**: Respects `prefers-reduced-motion` media query; toggle button toggles `body.reduced-motion` class
- **Color Contrast**: HUD text uses shadows and solid backgrounds for readability
- **Touch Targets**: Fire/Bomb buttons are 78×78px (easy to tap)
- **Keyboard Accessible**: All controls have keyboard equivalents; no mouse-only mechanics
