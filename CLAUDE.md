# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file HTML5 game — a Raiden-style vertical shoot-'em-up with procedurally generated audio. The entire game (HTML, CSS, JavaScript, and WebAudio synth) is contained in `index.html` (~1700 lines).

### Key Features
- Canvas-based 2D rendering with gradient skies and parallax clouds
- Six progressive stages, each with a unique boss encounter
- Weapon system: Vulcan (spreads at higher levels) and Laser (piercing shots)
- Difficulty settings: Easy, Normal, Hard (affects spawn rate, enemy speed, boss HP, lives/bombs)
- Mobile-first responsive design with touch controls (joystick + fire/bomb buttons)
- Adaptive performance: quality scaling (0=full, 1=medium, 2=low) based on frame time
- Accessibility: screen-reader announcements, prefers-reduced-motion support, mute toggle
- Local storage: persists best score, difficulty, mute state, reduced motion preference

## Architecture

The game follows a classic game loop pattern with state management and immediate-mode rendering:

```
Initialization (resetGame)
  ↓
Loop (requestAnimationFrame)
  ├─ Update (input, physics, collisions, AI)
  ├─ Render (draw to canvas)
  └─ Repeat
```

### Core Systems

**Player & Input**
- Keyboard (Arrow keys / WASD for movement, Space for fire)
- Mouse (pointer tracks, click fires)
- Touch (left 55% of screen for joystick, right 45% for action buttons)
- Priority: keyboard/joystick > mouse (prevents fighting between input methods)

**Game State**
- Global state: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Gameplay variables: `score`, `lives`, `stage`, `kills`, `chain` (for multiplier), `multiplier`
- UI state: `running`, `paused`, `gameOver`

**Enemy & Boss AI**
- Enemies spawn on a timer that accelerates each 300 frames (difficulty-dependent spawn base)
- Enemy types: `grunt` (1 HP, fast), `interceptor` (1 HP, very fast with weaving), `tank` (4 HP, slow, fires less)
- Enemies shoot in two patterns: `spread` (5-shot fan) or `aimed` (homing toward player)
- Bosses enter from top, then patrol side-to-side, telegraph attacks (22 frame wind-up), then fire patterns
- Six boss designs with unique attack patterns (radial bursts, homing volleys, teleport attacks, etc.)

**Weapons & Powerups**
- Vulcan: spread bullet pattern (1-3 lanes based on level), 150ms fire rate
- Laser: piercing shots, faster 100ms fire rate, 1-3 lanes based on level
- Powerups (6% life, 8% bomb, 20% weapon): spawn randomly on enemy death, descend toward bottom
- Weapon progression: Vulcan/Laser Lvl 1 → Lvl 2 → Lvl 3 → switch to opposite weapon, reset to Lvl 1

**Collision & Scoring**
- Simple AABB overlap detection (`rectsOverlap` function)
- Enemy destruction: player bullet hit (costs pierce or destroys bullet) or bomb (instant kill all)
- Boss destruction: 3 damage per player bullet, 24 damage per bomb
- Scoring system: base score × multiplier (1 to 10x, increases every 5 kills in a chain)
- Chain breaks if no enemy dies for 2.2 seconds

**Audio (WebAudio Synth)**
- No external files; all sounds are synthesized real-time using Web Audio API
- Oscillators (sine/square/sawtooth) + noise bursts + filters + panning
- Sfx categories: shoot, laser, explode (grunt/interceptor/tank), hit, bomb, powerup, boss (hit/telegraph/down), stage clear, UI click, gameover
- Dynamic pan based on screen X position for spatial audio

**Visual Effects**
- Screen shake (magnitude and duration, reduced under reduced-motion mode)
- Flash overlay on hit (0.25s) or bomb (0.5s)
- Combo pop-up at center screen (shows kill milestones every 10 kills)
- Stage banner announcement (2s fade in/out)
- Vignette pulse when health is critical (1 life remaining)
- Particle explosion system: 3-22 particles per entity, varies by quality setting

**Rendering Pipeline**
- Sky gradient (6 themes, cycling per stage)
- Sun radial gradient overlay
- Parallax clouds (two layers, different speeds)
- Player ship (with glow aura for laser weapon)
- Bullets (with glow shadows on high quality)
- Enemy ships (plane shape with type-specific colors)
- Enemy bullets (circular red projectiles)
- Boss ship (complex multi-part design with core, hull, wings, animations)
- Powerups (rotating squares, glow effect)
- Particles (fading circles)
- Mouse crosshair (if mouse is active and game is running)

## Development

### Quick Start
1. Open `index.html` in a web browser (no build step required)
2. The game runs immediately; press **Start Game** to begin

### Common Tasks

**Test Single Feature**
- Weapon system: Start game, destroy enemies to collect weapon powerups (3rd tier opens weapon switch)
- Boss encounters: Reach stage 2+ (18 enemy kills to trigger boss spawn warning)
- Difficulty: Select from main menu before pressing Start Game
- Mobile controls: Use Chrome DevTools device emulation (Ctrl+Shift+M) or test on actual device

**Debug Input**
- Keyboard/mouse: Check `keys` object and `mouseActive` flag in the input section
- Touch: Debug joystick via `joyVec` and `joyActive`
- Test both: Open DevTools → Sensors tab, enable "Emulate device with touch events"

**Adjust Game Balance**
- Difficulty parameters: `DIFFICULTIES` object (lines 485-489)
- Enemy types: `ENEMY_TYPES` object (lines 502-506)
- Boss patterns: `BOSS_PATTERNS` array (lines 1372-1409); each boss maps to a pattern set
- Spawn timing: `spawnEnemy()` function controls spawn delay and enemy composition
- Weapon properties: `fireWeapon()` function (bullet speed, spread, cooldown)
- Boss HP: `spawnBoss()` formula: `baseHp = 70 + stage * 35`, multiplied by difficulty and boss design

**Modify Visuals**
- Colors: Search `THEMES` array for sky/cloud/sun values; find enemy `color`/`dark` in `ENEMY_TYPES`
- Canvas size: `BASE_W = 480, BASE_H = 720` (line 753); `resize()` maintains aspect ratio
- Drawing functions: `drawPlane()` (player/enemies), `drawBossShip()` (boss), `draw()` (main render loop)

### Code Organization

**Global Constants (lines 485–516)**
- `DIFFICULTIES`: spawn rates, enemy speeds, shoot chances, lives/bombs, boss HP multiplier
- `KILLS_PER_STAGE`: 18 kills trigger boss warning
- `THEMES`: 6 sky/cloud color palettes cycling per stage
- `ENEMY_TYPES`: grunt, interceptor, tank with HP, speed, score, appearance
- `BOSSES`: 6 boss designs (names, colors, hull shapes, core, HP multiplier)

**Initialization & Setup (lines 517–762)**
- Storage keys, best score load, difficulty/mute/motion defaults
- Canvas sizing and resize handler
- HUD/overlay element references

**Input Handling (lines 767–861)**
- Keyboard listener, touch joystick tracking, mouse steering
- Priority logic ensures keyboard/joystick override mouse

**Game Loop (lines 1148–1652)**
- `update(dt)`: player movement, shooting, enemy AI, collisions, scoring, boss behavior
- `draw()`: canvas clear, background gradients, entity rendering with effects
- `loop(now)`: requestAnimationFrame callback, delta-time tracking, performance tracking

**Collision & Gameplay (lines 1069–1322)**
- Bullet-enemy, bullet-boss, enemy-player, bomb-all interactions
- Powerup pickup and application
- Score/chain/multiplier system
- Boss destruction and stage progression

**Audio & Visual Effects (lines 542–730)**
- WebAudio synthesis setup and sfx catalog
- Screen shake, flash, vignette, particle spawning
- Motion preference handling

### Performance Considerations

- **Quality scaling**: Frame time tracking adjusts particle count and shadow blur
- **Particle limits**: Explosion particle count scales: 100% full quality, 60% medium, 35% low
- **Canvas rendering**: Direct 2D context calls; no off-screen buffering
- **Collision checks**: O(N²) bullet-vs-enemy, justified by typical entity counts (~30 enemies max)
- **Animation frames**: Reduced-motion CSS animations disabled; JavaScript fallbacks omit shake/flash

### Browser Compatibility

- **Required**: HTML5 Canvas 2D, ES6+ (arrow functions, const/let, template literals)
- **Optional**: Web Audio API (graceful fallback if unavailable or suspended)
- **Mobile**: Touch events, `prefers-reduced-motion` media query, viewport-fit safe area

## Customization & Extension Points

**Add a New Boss**
1. Add entry to `BOSSES` array with name, colors, hpMul
2. Create attack pattern functions (e.g., `fireRadial`, `fireAimedSpread`)
3. Add pattern set to `BOSS_PATTERNS` array at matching index

**Add a New Enemy Type**
1. Add entry to `ENEMY_TYPES` with w, h, hp, speedMul, color, dark, score, minStage, optional weave
2. Update `spawnEnemy()` probability checks if needed (lines 1028–1030)

**Add a Sound Effect**
1. Create synthesis function in `sfx` object using `beep()` and/or `noiseBurst()`
2. Call it from relevant game event (e.g., `sfx.uiClick()` on button press)
3. Check muted state automatically (both functions return early if muted)

**Add a New Difficulty**
1. Add entry to `DIFFICULTIES` object with appropriate parameters
2. New difficulty appears in menu automatically (`.diffBtn` buttons loop from data attributes)
3. Difficulty persists via localStorage

## Storage & Persistence

Local storage keys (configurable at top):
- `skyfighter.bestScore`: number (best score achieved)
- `skyfighter.muted`: '1' or '0' (audio mute state)
- `skyfighter.difficulty`: 'easy'|'normal'|'hard'
- `skyfighter.reducedMotion`: '1'|'0' (overrides prefers-reduced-motion)

## Testing Checklist

- **Gameplay**: All 6 difficulties work; boss patterns fire correctly; weapon upgrades function
- **Collision**: Player bullets hit enemies; bombs clear screen; enemy fire hurts player; powerups collectable
- **Mobile**: Joystick movement, fire button, bomb button, landscape/portrait rotation
- **Accessibility**: Screen reader announcements (lives, stage, score changes); keyboard works fully; reduced motion disables animations
- **Performance**: Maintains ~60fps on low-end devices (Moto G or iPad Air 2 baseline)
- **UI**: Score/multiplier/lives/bombs update; pause works; game-over screen shows new best
