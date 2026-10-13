# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The player controls a fighter jet, destroys waves of enemies, collects weapon power-ups, and defeats stage bosses. The game features:

- **Single Canvas-based renderer** using 2D context for all graphics
- **Web Audio API synth** for all sound effects (no external audio files)
- **Multiple difficulty modes** (Easy, Normal, Hard) with different enemy spawn rates and player lives
- **6 themed stages** with unique sky/cloud color schemes
- **6 boss types** with distinct visual designs and HP multipliers
- **Touch and keyboard controls** with full mobile support
- **LocalStorage persistence** for best scores and user settings
- **Accessibility support** including ARIA labels, screen reader announcements, and reduced-motion preference handling

## Architecture

### Single-File Structure

The entire game (HTML, CSS, JavaScript) is in `index.html`. This monolithic approach is intentional:
- **Self-contained**: no external dependencies, assets, or build process required
- **Portable**: single file that works offline and can be distributed easily
- **Performant**: no network requests, instant load time

### Main Game Systems

The JavaScript game code is organized into these conceptual areas (search for key constants):

1. **Configuration & Constants** (lines 463-515):
   - `DIFFICULTIES`: Spawn rates, enemy stats, lives, bombs per difficulty
   - `KILLS_PER_STAGE`: Enemy count before advancing to next stage
   - `THEMES`: 6 sky gradient color schemes
   - `ENEMY_TYPES`: Grunt, Interceptor, Tank with HP, speed, score values
   - `BOSSES`: 6 boss patterns with unique colors and HP multipliers

2. **Canvas & Rendering**:
   - Canvas setup and 2D context initialization
   - Adaptive performance tracking: quality levels 0-2 adjust particle/effect density
   - Theme application via gradient rendering in game loop

3. **Input Handling**:
   - Keyboard input: Arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
   - Mouse input: Move to aim, click to fire
   - Touch input: Left zone for movement, right zone for fire/bomb buttons with virtual joystick

4. **Audio Synthesis**:
   - WebAudio synth with oscillators, filters, and noise bursts
   - No sample files—all sounds procedurally generated
   - `beep(freq, dur, type, vol, opts)`: Sine/square wave with optional sweep, filter, pan, delay
   - `noiseBurst()`: Noise-based sound for explosions/impacts
   - Mute toggle and master gain control via localStorage

5. **Player & Weapons**:
   - Player ship with position, velocity, lives, bombs
   - Multiple weapon types: Vulcan (rapid fire), Plasma (charged), Laser (piercing)
   - Weapon level system: collecting power-ups increases weapon level and damage
   - Bomb mechanic: limited-use screen-clear ability

6. **Enemy & Boss System**:
   - Dynamic enemy spawning based on difficulty spawn rates
   - Three enemy types with different movement patterns (straight, weaving, slow tank)
   - Stage progression: kill `KILLS_PER_STAGE` enemies to advance, then boss fight
   - Boss logic: boss types rotate, HP scales with difficulty and stage

7. **Game Loop**:
   - RequestAnimationFrame-based main loop with deltaTime tracking
   - Update phase: player position, enemy positions, collision detection
   - Render phase: clear canvas, draw gradient sky, render sprites, update HUD
   - State management: title screen, playing, paused, game over

8. **Persistence & UI**:
   - LocalStorage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
   - HUD display: score, best score, multiplier, weapon level, hearts (lives), bombs
   - Boss health bar (shown during boss stage)
   - Combo popup (brief text display on combo hits)
   - Stage banner (announces new stage)
   - Overlay screen: title, controls hint, difficulty selection, game over with stats

## Common Development Tasks

### Testing the Game

Open `index.html` directly in a browser:
```bash
# Simple HTTP server (Python)
python3 -m http.server 8000

# Or use any other local server and visit http://localhost:8000/index.html
```

No build step required. Edit the file directly in your editor, save, and refresh the browser to see changes.

### Adjusting Difficulty Balance

Modify `DIFFICULTIES` object (lines 485-489):
- `spawnBase` / `spawnMin`: Control how frequently enemies spawn (lower = more frequent)
- `enemySpeed`: Multiplier for enemy movement speed (1.0 = baseline)
- `enemyShootChance`: Probability (0-1) that an enemy fires at the player each frame
- `lives`: Starting player lives
- `bombs`: Starting smart bomb count
- `bossHpMul`: Boss HP multiplier (1.0 = baseline)

### Adding New Enemies or Bosses

**New Enemy Type** (add to `ENEMY_TYPES`):
```javascript
newtype: { w: 40, h: 40, hp: 2, speedMul: 1.2, color: '#ff0000', dark: '#990000', score: 25, minStage: 3, weave: false }
```

Then reference it in the enemy spawn logic (search for `ENEMY_TYPES[...]`).

**New Boss** (add to `BOSSES`):
```javascript
{ name: 'PHOENIX', hullTop: '#ff6600', hullBottom: '#cc3300', wing: '#ff9900', core: '#ffdd00', hpMul: 1.25 }
```

Bosses rotate in order; each new boss is encountered on the next stage.

### Changing Themes

`THEMES` array (lines 493-500) defines sky colors. Each theme has:
- `sky`: Array of 3 colors for the gradient [top, middle, bottom]
- `cloud`: RGB string for cloud color (rendered as semi-transparent circles)
- `sun`: RGB string for sun glow color
- `name`: Display name

Themes rotate stage-by-stage in order.

### Adjusting Visual Effects

- **Screen shake**: Search for `vignette` class toggle and `ctx.translate()` camera offset
- **Flash on hit**: Controlled by `#flash` element animation and `flash.hit` class
- **Particle effects**: Adaptive quality system reduces particles on low FPS (quality variable)

### Weapons Tuning

Weapon objects are created during game init. Search for weapon stats (fire rate, damage, spread). Modify:
- Damage values
- Fire rate (time between shots)
- Bullet speed
- Spread pattern

### Accessibility Notes

- Screen reader announcements use `#srLive` (aria-live="polite")
- Reduced motion preference is detected via `prefers-reduced-motion` media query
- User can override via UI button (stored in localStorage)
- All animations are disabled when reduced motion is active
- Color contrast meets WCAG standards

## Known Constraints

1. **Single File**: All changes go into `index.html`. Large additions may become unwieldy.
2. **Canvas-Only Graphics**: No SVG or DOM-based rendering. All drawing is imperative canvas 2D code.
3. **No External Assets**: Audio, fonts, and images are all embedded or procedurally generated.
4. **No Build Tool**: Direct browser execution. Minification/bundling must be done externally if needed.

## Testing Checklist

Before committing game balance or feature changes:

- [ ] Test all three difficulties and confirm spawn rates feel correct
- [ ] Play through at least 2 full stages to verify progression
- [ ] Test all weapon types and confirm power-up pickup feels right
- [ ] Use bomb during a crowded enemy wave and confirm it clears screen
- [ ] Defeat a boss and verify stage advance/theme change
- [ ] Test pause (P key) and resume
- [ ] Toggle sound mute and reduced motion, verify localStorage persists on reload
- [ ] Test on mobile: drag left side to move, tap right side buttons to fire/bomb
- [ ] Verify best score updates and persists across sessions

## Performance Notes

The game includes adaptive quality tuning:
- Frame times are tracked in a rolling 45-frame window
- If average frame time exceeds 1/40 (25ms), quality increases (reduces particles/effects)
- If average frame time falls below 1/56 (~17.8ms), quality decreases (enables more effects)
- This helps maintain playability on lower-end devices while looking good on high-end ones
