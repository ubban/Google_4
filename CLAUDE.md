# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game, built as a self-contained HTML5 application (~1669 lines). It features a player aircraft that must survive enemy waves, defeat bosses across multiple stages, and accumulate the highest score.

## Architecture & Key Systems

### Structure
- Single `index.html` file containing all CSS and JavaScript
- Canvas-based rendering at 480x720 base resolution (responsive scaling)
- No build process, no dependencies, no external assets
- Game runs entirely in the browser with WebAudio API for sound synthesis

### Core Game Loop
```
requestAnimationFrame(loop) → update(dt) → draw() → render HUD
```
- `update(dt)`: Physics, collision detection, AI, state mutations
- `draw()`: Canvas rendering with theme-based sky gradients and effects
- Game state: `running`, `paused`, `gameOver` flags; score, lives, stage tracking

### Entity Systems

**Player (`player` object)**
- Position, velocity (keyboard/mouse/touch input driven)
- Two weapon types: Vulcan (rapid fire) and Laser (piercing beam with glow effect)
- Weapon level upgrade system (1-3, cycling between weapons at level cap)
- Invulnerability cooldown post-hit; visual blink during invuln
- Movement: keyboard (WASD/Arrows) → priority over mouse; mouse smooths toward crosshair; touch joystick on left half of screen

**Enemies (3 types defined in `ENEMY_TYPES`)**
- `grunt`: Standard fighter, 1 HP
- `interceptor`: Fast, weaving pattern, aggressive
- `tank`: Heavy, 4 HP, has health bar overlay
- Spawned via `spawnEnemy()` with difficulty-based frequency
- Enemies shoot at player (aim + spread patterns depending on type)
- Destroyed by player bullets; drop power-ups probabilistically via `maybeSpawnPowerUp()`

**Bosses (6 designs in `BOSSES` array, selected by stage modulo 6)**
- Each boss has unique attack pattern set (telegraphed attacks, cooldown cycles)
- Boss patterns defined in `BOSS_PATTERNS`: radial bursts, aimed spreads, columns, homing, teleport dashes
- Boss telegraph system: 22-frame countdown before attack, visual whitening of core
- Health bar HUD shows boss HP; damage indicator `boss.telegraph` controls core glow
- Boss enters from top with smooth ease-in; bounces horizontally

**Bullets & Particles**
- Player bullets: stored in `bullets` array; may be laser (piercing) or vulcan (single-hit)
- Enemy bullets: `enemyBullets` array; various velocities and patterns
- Particles: short-lived visual effects (`particles` array) spawned on hits/explosions

### Difficulty & Progression

**Three difficulty presets** (`DIFFICULTIES` object):
- `easy`: Lower spawn rate, slower enemies, 4 lives, reduced boss HP
- `normal`: Baseline; 3 lives, standard boss HP
- `hard`: Aggressive spawning, fast enemies, only 2 lives, boosted boss HP

**Stage Progression:**
- Player must kill `KILLS_PER_STAGE` (18) enemies per stage
- After 18 kills, boss spawns after 1.2s delay
- Defeating boss increments stage, resets kill counter, applies next theme
- 6 unique visual themes cycle: Daylight Front → Sunset Corridor → Night Approach → Storm Front → Arctic Wastes → Volcanic Ridge

**Scoring & Multiplier:**
- Base score: `enemy.score` (10/15/35 depending on type) × `multiplier` (starts 1, increases with kill chain)
- Every 10 kills milestone triggers `showComboPop()` UI announcement
- Chain breaks on hit; resets multiplier to 1
- Boss kill awards 500 points

### Input Systems

**Keyboard:**
- WASD / Arrow Keys: Move player
- Space / Click: Fire
- X / Shift: Use bomb
- P / Esc: Pause

**Mouse:**
- Move: Player glides toward cursor position (lower priority than keyboard)
- Click: Continuous fire (fire flag set while button held)

**Touch (mobile):**
- Left half of screen: Joystick input (move zone)
- Bottom-right quadrant: Fire & Bomb buttons (78px circles)
- Joystick renders base + stick during active touch

### Audio

**WebAudio Synthesis** (no external files):
- Synth beeps with frequency sweeps, filters, panning
- Noise bursts for explosions
- Compression for dynamic range limiting
- Per-sound effect: `sfx.playerShoot()`, `sfx.explode()`, `sfx.bossTelegraph()`, etc.
- Mutable via Settings button; state persists in LocalStorage

### Visual Systems

**Themes:**
- Sky gradient: 3-stop linear gradient (top to bottom)
- Cloud layers: 2 parallax layers at different speeds
- Sun glow: Radial gradient overlay
- Each theme ties to stage cycle, reset every 6 stages

**Effects:**
- Screen shake: Magnitude & duration; reduced motion mode cuts by 85%
- Flash overlay: White flash on hit (0.25s), yellow flash on bomb (0.5s)
- Vignette pulse: Red pulsing vignette when player health is critical (1 life remaining)
- Combo pop: Large animated text when hitting 10/20/30 kills
- Stage banner: Slide-in title on new stage

**Reduced Motion Mode:**
- Accessible via Settings button (🎆)
- Disables shake, flash, pulse animations; reduces animation durations
- Respects system `prefers-reduced-motion` by default
- Stored in LocalStorage

### Accessibility

- Screen reader announcements via `announce(text)` → `#srLive` live region
- Announcements on: boss defeat, stage progression, settings changes, game over + final score
- Semantic HTML structure with roles/aria-live

## Common Modifications & Extension Points

### Adding a New Boss Pattern
1. Add entry to `BOSSES` array with colors (`hullTop`, `hullBottom`, `wing`, `core`, `hpMul`)
2. Add pattern set to `BOSS_PATTERNS` array (same index)
3. Patterns call firing functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`, or define custom inline
4. Boss enters at stage where `(stage - 1) % 6 === BOSSES.indexOf(newBoss)`

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with stats (`w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`)
2. Adjust spawn logic in `spawnEnemy()` to include new type based on stage
3. Rendering handled automatically by `drawPlane()` or add custom logic in draw loop

### Tweaking Difficulty
- Modify `DIFFICULTIES` object: `spawnBase` / `spawnMin` (spawn rate), `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`
- Difficulty scales dynamically: every 300 frames (~5s at 60fps), enemy spawn frequency decreases by 4 frames up to `spawnMin`

### Game Balance Knobs
- `BASE_W`, `BASE_H`: Canvas resolution (affects hitbox scaling)
- `KILLS_PER_STAGE`: Enemies needed before boss
- Individual `ENEMY_TYPES` stats: `hp`, `speedMul`, `score`
- Weapon cooldowns in player firing logic (search `player.cooldown`)
- Power-up drop rates: `maybeSpawnPowerUp()` has hardcoded percentages (40% life, 40% bomb, 20% weapon)

## Testing & Verification Workflow

1. **Open in Browser:** `index.html` directly or through a local server
2. **Test Each Difficulty:** Play through Easy/Normal/Hard to verify spawn rates, enemy behavior, boss patterns
3. **Test All Input Methods:** Keyboard, mouse, touch (use browser DevTools mobile emulation)
4. **Check Accessibility:** Toggle reduced motion; use screen reader (browser accessibility inspector)
5. **Verify Audio:** Test mute/unmute; confirm each SFX plays (beeps, explosions, boss telegraphs)
6. **Stages & Bosses:** Reach stage 2+ to confirm theme switch, boss entry, telegraph/attack cycle
7. **Edge Cases:**
   - Hit invulnerability: Get hit once, verify blink + collision immunity
   - Chain break: Get hit mid-chain, confirm multiplier resets
   - Boss defeat: Confirm stage banner, health bar disappears, next boss spawns
   - Game over: Verify final score display, "New Best Score" message if applicable

## Performance Notes

- Adaptive quality system: Tracks frame times, degrades particle glow effects if FPS drops
- Canvas clamped at max 1.6× scale to prevent memory pressure on large screens
- Particle pooling via `particles` array splice (lifecycle-based cleanup)
- No asset loading (all synthesis/drawing in-memory)
- Shadow blur disabled on low-quality setting

## File Storage (LocalStorage Keys)

- `skyfighter.bestScore`: Best score (integer)
- `skyfighter.muted`: Audio mute state ('0' or '1')
- `skyfighter.difficulty`: Last selected difficulty ('easy', 'normal', 'hard')
- `skyfighter.reducedMotion`: Accessibility toggle ('0' or '1')
