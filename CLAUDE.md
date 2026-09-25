# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML5 file. It's a browser-based game with procedurally generated levels, multiple difficulty modes, boss fights, and full mobile support.

## Running & Development

### Running the Game
- Open `index.html` directly in a web browser (no build tools required)
- Game runs completely client-side with no server dependencies
- Progress is saved to `localStorage`

### Testing/Verification
- Test across desktop (keyboard + mouse controls) and mobile (touch controls)
- Verify responsive design works at various viewport sizes
- Check audio works (WebAudio API synthesized SFX — no external assets)
- Test across browsers (Chrome, Firefox, Safari, Edge) as WebAudio API support varies slightly

### No Build Step Required
This is a single-file game. Changes to `index.html` take effect immediately on reload. No linting, compilation, or bundling needed.

---

## Game Architecture

### Core Game Loop
The main loop runs via `requestAnimationFrame` (line 1640). Each frame:
1. **Update**: Simulate player, enemies, bullets, particles, collisions (line 1148)
2. **Draw**: Render everything to canvas (line 1520)

The game is paused or running based on the `running` and `paused` state flags (line 865).

### Game State
All game state lives in the global scope of the IIFE (Immediately Invoked Function Expression). Key variables:
- `player`: player position, weapon, health, invulnerability
- `enemies`, `bullets`, `enemyBullets`, `particles`, `powerUps`: pooled entity arrays
- `boss`: current boss (null if no boss active)
- `score`, `lives`, `multiplier`, `chain`: gameplay metrics
- `difficulty`: easy/normal/hard (line 485, persisted to localStorage)

### Three Game States
1. **Menu**: Overlay visible, game not running. Player picks difficulty and clicks "Start Game"
2. **Playing**: Game running and not paused. Update and draw loop active
3. **Game Over**: Running flag false, overlay shows final stats

Pause is a sub-state of playing (toggle via P or Esc). When paused, update loop stops but overlay appears.

### Difficulty Configuration
Defined at line 485 as `DIFFICULTIES` object. Each difficulty affects:
- Enemy spawn rate and density
- Enemy speed, shoot frequency, aim accuracy
- Player lives and bomb count
- Boss HP multiplier

Difficulty setting persists to localStorage (line 519, key: `skyfighter.difficulty`).

---

## Game Systems

### Stage & Boss Progression
- Stages are numbered sequentially (line 1, 2, 3...)
- Each stage: kill 18 regular enemies (line 491), then a boss spawns (line 1204)
- Boss selection: cycles through 6 boss designs (line 508-515) based on stage % 6
- Each boss has unique attack patterns (line 1372-1409)
- On boss defeat, stage increments and theme/enemies refresh (line 1449-1453)

### Weapon System
Two weapon types (line 871-872):
- **Vulcan** (default): spread-shot at close range. Upgrades level 1→2→3 increase bullet count
- **Laser**: focused beam with pierce capability. Upgrades level 1→2→3 increase lanes
- **Power-ups**: `type: 'weapon'` cycles weapon or increases level (line 1311-1320)

### Scoring & Multiplier
- Kill multiplier: starts at 1x, increases by 1 for every 5 kills in the chain (line 1090)
- Chain timer: resets to 2.2s on each kill; if timer expires, chain/multiplier reset (line 1183-1186)
- Bomb kill: worth 5 × multiplier (line 1113)
- Boss kill: worth 500 × multiplier (line 1446)

### Collision Detection
Rectangle overlap check (line 1069):
```javascript
function rectsOverlap(a, b) {
  return Math.abs(a.x - b.x) < (a.w + b.w) / 2 && Math.abs(a.y - b.y) < (a.h + b.h) / 2;
}
```
Checked against enemies, enemy bullets, player, boss, power-ups (line 1232-1294).

### Enemy Spawning
- Wave-based: enemies spawn at intervals based on difficulty (line 1197-1203)
- Spawn rate accelerates over time (difficultyRamp, line 1201)
- Enemy types unlock at specific stages (line 1026, `minStage` field)
- Three enemy types (line 502-506): grunt, interceptor, tank with different stats

### Boss Attack Patterns
Boss patterns defined as functions in `BOSS_PATTERNS` array (line 1372). Each boss (6 total) has a unique pattern set of ~3 attack types. Patterns fire enemy bullets in radial, spread, or aimed formations. Boss telegraph (white glow) indicates incoming attack (line 998-1000).

### Particles & Visual Effects
- Screen shake: `screenShake(magnitude, duration)` (line 727)
- Flash effect: `flashHit()` or `flashBomb()` (line 698-709) — animates white flash overlay
- Combo pop: displays kill count milestone (line 712)
- Stage banner: displays stage number on stage entry (line 719)
- Vignette: red pulsing edge on critical health (1 life remaining) (line 687)

### Audio System
WebAudio API synth with no external files. All SFX generated procedurally:
- `beep()`: oscillator with frequency sweep (line 566)
- `noiseBurst()`: white noise with filter (line 597)
- `sfx` object (line 624): named sound effects (shoot, laser, explode, bomb, etc.)
- Master gain and compressor (line 544-557)
- Mute toggle persists to localStorage (line 736-739, key: `skyfighter.muted`)
- Reduced motion setting also mutes motion effects (line 537, 728-729)

---

## Key Implementation Details

### Mobile & Input Handling
- **Desktop**: Mouse position steers plane (line 847-861), click or spacebar fires, X/Shift bombs
- **Mobile**: Touch joystick in left zone (lines 814-826), fire and bomb buttons on right (lines 829-834)
- Keyboard arrow keys or WASD move (line 1153-1156)
- Input is combined (keyboard priority over mouse when keys active) (line 1161-1170)

### Adaptive Performance
Quality levels (0=full, 1=medium, 2=low) adjust based on frame time average (line 476-483). Affects particle count and shadow/glow rendering (line 1074, 1002, 1579-1591).

### Themes & Visuals
Six parallax sky themes (line 493-500) that cycle per stage. Each includes sky gradient colors, cloud color, and sun glow. Themes rotated as stages progress.

### Canvas Sizing
Base resolution 480×720px (line 753). Canvas scaled to fit window while maintaining aspect ratio (line 754-765). Accounts for device pixel ratio and safe areas (notches, etc.).

---

## Extending the Game

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502-506): define width, height, HP, speed multiplier, color, score value, minimum stage
2. Optionally add to `EXPLODE_SFX` mapping (line 678) for custom explosion sound
3. Adjust `spawnEnemy()` spawn probabilities (line 1029-1030) if needed

### Adding a New Boss
1. Add boss design to `BOSSES` array (line 508-515): name, hull colors, core color, HP multiplier
2. Create unique attack pattern function(s) using helper functions: `fireRadial`, `fireAimedSpread`, `fireColumns`, etc. (lines 1324-1370)
3. Add pattern array to `BOSS_PATTERNS` (line 1372)
4. Boss selection automatically cycles through list by stage

### Adjusting Game Balance
- Difficulty multipliers: edit `DIFFICULTIES` object (line 485)
- Enemy spawn rate: adjust `spawnBase` and `spawnMin` in difficulties
- Enemy HP/speed: edit `ENEMY_TYPES` object
- Boss HP: modify base formula (line 1046) or per-boss `hpMul` in `BOSSES`
- Scoring: adjust base values in `addScore()` or boss kill reward (line 1446)
- Player speed: edit `player.speed` in `resetGame()` (line 877)

### Adding Visual Effects
Use existing helper functions:
- `spawnExplosion(x, y, color, count)`: particle burst (line 1073)
- `screenShake(mag, dur)`: camera shake (line 727)
- `flashHit()` or `flashBomb()`: screen flash overlay (line 698-709)
- Add new animations to CSS (section inside `<style>` tag, lines 10-359)

### Adding New SFX
Add function to `sfx` object (line 624). Use `beep()` or `noiseBurst()` helpers with WebAudio parameters (frequency, duration, sweep, filter type/frequency, pan).

---

## Persistence & Settings

All user state saved to `localStorage`:
- `skyfighter.bestScore`: best score ever achieved
- `skyfighter.muted`: whether sound is off (1 = muted)
- `skyfighter.difficulty`: last selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: accessibility preference for motion effects

Constants defined at line 463-466. Clear localStorage to reset all settings.

---

## Browser Compatibility Notes

- **WebAudio API**: required for sound. Supported on all modern browsers but AudioContext may start in "suspended" state (line 560 resumes it on first user interaction).
- **CSS Gradients & Transforms**: widely supported, but test on older Safari for issues.
- **Canvas 2D**: solid support everywhere.
- **localStorage**: persistent storage; fails silently if disabled.
- **Responsive viewport**: uses `viewport-fit=cover` for notch-aware layout (line 5).

---

## Performance Considerations

- **Particle count** is adaptive based on frame performance (line 1074)
- **Shadow/glow effects** reduced on lower quality levels (line 1002, 1579)
- **Object pooling**: bullets, enemies, particles reuse arrays (not true pooling, but filtered efficiently)
- **requestAnimationFrame** ensures smooth 60 FPS sync with browser refresh
- **Canvas transforms** used for player/enemy/boss rotation (translate, rotate calls)

Older devices may struggle on hard mode or with many particles on-screen. Quality level auto-adjusts (line 481-482).
