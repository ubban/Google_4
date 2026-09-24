# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in vanilla HTML5 with Canvas 2D rendering and Web Audio API for sound. The entire game is contained in a single `index.html` file (~1670 lines) with no external dependencies, build steps, or package managers.

**Key Features:**
- Three difficulty levels (Easy, Normal, Hard) with configurable spawn rates and enemy behavior
- Six unique boss encounters with distinct attack patterns
- Weapon system with two weapon types (Vulcan + Laser) and upgradeable levels
- Smart bomb mechanic for clearing enemies and damaging the boss
- Combo/multiplier system based on kill chain
- Cross-platform input: mouse steering, keyboard, and mobile touch controls with joystick
- Procedurally generated audio (no external sound files)
- Responsive canvas scaling for mobile and desktop
- Adaptive performance that reduces visual effects on low-end devices
- Local storage for persistent best score, muted state, difficulty, and reduced-motion preference

## How to Run

**Development:**
```bash
# No build step needed. Simply open the file in a browser:
open index.html          # macOS
xdg-open index.html      # Linux
start index.html         # Windows
```

Or use a simple HTTP server:
```bash
# Python 3
python3 -m http.server 8000

# Node.js (if available)
npx http-server
```

Then visit `http://localhost:8000/`

**Testing:** The game is fully self-contained. No automated tests exist; test manually in the browser by:
- Starting a game on each difficulty level
- Collecting power-ups and switching weapons
- Using bombs to clear enemies
- Reaching and defeating bosses
- Verifying audio playback and screen shake effects

## Code Architecture

### High-Level Structure

The 1670-line script follows this organization:

1. **DOM Setup** (lines 362-419): HTML structure for canvas, HUD, overlay, and touch controls
2. **Styles** (lines 10-359): CSS for game UI, animations (heart beat, combo pop, stage banner), and responsive design
3. **Game State & Constants** (lines 422-523): Difficulty presets, enemy types, boss designs, themes, and local storage
4. **Audio System** (lines 542-676): Web Audio synthesis pipeline (beep, noise burst, reverb/filter chains) and sound effect library
5. **UI Rendering** (lines 680-724): HUD updates, hearts, bombs, combo/stage banners
6. **Input Handling** (lines 768-861): Keyboard, mouse, touch events, and joystick emulation
7. **Game Loop** (lines 1640-1653): Animation frame loop calling update() then draw()
8. **Update Logic** (lines 1148-1302): Player/enemy movement, collision detection, spawn timing
9. **Render System** (lines 1520-1638): Canvas drawing for sky, clouds, player, enemies, bullets, boss, particles
10. **Boss System** (lines 1372-1439): Boss movement, attack pattern dispatch
11. **Game Events** (lines 1103-1505): Bomb detonation, power-ups, life loss, game over, pause

### Key Data Structures

**Player:**
```javascript
player = {
  x, y, w, h,              // Position and dimensions
  speed: 260,              // Movement speed (pixels/sec)
  cooldown, rapid,         // Fire rate tracking
  weapon: 'vulcan'|'laser',// Current weapon type
  level: 1|2|3,            // Weapon upgrade level
  invuln: 0                // Invulnerability timer (seconds)
}
```

**Enemies:** Basic (grunt, interceptor) and tanks; each has hp, type-specific colors, and spawn conditionally by stage.

**Boss:**
```javascript
boss = {
  x, y, w, h,
  hp, maxHp,               // Health tracking
  entering: true|false,    // Entry animation state
  design,                  // BOSSES[index] reference
  telegraph,               // Time until next attack (frames)
  patternIndex,            // Current attack pattern in BOSS_PATTERNS
  vx                       // Horizontal velocity (bounces off walls)
}
```

**Bullets & Enemy Bullets:** Support pierce (laser) or single-hit (vulcan). Enemy bullets include radial, aimed spread, and homing variants.

**Power-ups:** Life, bomb, weapon (type, x, y, falling speed). Spawn probabilistically from dead enemies.

**Particles:** Explosion fragments with velocity, life timer, and color for visual feedback.

### Adaptive Performance System

Lines 474-482 track average frame time and adjust quality level (0=full, 1=medium, 2=low):
- **Quality 0:** Full shadows, glow effects, screen shake
- **Quality 1:** Reduced particle count, less blur
- **Quality 2:** Minimal visual effects

This is checked in draw() to conditionally enable shadow/glow based on `quality`.

### Difficulty & Boss Patterns

**Difficulties** (lines 485-489) define spawn rates, enemy speed, shoot chance, lives, bombs, and boss HP multiplier.

**Boss Patterns** (lines 1372-1409): Six boss types, each with three distinct attack patterns:
- VANGUARD: Radial bursts, aimed fans, V-formations
- BEHEMOTH: Heavy columns, homing spreads, dense radials
- SPECTER: Teleport dash, tight spirals, double aimed shots
- DREADNOUGHT: 8-way cross, homing volleys, dense walls
- WRAITH: Teleport barrage, expanding rings, cross-spreads
- TITAN: Heavy columns, full rings, relentless homing

Patterns dispatch via `pattern(b)` function reference during telegraph phase.

### Input Priority

- **Keyboard/Joystick > Mouse:** Keyboard and joystick input take priority; mouse is fallback for steering
- **Touch:** Separate zones for movement (left) and actions (right: fire & bomb buttons)
- **Action Buttons:** Space/Fire (button), X/Shift/Bomb (button) always fire/bomb when pressed

### Theme & Visual Variety

Six themes cycle per stage (lines 493-500), each with unique sky gradient, cloud color, and sun glow. Changes per stage to prevent visual fatigue.

### Browser Storage Keys

- `skyfighter.bestScore` — High score
- `skyfighter.muted` — Audio mute flag
- `skyfighter.difficulty` — Last selected difficulty
- `skyfighter.reducedMotion` — Reduced motion preference (respects prefers-reduced-motion)

## Common Development Tasks

### Adding a New Enemy Type

1. Define in `ENEMY_TYPES` (line 502-506):
```javascript
newType: { w: 40, h: 40, hp: 2, speedMul: 1.2, color: '#ff0000', dark: '#8b0000', score: 25, minStage: 3 }
```
2. Add spawn logic condition in `spawnEnemy()` (line 1026).
3. Add explosion sound in `EXPLODE_SFX` (line 678) if needed.

### Adding a New Boss Attack Pattern

1. Define pattern function in `BOSS_PATTERNS` array (line 1372):
```javascript
b => fireRadial(b, 18, 200)  // Example: 18-bullet ring at 200 speed
```
2. Reuse existing fire functions (`fireRadial`, `fireAimedSpread`, etc.) or create new ones.

### Tuning Difficulty

Modify `DIFFICULTIES` object (line 485):
- `spawnBase` / `spawnMin`: Spawn interval in frames; lower = harder
- `enemySpeed`: Multiplier on enemy velocity
- `enemyShootChance`: 0-1, probability enemies shoot
- `bossHpMul`: Multiplier on boss health

### Adding a Sound Effect

1. Create a function in `sfx` object (line 624):
```javascript
newSfx: () => beep(880, 0.1, 'sine', 0.05, { filterFreq: 3000 })
```
2. Call via `sfx.newSfx()` where needed.
3. Use `beep(freq, duration, type, volume, opts)` or `noiseBurst(duration, volume, opts)`.

### Modifying Player Weapon

Edit `fireWeapon()` (line 1124) to change bullet spread, speed, or fire rate:
```javascript
const fireDelay = player.weapon === 'laser' ? 0.1 : 0.15;  // Line 1177
```

### Responsive Design Tweaks

- Canvas scaling happens in `resize()` (line 754); `BASE_W`/`BASE_H` define logical resolution.
- CSS uses `clamp()` for responsive font sizes; adjust min/preferred/max values as needed.
- Safe area insets handle notches on mobile (`env(safe-area-inset-*)`).

## Key Files & Lines Reference

| Task | File & Lines |
|------|------|
| Game loop & state | `index.html` 863–911 (state), 1640–1653 (loop) |
| Physics & collision | 1188–1302 (update, collision detection) |
| Rendering | 1520–1638 (draw) |
| Audio setup | 542–595 (ensureAudio, beep, noiseBurst) |
| Boss AI | 1411–1439 (updateBoss) |
| Player input | 768–861 (keyboard, mouse, touch) |
| Canvas sizing | 754–765 (resize & scaling) |
| Difficulty config | 485–489 (DIFFICULTIES) |
| Boss designs & patterns | 508–515, 1372–1409 (BOSSES, BOSS_PATTERNS) |

## Debugging Tips

1. **Muted at startup?** Check `localStorage.getItem('skyfighter.muted')` in browser console.
2. **Frame rate issues?** Open DevTools → Performance and check frame time in `trackPerf()`.
3. **Collision off?** Verify `rectsOverlap()` logic (line 1069); uses center-based rectangle intersection.
4. **Audio not playing?** Verify `audioCtx` is initialized in `ensureAudio()` and not null; some browsers require user gesture first.
5. **Touch controls not working?** Check `isTouchDevice` (line 778); may need gesture to resume audio context.
6. **Screen shake too intense?** Adjust `screenShake(mag, dur)` calls or check `reducedMotion` flag.

## Notes for Future Developers

- **No build or dependencies:** This keeps the game fast to load and easy to fork.
- **Canvas coordinate system:** Origin (0, 0) is top-left; +X is right, +Y is down.
- **Frame-based and time-based:** Spawning uses frame count; physics uses dt (delta time). Both coexist.
- **Global scope:** Everything runs in a single IIFE to avoid polluting global scope.
- **LocalStorage:** Game state persists across sessions. Clear via browser DevTools if needed.
- **Accessibility:** Uses screen reader announcements (aria-live) for game events; respects prefers-reduced-motion.
