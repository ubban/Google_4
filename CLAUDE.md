# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in a single HTML file (`index.html`). It features:

- Player aircraft with weapon upgrades (Vulcan and Laser)
- Multiple enemy types (grunts, interceptors, tanks) with stage-based progression
- Six boss encounters with unique attack patterns
- Difficulty levels (Easy, Normal, Hard) affecting spawn rates, enemy speed, and boss HP
- Chain multiplier system for consecutive kills
- Smart bomb mechanic to clear all enemies
- Touch and keyboard controls
- WebAudio synthesized sound effects
- Adaptive performance scaling for FPS optimization
- Fully persistent game state (best scores, mute setting, difficulty, reduced motion preference)

## Architecture

The game uses a single-file architecture with no external dependencies. The structure is:

```
┌─ HTML/DOM Layer
│  ├─ Canvas (#gameCanvas) - main rendering surface
│  ├─ Overlay (#overlay) - menu, pause, game-over screens
│  ├─ HUD elements - score, lives, bombs, weapon, multiplier
│  ├─ Touch controls - virtual joystick, FIRE/BOMB buttons
│  ├─ Effects - vignette, flash, combo pop, stage banner
│  └─ SR-only announcements for accessibility
│
├─ Game State (global variables)
│  ├─ player, bullets, enemies, enemyBullets, particles, powerUps, boss
│  ├─ score, lives, running, paused, gameOver
│  ├─ stage, stageKills, kills, chain multiplier
│  └─ difficulty, muted, reducedMotion, bestScore (localStorage)
│
├─ Core Game Loop
│  ├─ `loop(now)` - requestAnimationFrame driver
│  ├─ `update(dt)` - game state simulation
│  └─ `draw()` - canvas rendering
│
├─ Input Handlers
│  ├─ Keyboard - arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
│  ├─ Mouse - movement and click-to-fire
│  └─ Touch - joystick in lower-left, FIRE/BOMB buttons lower-right
│
├─ Entity Systems
│  ├─ Player - movement, firing, weapon switching, invulnerability
│  ├─ Enemies - spawning, movement patterns (weaving), shooting
│  ├─ Boss - multi-phase attacks, pattern library, bullet choreography
│  ├─ Bullets (player) - pierce property, laser vs. vulcan variants
│  ├─ Bullets (enemy) - radial or aimed, homing variants
│  ├─ PowerUps - weapon upgrades, extra lives, bombs
│  └─ Particles - explosive debris, visual feedback
│
├─ Rendering
│  ├─ Canvas drawing with 2D transforms
│  ├─ Theme system - 6 dynamic sky/sun gradients, per-stage palette
│  ├─ Plane sprites - procedural path-based drawing (player, enemies, bosses)
│  ├─ Cloud parallax - scrolling layers at different speeds
│  └─ Screen shake and flash effects
│
├─ Audio (WebAudio synthesis, no assets)
│  ├─ Oscillator-based beeps with frequency sweeps
│  ├─ Noise bursts with filters
│  ├─ Stereo panning
│  └─ Master compressor for dynamic range
│
├─ Difficulty System
│  ├─ DIFFICULTIES - spawn rates, enemy speed, shoot chance, lives, boss HP multiplier
│  ├─ Dynamic spawn acceleration - enemies spawn faster as frame count grows
│  └─ Boss pattern variation by difficulty
│
└─ Persistent Storage (localStorage keys)
   ├─ skyfighter.bestScore
   ├─ skyfighter.muted
   ├─ skyfighter.difficulty
   └─ skyfighter.reducedMotion
```

## Key Data Structures

### Player
```javascript
player = {
  x, y,                // position
  w, h,                // dimensions (40x40)
  speed: 260,          // px/s movement speed
  cooldown,            // fire rate limiter
  weapon: 'vulcan'|'laser',
  level: 1-3,          // weapon power level
  invuln: 0-1.5        // invulnerability window (seconds) after taking damage
}
```

### Enemy
```javascript
enemies.push({
  x, y, w, h,
  type: 'grunt'|'interceptor'|'tank',  // defines HP, speed, score, appearance
  hp, speed, color, dark (for shading),
  shootTimer, canShoot, pattern ('aimed'|'spread'),
  weave: boolean,      // sine-wave lateral movement
  baseX, weaveSeed     // weaving parameters
})
```

### Boss
```javascript
boss = {
  x, y, w: 90, h: 80,
  hp, maxHp,
  design: BOSSES[index],  // defines visuals and pattern index
  entering: boolean,   // entry animation phase
  vx: 1|-1,            // horizontal movement direction
  attackTimer, telegraph: 0,  // attack timing and telegraph flash
  patternIndex         // which attack pattern to use next
}
```

### Bullet (Player)
```javascript
bullets.push({
  x, y, w, h,
  speed: 500|640,
  vx,                  // horizontal velocity (spread for vulcan)
  pierce: 0-2,         // how many enemies laser can pass through
  laser: boolean       // affects render and piercing
})
```

### Bullet (Enemy)
```javascript
enemyBullets.push({
  x, y, w, h,
  speed,               // forward speed
  vx, vy, OR dx, dy,   // velocity vectors
  radial: boolean      // uses dx/dy for boss bullets
})
```

## Game Flow

1. **Initialization** - load localStorage, set difficulty/mute/reduced-motion state, attach event listeners
2. **Menu** - overlay shown, player selects difficulty and clicks Start
3. **Reset Game** - `resetGame()` initializes player, stage=1, enemies=[]
4. **Main Loop** - `loop()` calls `update()` then `draw()` ~60 FPS
5. **Spawn Enemies** - `spawnTimer` decrements; when zero, spawn random enemy type (type/stage gating via `ENEMY_TYPES[k].minStage`)
6. **Enemy Patterns** - 18 kills on stage → boss spawning sequence (1.2s delay)
7. **Boss Battle** - boss enters from top, uses choreographed attack patterns, takes 3 damage per player bullet
8. **Stage Progression** - boss defeated → `stage++`, spawn rate increases, new theme/enemies unlock
9. **Game Over** - lives reach 0 → show end screen with score/stage/best
10. **Pause** - toggles overlay without stopping state, freezes game

## Common Modifications

### Adjust Difficulty Balancing
Edit `DIFFICULTIES` object (line 485-489):
- `spawnBase`/`spawnMin` - base and minimum spawn timer (lower = faster spawning)
- `enemySpeed` - multiplier on all enemy speeds
- `enemyShootChance` - probability enemy can shoot (0-1)
- `lives`, `bombs` - starting resources
- `bossHpMul` - health scaling for bosses

### Add/Modify Enemy Types
Edit `ENEMY_TYPES` (line 502-506):
- Add new key → new type
- Properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
- Add drawing logic in `enemies.forEach(en => { ... drawPlane(...) })` (line 1603) if custom visuals needed
- Add explosion SFX in `EXPLODE_SFX` (line 678) if custom sound

### Change Boss Attack Patterns
Edit `BOSS_PATTERNS` array (line 1372-1409):
- One array per boss design (index matches `BOSSES` order)
- Each pattern array contains functions that fire enemy bullets
- Helper functions: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`
- Boss telegraph animation is automatic (white flash 22 frames before attack fires)

### Adjust Player Weapon Stats
- Vulcan: spread at level 2-3 (line 1127), cooldown 0.15s (line 1177)
- Laser: lanes/pierce at level 2-3 (line 1136-1137), cooldown 0.1s (line 1177)
- Fire rate and spread controlled in `fireWeapon()` (line 1124-1146)

### Add/Modify Visual Themes
Edit `THEMES` array (line 493-500):
- `sky`: [top, middle, bottom] gradient stops (hex colors)
- `cloud`, `sun`: RGB values for parallax and glow
- `name`: displayed in stage announcements
- New theme auto-cycles via `(stage - 1) % THEMES.length`

### Adjust UI/HUD Styling
All CSS is inline (lines 10-358). Key selectors:
- `#hud` - score/bomb/heart display
- `#bossBarWrap`, `#bossBarFill` - boss health bar
- `#overlay` - menu/pause/gameover screen
- Animations use `@keyframes` (heartBeat, vignettePulse, comboPulse, stagePulse, flashHit, flashBomb)
- Reduced motion support via `body.reduced-motion` class toggling

### Control Audio
- Mute button toggles `muted` state and `localStorage.setItem(STORAGE_MUTE)`
- Individual SFX calls check `if (muted || !audioCtx) return` at start
- Master volume via `masterGain.gain.value` (line 556, currently 0.9)
- Add new sound in `sfx` object (line 624) using `beep()` or `noiseBurst()` helpers

### Screen Shake & Flash Effects
- `screenShake(mag, dur)` - triggers red vignette pulse and jitter in canvas rendering
- `flashHit()` / `flashBomb()` - white screen flash animation (CSS keyframe)
- Both respect `reducedMotion` preference (dampened or skipped)

## Performance Optimization

### Adaptive Quality Scaling
The `quality` variable (0=full, 1=medium, 2=low) auto-adjusts every 45 frames based on average frame time:
- Quality affects:
  - Particle count in explosions (line 1074)
  - Glow effects on bullets/powerups (lines 1579, 1622, 1583, 1594)
  - Boss core shadow blur (line 1002)
  - Screen shake magnitude (line 728)

### Particle Pooling
Particles are created on-demand and filtered when `life <= 0`. No pre-allocation, but explosions scale particle count based on quality.

### Render Optimization
- Canvas size is fixed (480×720), scaled via CSS
- `ctx.save()` / `ctx.restore()` used to avoid state leaks
- No animated GIFs or image assets—all procedural

## Browser APIs & Features Used

- **Canvas 2D** - `fillRect`, `beginPath`, `arc`, `ellipse`, `translate`, `rotate`, `globalAlpha`, `shadowBlur`, `createLinearGradient`, `createRadialGradient`
- **WebAudio** - `AudioContext`, `OscillatorNode`, `BiquadFilterNode`, `GainNode`, `DynamicsCompressorNode`, `StereoPannerNode`
- **localStorage** - 4 keys for persistence (score, mute, difficulty, reduced-motion preference)
- **requestAnimationFrame** - game loop driver
- **window.matchMedia** - detect prefers-reduced-motion
- **Touch/Pointer Events** - touch start/move/end, mouse events, keyboard events
- **window.getComputedStyle** - screen shake via translate transform

## Accessibility Features

- Semantic HTML with `aria-live="polite"` for status announcements
- Screen-reader-only live region (`#srLive`) for game events
- Reduced motion support (toggle at top-right, respects system preference)
- Keyboard-only playable (arrow keys + Space + X)
- Color choices provide contrast (yellow on dark, red for damage)
- Touch controls for mobile
- No flashing above 3 Hz (eye safety)

## Development Practices

- All code is vanilla JavaScript (no dependencies)
- Global scope minimized via IIFE (line 423)
- Constants defined at top (DIFFICULTIES, ENEMY_TYPES, BOSSES, THEMES)
- Helper functions grouped by concern (input, rendering, audio, game logic)
- Magic numbers are mostly acceptable here since it's a single file; add a comment if the meaning is non-obvious (e.g., line 728 shake dampening)
- No tests (game is inherently visual); verify changes by running locally and playing

## Running Locally

1. **Open in browser** - double-click `index.html` or use a local server:
   ```bash
   python3 -m http.server 8000
   # then open http://localhost:8000
   ```
2. **Edit** - modify `index.html` directly in your editor
3. **Refresh** - reload browser tab (Ctrl+R / Cmd+R) to see changes
4. **Debug** - use browser DevTools console to inspect `player`, `enemies`, `boss`, `score`, etc. at any time

## Git Workflow

- All changes go into `index.html`
- Commit messages should describe what changed (e.g., "Adjust Behemoth boss HP and pattern" or "Add new Storm Front stage theme")
- No build step needed; changes are live immediately after save+refresh
