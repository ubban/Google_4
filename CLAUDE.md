# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single self-contained HTML file. It requires no build process, external dependencies, or server—just open `index.html` in a browser to play.

- **Type:** Browser-based game
- **Tech Stack:** Vanilla JavaScript (ES6), Canvas 2D API, Web Audio API, HTML/CSS
- **File Structure:** Single `index.html` file (≈59KB) containing all game logic, rendering, audio, and UI
- **No Dependencies:** Pure browser APIs—no npm packages, no build tools

## Architecture & Game Flow

### Core Game Loop

The game uses `requestAnimationFrame` for a consistent update loop with delta-time calculations:

1. **Input Processing** → **Update Game State** → **Render Canvas** → loop

Key functions:
- `update(dt)` — Updates positions, collisions, spawning, state changes
- `draw()` — Renders all game elements to canvas
- `loop(now)` — Main game loop, handles timing and frame limiting

### Game State Structure

Global state tracked in these main arrays:
- `player` — Player ship object (position, health, weapon, cooldowns)
- `bullets` — Player projectiles
- `enemies` — Spawned enemy ships
- `enemyBullets` — Enemy projectiles
- `boss` — Current boss object (null when no boss active)
- `particles` — Explosion effects and visual debris
- `powerUps` — Dropped items (life, bomb, weapon)

Scalar state:
- `lives`, `score`, `stage`, `kills`, `stageKills` — Game progression
- `multiplier`, `chain`, `chainTimer` — Combo system
- `running`, `paused`, `gameOver` — Game mode flags

### Difficulty System

Three difficulty presets control game parameters:

```javascript
DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
};
```

Spawn rate is dynamic—gets tighter as frame count increases (difficulty ramps over time within a stage).

### Enemy & Boss System

**Enemy Types:**
- `grunt` — Basic red enemy, 1 HP
- `interceptor` — Fast orange weaver, 1 HP
- `tank` — Heavy purple tank, 4 HP, spawns from stage 2

**Enemy AI:**
- `pattern: 'spread'` — Fires a 3-way spread shot
- `pattern: 'aimed'` — Fires directly at player position
- `weave: true` — Sine-wave movement pattern for variety

**Boss System:**
- 6 boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), cycling per stage
- Each boss has unique attack patterns in `BOSS_PATTERNS` array
- Boss health scales with stage and difficulty (`baseHp = 70 + stage * 35`)
- Telegraph system: Boss shows visual warning (core flashing white) ~22 frames before attack

### Stage Progression

- **KILLS_PER_STAGE = 18** — Kill 18 enemies to trigger boss spawn
- Boss defeated → Stage incremented → New theme loaded
- Themes cycle through 6 environmental skins (Daylight, Sunset, Night, Storm, Arctic, Volcanic)

### Weapon System

**Vulcan (default):**
- Levels 1–3: Spread increases from single shot → 2-way → 3-way
- Cooldown: 0.15s

**Laser:**
- Levels 1–3: Lanes increase from center → 2 outer → 2 outer + center
- Pierce: Lasers ignore enemies after 1–3 hits depending on level
- Cooldown: 0.1s (faster)
- Weapon switching: Pick up power-up at level 3 to swap to other weapon at level 1

### Combo & Score System

**Combo Chain:**
- Each kill increments `chain` counter
- Counter resets if no kills for 2.2 seconds
- Multiplier = `1 + floor(chain / 5)` (capped at 1–10x)
- Score = base kill points × multiplier

Example: VANGUARD boss kill is 500 points base → 5000 at 10x multiplier.

### Input Handling

**Keyboard/Gamepad Priority:**
1. Arrow keys + WASD → Movement (normalized if multiple pressed)
2. Space → Fire (hold to continuous fire)
3. X or Shift → Use bomb
4. P or Escape → Pause
5. Joystick (virtual on touch) overrides keyboard magnitude

**Mouse:**
- Player glides toward cursor when mouse is over canvas
- Click to fire
- Keyboard/joystick input takes priority (no fighting)

**Touch:**
- Left 55% of screen: Virtual joystick for movement
- Right 45% of screen: Fire + Bomb buttons
- Fire button can be held for continuous fire

### Audio

All sound is procedurally generated using Web Audio API synth nodes—no audio files:

- `beep(freq, dur, type, vol, opts)` — Oscillator-based tones with sweep/filter/pan
- `noiseBurst(dur, vol, opts)` — Filtered noise for explosions
- Sound effects in `sfx` object: `shoot`, `laser`, `explodeGrunt`, `explodeTank`, `bomb`, `powerup`, `bossHit`, `hit`, `gameover`, etc.

Audio context is lazy-initialized on first user interaction (browser policy).

### Visual Effects & Performance

**Screen Shake:**
- `screenShake(mag, dur)` — Applied to whole canvas context in draw loop
- Reduced by 85% if reduced-motion preference is enabled

**Adaptive Quality:**
- Tracks frame times and adjusts quality level (0=full, 1=medium, 2=low)
- High quality: Full particle counts, glow effects, shadow blur
- Low quality: 35% particle counts, no glow/shadow effects

**Particle System:**
- Particles velocity-based, gravity-less
- Lifecycle-based removal

### Canvas & Responsive Design

- **Base resolution:** 480×720 (portrait)
- **Scaling:** Scales up to 1.6x based on viewport, maintaining aspect ratio
- **Safe areas:** Respects safe-area-inset for notched/folded devices

## Input/Output

**Input:**
- Keyboard: Arrow keys, WASD, Space, X, Shift, P, Escape
- Mouse: Move (steering), Click (fire)
- Touch: Joystick (left zone) + buttons (right zone)
- Gamepad: Virtual joystick through touch

**Output:**
- Canvas 2D rendering (all visuals)
- Web Audio API (all sound)
- localStorage: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- DOM updates: HUD (score, health, bombs, weapon), pause overlay, boss bar

## LocalStorage Keys

- `skyfighter.bestScore` — Best score across sessions
- `skyfighter.muted` — Mute state (boolean as '0' or '1')
- `skyfighter.difficulty` — Last selected difficulty ('easy', 'normal', 'hard')
- `skyfighter.reducedMotion` — Reduced motion preference ('0' or '1')

## Common Modifications

### Adding Enemy Types

Define in `ENEMY_TYPES` object with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optionally `weave`. Spawn logic in `spawnEnemy()` filters by `minStage`.

### Adding/Modifying Boss Patterns

Add pattern function to `BOSS_PATTERNS` array. Pattern functions take boss object and call fire helper functions:
- `fireRadial(b, count, speed, offset?)` — N-way burst
- `fireAimedSpread(b, count, speed, spreadStep)` — Aimed fan
- `fireColumns(b, speed)` — Vertical columns
- `fireSpiralBurst(b, speed)` — Rotating spiral
- `fireHomingSeeded(b, count, speed)` — Targeted spread

### Adjusting Difficulty Curves

Modify `DIFFICULTIES` object or the spawn ramp formula in `update()` (around line 1201).

### Theming

Visual themes defined in `THEMES` array—each has `sky` (3-stop gradient), `cloud`, and `sun` color definitions. Add new theme or modify existing for different stage appearances.

## Testing in Browser

Simply open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge). The game has no build step.

- To test reduced motion: Use browser DevTools to toggle `prefers-reduced-motion` media query
- To test touch: Use mobile device or DevTools device emulation
- To test frame rate: Open DevTools → Performance tab → record and check FPS

## Accessibility

- ARIA live region (`#srLive`) announces game events (stage clear, game over, etc.)
- Screen reader friendly buttons and labels
- Respects `prefers-reduced-motion` system preference by default
- Reduced motion toggle in UI overrides system preference
- High contrast text with text-shadow for readability
- Touch targets sized appropriately (buttons 36–78px)

## Common Issues & Debug

**Sound not playing:**
- Check if muted (🔇 icon active)
- Web Audio requires user gesture before playing—must click canvas or button first
- Some browsers block sound on muted tabs

**Lag/stuttering:**
- Check adaptive quality system (`quality` variable)
- Lower `QUALITIES` effect counts if needed
- Reduce particle limits in `spawnExplosion()`

**Touch controls not working:**
- Verify touch events are firing (check `ontouchstart in window`)
- Check `#moveZone` and `#actionZone` pointer-events are `auto`

**Boss HP bar not updating:**
- Check `bossBarWrap.classList.add('show')` is called in `spawnBoss()`
- Verify `boss.hp` is decremented on bullet/bomb hit
