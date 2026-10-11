# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. The entire game is implemented as a single self-contained HTML file with embedded CSS styling and JavaScript game engine. It features:

- Player aircraft that moves and shoots enemies
- Multiple enemy types with distinct behaviors (Grunt, Interceptor, Tank)
- Six boss encounters with unique attack patterns
- Weapon upgrade system (Vulcan spreads vs Laser piercing shots)
- Smart bomb ability to clear screen
- Combo multiplier system for consecutive kills
- Difficulty modes (Easy, Normal, Hard)
- Canvas-based rendering with adaptive performance
- Web Audio API synthesized sound effects
- Touch controls for mobile devices
- Accessibility features (screen reader support, reduced motion preference)

## Running the Game

Simply open `index.html` in a web browser. The game is fully playable immediately with no build or dependency installation required.

## Architecture Overview

The entire game logic lives in a single `<script>` block within `index.html`. The architecture follows these major sections:

### State Management & Game Loop
- **Game state**: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- **Session state**: `score`, `lives`, `bombCount`, `stage`, `kills`, `chain`, `multiplier`
- **Loop mechanism**: `requestAnimationFrame` drives the main update/draw cycle at `update(dt)` then `draw()`
- **Performance tracking**: `trackPerf()` monitors frame times and scales quality (fewer particles, reduced glow effects) on slower devices

### Core Game Systems

**Player Entity** (`player` object)
- Position, size, speed attributes
- `weapon`: switches between 'vulcan' (spread shots) and 'laser' (piercing)
- `level`: 1–3, affects spread pattern or lane count
- `cooldown`: fire rate limiter
- `invuln`: invincibility timer after taking damage

**Enemy Spawning & Behavior**
- `ENEMY_TYPES`: Grunt (basic), Interceptor (fast weaver), Tank (slow tanky)
- Each stage gates which enemy types appear via `minStage`
- Enemies follow spawn timer (`spawnBase`, `spawnMin` per difficulty) that ramps down as frames progress
- Enemy AI: either 'spread' pattern (fixed angles) or 'aimed' (toward player)
- Interceptor enemies have `weave: true` to oscillate side-to-side

**Boss System**
- Six boss designs in `BOSSES` array, cycled per stage
- Each boss has unique HP multiplier and visual design
- `BOSS_PATTERNS`: Array of 6 pattern sets; each set has 3 attack phases that rotate
- Boss pattern functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`
- Boss telegraph mechanic: 22-frame windup before firing, plays audio cue

**Weapon System**
- Vulcan: horizontal straight shots with spread at L2–L3 (up to 3 lanes)
- Laser: vertical piercing shots; at L2+ fires from offset lanes; pierce allows hitting multiple enemies before destroying bullet
- `fireWeapon()` called when cooldown <= 0 and fire button held

**Combo & Multiplier**
- `chain`: increments on each kill; resets on damage or after `chainTimer` expires (2.2s window)
- `multiplier`: `1 + floor(chain / 5)` capped at 10x
- Breaking chain: when player takes damage or timer expires

**Difficulty Modes** (`DIFFICULTIES` object)
- Scales: spawn rate, enemy speed, shoot frequency, lives, bombs, boss HP
- Stored in localStorage as `skyfighter.difficulty`

**Input Handling**
- Keyboard: Arrow keys / WASD for movement, Space for fire, X/Shift for bomb, P/Esc to pause
- Mouse: Move pointer to steer, click to fire (only when over canvas)
- Touch: Left zone (55% width) for joystick movement, right zone (45% width) for FIRE/BOMB buttons
- Joystick visual feedback with `#joystickBase` / `#joystickStick` elements

**Audio System** (WebAudio synthesis, no external assets)
- `ensureAudio()`: Lazy-initializes AudioContext on first interaction (mobile requirement)
- `beep()`: Oscillator-based tones with optional frequency sweep, filter, stereo pan
- `noiseBurst()`: White noise with envelope and filter
- `sfx` object: Named sound effects for all game actions (shoot, explode, hit, bomb, powerup, boss telegraph, etc.)
- Mute state stored in localStorage as `skyfighter.muted`

**Visual Effects**
- Screen shake: `screenShake(magnitude, duration)` applies random offset to canvas translation
- Flash overlay: White flash on hit, yellow-orange flash on bomb
- Vignette pulse: Red radial gradient on player when health = 1
- Particle system: `particles[]` for explosion graphics, velocity-based movement with life decay
- Combo pop-up & stage banner: Floating text animations

**Themes** (`THEMES` array)
- 6 stage themes with gradient sky colors, cloud color, sun color
- Each theme has a name; cycled per stage

### Rendering

**Canvas Setup**
- Base resolution: 480×720 (portrait)
- Scales to fit window up to 1.6x
- Aspect ratio maintained with flexbox centering

**Drawing Order**
1. Sky gradient + sun glow
2. Animated clouds (parallax at different speeds)
3. Player aircraft (with flame when accelerating, or laser aura)
4. Player bullets (yellow for vulcan, cyan for laser, with glow on high quality)
5. Enemy bullets (red circles)
6. Enemy aircraft
7. Boss aircraft
8. Power-up pickups (rotating colored squares)
9. Particles (explosion debris)
10. HUD overlay (score, lives, multiplier, etc. rendered via HTML elements, not canvas)

**HUD & Overlays** (DOM-based, not canvas)
- `#overlay`: Start screen, pause screen, game over screen
- `#hud`: Score, best, multiplier, weapon, hearts, bombs (at top center)
- `#bossBarWrap`: Boss health bar (shown when boss active)
- `#comboPop`, `#stageBanner`: Floating text animations
- Touch controls: Joystick base, action buttons

## Key Functions & Data Structures

**Game Flow**
- `resetGame()`: Initialize/reset all state, show start screen
- `update(dt)`: Physics, collision, scoring, enemy/boss behavior
- `draw()`: Render everything
- `loop(now)`: requestAnimationFrame driver, calls update & draw, tracks delta time

**Player Movement**
- Keyboard/joystick input accumulates velocity vector
- Mouse input: player glides toward pointer target (lower priority than keyboard/joystick)
- Clamped to canvas bounds

**Collision Detection**
- `rectsOverlap(a, b)`: AABB collision (checks distance, not pixel-perfect)
- Bullet-enemy, bullet-boss, enemy-player, enemy-bullet-player, bomb-enemy, bomb-boss

**Scoring & Progression**
- `addScore(base)`: Adds `base * multiplier` to score, increments chain
- `killBoss()`: 500 base points, advances stage, spawns new boss after 1.2s
- Stage progression: 18 kills per stage triggers boss spawn after warning
- Best score persisted in localStorage

**Power-ups**
- `maybeSpawnPowerUp(x, y)`: Random chance on enemy death (6% life, 8% bomb, 20% weapon)
- `applyPowerUp(type)`: Apply effect, play sound

**Pause/Resume**
- `togglePause()`: Show/hide overlay, stop game loop updates
- Pause state persists across pause/resume

**Difficulty Selection**
- Buttons on start screen (Easy, Normal, Hard)
- Only changeable when not running
- Persisted in localStorage

## Development Tips

### Modifying Game Difficulty
Edit the `DIFFICULTIES` object constants:
- `spawnBase/spawnMin`: Control enemy spawn frequency (lower = faster spawns)
- `enemySpeed`: Multiplier on enemy velocity
- `enemyShootChance`: Probability enemy fires per frame (0–1)
- `lives`, `bombs`: Starting resources
- `bossHpMul`: Boss health multiplier

### Adding/Modifying Boss Patterns
1. Define attack function in `BOSS_PATTERNS[bosIndex][phaseIndex]` 
2. Use helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.
3. Fire functions create entries in `enemyBullets[]` array
4. Set `b.telegraph = 22` to trigger telegraph phase before attack

### Changing Weapon Behavior
Edit `fireWeapon()` function:
- Vulcan: Adjust spread angle with `Math.sin(a)` term or lane count
- Laser: Adjust pierce value or lane offsets
- Adjust `fireDelay` to change fire rate per weapon

### Adding New Enemy Types
1. Add entry to `ENEMY_TYPES` object (define w, h, hp, speed multiplier, color, etc.)
2. Set `minStage` to control when it appears
3. Add weave pattern if needed (weaving interceptor is example)
4. Ensure explosion SFX key exists in `EXPLODE_SFX` mapping

### Audio Tuning
- `beep()` params: freq (Hz), duration (s), type ('square'/'sine'/'sawtooth'/'triangle'), vol (0–1), opts (sweep, filter, pan)
- `noiseBurst()` params: duration, vol, opts (filter type, frequency)
- All synthesis happens in real-time; no precomputed assets

### Performance Optimization
- Quality levels: 0 (full glow), 1 (medium), 2 (low)
- Auto-scales based on average frame time over 45 frames
- Particle count scales by quality (see `spawnExplosion()`)
- Reduce `shadowBlur` values on low quality

### Accessibility
- Respects `prefers-reduced-motion` media query
- Reduced motion button in top-right toggles effects (animations, shakes, flashes)
- Screen reader announcements via `announce()` function (updates `#srLive` aria-live region)
- Stored in localStorage as `skyfighter.reducedMotion`

### Storage Keys
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Reduced motion preference

## File Structure

- `index.html`: Entire game (HTML markup + inline CSS + inline JavaScript)
- `README.md`: Simple project name/timestamp
- `.git/`: Version history

## Debugging Tips

- Open browser DevTools to see console errors
- Check `console.log()` for game state snapshots
- Frame counter in code: `frame` variable increments every game loop
- Collision boxes can be visualized by drawing rects in `draw()` function
- Audio context state: check `audioCtx.state` (may be 'suspended' on mobile until first user interaction)
- Performance: Use DevTools Performance tab to profile frame times
