# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 Canvas application. No build process or external dependencies are required—the game runs directly in any modern browser.

## Architecture

### Single-File Structure
- **index.html** (1669 lines): Complete game implementation including HTML, CSS, and JavaScript
- No build system, no npm, no external assets
- Web Audio API synthesizes all sound effects (no audio files)
- Game state persisted in `localStorage`

### Core Game Loop
The main game loop (`loop()` at line 1640) runs at 60 FPS via `requestAnimationFrame`:
1. **Update Phase** (`update(dt)` at line 1148): Player/enemy movement, collision detection, state changes
2. **Draw Phase** (`draw()` at line 1520): Canvas rendering with parallax clouds, entities, effects
3. **Performance Tracking**: Adaptive quality scaling based on frame times (tracked at line 476)

### Key Systems

**Player** (line 877-878)
- Position, speed (260 px/s), collision box
- Weapon system: Vulcan (spread shot) or Laser (piercing), levels 1-3
- Invulnerability frames (1.5s after hit)
- Lives, bombs (smart bombs clear screen and damage bosses)

**Enemies** (line 1022-1041, line 1210-1228)
- Three types: Grunt (1 HP), Interceptor (1 HP, fast, weaving), Tank (4 HP, slow, armored)
- Stage-gated spawning (tanks appear at stage 2+)
- Two firing patterns: spread (radial) or aimed (at player)
- Difficulty-based spawn rates and behavior (line 485-489)

**Bosses** (line 1043-1056, line 1411-1439)
- Six unique designs, one per stage (cycling)
- Each has distinct attack pattern library (line 1372-1409):
  - VANGUARD: radial bursts, aimed spreads, V-fans
  - BEHEMOTH: heavy columns, homing spreads, dense radials
  - SPECTER: teleporting dashes, spirals, double aimed shots
  - DREADNOUGHT: 8-way crosses, homing missiles, radial walls
  - WRAITH: barrage teleports, double rings, cross spreads
  - TITAN: columns, full rings, relentless homing
- Telegraph mechanic (line 1421-1434): Boss flashes before attack, player has 22 frames to react

**Scoring & Progression**
- Base scores per enemy type (10-35 points)
- Kill chain multiplier: +1 per 5 consecutive kills (max 1.0 to 1.8x)
- Chain resets on damage or time (2.2s timeout at line 1089)
- Stages unlock after 18 kills (line 491); boss fight, then stage +1
- Difficulty levels affect spawn rate, enemy speed, life count, bomb count, boss HP

**Power-ups** (line 1058-1067, line 1304-1322)
- **Life**: Restore one heart (6% spawn chance)
- **Bomb**: Restore one smart bomb (8% spawn chance)
- **Weapon**: Level up current weapon or switch weapons (20% spawn chance)

### Input Handling
**Keyboard**: Arrow keys/WASD (move), Space (fire), X/Shift (bomb), P/Esc (pause)
**Mouse**: Move to steer player, click to fire
**Touch**: Left-side joystick for movement (line 785-826), FIRE/BOMB buttons (line 829-834)
- Joystick constrained to 50px radius (line 799-805)
- Keyboard input always takes priority over mouse (line 1161-1170)

### Visual Feedback
**Adaptive Quality** (line 474-483): Monitors frame times; downgrades rendering at 40fps, upgrades at 56fps
- Quality 0 (full): Glowing effects on projectiles/power-ups, shadow blur on bosses
- Quality 1 (medium): Reduced shadow blur
- Quality 2 (low): No glow/shadow effects

**Screen Effects**
- Screen shake on boss/bomb hits (line 727-729)
- Flash (white) on player hit (line 698-703)
- Red vignette pulse when player at 1 HP (line 71-87)
- Combo announcements (line 712-717)

**Themes** (line 493-500)
Six parallax sky themes (gradient, clouds, sun) cycling by stage:
1. Daylight Front (blue)
2. Sunset Corridor (orange/pink)
3. Night Approach (dark blue)
4. Storm Front (teal)
5. Arctic Wastes (cyan)
6. Volcanic Ridge (red/orange)

### Audio Synthesis
No external audio files—all sounds generated via Web Audio API (line 542-676):
- **Oscillators**: Square, sawtooth, triangle, sine waves with frequency sweeps
- **Filters**: Lowpass/highpass for tone shaping
- **Noise**: White noise buffer for impact sounds
- **Effects**: Panning, gain envelope (attack/decay)
- **SFX Library** (line 624-676): 10+ distinct sounds (shoot, laser, explosions, hit, bomb, power-up, boss, stage clear, game over)

### Persistence
Four `localStorage` keys (line 463-466):
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Animations/effects toggle (respects system prefers-reduced-motion)

### Accessibility
- Screen reader support: `.sr-only` HUD, live region announcements (line 390, 468-471)
- Respects `prefers-reduced-motion` (line 520-523, 349-358)
- High contrast UI with text shadows
- Keyboard fully playable

## Extending the Game

### Add a New Enemy Type
1. Add to `ENEMY_TYPES` (line 502-506): Define `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Add explosion SFX mapping to `EXPLODE_SFX` (line 678) if unique sound desired
3. Spawning auto-gates new type to `minStage` in `spawnEnemy()` (line 1026)

### Add a New Boss Pattern
1. Add boss design to `BOSSES` (line 508-515)
2. Add pattern array to `BOSS_PATTERNS` (line 1372-1409): Functions that fire enemy bullets
   - Access boss position (`b.x`, `b.y`), attack index (`b.patternIndex`), player position (`player.x`, `player.y`)
   - Patterns execute when telegraph timer expires (line 1423-1430)

### Modify Difficulty Tuning
Edit `DIFFICULTIES` (line 485-489):
- `spawnBase`: Frame delay between spawns (lower = more frequent)
- `spawnMin`: Minimum spawn delay as difficulty ramps (line 1201-1202)
- `enemySpeed`: Multiplier on enemy movement speed
- `enemyShootChance`: Probability each enemy can fire (0-1)
- `lives`: Starting heart count
- `bombs`: Starting bomb count
- `bossHpMul`: Boss HP multiplier

### Adjust Visual Styling
- Inline CSS (line 10-358): Modify colors, sizing, animations
- Canvas drawing functions: `drawPlane()` (line 913-968), `drawBossShip()` (line 970-1020), `drawClouds()` (line 1507-1518)
- Collision boxes are simple AABB; adjust `w`, `h` properties to change hit areas

## Testing & Debugging

**Canvas Math**: Collision detection uses axis-aligned bounding boxes (line 1069-1071). Canvas coordinate (0,0) is top-left; +Y down.

**Frame Inspection**: Global `frame` counter (line 889) increments every update; useful for time-locked logic (e.g., telegraph timing at line 1421-1434).

**Local Storage**: Open browser DevTools > Application > Local Storage to inspect/clear saved state.

**Performance**: Check `quality` variable in DevTools console; if consistently > 0, consider reducing enemy count, particle effects, or shadow blur.

**Sound Debugging**: Web Audio context may not initialize until first user interaction (line 545-561). If no sound, ensure `ensureAudio()` was called on a click/key event.

## Deployment

The game is production-ready as-is:
- Single HTML file → copy and serve over HTTP/HTTPS
- No build step
- Responsive design scales to any viewport
- Safe to open locally (file:// protocol works for most features; Audio Context may need HTTP for some browsers)

---

*This document is the single source of truth for Sky Fighter's architecture. When making changes, update this file to reflect new systems or significant refactors.*
