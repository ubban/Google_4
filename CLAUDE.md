# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 Canvas application. No build tools or external dependencies are required — the entire game is contained in `index.html` (~1,670 lines).

The game features:
- 6 difficulty-tuned stages with procedurally spawned enemies
- 6 unique boss encounters with distinct attack patterns
- Adaptive weapon system (Vulcan gun + Laser, 3 levels each)
- Smart bombs for screen clearing
- Procedural audio synthesis (no external audio assets)
- Touch and keyboard controls
- Difficulty settings (Easy/Normal/Hard)
- Local storage for best scores

## How to Run

Simply open `index.html` in a modern web browser (Chrome, Firefox, Safari, Edge). No server or build step required. The game will:
1. Load immediately
2. Display the start screen with difficulty selector
3. Initialize audio context on first user interaction (click or key press)

For development, edit `index.html` directly, save, and refresh the browser.

## Architecture Overview

The codebase is organized as a single IIFE (Immediately Invoked Function Expression) in strict mode, with distinct functional sections:

### Core Systems

**Game State & Loop** (~line 863)
- `running`, `paused`, `gameOver` flags
- Score, lives, multiplier, stage tracking
- Main update/draw loop via `requestAnimationFrame`
- Delta-time handling with frame capping (max 0.05s per frame)

**Canvas & Rendering** (~line 752)
- Fixed logical resolution: 480x720 canvas
- Responsive scaling to window (max 1.6x) while maintaining aspect ratio
- `draw()` function: renders sky theme, player, enemies, bullets, boss, particles
- Theme system: 6 sky themes that cycle per stage

**Input System** (~line 767)
- Keyboard: Arrow keys / WASD (movement), Space (fire), X/Shift (bomb), P/Esc (pause)
- Mouse: Hover to steer, click to fire
- Touch: Drag left zone to move, tap FIRE/BOMB buttons (mobile detected via `ontouchstart`)
- Joystick simulation for touch with visual base + stick feedback

**Audio (WebAudio Synthesizer)** (~line 542)
- No external audio files — all sound is procedurally generated
- Oscillators, filters, noise bursts, panning
- Audio context only initialized on first interaction (`ensureAudio()`)
- Mute toggle stored in localStorage

**Collision Detection** (~line 1069)
- Simple AABB: `rectsOverlap(a, b)` — compares object centers and half-widths
- Used for: bullets vs enemies, bullets vs boss, enemies vs player, enemy fire vs player, pickups vs player

### Game Logic

**Enemy System** (~line 1022)
- Three enemy types: grunt (weak, fast), interceptor (fast, weaving), tank (slow, durable)
- Difficulty-tuned spawn rate increases as stage progresses
- Enemies fire in two patterns: spread (fan) or aimed (at player)
- Each enemy stores baseX to enable weaving motion

**Boss System** (~line 1043)
- 6 unique boss designs (VANGUARD → TITAN) cycling per stage
- Multi-phase attack patterns per boss type (~line 1372)
- Telegraph system: 22-frame wind-up before each attack
- Boss spawned after 18 kills per stage (`KILLS_PER_STAGE`)

**Weapon System** (~line 1124)
- Vulcan: spread projectiles, fire-and-forget, lower cooldown
- Laser: piercing projectiles, higher fire rate, can pass through multiple enemies
- Each weapon has 3 levels; switching weapons resets to level 1
- Power-ups: life restore, bomb, weapon upgrade

**Scoring & Multiplier** (~line 1087)
- Chain counter increments on kills; resets if >2.2 seconds pass without kill
- Multiplier: 1 + floor(chain / 5), capping at 10x
- Score displayed live in HUD; best score stored in localStorage

**Performance Adaptation** (~line 473)
- Frames sampled into rolling 45-frame buffer
- Quality levels: 0 (full), 1 (medium), 2 (low)
- Triggers reduce particle counts, glow effects, animation smoothness
- Respects `prefers-reduced-motion` media query on load

### Rendering Systems

**Drawing Functions**
- `drawPlane(x, y, w, h, color, darkColor, flame)`: Draws player or enemy aircraft
- `drawBossShip(b)`: Draws boss with animated thrusters and core glow
- `drawClouds(speed, fillColor, rx, ry)`: Parallax scrolling cloud layers
- Background is multi-color gradient per theme; clouds and sun glow layer on top

**Particle System** (~line 1296)
- Particles spawn on enemy/boss death, explosions, bomb detonations
- Simple physics: position += velocity × dt
- Fade-out: life decreases each frame; particles removed at life ≤ 0

**Visual Effects**
- Screen shake: amplitude & duration set by impacts; applied via canvas translation
- Flash (hit/bomb): CSS animation classes on #flash element
- Vignette pulse: Red vignette when player at 1 HP
- Combo pop: "+X KILLS!" text at screen center with scale animation

## Key Variables & Systems

**Player Object**
```javascript
player = {
  x, y,              // Position (clamped to canvas)
  w, h,              // Width/height (40×40)
  speed: 260,        // Pixels/sec movement
  cooldown,          // Weapon fire cooldown
  weapon: 'vulcan'|'laser',
  level: 1|2|3,
  invuln             // Invulnerability timer (frames after hit)
}
```

**Difficulty Settings** (~line 485)
- `spawnBase`, `spawnMin`: Enemy spawn rate (frames)
- `enemySpeed`, `enemyShootChance`: Enemy behavior
- `lives`, `bombs`: Starting quantities
- `bossHpMul`: Boss health multiplier (0.8–1.3)

**Local Storage Keys**
- `skyfighter.bestScore`: Best score
- `skyfighter.muted`: Mute state (0|1)
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Reduced motion toggle

## Common Development Tasks

### Adjust Difficulty / Game Balance
1. Edit `DIFFICULTIES` object (line 485)
2. Tweak `ENEMY_TYPES` health/speed (line 502)
3. Modify `KILLS_PER_STAGE` (line 491) to change boss encounter timing
4. Adjust boss HP multiplier via boss `.hpMul` values (line 508)

### Add New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502)
2. Set `minStage` to limit spawn threshold
3. Update `spawnEnemy()` chance weights (line 1028)
4. Optional: Add explosion SFX to `EXPLODE_SFX` map (line 678)

### Add New Boss
1. Add design to `BOSSES` array (line 508)
   - Define color scheme (`hullTop`, `hullBottom`, `wing`, `core`)
   - Set `hpMul` for balance
2. Add attack pattern array to `BOSS_PATTERNS` (line 1372)
3. Patterns receive boss object; call helper functions:
   - `fireRadial(b, count, speed)` — omnidirectional burst
   - `fireAimedSpread(b, count, speed, spreadStep)` — aimed fan
   - `fireColumns(b, speed)` — vertical columns
   - `fireSpiralBurst(b, speed)` — rotating burst
   - `fireHomingSeeded(b, count, speed)` — homing projectiles

### Modify Weapon Behavior
- Weapon firing: `fireWeapon()` (line 1124)
- Fire rate: `fireDelay` constant (line 1177)
- Spread patterns: Modify arrays in weapon branches
- Piercing: Set `pierce` property on bullet

### Change Audio
- Sound effects defined in `sfx` object (line 624)
- Uses `beep(freq, dur, type, vol, opts)` for tones
- Uses `noiseBurst(dur, vol, opts)` for noise-based sounds
- No external audio assets; all procedurally generated

### Adjust Visual Theme or Appearance
- Sky themes: `THEMES` array (line 493)
- Player/enemy colors: Defined in type definitions and draw calls
- UI colors: Embedded in CSS (lines 10–358)
- Particle colors: Set in `spawnExplosion()` calls

## Performance Considerations

1. **Particle count** scales with quality level (line 1074)
2. **Shadow blur** disabled at low-quality (line 1002)
3. **Glow effects** on bullets/pickups only at full quality
4. **Animation smoothness** reduced when `prefers-reduced-motion` is active
5. **Canvas size** fixed (480×720); CSS scaling is hardware-accelerated
6. **Object pooling** not used; objects created/discarded each frame (acceptable for this scale)

## Testing Notes

- Test on desktop (keyboard + mouse) and mobile (touch)
- Verify boss patterns cycle correctly across multiple stages
- Check difficulty tuning: Easy should be forgiving, Hard should pressure the player
- Audio initialization must wait for user interaction (browser policy)
- Screen shake and animations respect reduced-motion preference
- Local storage persists across sessions

## File Structure

```
Google_4/
├── CLAUDE.md          (This file)
├── README.md          (Basic project description)
└── index.html         (Complete game, 1,670 lines)
    ├── <meta> & <link> tags
    ├── <style> (CSS: layout, animations, responsive)
    ├── <body> (HTML: canvas, HUD, overlay, touch controls)
    └── <script> (JavaScript: entire game logic)
```

## Browser Compatibility

- Requires: Canvas 2D Context, Web Audio API, localStorage
- Works: Chrome 60+, Firefox 55+, Safari 11+, Edge 79+
- Touch support detected via `ontouchstart` or `maxTouchPoints`
- Audio context uses both `AudioContext` and `webkitAudioContext` fallback
