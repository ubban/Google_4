# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in HTML5. It's a single-file game (index.html) that combines Canvas 2D rendering, WebAudio synthesis, and vanilla JavaScript. No build step or external dependencies required.

## Running the Game

1. **Direct browser**: Open `index.html` in any modern web browser
2. **Local server**: For best results, serve over HTTP (some features like audio may be restricted from `file://`)
   ```bash
   # Python 3
   python3 -m http.server 8000
   # Then visit http://localhost:8000
   ```

## Gameplay Features

- **Difficulties**: Easy, Normal (default), Hard — affects enemy spawn rate, speed, shoot frequency, lives, bombs, and boss HP
- **Three enemy types**: Grunt (1 HP, red), Interceptor (1 HP, fast, orange), Tank (4 HP, purple)
- **Six boss types**: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN (one per stage, cycling)
- **Two weapons**: Vulcan (spread, 3 levels) and Laser (piercing, 3 levels); power-ups switch/upgrade
- **Stage progression**: 18 enemy kills per stage triggers boss, infinite stages
- **Scoring**: Base points × multiplier (1–10x, resets after 2.2s without kills)
- **Input**: Keyboard (WASD/arrows + Space/click), mouse aim + click, touch joystick, mobile buttons

## Architecture

### Code Structure (all in index.html)

**HTML/CSS** (lines 1–359)
- Minimal markup: canvas, overlay UI, HUD displays (score, hearts, bombs, weapon, boss bar)
- Responsive layout with `clamp()` and safe-area insets for mobile notches
- Animations: heartbeat, screen flash, combo pop, stage banner, pause card
- Accessibility: screen-reader live region, reduced-motion preferences

**Game State** (lines 863–872)
- `player`: position, size, speed, cooldown, weapon/level, invulnerability frames
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- `score`, `lives`, `stage`, `kills`, `chain`, `multiplier`
- Difficulty settings persist to `localStorage` (best score, mute, difficulty, reduced motion)

**Core Game Loop** (line 1640–1653)
- `loop()`: requestAnimationFrame callback; calls `update()` then `draw()` every frame
- `update(dt)`: player movement, firing, collision detection, enemy/boss AI
- `draw()`: canvas rendering (background, player, enemies, bullets, particles, effects)
- Adaptive quality: tracks frame time; downgrades particle/shadow count if FPS drops

### Key Subsystems

**Input Handling** (lines 768–862)
- Keyboard: keys object tracks all pressed keys; checked in `update()`
- Mouse: canvas mousemove sets `mouseTarget`; player glides there if no keyboard input
- Touch: two zones—left side (move joystick), right side (fire + bomb buttons)
- Input priority: keyboard/joystick > mouse (never fight each other)

**Audio (Synthesized, no assets)** (lines 542–676)
- `ensureAudio()`: lazy-init WebAudio context (waits for user interaction)
- `beep()`: oscillator with optional frequency sweep, filter, panning, ADSR envelope
- `noiseBurst()`: white noise with lowpass filter and envelope
- `sfx` object: sound cues for shoot, explode (3 enemy types), bomb, hit, boss, UI, game over
- Mute toggle: persists to localStorage

**Rendering** (lines 1507–1638)
- `drawPlane()`: player and enemy ships (triangular fuselage, wings, cockpit, engines)
- `drawBossShip()`: boss with animated core, wings, and telegraph phase
- `drawClouds()`: parallax background layers
- Canvas quality flags: full shadows/glow at quality 0; reduced at quality 1–2

**Enemy & Boss AI** (lines 1022–1439)
- Enemy spawning: random location, patternRoll determines spread-fire or aimed-fire
- Weaving interceptors: sinusoidal movement offset by frame count
- Boss: enters from top, bounces side-to-side, telegraphs before attack patterns
- **Boss attack patterns** (lines 1372–1409): each boss has 3 patterns (radial, homing, columns, etc.)
  - Fire functions emit enemy bullets with `radial: true` flag; updated every frame in `updateBoss()`
  - Pattern index cycles; telegraph timer (22 frames) before each attack

**Collision Detection** (line 1069)
- `rectsOverlap(a, b)`: AABB—checks if centers < sum of half-widths
- Checked for: bullet–enemy, bullet–boss, enemy–player, bullet–player, boss–player, pickup–player

**Power-ups** (lines 1058–1067, 1304–1322)
- Spawn chances: 6% life (+1 up to max), 8% bomb (+1), 20% weapon (upgrade level or switch)
- `applyPowerUp()`: modifies player state and updates HUD

**Scoring & Combo** (lines 1087–1101)
- `addScore()`: increments chain; multiplier = 1 + floor(chain / 5) (max 10x)
- `chainTimer`: resets to 2.2s per kill; breaks to 1x after timeout
- Boss kill: +500 points

### Performance Tuning

- **Quality levels** (0 = full, 1 = medium, 2 = low):
  - Particle count: spawned less at low quality
  - Shadows/glows: disabled at quality 2
  - Reduced motion: disables animations (screen shake, flash, pulse)
- **Frametime tracking**: averaged over 45 frames; upgrades quality if avg > 25ms, downgrades if < ~18ms

## Modifying the Game

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502–506): define `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Add death animation to `EXPLODE_SFX` (line 678) if unique
3. Add SFX to `sfx` object (line 624–676)
4. Enemy automatically considered in `spawnEnemy()` filter by `minStage`

### Add a New Boss Attack Pattern
1. Define a pattern function in `BOSS_PATTERNS` (e.g., `b => fireRadial(b, 12, 150)`)
2. Add it to the array for the boss index (0–5 correspond to BOSSES order)
3. Pattern receives boss object; call fire functions (`fireRadial`, `fireAimedSpread`, etc.) to emit bullets

### Adjust Difficulty
- `DIFFICULTIES` object (line 485–489): tweak `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`
- Stage progression: change `KILLS_PER_STAGE` (line 491)

### Add Visual Theme
- Add entry to `THEMES` (line 493–500): define `sky` (3-stop gradient), `cloud` RGB, `sun` RGB, human-readable `name`
- Theme auto-cycles per stage

### Adjust Canvas Size
- `BASE_W`, `BASE_H` (line 753–754): default 480×720; respects device pixel ratio and safe areas
- Scaling: `Math.min(maxW / BASE_W, maxH / BASE_H, 1.6)` ensures responsive fit with max 1.6x zoom

## Key Code Patterns

**Game Loop Timing**: `dt` is clamped to 0.05s (20 FPS minimum) to prevent spiral on frame drops.

**Screen Shake**: `shakeTime` and `shakeMag` track intensity; applied in `draw()` via `ctx.translate()`. Respects reduced-motion preference.

**Player Invulnerability**: post-hit blink (alternating draw) + collision ignore for 1.5s via `player.invuln`.

**Boss Telegraph**: 22-frame visual wind-up before attack; SFX and boss core pulse during telegraph.

**Particle System**: simple structure—store velocity, life, decay by dt; filtered each frame.

## Testing & Debugging

- Open browser DevTools console (F12); no minification, easy to read
- Check localStorage (DevTools Application tab) for `skyfighter.*` keys (bestScore, muted, difficulty, reducedMotion)
- Performance: DevTools Performance tab; frame timeline visible in loop count
- Accessibility: disable colors in DevTools; reduced-motion test via `prefers-reduced-motion: reduce` media query

## Browser Support

- Modern browsers (Chrome, Firefox, Safari, Edge): WebAudio API, Canvas 2D, flexbox, CSS clamp, CSS variables
- Mobile: iOS 13+, Android Chrome 80+; fullscreen notch support via `env(safe-area-inset-*)`
- Graceful fallback: no WebAudio → no sound, game continues
