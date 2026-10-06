# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 Canvas application (~1,670 lines). The entire game—rendering, physics, gameplay logic, audio synthesis, and UI—is implemented in vanilla JavaScript with no external dependencies.

## Running and Testing

**To run the game:**
- Open `index.html` in any modern web browser (Chrome, Firefox, Safari, Edge)
- No build process, no server required—the game runs client-side
- Use `/run` skill or open locally: `file:///home/user/Google_4/index.html`

**To test in a browser:**
1. Launch the game via `/run` or open the HTML file
2. Click "Start Game" to play
3. Test difficulty modes by clicking Easy/Normal/Hard before starting
4. Test controls:
   - **Mouse:** Move to steer, click to fire
   - **Keyboard:** Arrow keys/WASD to move, Space to fire, X/Shift to use bomb, P/Esc to pause
   - **Mobile:** Drag left zone to fly, tap FIRE/BOMB buttons
5. Test features: mute button (🔊), reduced motion toggle (🎆), pause button (⏸)

## Architecture

The codebase is organized as one large IIFE (Immediately Invoked Function Expression) that encapsulates all game state. Key sections:

### **Core Game Loop**
- `loop(now)` — requestAnimationFrame loop; calls `update(dt)` then `draw()`
- Frame tracking via `frame` counter used for animation timing
- Delta-time capped at 50ms to prevent large jumps

### **State Management**
- **Game state:** `running`, `paused`, `gameOver`
- **Game objects:** `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- **Scoring:** `score`, `chain`, `chainTimer`, `multiplier` (combo system)
- **Player:** position, weapon type/level, cooldown, invulnerability frames
- **Stage:** `stage`, `stageKills`, `kills`; boss spawns after `KILLS_PER_STAGE` (18) kills

### **Gameplay Systems**

**Enemy System**
- Three enemy types: `grunt` (basic), `interceptor` (fast, weaving), `tank` (slow, tanky)
- Each stage unlocks harder enemies; spawning rate increases via `spawnTimer` (difficulty-scaled)
- AI: shoot patterns (spread vs. aimed), weaving movement for interceptors
- Collision: `rectsOverlap()` for all AABB checks (bullets, enemies, powerups, player)

**Weapon System**
- **Vulcan** (default): straight shots, spread increases with level (1=single, 2=two-lane, 3=three-lane)
- **Laser**: straight shots with pierce mechanic (can hit multiple times), faster fire rate
- Level 3 → weapon swap: Vulcan→Laser→Vulcan (resets to level 1 each swap)
- Toggled via power-ups; displayed in HUD as name + level

**Boss System**
- Six boss designs, each with unique visual and attack patterns
- Each boss has 6 attack patterns defined in `BOSS_PATTERNS` array
- Boss spawns after defeating 18 stage enemies (with warning message)
- Boss HP scales by difficulty and by a per-boss multiplier (`hpMul`)
- Pattern types: radial bursts, aimed spreads, homing, columns, teleport dashes, spirals, crosses
- Boss enters from top with `entering: true` state, then begins attacks

**Scoring/Combo**
- Kill chain: each kill increments `chain`, resets `chainTimer` to 2.2s
- Multiplier: `1 + floor(chain / 5)`, caps at 10x
- Chain breaks if 2.2s passes without a kill
- Milestone messages at 10, 20, 30… kills

**Difficulty System** (localStorage-persisted)
- **Easy:** slower enemies, longer spawn intervals, 4 lives, 3 bombs, 0.8x boss HP
- **Normal:** moderate challenge, 3 lives, 2 bombs, 1.0x boss HP
- **Hard:** fast enemies, tight intervals, 3 lives, 2 bombs, 1.3x boss HP
- Power-ups (life/bomb/weapon) spawn at low rates during play

**Adaptive Performance**
- `quality` variable: 0 (full glow/shadows), 1 (medium), 2 (low)
- Tracks average frame time; downgrades quality if avg > 25ms, upgrades if < 17.8ms
- Affects particle count, shadow blur, glow effects

### **Audio System**
- **WebAudio Synth:** no external files; sound effects generated via oscillators + noise bursts + filters
- `ensureAudio()` lazily initializes `audioCtx` on first interaction (user gesture required)
- `beep(freq, dur, type, vol, opts)` — oscillator with optional frequency sweep, filter, stereo pan
- `noiseBurst(dur, vol, opts)` — filtered noise for impact/explosion sounds
- Mute state and reduced-motion state stored in localStorage
- Sound effects: shoot, laser, explosion (enemy/tank-specific), hit, bomb, power-up, boss telegraph, stage clear, UI click, game over

### **Rendering**
- Canvas: 480×720 base resolution; scaled to fit viewport (max 1.6x)
- **Layering (z-order):**
  1. Sky gradient + clouds (parallax at two speeds)
  2. Player ship (with laser aura if weapon-switched)
  3. Enemies with optional HP bar (tanks)
  4. Boss ship (with glowing core telegraph animation)
  5. Bullets (player: yellow/cyan; enemy: red circles)
  6. Power-ups (rotating, color-coded: red/yellow/green)
  7. Particles (expanding, fading circles)
  8. UI HUD overlay (top center: score, best, multiplier, weapon, hearts, bombs)
  9. Boss bar (top center when boss active)
  10. Flash/vignette overlays (damage/bomb feedback)

**Shaders/Effects:**
- Screen shake via `screenShake(mag, dur)` applied in draw via translate
- Flash animation on hit/bomb via CSS animation (white full-screen fade)
- Vignette red pulse on low health
- Cloud parallax: two speed layers
- Theme system: 6 color themes for sky/sun/clouds, one per stage (cycles)

**Canvas Resize:**
- `resize()` called on window resize/orientation change
- Maintains aspect ratio; scales to fit with max 1.6x zoom

### **Input Handling**
- **Keyboard:** key press stored in `keys` object; movement prioritizes WASD/arrows, then mouse
- **Mouse:** `mouseTarget` tracks pointer; player glides toward it when keyboard/joystick not active
- **Touch:** virtual joystick (left zone) + fire/bomb buttons (right zone)
  - Joystick drawn when active; constrains to 50px radius
  - Buttons use `touch-action: none` and prevent defaults
- **Firing:** `Space`, mouse click, or fire button; controlled by `firing` boolean
- **Bombs:** X, Shift, or bomb button; calls `useBomb()` if available

### **UI and Overlays**
- **HUD:** Always-visible stats at top (score, best, multiplier, weapon, hearts, bombs)
- **Overlay:** Modal at game start, pause, and game-over; shows title card, difficulty selector, start button
- **Boss bar:** Health bar shown when boss present
- **Screen feedback:** Combo pop-ups (kills), stage banners, flash/vignette for damage
- **Accessibility:** ARIA live region for screen reader announcements
- **Reduced motion:** Respects `prefers-reduced-motion` media query; can be toggled; disables animations if enabled

## Key Constants and Tuning

- `BASE_W` / `BASE_H`: 480×720 (game resolution)
- `KILLS_PER_STAGE`: 18 (enemies to defeat before boss)
- `DIFFICULTIES`: spawn rates, enemy speeds, shoot chances, lives/bombs per mode
- `THEMES`: 6 sky/cloud color palettes, one per stage
- `ENEMY_TYPES`: grunt, interceptor, tank (stats: size, HP, speed, score, visual colors)
- `BOSSES`: 6 boss designs with unique colors, patterns, and HP multipliers
- `BOSS_PATTERNS`: 6 arrays of attack pattern functions; indexed by boss type

## Common Editing Tasks

**Add a new enemy type:**
1. Add entry to `ENEMY_TYPES` with w/h/hp/speedMul/color/dark/score/minStage
2. Update spawn logic in `spawnEnemy()` if new conditions needed
3. Add explosion SFX mapping in `EXPLODE_SFX` if custom sound desired
4. Optionally add weave behavior via `weave: true` flag

**Add a new boss attack pattern:**
1. Create a function `firePattern(b)` that pushes enemies to `enemyBullets` array
2. Add to `BOSS_PATTERNS` sub-array for that boss (0–5 index)
3. Patterns can use helpers like `fireRadial()`, `fireAimedSpread()`, `fireCross()`, etc.

**Add a new power-up type:**
1. Add entry to `maybeSpawnPowerUp()` spawn rates
2. Handle in `applyPowerUp(type)` with desired effect
3. Add rendering code in `draw()` with color and logic

**Tweak difficulty:**
- Modify `DIFFICULTIES` object spawn rates, enemy stats, lives, or boss HP
- Test all three modes to verify progression feel

**Add a new theme (sky palette):**
- Add object to `THEMES` array with `sky`, `cloud`, `sun` RGB color strings and `name`
- Stages cycle through themes automatically; no other code changes needed

## Common Bugs and Patterns

**Collision detection:** All object collision uses `rectsOverlap()`, which checks AABB overlap with half-widths. Ensure objects have `x`, `y`, `w`, `h` properties.

**Chain/multiplier reset:** Breaking chain (e.g., player takes damage) calls `breakChain()`. Some systems (like bomb) intentionally don't break chain.

**Boss telegraph:** Visual telegraph (white core flash) happens before pattern fires. The `telegraph` counter counts down; when it reaches 0, pattern fires and attack timer resets.

**Particle performance:** Adaptive quality downgrades particle counts on slower devices. Explosion function `spawnExplosion()` respects quality setting.

**Storage keys:** Game state persists (best score, mute, difficulty, reduced motion) via localStorage with keys starting with `skyfighter.*`. Clear if testing reset scenarios.

**Canvas coordinate system:** Origin (0,0) is top-left. Y increases downward. Player starts at bottom-center; enemies/boss enter from top.

## Development Tips

- Use `/run` to launch and test live
- Press F12 to open DevTools; check console for any errors
- Modify constants at the top of the IIFE for quick tweaks (KILLS_PER_STAGE, DIFFICULTIES, etc.)
- The `frame` counter increments every loop; use it for time-based logic and animations
- Audio context requires a user gesture; `ensureAudio()` handles lazy init on first click/key
- LocalStorage can be cleared in DevTools to reset saved state (best score, settings)
- Reduce motion toggle disables all CSS/JS animations for accessibility testing
- Test all three difficulty modes and all boss patterns to catch tuning issues

