# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML file with embedded CSS and JavaScript. No external dependencies or asset files are required — all game logic, rendering, audio synthesis, and UI are self-contained.

## Architecture Overview

The entire game runs in a single `index.html` file organized into these major systems:

### Rendering & Display
- **Canvas-based 2D rendering** at 480×720 base resolution, scaled responsively to fit the window
- **HUD overlay** showing score, best score, multiplier, weapon level, health, and bomb count
- **Dynamic theming** with 6 different sky gradients that cycle through stages
- **Visual effects**: Screen shake, flash on damage, combo pop-ups, stage banners, vignette on low health
- **Particle system** for explosions and visual feedback

### Audio System
- **WebAudio API synthesis** with no external sound files
- **Modular SFX effects**: `sfx.shoot()`, `sfx.laser()`, `sfx.explode()`, `sfx.bomb()`, etc.
- **Stereo panning** based on entity position for spatial audio
- **Mute toggle** with localStorage persistence
- All synthesized through oscillators, filters, and noise bursts

### Input Handling
- **Keyboard**: Arrow keys/WASD for movement, Space/Shift/X for actions, P/Esc for pause
- **Mouse**: Move player to cursor position, click to fire
- **Touch/Mobile**: Virtual joystick for movement, FIRE/BOMB buttons with adaptive UI
- **Joystick state** tracked separately from keyboard for smooth input merging
- **Event-based key tracking** via keydown/keyup with state object

### Game State Management
```
Game States: MENU → RUNNING (with entities) → PAUSED → GAME OVER → MENU

Core Entities:
- player: position, weapon type/level, health, bombs, invulnerability timer
- enemies[]: array of spawned enemy objects (grunt/interceptor/tank)
- boss: current boss object with HP, patterns, attack timers
- bullets[]: player projectiles with pierce/laser properties
- enemyBullets[]: projectiles from enemies and boss
- powerUps[]: dropped items (life/bomb/weapon)
- particles[]: explosion/visual effect particles
```

### Difficulty System
Three difficulty tiers (Easy/Normal/Hard) scale:
- Enemy spawn rate and minimum spawn interval
- Enemy movement speed and shooting frequency  
- Player starting lives, bombs, and max health
- Boss HP multiplier

Set at game start, persisted to localStorage.

### Stage Progression
1. **Wave system**: Each stage spawns `KILLS_PER_STAGE` (18) enemies before boss appears
2. **Boss progression**: Cycles through 6 unique boss designs with increasing HP per stage
3. **Difficulty ramp**: Enemy spawn rate increases as stage progresses (`Math.floor(frame / 300) * 4`)
4. **Theme cycling**: Sky gradient changes per stage for visual variety

### Weapon System
- **Vulcan**: Spread shot, upgrades add horizontal spread lanes (level 1/2/3)
- **Laser**: Piercing shots, upgrades add extra lanes and pierce through multiple enemies (level 1/2/3)
- Weapon drops from defeated enemies; damage is consistent, mechanics vary by type
- Fire cooldown: 0.15s (Vulcan), 0.1s (Laser)

### Scoring & Multiplier
- **Base scoring**: Each enemy type has fixed points (grunt=10, interceptor=15, tank=35)
- **Chain/multiplier system**: Kill chain increases multiplier (1-10x) at 5-kill intervals
- **Chain timer**: 2.2 seconds between kills to maintain chain
- **Bomb points**: Using bombs adds 5 points per enemy cleared and maintains the chain

### Collision Detection
- Simple AABB (axis-aligned bounding box) overlap: `rectsOverlap(a, b)` checks distance-based collision
- Used for: player/enemy bullets, player/power-ups, player projectiles/enemies, player projectiles/boss

### Adaptive Performance
Quality levels (0=full, 1=medium, 2=low) auto-adjust based on frame time:
- Tracks last 45 frame times
- Scales particle/explosion counts by quality
- Reduces shadow blur and effect intensity at lower quality
- Increases quality if frames exceed 1/40s (25ms), decreases if under 1/56s

### Accessibility
- **Reduced motion mode**: Disables animations, screen shake, and flash effects on user preference or by toggle
- **Mute toggle**: Silences all synthesized audio
- **Screen reader support**: ARIA live region for announcements (`#srLive`)
- **Safe area insets**: Respects iPhone notches and safe zones via `env(safe-area-inset-*)`

## Common Development Tasks

### Running the Game
Simply open `index.html` in a modern browser. No build step, no server required.

**Local testing**: Use `python -m http.server 8000` or any local web server (browsers enforce CORS on canvas data URLs in some contexts).

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with properties: `w, h, hp, speedMul, color, dark, score, minStage, weave` (weave optional)
2. Add optional explosion SFX to `EXPLODE_SFX` mapping if custom sound is needed
3. Add optional rendering in `renderEnemy()` if custom visuals are needed
4. Enemy will automatically spawn in `spawnEnemy()` based on `minStage` and probability tuning

### Adjusting Difficulty Parameters
Edit the `DIFFICULTIES` object entries for `easy/normal/hard`:
- `spawnBase`: Initial spawn interval in frames
- `spawnMin`: Minimum spawn interval (lower = faster spawning)
- `enemySpeed`: Movement speed multiplier for all enemies
- `enemyShootChance`: Probability (0-1) that enemy shoots
- `lives`: Player starting lives
- `bombs`: Player starting bombs
- `bossHpMul`: Boss HP multiplier

### Adding New Visual Effects
1. **Particle bursts**: Call `spawnExplosion(x, y, color, count)` 
2. **Combo/stage text**: Use `showComboPop()` or `showStageBanner()`
3. **Screen effects**: `screenShake(mag, dur)` for vibration, `flashHit()` or `flashBomb()` for white flashes
4. **Vignette**: Automatically enabled when health ≤ 1, can be toggled via `vignetteEl.classList`

### Adding New Audio
1. Create a new function in the `sfx` object: `sfx.newEffect = () => { beep(...) or noiseBurst(...) }`
2. Use `beep(freq, duration, type, volume, opts)` for tonal sounds or `noiseBurst(duration, volume, opts)` for noise
3. Reference in game: `sfx.newEffect()`
4. Optional parameters: `{ sweepTo, filterType, filterFreq, pan, delay }` for modulation and spatial effects

## File Structure

Single file: `index.html`
- Lines 1-359: HTML structure and embedded CSS styling
- Lines 422-1669: JavaScript game engine and logic

## Performance Notes

- **Frame time tracking** maintains 45-frame average; targets 40 FPS minimum
- **Particle/collision culling**: Off-screen entities removed each frame
- **Quality adaptation** disables effects at `quality > 0` to maintain performance
- **Canvas resolution fixed** at 480×720; browser scaling handles display size
- **No async operations** in game loop; all rendering and logic are synchronous

## Debugging Tips

- Open browser DevTools console to check for errors
- Check `quality` variable to see adaptive performance state
- Use `localStorage.getItem('skyfighter.*')` to inspect saved settings
- Monitor `frameTimes` array in console to check frame performance
- Player position accessible as `player.x, player.y` in console during gameplay
