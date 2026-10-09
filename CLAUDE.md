# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single HTML5 file (~1670 lines). It features:
- Keyboard/mouse/touch controls for desktop and mobile
- Multiple difficulty levels (Easy, Normal, Hard)
- Enemy types with different behaviors and health
- Boss encounters with pattern-based attacks
- Weapon progression (Vulcan gun → Laser) with levels
- Power-ups (health, bombs, weapons)
- Score multiplier based on kill chains
- WebAudio synthesis for all sound effects (no external audio files)
- Adaptive performance quality scaling
- LocalStorage for persistent scores and settings

## Code Architecture

The entire game runs in a single self-contained IIFE (Immediately Invoked Function Expression) starting at line 423. All state is encapsulated within this closure.

### High-Level Structure

1. **DOM & Canvas Setup** (lines 426-461): Element references and canvas initialization
2. **Storage & State Initialization** (lines 463-527): LocalStorage keys, settings, difficulty loading
3. **Audio System** (lines 542-677): WebAudio API wrapper for sound effects
4. **UI Rendering** (lines 680-709): HUD updates (hearts, bombs, score), screen effects (flash, vignette)
5. **Input Handling** (lines 767-861): Keyboard, mouse, and touch input processing with joystick simulation
6. **Game State** (lines 863-911): Player, enemies, bullets, particles initialization and reset
7. **Drawing Functions** (lines 913-1020): Rendering for player, enemies, bosses
8. **Enemy & Boss Logic** (lines 1022-1439): Spawning, updating, movement patterns, attack patterns
9. **Game Loop** (lines 1640-1665): Update → Draw → requestAnimationFrame

### Key Game Systems

**Difficulty Scaling** (lines 485-489):
- Three difficulty presets control spawn rates, enemy speed, shoot probability, lives, bombs, boss HP
- Spawn rate decreases dynamically every 300 frames (line 1201)

**Weapon System** (lines 871-1146):
- Two weapons: Vulcan (spread shots) and Laser (piercing shots)
- Weapon levels 1-3 unlock wider spreads or additional fire lanes
- Level 3 + next power-up = weapon switch
- Cooldown: 0.15s (vulcan) / 0.1s (laser)

**Enemy Types** (lines 502-506):
- Grunt: Basic slow enemy (hp: 1)
- Interceptor: Fast weaving enemy (hp: 1)
- Tank: Heavily armored (hp: 4), appears stage 2+
- Health bars only render for tanks (line 1605)

**Boss System** (lines 508-515, 1372-1439):
- 6 boss designs cycle through stages (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Each boss has a unique 3-attack pattern
- Patterns use helper functions: fireRadial, fireAimedSpread, fireColumns, fireSpiralBurst, fireCross, fireHomingSeeded, fireDoubleRing
- Boss telegraph animation plays before each attack (line 1433)
- Boss damage: 3 HP per player bullet, 24 HP per bomb

**Chain/Multiplier System** (lines 1087-1101):
- Kill chain increments on each enemy death
- Chain resets if 2.2 seconds pass without a kill (chainTimer)
- Multiplier = 1 + floor(chain / 5), capped at 10x
- Score calculation: base * multiplier

**Performance Adaptation** (lines 473-483):
- Tracks frame times over 45 frames
- Quality levels: 0 (full), 1 (medium), 2 (low)
- Lower quality reduces particle counts (line 1074) and disables glow effects

**Input Priority** (lines 1152-1169):
- Keyboard/joystick always takes priority over mouse steering
- Mouse steers the plane smoothly toward cursor when active
- Touch controls: left side for movement, right side for fire/bomb buttons

## Common Development Tasks

### Adjusting Game Balance

**Enemy Spawn Rate**: Modify `DIFFICULTIES` object (lines 485-489). Lower `spawnBase` = more frequent spawns.

**Boss Difficulty**: Adjust `hpMul` in BOSSES (lines 508-515) or `bossHpMul` in DIFFICULTIES (lines 486-488).

**Weapon Balance**: 
- Fire cooldown: line 1177 (`fireDelay`)
- Bullet speed & spread: lines 1127-1145
- Laser pierce count: line 1141

**Enemy Behavior**:
- Move speed multipliers: ENEMY_TYPES (line 504)
- Shoot probability: `enemyShootChance` in DIFFICULTIES
- Shoot patterns: `pattern` field in spawnEnemy (line 1037)

### Adding New Content

**New Enemy Type**:
1. Add to ENEMY_TYPES (lines 502-506) with w, h, hp, speedMul, color, dark, score, minStage
2. Update typeKey logic in spawnEnemy (lines 1026-1030)
3. Add explosion sound in EXPLODE_SFX (line 678)
4. Define sfx function if needed (lines 624-676)

**New Boss Pattern**:
1. Add pattern function to BOSS_PATTERNS (lines 1372-1409)
2. Pattern functions receive boss object `b` and fire bullets via enemyBullets array
3. Helper functions available: fireRadial, fireAimedSpread, fireColumns, etc. (lines 1324-1370)

**New Power-up Type**:
1. Add spawn chance in maybeSpawnPowerUp (lines 1058-1067)
2. Handle in applyPowerUp (lines 1304-1322)
3. Add color case in powerUps rendering (line 1621)

**New Theme/Background**:
1. Add to THEMES array (lines 493-500) with sky gradient stops, cloud color, sun color
2. Theme cycles based on stage: `THEMES[(stage - 1) % THEMES.length]` (line 1529)

### Modifying Audio

All sound effects use WebAudio synth (no samples). Main SFX functions:
- `beep(freq, dur, type, vol, opts)`: Tone generator with frequency sweep, filter, pan
- `noiseBurst(dur, vol, opts)`: Pink noise generator

To modify a sound:
1. Find it in `sfx` object (lines 624-676)
2. Adjust frequency, duration, volume, and options (sweepTo, filterFreq, filterType, pan, delay)
3. Example: `sfx.shoot()` uses 760→480 Hz sweep at 70ms (lines 625)

### Canvas & Rendering

- **Base resolution**: 480×720 (lines 753-761)
- **Scaling**: Maintains aspect ratio up to 1.6x zoom
- **Drawing context**: `ctx` is canvas 2D context
- **Coordinate system**: (0,0) at top-left; y+ downward

### UI & Overlays

- Main overlay (menu, pause, game-over): Controlled by `overlay.hidden` (line 1487)
- Boss health bar: `bossBarWrap` visibility (line 1054)
- HUD stats (score, multiplier, hearts, bombs): Auto-updated via renderHearts/renderBombs
- Screen shake: `screenShake(magnitude, duration)` (line 727)
- Flash effects: `flashHit()` and `flashBomb()` (lines 698-709)

### Accessibility

- Reduced motion preference: Respects `prefers-reduced-motion` media query (line 520)
- Screen reader announcements: `announce(text)` pushes to aria-live region (lines 468-470)
- Keyboard support: P/Esc pause, X/Shift bomb, Space/Click fire, Arrows/WASD move

## Testing Your Changes

1. **Open in Browser**: Simply open `index.html` in a web browser
2. **Test on Mobile**: Use device's touch controls or browser DevTools device emulation
3. **Check Audio**: Ensure WebAudio context initializes (click canvas to trigger)
4. **Verify Performance**: Open DevTools → Performance tab, record a run, check frame times
5. **Accessibility**: Test with keyboard only; enable "reduce motion" setting and verify effect

## Performance Notes

- Particle count scales with quality level (line 1074)
- Glow/shadow effects disabled at low quality (line 1579)
- Boss telegraph glow varies by quality (line 1002)
- Frame cap: 50ms per frame (line 1644) prevents timing issues
- On mobile/low-end devices, quality auto-reduces if avg frame time > 25ms
