# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The game features multiple difficulty levels, 6 unique boss designs with distinct attack patterns, procedurally varied enemy types, power-up systems, and WebAudio-based sound effects. It runs natively in all modern browsers with responsive touch controls for mobile devices.

## Codebase Architecture

All code lives in a single `index.html` file (~1670 lines) organized as a single large IIFE (Immediately Invoked Function Expression) to avoid global namespace pollution. The file contains three sections:

### 1. **Styling & DOM** (Lines 10-420)
- Comprehensive CSS with responsive design using `clamp()` for fluid scaling
- CSS animations for UI feedback (heartBeat, comboPulse, vignettePulse, etc.)
- Accessibility features: reduced motion preference support, screen reader announcements, touch controls
- Layout: fullscreen canvas with overlaid HUD, boss health bar, combo popups, stage banners
- Touch controls: virtual joystick in left zone, fire/bomb buttons in right zone

### 2. **Game Engine (Lines 422-1667)**
The game loop uses `requestAnimationFrame()` for 60 FPS rendering. Core subsystems:

#### **Input System** (Lines 768-861)
- Keyboard: Arrow keys/WASD for movement, Space for fire, X/Shift for bomb, P/Esc for pause
- Mouse: Move to steer, click to fire (takes priority over keyboard only when mouse is active)
- Touch: Joystick on left 55% of screen, Fire/Bomb buttons on right 45%
- Dual input model: keyboard/joystick take priority when active; mouse is secondary steering

#### **Audio Engine** (Lines 542-676)
- WebAudio API-based procedural sound synthesis (no external assets)
- Master gain node with dynamics compressor for polishing
- Core functions:
  - `beep(freq, dur, type, vol, opts)`: Oscillator-based tone with optional frequency sweep, lowpass filter, stereo panning
  - `noiseBurst(dur, vol, opts)`: White noise with envelope, used for impact sounds
- Sound design captures arcade aesthetic (retro square/sawtooth waves, noise bursts)
- All SFX defined in `sfx` object (shoot, laser, explode variants, hit, bomb, powerup, boss effects, gameover)

#### **Game State** (Lines 863-911)
- **Player**: Position, velocity, weapon type (vulcan/laser) and level, cooldown, invulnerability timer
- **Entities**: Arrays for bullets, enemies, enemy bullets, particles, power-ups, boss
- **Game variables**: Score, lives, stage, difficulty, paused/running/gameOver flags
- **Difficulty system**: Three presets (easy/normal/hard) modify spawn rate, enemy stats, boss HP, starting lives/bombs
- **Combo chain system**: Builds multiplier (1-10x) when enemies killed within 2.2s window; breaks on taking damage

#### **Rendering** (Lines 1507-1638)
- **Drawing functions**: `drawPlane()` for allied/enemy aircraft, `drawBossShip()` for bosses with animated core glow
- **Theme system**: 6 different sky gradients (Daylight, Sunset, Night, Storm, Arctic, Volcanic) with parallax cloud layers
- **Visual effects**: Screen shake (stored as `shakeMag`/`shakeTime`), vignette pulse on low HP, flash on hit/bomb, combo popups
- **Performance-aware rendering**: `quality` variable (0=full, 1=medium, 2=low) disables glow effects and reduces particles based on frame rate

#### **Physics & Collision** (Lines 1148-1302)
- `update(dt)` handles all per-frame state changes with delta-time for smooth 60fps gameplay
- `rectsOverlap()`: AABB collision detection used for all projectile/entity interactions
- Movement: Constrained to canvas bounds, joystick input normalized
- Particle system: Spawned on hits/explosions, momentum-based decay
- Bullet pierce system: Laser weapons can penetrate multiple enemies based on weapon level

#### **Enemy System** (Lines 1022-1040, 1210-1227)
Three enemy types with stage-gating:
- **Grunt** (minStage 1): Fast, weak, easy kill - red colored
- **Interceptor** (minStage 1): Faster, weaving pattern, single-hit - orange colored
- **Tank** (minStage 2): Slow, 4 HP, shows health bar, high score - purple colored

Enemy behavior:
- Spawn from top with random X in 30-canvas.width-30 range
- Weaving interceptors follow sine wave around spawn position
- Shoot pattern: either "spread" (3-way fan) or "aimed" (directly at player)
- Shoot chance and frequency scale by difficulty
- Removed when off-screen (y > canvas.height + 30)

#### **Weapon System** (Lines 1124-1146)
Two weapon types that upgrade separately:
- **Vulcan** (default): Rapid fire square bullets, spread increases at level 2-3
  - L1: Single center stream
  - L2: Dual-lane spread
  - L3: Triple-lane spread, can be toggled to Laser
- **Laser**: Slower sustained fire with pierce ability
  - L1: Single center beam
  - L2: Dual beams at -6/+6 offset
  - L3: Triple beams (dual + center), triple pierce
  - Can toggle back to Vulcan at L3

#### **Boss System** (Lines 508-515, 1043-1056, 1372-1439, 1441-1454)
Six unique bosses cycle by stage number (stage % 6):
- **VANGUARD**: Radial burst, aimed fan, wide V-fan
- **BEHEMOTH**: Heavy columns, homing spread, dense radial
- **SPECTER**: Teleport dash, spiral burst, double aimed
- **DREADNOUGHT**: 8-way cross, homing missiles, dense radial
- **WRAITH**: Teleport barrage, expanding double rings, cross-spread
- **TITAN**: Heavy columns, full ring, relentless homing

Each boss has unique color scheme and HP multiplier (0.8x-1.7x). Attack patterns use helper functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireHomingSeeded`, etc.). Boss enters from top, bounces left-right, telegraphs attacks (22 frames warning), executes pattern, repeat. Defeated at 0 HP after taking 3 damage per player bullet.

#### **Scoring & Multiplier** (Lines 1087-1101)
- Base scores per enemy type (10-35 points)
- Kill chain builds multiplier: +1x per 5 consecutive kills
- Max 10x multiplier, resets on taking damage or 2.2s timeout
- Boss kills award flat 500 points

#### **Power-up System** (Lines 1058-1067, 1289-1322)
Three power-up types spawn randomly on enemy death:
- **Life** (6% chance): Restore 1 heart to max
- **Bomb** (8% chance): Restore 1 bomb to max
- **Weapon** (20% chance): Upgrade weapon level or swap weapon type
- Collected when player overlaps; play powerup SFX

#### **Bomb Mechanic** (Lines 1103-1122)
- Limited resource (2-3 per run based on difficulty)
- On use: Clear all enemy bullets, damage all enemies (+5 points each), damage boss (-24 HP), strong screen shake, explosion particles
- Audio feedback: Heavy bass/noise burst SFX

#### **Difficulty Balancing** (Lines 485-489)
Difficulty affects:
- Enemy spawn rate: Easy 75/40, Normal 58/26, Hard 40/18 (base/min per frame)
- Enemy speed multiplier: Easy 1.0x, Normal 1.4x, Hard 1.9x
- Enemy shoot chance: Easy 45%, Normal 70%, Hard 95%
- Starting lives: Easy 4, Normal/Hard 3
- Starting bombs: Easy 3, Normal/Hard 2
- Boss HP multiplier: Easy 0.8x, Normal 1.0x, Hard 1.3x
- Spawn rate accelerates as game progresses (difficultyRamp based on frame count)

#### **Performance Adaptation** (Lines 473-483)
- Tracks average frame time over 45 frames
- Adjusts quality level (0=full glow, 1=reduced glow, 2=no glow/fewer particles)
- Quality improves if avg frame time < 1/56 (~17.9ms), degrades if > 1/40 (25ms)

#### **UI & UX** (Lines 468-751)
- **Screen reader support**: `announce()` function pushes messages to aria-live region for accessibility
- **Mute button**: Toggles audio on/off, stored in localStorage
- **Motion preferences**: Respects OS reduced-motion setting, disables animations when enabled, has toggle button
- **Pause system**: P/Esc hotkey, pause button toggles overlay; game state frozen when paused
- **Difficulty selection**: Three radio-style buttons on start screen, active difficulty stored in localStorage
- **HUD updates**: Score, best score, multiplier, weapon name/level, hearts, bombs, boss health bar all update in real-time

## Development Workflow

### Running the Game
Simply open `index.html` in a modern browser. No build step, no server needed.

### Testing Specific Scenarios
- **Test difficulty balance**: Use difficulty buttons on start screen, observe enemy spawn/shoot rates, test bomb damage
- **Test responsive design**: Open DevTools, toggle device emulation (mobile sizes)
- **Test accessibility**: Enable reduced motion OS setting, use keyboard navigation (Tab/Enter), verify audio on/off
- **Test mobile controls**: Use DevTools device emulation with touch events, or test on actual device
- **Test performance**: Check browser DevTools Performance tab, look for frame drops, verify quality adaptation
- **Test boss patterns**: Reach stage where specific boss appears (stage N selects BOSSES[N % 6]), verify attack patterns match design

### Making Changes
**Key invariants to preserve:**
- **Canvas resolution**: BASE_W=480, BASE_H=720 (aspect ratio 2:3 portrait)
- **Physics scale**: Player speed 260px/s, enemy speeds 1.0-1.9x multiplier, bullet speeds 200-640px/s
- **Game balance**: Enemy spawn rates, shoot chances, and multiplier math affect difficulty curve; test across all three difficulties
- **Accessibility**: Keep screen reader announcements, reduced motion support, keyboard controls always functional
- **Audio**: All sounds procedural; no external assets means sounds must be generated at runtime
- **Single-file constraint**: Keep all code in index.html (no external scripts/stylesheets) to maintain portability

**Common edits:**
- **Adjust enemy spawning**: Change `spawnBase`/`spawnMin` in DIFFICULTIES object or modify `spawnEnemy()` function
- **Modify weapon balance**: Edit VULCAN/LASER spread patterns or fire rates in `fireWeapon()` and `fireDelay` constant
- **Add/modify boss pattern**: Create new pattern function (e.g., `fireWave()`) and add to BOSS_PATTERNS array
- **Tune sound effects**: Modify frequency, duration, envelope parameters in `sfx` object (beep/noiseBurst calls)
- **Add visual theme**: Insert new entry to THEMES array with sky gradient colors, cloud color, sun color
- **Change difficulty constants**: Modify DIFFICULTIES object, KILLS_PER_STAGE, or HP/score multipliers

### Debugging Tips
- **Monitor performance**: Open DevTools → Performance tab, record a play session, look for frame drops
- **Check console for errors**: All audio errors are silently caught; if sound breaks, check audioCtx !== null
- **Test collision**: Verify `rectsOverlap()` logic; bullet hitboxes are w×h rectangles centered at (x,y)
- **Trace game state**: Add `console.log(player, enemies.length, score)` in update loop
- **Pause and inspect**: Use pause button to freeze game, inspect canvas state in DevTools

## Storage & Persistence

Three localStorage keys (prefixed `skyfighter.`):
- `bestScore`: Highest score achieved
- `muted`: Audio on/off state (stored as '1' or '0')
- `difficulty`: Last selected difficulty (easy/normal/hard)
- `reducedMotion`: Accessibility preference (stored as '1' or '0')

## Browser Compatibility

- **Requires**: ES6 (arrow functions, `const`/`let`, template literals, destructuring)
- **Audio**: WebAudio API (all browsers except IE11)
- **Input**: Touch events API for mobile joystick
- **CSS**: Viewport units (`vw`/`vh`), `clamp()` for responsive scaling, CSS animations

## Performance Targets

- **Target FPS**: 60 (enforced via requestAnimationFrame)
- **Quality adaptation**: Disables glow/glowBlur on mobile/low-end devices automatically
- **Particle count**: Capped and scaled by quality level (fewer on low-end)
- **No asset loading**: All graphics procedural (canvas drawing), no images/audio files

## Accessibility

- **Screen reader support**: Announcements for game state changes (level start, boss defeated, game over, lives remaining)
- **Reduced motion**: Checkbox in UI; respects OS preference; disables all animations when enabled
- **Keyboard controls**: Full game playable with keyboard only (arrows/WASD, Space, X/Shift, P/Esc)
- **Color contrast**: Dark theme with high-contrast UI text
- **Touch support**: Virtual joystick + button controls for mobile
