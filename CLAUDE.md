# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. It's a single-file HTML5 application (index.html) with embedded JavaScript, CSS, and WebAudio synthesis for sound effects. The game features multiple difficulty levels, boss encounters, weapon upgrades, and procedural visual themes.

## Running the Game

- **Browser**: Open `index.html` directly in any modern web browser. No build step, no server required.
- **Local development**: Use a simple HTTP server if needed (e.g., `python -m http.server` or `npx serve`), though the file can be opened directly with `file://`
- **Testing**: Open in browser and play through stages, test controls, verify audio and visual effects work

## Code Architecture

The entire game runs inside a single IIFE (Immediately Invoked Function Expression) from line 423 onward. The structure follows these major sections:

### 1. **Canvas & Input System** (lines 427–862)
- Canvas setup and responsive resizing (BASE_W=480, BASE_H=720)
- Keyboard input handling (arrows, WASD, space, X/Shift for bomb)
- Mouse steering: plane glides toward cursor when hovering the canvas; keyboard/joystick takes priority
- Touch controls: virtual joystick on left side (moveZone), fire/bomb buttons on right (actionZone)
- Pause/mute buttons and difficulty selector in the UI overlay

### 2. **Game State** (lines 863–912)
- Player object: position, velocity, weapon, cooldown, invulnerability timer
- Collections: bullets, enemies, enemyBullets, particles, powerUps, boss
- Scoring system with combo chain multiplier
- Stage progression and kill tracking
- resetGame() initializes all state and applies difficulty settings

### 3. **Audio System** (lines 542–676)
- WebAudio synthesized effects (no external files)
- `beep()` generates tonal sounds with pitch sweep, filter, and panning
- `noiseBurst()` creates explosive/impact sounds
- `sfx` object maps 12+ named sounds to sound generation calls
- Audio context is lazily initialized on first user interaction (ensureAudio)
- Muting can be toggled and persists to localStorage

### 4. **Rendering & Animation** (lines 680–1020)
- **drawPlane()**: Renders player and enemy fighter jets (wings, fuselage, canopy, optional flame)
- **drawBossShip()**: Procedurally draws boss with gradient hull, rotating turrets, glowing core
- **drawClouds()**: Parallax cloud layers with sine-wave scroll
- **Sky themes**: 6 themes (Daylight, Sunset, Night, Storm, Arctic, Volcanic) with custom sky gradients, sun glow, and cloud colors
- Glow effects (shadow blur) enabled only at full quality (quality === 0)
- Particle system: spawns colored explosion particles with physics on kill

### 5. **Game Update Loop** (lines 1148–1302)
- Called each frame via requestAnimationFrame
- Player movement: keyboard/joystick input takes priority over mouse target
- Boundary clamping keeps player within canvas
- Weapon firing with cooldown (vulcan 0.15s, laser 0.1s)
- Enemy spawn timer decreases; spawns new enemies or triggers boss
- Enemy behavior: weaving, shooting (aimed vs spread patterns), removal off-screen
- Bullet collision with enemies: piercing lasers reduce pierce count; regular bullets are consumed
- Powerup collision applies buffs (life, bombs, weapon upgrade)
- Screen shake and flash effects on hits
- Boss health bar update

### 6. **Boss System** (lines 1043–1434)
- 6 boss designs: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Each stage cycles through bosses: `BOSSES[(stage - 1) % BOSSES.length]`
- HP scales with difficulty (easy 0.8×, normal 1.0×, hard 1.3×) and boss design multiplier
- **Boss patterns**: Each boss has 3 unique attack patterns (BOSS_PATTERNS array, lines 1372–1409)
  - Radial bursts, aimed spreads, homing volleys, teleport dashes, cross-fire, double rings, etc.
- **Telegraph system**: Boss shows a visual telegraph (white core glow + sound) before attacking
- Bosses move back and forth (vx speed toggle at boundaries)
- Boss bullet types use `radial: true` flag with `dx`/`dy` velocity instead of `speed`/`vx`

### 7. **Difficulty & Persistence** (lines 485–527)
- **DIFFICULTIES**: Easy, Normal, Hard with spawn rates, enemy speed/shoot chance, lives, bombs, boss HP multiplier
- **Themes**: 6 visual themes cycle by stage
- **Enemy types**: Grunt (basic), Interceptor (fast weaving), Tank (slow, tanky, high-scoring)
- localStorage keys: bestScore, muted, difficulty, reducedMotion

### 8. **Effects & Polish**
- **Screen shake**: `screenShake(magnitude, duration)` adds random camera jitter when paused
- **Vignette**: Red pulsing vignette when health is critical (1 life left)
- **Combo counter**: Shows kill milestones (10, 20, 30…) in the center
- **Stage banner**: Displays stage name on stage start with animation
- **Accessibility**: Reduced motion respects prefers-reduced-motion CSS media query; screen reader announcements via #srLive

## Key Game Variables & Constants

| Name | Purpose |
|------|---------|
| `BASE_W`, `BASE_H` | Canvas logical size (480×720); CSS scales it responsively |
| `KILLS_PER_STAGE` | 18 kills triggers boss spawn |
| `quality` | 0 (full), 1 (medium), 2 (low); adapted per-frame based on perf |
| `difficulty` | "easy", "normal", "hard"; read from difficulty selector |
| `chain`, `multiplier` | Combo system; breaks after 2.2s without a kill |
| `player.weapon` | "vulcan" or "laser"; affects spread, cooldown, pierce |
| `player.invuln` | Invulnerability timer after hit (1.5s) |
| `boss.telegraph` | Countdown before boss attack pattern fires |

## Extending the Game

### Adding a New Difficulty
1. Add entry to `DIFFICULTIES` object with spawn rates, enemy multipliers, lives, bombs, boss HP
2. Extend the difficulty selector HTML if needed

### Adding a New Boss
1. Add entry to `BOSSES` array with name and colors (hullTop, hullBottom, wing, core, hpMul)
2. Add 3 attack patterns to `BOSS_PATTERNS` array (must match order)
3. Boss automatically cycles through via stage modulo

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` with w, h, hp, speedMul, color, dark, score, minStage, optional weave flag
2. Modify `spawnEnemy()` to include it in the random choice pool
3. Add explosion SFX mapping in `EXPLODE_SFX` if unique sound needed

### Adding a New Weapon or Powerup Type
1. Extend `fireWeapon()` with a new weapon condition or modify `applyPowerUp()` for powerup behavior
2. Add corresponding sound in `sfx` object

### Modifying Visuals
- Sky themes: Edit `THEMES` array (sky gradient colors, cloud/sun colors, name)
- Plane design: Modify `drawPlane()` path logic
- Boss appearance: Edit `drawBossShip()` and boss color schemes in `BOSSES`
- Canvas scale: Adjust `BASE_W`, `BASE_H`, and CSS max-width/height

## Performance Optimization

- **Adaptive quality**: Frame time averaged over 45 frames; quality decreases if avg > 25ms (40 FPS target), increases if < 17.8ms (56 FPS)
- **Glow effects**: Shadow blur only rendered at quality 0; disabled at lower quality settings
- **Particle count**: Explosions spawn fewer particles at lower quality
- **Touch vs mouse**: Mouse input doesn't auto-fire to avoid mobile jank; requires canvas click

## Accessibility Features

- Screen reader announcements: `announce()` updates #srLive aria-live region
- Reduced motion mode: Disables animations, screen shake, flash effects; respects OS preference
- Keyboard-only playable: No mouse required (arrows/WASD for movement, space to fire, X to bomb)
- Touch controls: Large buttons, haptic feedback capable (not yet used)

## Storage Keys

- `skyfighter.bestScore`: Best score across sessions
- `skyfighter.muted`: Audio mute toggle (0/1)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Reduced motion toggle (0/1)

## Common Modifications

### Slow down enemy bullets
Decrease `speed` values in fireRadial/fireAimedSpread/fireColumns calls (e.g., 150 → 100)

### Add health regeneration
Add a powerup type "regen" or modify `applyPowerUp()` to give temporary healing

### Change boss attack frequency
Adjust `b.attackTimer` in `updateBoss()` (currently 85 frames ≈ 1.4s at 60 FPS)

### Disable weapon switching
Remove the logic in `applyPowerUp()` that toggles between vulcan and laser

### Increase wave spawn rate
Lower `spawnBase` and `spawnMin` in `DIFFICULTIES`
