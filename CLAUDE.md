# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file web application (index.html). The game features player-controlled aircraft, multiple enemy types, boss battles, power-ups, and a score/combo system.

## Architecture

The entire game logic is contained in a single HTML file with embedded Canvas rendering and WebAudio synthesis. The architecture follows these main sections:

### 1. **Game State Management**
- **Player object**: Position, velocity, weapon type/level, health, invincibility timer
- **Entities**: bullets, enemies, enemy bullets, particles, power-ups, boss
- **Game flags**: running, paused, gameOver, stage, stageKills
- **Score/combo system**: chain counter, multiplier (1-10x), best score persistence

### 2. **Rendering Pipeline** (Canvas-based)
- **Background**: Sky gradient + sun halo + parallax clouds (3 layers)
- **Game objects**: Plane drawing (procedural), enemies (procedural plane graphics), boss ships (parameterized designs)
- **Effects**: Screen shake, flash vignette on low health, particle explosions
- **HUD**: Score, best, multiplier, hearts (lives), bombs, weapon status, boss health bar

### 3. **Input System**
- **Keyboard**: Arrow keys / WASD for movement, Space/click for fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Hover-based steering (optional, lower priority than keyboard)
- **Touch**: Left-side joystick for movement, right-side buttons for fire/bomb
- **Input priority**: Keyboard/joystick > mouse, so they never conflict

### 4. **Enemy System**
- **Types**: Grunt (basic red), Interceptor (fast orange), Tank (heavy purple, multi-hit)
- **Behavior**: Spawn from top, move downward with optional weaving, shoot either aimed or spread patterns
- **Difficulty scaling**: Enemy count, shoot chance, and speed ramp up in DIFFICULTIES config

### 5. **Boss System**
- **6 boss designs**: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- **Pattern sets**: Each boss has 3 distinct bullet patterns (stored in BOSS_PATTERNS array)
- **Attack phases**: Entering → teleport to Y=120 → patrol side-to-side → telegraph phase → attack phase (cycle)
- **Visual indicator**: Core glows white during telegraph, different for each boss (hull colors, core glow)

### 6. **Weapon System**
- **Two weapon types**: VULCAN (spread shots, fast fire) and LASER (single/dual/tri lanes, pierces enemies, slower)
- **Power-up upgrades**: Level 1-3 for each weapon (spread width/laser lanes), Level 3 → switch weapon
- **Cooldown**: 150ms for Vulcan, 100ms for Laser

### 7. **Audio (Web Audio API)**
- **No external assets**: All sounds synthesized using oscillators, filters, noise, and gain envelopes
- **SFX categories**: shoot, laser, explode (grunt/interceptor/tank variants), hit, bomb, powerup, bossHit, bossTelegraph, bossDown, stageClear, uiClick, gameover
- **Master volume control**: Mute button persists to localStorage

### 8. **Difficulty Levels** (localStorage-persisted)
- **Easy**: Slower spawn, fewer/weaker enemies, 4 lives, 3 bombs, 0.8x boss HP
- **Normal**: Baseline difficulty, 3 lives, 2 bombs, 1.0x boss HP
- **Hard**: Fast spawn, aggressive enemies, 3 lives, 2 bombs, 1.3x boss HP

### 9. **Performance Adaptation**
- `quality` flag (0=full, 1=medium, 2=low) adjusts particle counts and glow effects based on frame time
- Tracks last 45 frames; if avg > 25ms target, step up; if < ~17.8ms, step down

### 10. **Persistence** (localStorage keys)
- `skyfighter.bestScore`: All-time high score
- `skyfighter.muted`: 1 if muted
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: User reduced-motion preference override

## Code Organization

The single file is organized as an IIFE (immediately invoked function expression) to avoid global scope pollution. Key sections in order:

1. **Constants & Configurations** (lines ~425–515)
   - DIFFICULTIES, KILLS_PER_STAGE, THEMES, ENEMY_TYPES, BOSSES, BOSS_PATTERNS, storage keys

2. **Audio Setup** (lines ~542–676)
   - Web Audio synth functions: `beep()`, `noiseBurst()`, `sfx.*` effects

3. **UI & Rendering Helpers** (lines ~680–750)
   - Heart/bomb displays, combo pop-up, stage banner, flash effects, button updates

4. **Input Handling** (lines ~752–861)
   - Canvas resize, keyboard/mouse/touch event listeners, joystick logic

5. **Game State & Reset** (lines ~863–911)
   - Global game variables, `resetGame()` function

6. **Drawing Functions** (lines ~913–1020)
   - `drawPlane()` (procedural), `drawBossShip()`, `drawClouds()`

7. **Spawning & Collision** (lines ~1022–1085)
   - `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`, `rectsOverlap()`, `spawnExplosion()`

8. **Game Update Logic** (lines ~1087–1302)
   - `update(dt)`: Movement, firing, collision detection, scoring
   - `addScore()`, `breakChain()`, `useBomb()`, `fireWeapon()`

9. **Boss AI** (lines ~1304–1439)
   - `applyPowerUp()`, attack patterns (fireRadial, fireAimedSpread, etc.), `updateBoss()`

10. **Game Flow** (lines ~1441–1505)
    - `killBoss()`, `loseLife()`, `endGame()`, `togglePause()`

11. **Rendering Loop** (lines ~1507–1665)
    - `draw()` (sky, clouds, sprites, particles), `loop()` (animation frame callback)

## Common Development Tasks

### Run the Game
Open `index.html` in a web browser. No build step required—it's a standalone HTML file.

### Test Locally
```bash
# Option 1: Use a local HTTP server (required for some browser features)
python3 -m http.server 8000
# Then visit http://localhost:8000/

# Option 2: Just open the file
open index.html  # macOS
xdg-open index.html  # Linux
```

### Modify Game Balance
- **Enemy spawn rate**: Edit `DIFFICULTIES` object (spawnBase, spawnMin)
- **Enemy stats**: Adjust ENEMY_TYPES (hp, speed, shootChance)
- **Boss difficulty**: Modify boss hp multipliers in DIFFICULTIES, or add hpMul to BOSSES
- **Player movement speed**: Line ~877, `player.speed = 260`
- **Weapon cooldowns**: Line ~1177, `fireDelay` variable
- **Lives/bombs/stage progression**: DIFFICULTIES or KILLS_PER_STAGE constant

### Add New Visual Theme
Append to THEMES array (line ~493). Requires: sky gradient colors, cloud color, sun color, theme name.

### Add New Boss
1. Add entry to BOSSES array with hull/wing/core colors and hpMul
2. Add 3-function pattern set to BOSS_PATTERNS (must match array index)
3. Boss will cycle through patterns during attack phase

### Adjust Audio
- Change frequency/duration in sfx functions (beep params)
- Mute flags check `audioCtx` existence; WebAudio won't work on http://, use https:// or localhost
- Master gain at line ~556 (currently 0.9)

### Fix Particle/Visual Performance
Modify quality scaling thresholds in trackPerf() (~line 476–482). Currently adapts to maintain ~60fps.

### Enable Accessibility Features
- **Reduced motion**: Already implemented (button in top-right), respects prefers-reduced-motion media query
- **Screen reader**: Uses `#srLive` element for announcements (aria-live polite)
- **Keyboard-only play**: Fully supported (keyboard takes priority over mouse)

## Key Mechanics

### Combo/Chain System
- Each kill increments `chain` counter and resets `chainTimer` (2.2s window)
- Multiplier = 1 + floor(chain / 5), caps at 10x
- Breaking the chain (gap > 2.2s) resets to 1x

### Boss Telegraph
- Boss enters "telegraph" phase (visual indicator: core glows white)
- After 22 frames (0.37s) of telegraph, fires attack pattern
- Each boss has 3 rotating patterns, cycles through them

### Weapon Upgrade Mechanic
- Level 1-3 increases spread/lanes per weapon type
- At Level 3, next pickup switches to other weapon (resets to Level 1)
- Laser pierces enemies; Vulcan is purely spread

### Invincibility & Safety
- Player gains 1.5s invincibility after hit
- Blinks while invincible (frame-based flicker)
- Chain breaks on hit

## Caveats & Common Pitfalls

1. **Single-file bottleneck**: All changes go into one 1670-line file. No build step = fast iteration but harder to navigate.
2. **Canvas rendering only**: No DOM manipulation for sprites; all procedural drawing. Change rendering by editing `draw()` and `drawPlane()` / `drawBossShip()`.
3. **No collision against stage boundaries**: Enemies don't stop at top; they spawn above and fade at bottom. This is intentional for scrolling shooter feel.
4. **localStorage is origin-dependent**: Game data won't persist across different origins (http vs https, different domains, different ports).
5. **Web Audio requires user interaction**: Audio won't play until user clicks/presses key (`ensureAudio()` in input handlers).
6. **Touch controls are touch-specific**: Desktop users see mouse cursor; mobile users see joystick/buttons. No cross-platform fallback.

## References

- **Canvas API**: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API
- **Web Audio API**: https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API
- **requestAnimationFrame**: Animation loop driver (line ~1665)
- **localStorage**: Persistence layer (lines ~517–523, various setItem/getItem calls)
