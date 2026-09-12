# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single-file HTML5 application. It features:
- Canvas-based graphics rendering
- WebAudio API for procedurally-generated sound effects (no external audio files)
- Touch, keyboard, and mouse input support
- Three difficulty levels with tuned enemy and boss behavior
- Boss fights with distinct attack patterns per boss type
- Power-ups (weapon upgrades, bombs, health)
- Score tracking and high-score persistence via localStorage
- Accessibility features (reduced motion mode, screen reader announcements)
- Adaptive performance optimization (quality levels based on frame time)

## Running the Game

The game requires **no build step**. Simply open `index.html` in a modern browser:

```bash
# Using a local web server (recommended for full functionality)
python3 -m http.server 8000
# Then navigate to http://localhost:8000

# Or directly open the file (some features may be restricted by CORS/security)
open index.html
```

## Development

### Understanding the Architecture

The entire game logic lives in a single `<script>` tag (lines 422-1666). The architecture follows a classic game loop pattern:

**Game State** (lines 864-869):
- Player, bullets, enemies, enemy bullets, particles, power-ups, boss, score, lives, stage
- Stored in module scope; no external state management

**Main Loop** (lines 1640-1653):
- `requestAnimationFrame(loop)` calls `update(dt)` then `draw()`
- Delta time capped at 50ms to prevent large jumps
- Adaptive performance tracking adjusts graphics quality based on frame time

**Input Handling** (lines 768-861):
- Keyboard: arrow keys, WASD for movement; Space/X/Shift for fire/bomb; P/Esc for pause
- Mouse: hover to steer, click to fire (mouse input takes priority only when no keyboard/joystick input)
- Touch: left zone for movement joystick, right zone for fire and bomb buttons
- All input events trigger `ensureAudio()` to handle browser audio context suspension

**Update Phase** (lines 1148-1302):
- Player movement (keyboard/joystick/mouse priority)
- Weapon firing and cooldown
- Enemy spawning and behavior (weaving, shooting)
- Boss behavior updates (telegraph, attack patterns)
- Collision detection (bullet-to-enemy, bullet-to-boss, enemy-to-player, powerup-to-player)
- Particle lifecycle
- Chain/combo tracking for score multipliers

**Draw Phase** (lines 1520-1638):
- Sky gradient and cloud parallax based on current stage theme
- All game objects (player, enemies, boss, bullets, particles, powerups)
- Boss health bar and HUD display via DOM (not canvas)
- Screen shake offset applied via canvas transform

### Key Systems

**Difficulty System** (lines 485-489):
- Easy/Normal/Hard preset tables adjust enemy spawn rate, speed, shooting chance, lives, bombs, boss HP
- Dynamically ramped difficulty increases enemy spawn frequency over time
- Affects boss health multiplier per boss design

**Enemy Spawning and Types** (lines 502-506, 1022-1041):
- Three enemy types: Grunt (weak), Interceptor (fast), Tank (slow, high HP)
- Tank appears only from Stage 2+
- Weaving interceptors follow sine wave patterns
- Two shooting patterns: "spread" (radial spray) and "aimed" (tracks player)

**Boss System** (lines 508-515, 1043-1056, 1411-1454):
- Six unique boss designs cycle through stages (Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan)
- Each boss has 3 unique attack patterns (lines 1372-1409)
- Patterns include: radial bursts, aimed spreads, teleport dashes, homing missiles, rotating rings, columns
- Telegraph phase before each attack (visual core flash + audio cue)
- Boss defeated spawns large particles and transitions to next stage

**Weapon System** (lines 871-873, 1124-1146, 1304-1322):
- Two weapons: Vulcan (spread bullets, faster fire) and Laser (piercing, slower fire, glow aura)
- Three upgrade levels per weapon
- Level 3 unlock alternate weapon; upgrading level 3 switches weapon to Level 1
- Spread and pierce upgrade with levels

**Audio System** (lines 542-676):
- WebAudio synthesizer creates all sound effects procedurally (no asset files)
- `beep()` generates tones with optional frequency sweep, filter, panning, and ADSR envelope
- `noiseBurst()` generates filtered white noise for explosion and impact sounds
- Master compressor and gain node for consistent volume
- All sounds respect mute toggle and check audio context state

**Visual Effects**:
- Screen shake (magnitude and duration): enemy hits, player hit, bomb, boss defeated
- Flash effects: hit flashes (white 0.25s) vs bomb flashes (white 0.5s)
- Combo pop animation: shows kill counts at 10/20/30/etc
- Stage banner animation: fade in/out with scale pulse
- Vignette pulse: red screen when critical health (1 life remaining)
- Reduced motion mode: disables animations, reduces shake, simplifies flash

**Performance Adaptation** (lines 473-483):
- Tracks last 45 frame deltas
- If average > 1/40s → quality level 1 (medium)
- If average > 1/40s while quality 1 → quality level 2 (low)
- If average < 1/56s → decrease quality level
- Quality affects shadow rendering (glow effects) and particle count

**Persistence** (lines 463-467):
- Best score, mute state, difficulty, reduced motion preference stored in localStorage
- Keys prefixed with `skyfighter.` to avoid collisions
- Reduced motion uses system preference as initial value

### Common Modifications

**Adjusting Enemy Difficulty**:
- Edit `DIFFICULTIES` object (lines 485-489): `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`
- Edit `ENEMY_TYPES` (lines 502-506): `hp`, `speedMul`, `score`
- Enemy spawn rate ramps every 300 frames (line 1201): increase divisor for slower ramp

**Adding Boss Attack Patterns**:
- Add new pattern function to match signature of `fireRadial`, `fireAimedSpread`, etc (lines 1324-1370)
- Add 3-pattern array to `BOSS_PATTERNS` (lines 1372-1409) matching boss index
- Patterns execute in order and loop

**Tweaking Weapon Balance**:
- Vulcan spread array (line 1127): more/fewer angles for wider/narrower fire
- Laser pierce value (line 1141): higher = more enemies per shot
- Fire cooldown (line 1177): lower = faster fire rate

**Adding Stage Themes**:
- Add object to `THEMES` array (lines 493-500): `sky` (gradient stops), `cloud`, `sun` colors, `name`
- Themes cycle by stage modulo

**Screen Shake and Visual Feedback**:
- `screenShake(magnitude, duration)` (line 727): called by damage events
- `flashHit()` / `flashBomb()` (lines 698-709): trigger white flash animation
- Vignette pulse tied to `lives === 1` (lines 687, 530)

### Testing the Game

Since there's no test framework:
- **Play test manually**: Open in browser, try all input methods, verify collisions
- **Edge cases to check**:
  - Multi-stage progression (boss patterns, stage themes)
  - Difficulty setting persistence across page reload
  - Touch input on mobile devices
  - Reduced motion accessibility (all animations disabled)
  - High score beating previous best
  - Bomb usage and refill from powerups
  - Weapon switching at level 3
  - Screen shake and flash effects during gameplay

### Code Style Notes

- No external libraries; vanilla DOM, Canvas 2D, WebAudio
- Procedural drawing (no sprites): all graphics generated via canvas paths and gradients
- State mutations are direct (not immutable); collision detection mutates arrays in-place
- Collision checks use AABB (axis-aligned bounding box) overlap
- Radial enemy fire uses trigonometry for pattern generation
- Animation timing uses frame counter or elapsed time depending on context
