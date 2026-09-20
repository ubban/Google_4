# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. A single-player arcade game where the player controls a fighter jet to destroy waves of enemies and defeat stage bosses. The game includes three difficulty levels (Easy, Normal, Hard), weapon variety, score multipliers based on kill chains, and procedurally generated visual themes.

The entire game is implemented as a single HTML file (`index.html`) with embedded CSS and JavaScript.

## Architecture

The game uses a classic game loop pattern with a single canvas-based rendering pipeline:

```
Game Loop → Update Game State → Render to Canvas → Request Next Frame
```

**Key Systems:**

1. **Game State Management** (line 864-869)
   - Player position, health, weapons, invulnerability timer
   - Enemy array with individual properties (position, type, HP, behavior)
   - Bullets (player and enemy), particles, power-ups
   - Score, lives, kills, stage progression, combo multiplier
   - Boss state when active

2. **Input Handling** (line 768-861)
   - Keyboard: Arrow keys/WASD for movement, Space for fire, X/Shift for bomb, P/Esc for pause
   - Mouse: Position-based steering and click-to-fire
   - Touch: Joystick on left side, fire/bomb buttons on right side for mobile

3. **Rendering** (line 1520-1638)
   - Canvas setup: base 480×720, scaled to fit window up to 1.6× (line 753-765)
   - Sky gradient with parallax clouds based on current stage theme
   - Draws: player plane, enemies, bullets, boss, particles, power-ups, HUD overlay
   - Visual quality automatically degrades if frame times exceed ~40 FPS (line 474-483)

4. **Game Systems**

   - **Update Loop** (line 1148-1302): Handles player movement, weapon fire, collision detection, spawning
   - **Enemy Spawning** (line 1022-1041): Enemies spawn based on difficulty settings; tank/interceptor types appear at stage 2+
   - **Difficulty Scaling** (line 485-489): Three presets control spawn rate, enemy speed, enemy fire chance, lives, bombs, boss HP
   - **Stage Progression** (line 1204-1207): After 18 enemy kills, a boss spawns; defeating a boss advances to the next stage
   - **Boss Patterns** (line 1372-1409): Each of 6 bosses has unique attack patterns (radial bursts, aimed spread, columns, homing, etc.)
   - **Weapon System** (line 1124-1146): Vulcan (spread shots) and Laser (pierce shots); upgrades at 3 levels per weapon
   - **Combo Multiplier** (line 1087-1101): Kill within 2.2s of last kill to build chain; multiplier (1×–10×) applies to all scores
   - **Power-ups** (line 1058-1067): Life restore, bomb restore, weapon upgrade; spawn randomly when enemies die
   - **Audio** (line 542-676): WebAudio synthesis; procedurally generated SFX for shoot, explosions, boss patterns, UI feedback

## Common Development Tasks

### Adjusting Difficulty
Edit the `DIFFICULTIES` object (line 485-489):
- `spawnBase` / `spawnMin`: Base/minimum frames between enemy spawns
- `enemySpeed`: Multiplier on base enemy speed
- `enemyShootChance`: Probability an enemy can shoot (0–1)
- `lives` / `bombs`: Starting resources
- `bossHpMul`: Boss HP multiplier

### Adding/Modifying Enemies
Edit `ENEMY_TYPES` (line 502-506):
- `w` / `h`: Collision box
- `hp`: Hit points (removed when hp ≤ 0)
- `speedMul` / `color` / `dark`: Visual and behavioral properties
- `score`: Points awarded on kill
- `minStage`: First stage this enemy appears
- `weave`: Sine-wave movement pattern

To spawn a custom type, update `spawnEnemy()` (line 1022-1041).

### Adjusting Boss Behavior
- **Boss HP & Visuals** (line 508-515): Edit `BOSSES` array for name, hull colors, core glow, HP multiplier
- **Attack Patterns** (line 1372-1409): Each boss has 3 pattern functions called cyclically; patterns use helpers: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`
- **Attack Timing** (line 1421-1434): Telegraph delay before pattern fires is 22 frames; attackTimer between patterns is 85 frames

### Adjusting Stage Themes
Edit `THEMES` array (line 493-500):
- `sky`: [top, mid, bottom] gradient colors (hex)
- `cloud` / `sun`: RGB components for parallax cloud and sun glow
- `name`: Display name shown on stage entry

### Tweaking Weapon Behavior
- **Vulcan** (line 1126-1134): Spread pattern at levels 2–3; fire delay 0.15s
- **Laser** (line 1135-1145): Pierce property removes on spray or after N hits; fire delay 0.1s; visual aura when active

Edit fire delay, spread angles, speeds, and pierce values.

### Sound & Audio
- Audio synthesis happens in `beep()` (line 566-595) and `noiseBurst()` (line 596-623)
- Each SFX is defined in `sfx` object (line 624-676)
- Mute button toggles `muted` flag; stored in localStorage
- To silence a sound, set `muted = true` in `ensureAudio()` or remove the call

### Screen Shake & Flash Effects
- `screenShake(magnitude, duration)` (line 727-730): Applied to collision hits, bomb use, boss defeat
- Flash effects (line 698-709, 1107-1108): Used for hit feedback and bomb flash
- Reduced motion preference (line 520-540): Respects system `prefers-reduced-motion`; dampens all shake/flash/animation

### Particle System
- Particles spawn on hit/explosion via `spawnExplosion(x, y, color, count)` (line 1073-1085)
- Each particle is a small circle expanding and fading outward
- Quality setting (line 474) scales particle count on lower-end devices

### Local Storage Keys
- `skyfighter.bestScore`: High score persistence
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Accessibility preference

## Key Game Constants

- `BASE_W / BASE_H`: Canvas base size (480×720)
- `KILLS_PER_STAGE`: 18 kills trigger boss spawn
- `STORAGE_*`: localStorage keys for persistence
- Enemy bullet radius used for collision: `w / 1.6` (line 1598)
- Boss damage per hit: 3 HP (line 1259); per bomb: 24 HP (line 1117)

## File Structure

The entire project is contained in `index.html`:
- **Lines 1–359**: HTML & CSS (layout, animations, responsive design, dark mode)
- **Lines 422–676**: Audio system (synthesis, SFX library)
- **Lines 680–750**: UI updates (HUD rendering, button handlers)
- **Lines 752–862**: Input handling (keyboard, mouse, touch)
- **Lines 863–1302**: Game state & update logic
- **Lines 1304–1438**: Boss AI, power-ups, boss patterns
- **Lines 1439–1505**: Game flow (pause, game over, stage progression)
- **Lines 1520–1652**: Rendering pipeline
- **Lines 1655–1666**: Initialization & event loop

## Performance Considerations

- Quality level auto-tunes based on frame time (target ~40 FPS minimum)
- Particle count scales with quality setting
- Shadow/glow effects disabled at lower quality levels
- Canvas size clamped to 1.6× scale to prevent excessive rendering on large displays
- Collision detection uses fast AABB (axis-aligned bounding box) via `rectsOverlap()` (line 1069-1071)

## Testing Notes

- Use browser DevTools to throttle CPU or check frame rate
- localStorage is cleared if you want to reset high scores/settings
- Touch testing: emulate mobile in DevTools or test on actual device
- Difficulty testing: Switch difficulties before starting a run; settings persist
