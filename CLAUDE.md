# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file web application. The entire game—rendering, game logic, audio synthesis, input handling, and UI—lives in `index.html`.

## Architecture

The game is structured as a self-contained vanilla JavaScript application with:

- **Canvas 2D rendering** for all graphics (no external assets)
- **Web Audio API** for synthesized sound effects (no audio files)
- **localStorage** for persistence (best score, settings)
- **Responsive design** supporting desktop (keyboard/mouse) and mobile (touch) controls

### Core Game Loop

The main update-render loop runs in `requestAnimationFrame` via the `loop()` function:
1. `update(dt)` — game logic (player movement, collision detection, spawning, scoring)
2. `draw()` — canvas rendering with theme-based visuals
3. Performance tracking adjusts visual quality on slower devices

### Key Game Systems

**Player & Input**
- Player plane steers toward mouse/joystick, keyboard (arrows/WASD), or static position
- Firing: continuous (Space bar or touch button) with weapon-dependent cooldown
- Bombs: discrete (X/Shift or touch button); limited quantity but rechargeable
- Input priority: keyboard/joystick > mouse when both active

**Enemies**
- Three types spawned based on stage: `grunt`, `interceptor` (fast, evasive), `tank` (slow, tanky)
- Enemies spawn randomly from top, move downward with optional weaving
- Shoot on timers with two patterns: spread (outward) or aimed (at player)
- When destroyed, drop power-ups; defeated enemies award score multiplied by kill chain

**Bosses**
- Spawn after defeating 18 enemies per stage; 6 unique designs with distinct attack patterns
- Each boss design has 3 attack patterns that cycle; telegraph animation before each attack
- Boss bullets are radial (moving via `dx`/`dy` not `speed`/direction like enemy bullets)
- Defeat boss → next stage, theme rotates, spawn timer resets

**Scoring & Progression**
- `chain` counter increments on each kill; breaks after 2.2 seconds
- `multiplier` = 1 + floor(chain/5), capped at 10x
- Difficulty settings scale enemy spawn rate, speed, health, and boss HP
- Storage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`

**Collision & Hit Detection**
- All collisions use AABB overlap: `rectsOverlap(a, b)` checks center-based rect intersection
- Bullets pierce bosses by `pierce` count (laser weapon)
- Invulnerability after hit: 1.5s window, player blinks visually
- Bombs clear all on-screen enemy bullets + damage all enemies/boss

**Rendering & Themes**
- Six scrolling themes with gradient skies, sun glows, and animated cloud layers
- Theme selection cycles by stage: `THEMES[(stage - 1) % THEMES.length]`
- Boss designs and enemy colors are theme-independent constants
- Visual quality downgraded on slow devices: shadow/glow effects and particle counts reduced

### Weapon System

Two weapons with upgrade levels:

- **Vulcan** (default): fast, spread pattern widens at level 3
  - Cooldown: 0.15s, single shot + 2 at level 2 + 3 at level 3
- **Laser** (power-up): piercing, can penetrate enemies
  - Cooldown: 0.1s, pierce value allows passing through multiple targets
  - Weapon swap resets level to 1

## Common Development Tasks

### Running the Game

The game runs in any modern browser. To test locally:

```bash
# Simple HTTP server (Python 3)
python3 -m http.server 8000
# Then open http://localhost:8000

# Or Node http-server
npx http-server
```

### Code Organization Within index.html

The file is structured in logical sections:

| Lines | Section |
|-------|---------|
| 1–359 | HTML structure + CSS (layout, animations, UI states) |
| 361–421 | DOM elements query and setup |
| 423–677 | Constants & audio synth (difficulty, enemy types, bosses, sound effects) |
| 679–765 | Input handling & canvas setup |
| 863–912 | Game state initialization |
| 913–1020 | Drawing functions (planes, bosses, effects) |
| 1022–1122 | Spawning & destruction (enemies, power-ups, explosions) |
| 1124–1302 | Update logic (player, bullets, enemies, collision) |
| 1324–1439 | Boss attack patterns & behavior |
| 1440–1505 | Game events (game over, pause, difficulty) |
| 1507–1638 | Rendering pipeline (backgrounds, sprites, particles) |
| 1640–1666 | Main loop & startup |

### Making Changes

**To modify game balance:**
- Adjust `DIFFICULTIES` object (spawn rates, enemy speeds, boss HP multipliers)
- Edit `KILLS_PER_STAGE` to change when bosses appear
- Modify weapon cooldowns or spread angles in `fireWeapon()`

**To add/change boss attacks:**
- Add a pattern function (e.g., `fireRadial`, `fireAimedSpread`) or compose existing ones
- Insert into the boss's array in `BOSS_PATTERNS`
- Pattern functions mutate `enemyBullets` and use `radial` movement

**To adjust scoring:**
- Edit `addScore()` multiplier logic
- Modify `chain` breakup timer or thresholds
- Adjust base scores in `ENEMY_TYPES` and `killBoss()` reward

**To change visual themes:**
- Add to or edit `THEMES` array (sky gradients, cloud color, sun)
- Boss and enemy colors are in `BOSSES` and `ENEMY_TYPES` constants (independent of theme)

**To add audio:**
- New SFX functions in `sfx` object use `beep()` and `noiseBurst()` helpers
- Synth parameters: frequency, duration, type, volume, sweep, filter, panning
- Call with `ensureAudio()` to initialize Web Audio context on first interaction

### Performance Optimization

The game adapts visual quality automatically:

- `quality` variable (0=full, 1=medium, 2=low) adjusts in response to frame times
- Shadow/glow effects skip at `quality > 0`
- Particle counts scale with quality
- Particle generation (explosions) capped based on `quality`

To test performance on a slow device, manually set `quality = 1` or `2` in browser console.

### Accessibility

- Screen reader live region (`#srLive`) for status announcements
- Reduced motion preference detected from OS and toggleable in UI
- Animations and flash effects skipped when reduced motion is enabled
- Color scheme respects `prefers-color-scheme: dark`
- Touch controls with large hit areas (78px buttons)

## Key Code Patterns

**Collision Detection:**
```javascript
if (rectsOverlap(objectA, objectB)) {
  // Handle collision
}
```

**Spawning Particles:**
```javascript
spawnExplosion(x, y, color, count);
// Creates particles with random velocity
```

**Audio Synthesis:**
```javascript
beep(frequency, duration, waveType, volume, {sweepTo, filterFreq, pan});
noiseBurst(duration, volume, {filterFreq, pan});
```

**Boss Pattern Definition:**
```javascript
const BOSS_PATTERNS = [
  [ // Boss 0
    b => fireRadial(b, count, speed),
    b => fireAimedSpread(b, count, speed, spreadStep),
    // ... more patterns
  ],
  // ... more bosses
];
```

## Testing & Debugging

- Open browser DevTools (F12)
- Set breakpoints in the game loop (`update`, `draw`)
- Modify game state in console: `score += 1000`, `bombCount = 5`, `stage = 5`
- Manually set `quality = 2` to test low-end performance
- Check `localStorage` entries for persistence data
- Press P or Esc to pause mid-game for observation

## Deployment

The game is production-ready as a static file. Simply serve `index.html`:

- GitHub Pages: commit and push (index.html will be served)
- Any static host: upload `index.html`
- No build step, no dependencies, no server required

## Notable Browser Compatibility

- **Canvas 2D**: all modern browsers
- **Web Audio API**: all modern browsers (may need `webkitAudioContext` fallback)
- **localStorage**: all modern browsers
- **Touch events**: all modern browsers
- **Media query `prefers-reduced-motion`**: widely supported; graceful fallback on older browsers
