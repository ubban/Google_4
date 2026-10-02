# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a browser-based, Raiden-style vertical shoot-'em-up game. It's a single-file HTML/CSS/JavaScript application (~1669 lines) designed to run in modern browsers with no external dependencies beyond the browser's standard APIs (Canvas 2D, Web Audio, localStorage).

## How to Run and Test

**Running the game:**
- Open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge)
- The game auto-starts with the title screen overlay; select difficulty and click "Start Game"
- No build step, no server required — it's ready to run immediately

**Testing during development:**
- Open the browser DevTools (F12) to check console for any errors
- Test all input methods:
  - **Keyboard**: Arrow keys or WASD to move, Space to fire, X or Shift for bomb
  - **Mouse**: Move pointer to steer (while hovering canvas), click to fire
  - **Touch**: Drag on left side for movement, tap FIRE button for weapon, tap BOMB button
- Test across screen sizes (use DevTools device emulation) to verify responsive layouts
- Check localStorage (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`) to verify persistence
- Listen for Web Audio output when sounds are enabled

**Key browser features used:**
- Canvas 2D rendering (entire game rendered via `ctx.` methods)
- Web Audio API for procedural sound generation (no audio files)
- localStorage for settings and best score persistence
- requestAnimationFrame for the game loop

## Codebase Architecture

The entire game runs in one IIFE (Immediately Invoked Function Expression) within a `<script>` tag. The main architectural components:

### Game State
All game entities and state live as global variables within the IIFE scope:
- `player`: Player ship object with `{x, y, w, h, speed, cooldown, rapid, weapon, level, invuln}`
- `enemies`: Array of enemy ships; each has `{x, y, w, h, hp, type, cooldown, rapid}`
- `bullets`: Array of player-fired bullets with `{x, y, vx, vy, w, h, type}`
- `enemyBullets`: Array of enemy-fired bullets
- `particles`: Array of visual effects (explosions, sparks)
- `powerUps`: Array of weapon/bomb pickups
- `boss`: Current stage boss (null when not present)

Supporting state:
- `score`, `lives`, `maxLives`: Player progression
- `bombCount`, `maxBombs`: Bomb system
- `stage`, `stageKills`: Stage progression (boss spawns after KILLS_PER_STAGE)
- `chain`, `chainTimer`, `multiplier`: Combo/scoring multiplier
- `running`, `paused`, `gameOver`: Game loop control flags
- `quality`: Adaptive performance quality level (0=full, 1=medium, 2=low)

### Core Systems

**Rendering**
- `ctx` (2D canvas context) draws all sprites procedurally:
  - Player ship: `drawPlane()`
  - Boss ships: `drawBossShip()` with animated cores
  - Enemies, bullets: simple rectangles and circles
  - Backgrounds: `drawClouds()` with parallax scrolling
  - Particles: individual rectangles with fade

**Game Loop** (invoked via `requestAnimationFrame`)
- Calls `update(dt)` to advance game state by delta time `dt` (in seconds)
- Clears canvas and redraws all entities
- Handles pause/resume; applies adaptive quality scaling if frame time exceeds threshold

**Input Handling**
- Keyboard: Direct event listeners on key state
- Mouse: Canvas pointer events; player glides toward mouse target (overridden by keyboard/joystick)
- Touch: Joystick simulation on left side via `touchstart`/`touchmove`/`touchend` with visual feedback

**Physics & Collision**
- Movement: Direct position updates (`x += vx * dt`)
- Collision: `rectsOverlap()` checks bounding boxes (not pixel-perfect)
- Damage: On collision, entities lose HP; at 0 HP, spawn particles and remove from array

**Enemy AI**
- Spawning: Timer-based (`spawnTimer`), difficulty-scaled spawn rates
- Behavior: Move downward; random chance to fire; some types weave left/right
- Boss patterns: Hardcoded attack sequences (`fireRadial`, `fireColumns`, `fireSpiralBurst`, etc.) with telegraph animations

**Scoring & Progression**
- Base score per kill (10–35 depending on type) × multiplier
- Chain system: consecutive kills (without being hit) increase multiplier; chain breaks on damage
- Stages: After KILLS_PER_STAGE kills, boss spawns; defeating boss advances stage and resets spawn rates

**Audio**
- Web Audio synth: `beep()` generates tone-based sounds (frequency, duration, waveform)
- `noiseBurst()` creates white-noise explosions (pitch-shifted random)
- All audio is procedurally generated; no external sound files

### Key Functions by Responsibility

**Game State Management**
- `resetGame()`: Initialize player, clear entities, reset score/lives
- `endGame()`: Show final score overlay, save best score

**Spawning & Entities**
- `spawnEnemy()`: Random enemy type (based on stage and difficulty), random X position
- `spawnBoss()`: Create boss object with current stage design and initial HP
- `maybeSpawnPowerUp(x, y)`: 25% chance to drop weapon/bomb upgrade at kill location

**Player Actions**
- `fireWeapon()`: Fire player bullets based on current weapon (vulcan spray vs. laser burst)
- `useBomb()`: Smart bomb that damages all enemies, shows flash/vignette effect
- Movement: Direct state update from input; clamped to canvas bounds

**Combat & Progression**
- `applyPowerUp(type)`: Upgrade weapon level or bomb count
- `addScore(base)`: Increment score by (base × multiplier); update chain timer
- `breakChain()`: Reset chain to 0, set multiplier to 1
- `loseLife()`: Decrement lives; if 0, trigger `endGame()`

**Visual Effects**
- `flashHit()`: Brief white flash on player damage
- `flashBomb()`: Longer white flash for bomb detonation
- `showComboPop(text)`: Animated text popup (e.g., "10x COMBO!")
- `showStageBanner(text)`: Large stage title slide-in
- `screenShake(mag, dur)`: Apply canvas transform shake (disabled if reduced motion)
- `spawnExplosion(x, y, color, count)`: Create particles at location

**Boss Attack Patterns** (called from `updateBoss()`)
- `fireRadial(b, count, speed, offset)`: Bullets in circle, all directions
- `fireAimedSpread(b, count, speed, spreadStep)`: Cone-shaped spread toward player
- `fireColumns(b, speed)`: Vertical columns from wing tips
- `fireSpiralBurst(b, speed)`: Rotating spiral
- `fireHomingSeeded(b, count, speed)`: Bullets that track player (with seed for predictability)
- `fireDoubleRing(b, speedA, speedB)`: Two concentric circles of bullets

**UI & Accessibility**
- `renderHearts(count, max)`: Draw heart/empty-heart indicators
- `renderBombs(count, max)`: Draw bomb indicators
- `announce(text)`: Post text to screen reader via `srLive` for accessibility
- `updateMotionBtn()`, `updateMuteBtn()`: Toggle UI button state and store preference

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. In `spawnEnemy()`, update the type-selection logic to include your new type at appropriate stages
3. (Optional) Add custom rendering in `update()` loop if different from the simple colored rectangle

### Adding a New Boss Attack Pattern
1. Define a new function (e.g., `firePatternName(b, speed)`) that spawns enemyBullets into the array
2. In `updateBoss()`, add a new condition in the pattern selector (the `switch`-like structure using telegraph/cooldown)
3. Tip: Boss patterns use `b.telegraph` to animate a warning pulse before firing

### Tweaking Difficulty
1. Edit the `DIFFICULTIES` object: adjust `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`
2. Test by selecting the difficulty and playing through 2–3 stages to verify pacing feels right

### Modifying Game Balance
- **Player speed**: `player.speed` in `resetGame()`
- **Enemy HP**: Edit `ENEMY_TYPES[type].hp`
- **Weapon damage**: Each bullet type has implicit damage; increase by reducing enemy HP or increasing fire rate
- **Scoring**: Adjust base scores in `ENEMY_TYPES` or the multiplier ramp in `addScore()`
- **Boss HP**: Scale via `BOSSES[idx].hpMul`

### Visual Tweaks
- **Canvas background gradient**: Edit the `<style>` section's `#gameCanvas` or dynamically in `drawClouds()`
- **Theme colors**: `THEMES` array holds sky gradients, cloud color, sun color per stage
- **Sprite colors**: `drawPlane()`, `drawBossShip()` use color variables; edit or parameterize them

## Performance Considerations

**Adaptive Quality System**
- The game tracks frame times over 45 frames
- If average frame time exceeds 1/40 (25ms), quality is degraded: fewer particles, simpler effects
- If frame time drops below 1/56 (~18ms), quality can be restored
- Quality levels: 0 (full), 1 (medium particle count), 2 (minimal effects)
- Check `quality` variable in `update()` and conditional logic for effect rendering

**Optimization Tips**
- Particle count scales with quality; don't spawn unlimited particles
- Collision checks are O(n²) but n is small (~50 entities typical), so acceptable
- Web Audio synthesis is cheap; no external audio loading bottleneck
- Canvas is rendered every frame; no dirty-rect optimization (full redraw)

## Accessibility & Compatibility

**Accessibility Features**
- Screen reader announcements via `#srLive` (aria-live polite)
- Reduced motion mode (respects OS preference, can be toggled) — disables animations/shakes/flashes
- Semantic HTML in overlay cards
- Keyboard controls fully functional (no mouse-only features)

**Browser Compatibility**
- Requires ES6 (arrow functions, const/let, template literals)
- Canvas 2D API (universal in modern browsers)
- Web Audio API (not available in very old browsers; gracefully degrades to silent mode)
- localStorage (standard; safe to assume available)
- No external CDN dependencies; all code inline

**Responsive Design**
- Uses `clamp()` for font sizes and padding
- Canvas uses `max-width: 100%; max-height: 100%;` and is centered
- HUD and overlay scale with viewport
- Touch controls hidden on desktop, shown on mobile

## Data Structures & Constants

**Difficulty presets** (`DIFFICULTIES` object):
- Define spawn rates, enemy speed multiplier, player lives, bomb count, boss HP scaling
- Key: 'easy', 'normal', 'hard'

**Theme presets** (`THEMES` array):
- Each theme: sky gradient colors, cloud color (RGB), sun color (RGB), theme name
- One theme per stage; cycles through 6 themes

**Enemy types** (`ENEMY_TYPES` object):
- Key examples: 'grunt' (weak, common), 'interceptor' (fast, weaves), 'tank' (slow, tanky)

**Boss designs** (`BOSSES` array):
- Each boss: name, hull colors (top/bottom), wing color, core color, HP multiplier
- Six unique bosses; stages cycle through them

**Local storage keys:**
- `skyfighter.bestScore`: Best score (number)
- `skyfighter.muted`: Audio mute state (0 or 1)
- `skyfighter.difficulty`: Last selected difficulty ('easy', 'normal', 'hard')
- `skyfighter.reducedMotion`: Reduced motion toggle (0 or 1)

## Testing Checklist for Changes

1. **Game loop**: Does the game run without errors? Check console.
2. **Input**: Test keyboard (arrows, space, X), mouse (move + click), touch (joystick + buttons).
3. **Spawning**: Verify enemies appear at correct rates; boss appears after stage kill count.
4. **Collisions**: Hit enemies, get hit by bullets, pick up power-ups — verify state changes.
5. **Scoring**: Kill enemies and confirm score updates correctly with multiplier; chain breaks on hit.
6. **Difficulty**: Test all three difficulties; verify spawn rate and enemy behavior differs.
7. **Audio**: Toggle mute, verify sound state persists in localStorage.
8. **Accessibility**: Enable reduced motion mode; verify animations are disabled.
9. **Responsive**: Test in portrait and landscape on mobile and desktop; verify HUD remains readable.
10. **Performance**: Monitor frame rate in DevTools; if dropping below 30 FPS, check quality scaling is active.

## Future Enhancement Ideas

- Particle pooling to reduce GC pressure (currently creates/destroys freely)
- More sophisticated boss AI with telegraph animations for projectile patterns
- Weapon variety beyond Vulcan/Laser (spread shot, mines, etc.)
- Leaderboard (would require backend; currently only localStorage per-device)
- Mobile app wrapper (Cordova/Capacitor) to distribute on app stores
- Sound toggle persistence (already saved to localStorage, just needs UI)
- Haptic feedback on mobile (vibration on hits/bomb)
