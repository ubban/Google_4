# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game, implemented as a single self-contained HTML file (`index.html`). It features:
- 6 stages with unique visual themes (Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge)
- 6 boss encounters with distinct attack patterns
- 3 difficulty levels (Easy, Normal, Hard)
- 2 weapon types: Vulcan (rapid-fire, spreads at higher levels) and Laser (piercing shots)
- Smart bomb mechanic for clearing enemies and damaging bosses
- Combo multiplier system for consecutive kills
- Mobile-optimized touch controls with virtual joystick

## Running the Game

- **No build process required.** Simply open `index.html` in a modern web browser.
- The game uses local storage to persist high scores, difficulty preference, mute state, and motion preferences.
- Keyboard: Arrow keys/WASD to move, Space to fire, X/Shift to bomb, P/Esc to pause
- Mouse: Hover to aim, click to fire
- Mobile: Touch left side to move, FIRE/BOMB buttons on right

## Architecture

### Single-File Design
All code (HTML, CSS, JavaScript) is contained in `index.html`. This simplifies distribution and eliminates build/deployment complexity. The game is fully functional when opened locally or served from any static host.

### Game Loop
The core loop (lines 1640–1665) runs at 60 FPS using `requestAnimationFrame`:
1. **Update phase** (`update(dt)`): Process input, move entities, handle collisions, spawn enemies/bosses
2. **Draw phase** (`draw()`): Render background, entities, effects, HUD
3. **Performance tracking** (`trackPerf(dt)`): Monitor frame time and auto-reduce quality (particles, shadows) if average FPS drops

### Entity Systems

**Player** (line 877): Single player ship at the bottom. Tracks position, weapon type, level, cooldown, and invulnerability timer.

**Enemies**: Three types (lines 502–506):
- **Grunt**: Fast, weak (1 HP), worth 10 pts
- **Interceptor**: Faster still, weaves horizontally (line 1212)
- **Tank**: Slow, durable (4 HP), worth 35 pts, appears from stage 2

Enemies spawn based on difficulty settings (line 1023) with randomized patterns (spread fire or aimed at player).

**Boss** (line 1047): One active boss per stage. Each boss has:
- A design (color scheme, name) from `BOSSES` array (lines 508–515)
- Multiple attack patterns from `BOSS_PATTERNS` (lines 1372–1409)
- Telegraph phase (red core flashing, `b.telegraph`) before attacking
- Entering animation on spawn

**Bullets & Projectiles**:
- Player bullets: Track position, speed, and pierce count (for lasers)
- Enemy bullets: Radial, aimed, or spread patterns; cleared by bombs
- Particles: Explosion effects, fade over time

**Power-ups**: Life (6% drop), bomb (8% drop), weapon (20% drop). Spawn at defeated enemy positions.

### Collision Detection
`rectsOverlap(a, b)` (line 1069) uses axis-aligned bounding box collision for all entities. Checks occur in:
- Player bullets vs. enemies (line 1232)
- Player bullets vs. boss (line 1255)
- Enemies vs. player (line 1268)
- Enemy bullets vs. player (line 1276)
- Boss vs. player (line 1284)
- Power-ups vs. player (line 1289)

### Audio System (Lines 542–676)
WebAudio synth generates all sound effects procedurally (no external audio files):
- **Synthesis functions**:
  - `beep(freq, dur, type, vol, opts)`: Oscillator-based synth with optional sweep, filter, pan
  - `noiseBurst(dur, vol, opts)`: Noise-based percussion
- **Effects**: compression, dynamic filtering, stereo panning, amplitude envelopes
- **Sound library** (`sfx` object): shoot, laser, explode variants, bomb, powerup, bossHit, etc.
- Mute state persists via local storage

### Visual Effects & Rendering

**Screen Shake** (lines 726–730): Horizontal/vertical jitter applied to canvas context; magnitude and duration reduce with quality setting and reduce-motion preference.

**Flash Effects** (lines 698–709): White screen overlay triggered on player hit or bomb detonation; animations respect reduce-motion.

**Combo Pop** (line 712–717): Large floating text ("X KILLS!") appears at 1/10th of kills; uses CSS animation.

**Stage Banner** (line 719–724): "STAGE N" and "WARNING — BOSS NAME" messages fade in/out.

**Boss Health Bar** (lines 149–180): Positioned at top, shows real-time HP drain.

**Canvas Scaling** (lines 753–765): Base resolution 480×720; scales to fit viewport with max 1.6× magnification to maintain crisp rendering.

**Background Gradient**: Per-stage theme (lines 1529–1545) with sun halo and animated cloud layers at two depths.

### Game State Management

**Difficulty** (lines 485–489): Three configurations scale enemy spawn rates, HP, shoot chance, and boss multiplier.

**Stage Progression**: 18 kills per stage (line 491) triggers boss spawn. Boss defeat advances stage, resets kill counter, changes theme.

**Chain & Multiplier** (lines 1087–1101): Each kill within 2.2 seconds extends the chain; multiplier caps at 10× (chain ≥ 50).

**Invulnerability** (line 1174): Player gets 1.5 seconds after hit; blinks but unharmed.

**Lives & Game Over** (lines 1456–1489): Running out of lives triggers end-game screen with stats and new-best detection.

### Input Handling

**Keyboard** (lines 768–776): `keys` object tracks state. P/Esc toggle pause; X/Shift use bomb.

**Mouse** (lines 837–861): When mouse is over canvas, player glides toward cursor (priority lower than keyboard/joystick to avoid fighting inputs).

**Touch** (lines 778–834): 
- Left 55% of screen: virtual joystick with draggable stick
- Right 45% of screen: FIRE and BOMB buttons
- Joystick visualized with base circle and stick position

Joystick clamped to 50px radius; movement converted to normalized vector.

### Configuration & Persistence

**Local Storage Keys** (lines 463–466):
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Mute toggle
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: User or system reduced-motion preference

## Key Constants & Tuning

| Constant | Purpose | Where |
|----------|---------|-------|
| `BASE_W`, `BASE_H` | Canvas resolution | 753 |
| `KILLS_PER_STAGE` | Enemy kills required to trigger boss | 491 |
| `DIFFICULTIES` | Difficulty configs (spawn, enemy speed, lives, bombs) | 485–489 |
| `THEMES` | 6 stage themes with sky gradients, cloud colors | 493–500 |
| `ENEMY_TYPES` | 3 enemy archetypes (grunt, interceptor, tank) | 502–506 |
| `BOSSES` | 6 boss designs with HP multipliers | 508–515 |
| `BOSS_PATTERNS` | Attack patterns per boss (6 patterns × 3 attacks each) | 1372–1409 |

## Common Modifications

**Add an enemy type**: Add to `ENEMY_TYPES` (line 502), update `minStage`, update spawn logic (line 1028).

**Add a boss attack pattern**: Append function to appropriate `BOSS_PATTERNS` entry (line 1372). Pattern functions take boss `b` and call firing helpers (`fireRadial`, `fireColumns`, etc., lines 1324–1370).

**Adjust difficulty scaling**: Edit `DIFFICULTIES` (line 485).

**Change stage visuals**: Edit `THEMES` (line 493) with new sky/cloud/sun colors and name.

**Tune weapon balance**: Adjust fireWeapon spread/count (line 1124), laser pierce value (line 1141), or cooldown (line 1177).

**Adjust particle/quality effects**: Search "quality" (lines 474, 1074, 1579, 2000) to see where quality level affects rendering.

## Accessibility Features

- **Reduced Motion**: Toggle via UI button; disables screen shake, flashes, animations (applied with `.reduced-motion` body class)
- **Screen Reader Support**: `#srLive` element announces game state (lives, boss defeated, stage cleared)
- **Semantic HTML**: Form controls use `<button>`, live region uses `aria-live="polite"`
- **Keyboard Navigation**: All controls accessible via keyboard

## Testing Tips

- Open browser DevTools (F12) and throttle network/CPU to see quality downgrade in action
- Use `localStorage.clear()` in console to reset high score and preferences
- Modify `DIFFICULTIES` to quickly test balance changes
- Disable audio: set `muted = true` in console to silence synth for testing
