# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. Single HTML file with embedded CSS and JavaScript. No external dependencies. Runs entirely in the browser using Canvas 2D and WebAudio API for sound synthesis.

## Running the Game

### Development
- Open `index.html` directly in a browser, or serve locally with any HTTP server
- Game runs at 480×720px logical resolution, scales to fit window
- Works on desktop (keyboard/mouse) and mobile (touch controls)

### Testing Features
- **Difficulty**: Easy/Normal/Hard (selectable in menu)
- **Audio**: Mute button in top-right corner
- **Motion**: Toggle reduced-motion effects (screen shake, flash, pulse)
- **Pause**: P or Esc key (or pause button)

## Code Architecture

The entire game logic lives in one `<script>` block (lines 422–1667). Code is organized by subsystem:

### 1. **Canvas & Graphics (lines 752–765, 1507–1638)**
- Canvas element rendered to 480×720px, then CSS-scaled to fit window
- `draw()` function renders one frame
- Uses 6 visual themes that cycle per stage (different sky gradients, cloud colors)
- Dynamic glowing effects for bullets and power-ups (reduced on low-quality devices)
- Particle system for explosions (count scales with quality setting)

### 2. **Audio (lines 542–676)**
- WebAudio synth: generates all sounds procedurally (no audio files)
- `beep()` and `noiseBurst()` create tones with frequency sweeps and filtering
- Sound effects: shoot, laser, explosions (by type), hit, bomb, powerup, boss telegraph, boss defeat, stage clear, UI click, game over
- Master gain and compressor for dynamic range control
- Can be muted; respects OS reduced-motion preference

### 3. **Input & Controls (lines 767–862)**
- **Keyboard**: Arrows/WASD to move, Space to fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Move to steer, click to fire (takes priority over keyboard when active)
- **Touch**: Left 55% of screen = joystick (drag to move), Right 45% = FIRE and BOMB buttons
- Joystick radius clamped to 50px max distance
- Input state merged per-frame; keyboard/joystick takes priority over mouse steering

### 4. **Game State (lines 863–910)**
- Global vars: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Session stats: `score`, `lives`, `stage`, `kills`, `stageKills`, `chain`, `multiplier`
- `resetGame()` initializes all state for a new game run

### 5. **Enemy System (lines 502–506, 1022–1041, 1210–1228)**
- **Three enemy types**: Grunt (slow, 1 HP), Interceptor (fast, weaving, 1 HP), Tank (slow, 4 HP, shows health bar)
- Each type has spawn conditions (minStage), score value, color
- Enemies spawn at top, move downward; removed when off-screen
- Two firing patterns: **Spread** (fixed 3-way fan) or **Aimed** (homing at player)
- Weaving: Interceptors sine-wave around their spawn X while moving down

### 6. **Player & Weapon System (lines 877, 1124–1146, 1304–1322)**
- Player position, size, speed (260 px/s), cooldown timer, invulnerability timer
- **Two weapons**: Vulcan (rapid-fire bullets) and Laser (piercing slow beams)
- Each weapon has 3 levels that unlock shot patterns:
  - **Vulcan L1**: Single center, **L2**: Dual, **L3**: Triple spread
  - **Laser L1**: Single, **L2**: Dual lanes, **L3**: Triple with center
- Weapons picked up via power-ups; hitting 3 levels cycles to next weapon
- **Power-ups**: Life (restore HP), Bomb (add bomb charge), Weapon (upgrade/switch)

### 7. **Boss System (lines 508–515, 1043–1056, 1372–1439, 1441–1454)**
- **Six boss types** (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) cycle per stage
- Each boss has unique hull colors, core glow, and attack pattern set
- Boss enters from top (y = -80 → 120 over ~2 seconds), then bounces side-to-side
- Enters after killing 18 enemies per stage (`KILLS_PER_STAGE = 18`)
- **Telegraph system**: Boss flashes white for ~22 frames before firing (players learn to expect attacks)
- Each boss fires from one of 3 attack patterns each turn (radial burst, aimed spread, special patterns)
- Attack patterns are phase-locked: BOSS_PATTERNS[bossIndex][patternIndex % 3]

### 8. **Difficulty System (lines 485–489, 741–750)**
- **Easy**: 75 spawn base, 40 min spawn, 1.0× speed, 0.45 shoot chance, 4 lives, 3 bombs, 0.8× boss HP
- **Normal**: 58 spawn base, 26 min spawn, 1.4× speed, 0.7 shoot chance, 3 lives, 2 bombs, 1.0× boss HP
- **Hard**: 40 spawn base, 18 min spawn, 1.9× speed, 0.95 shoot chance, 3 lives, 2 bombs, 1.3× boss HP
- Spawn timer decreases over time (difficulty ramps every 300 frames)
- Saved to localStorage; persists across sessions

### 9. **Scoring & Combo (lines 1087–1101, 1244–1248)**
- Kill an enemy → `addScore(baseScore * multiplier)`
- **Chain timer**: 2.2 seconds per kill; hits break chain
- **Multiplier**: 1 + floor(chain/5), caps at 10× (50+ kills in chain)
- Bombs also award score (5 per enemy destroyed in bomb explosion)
- Boss defeat awards flat 500 points
- Best score saved to localStorage; shown in HUD and end-game screen

### 10. **Collision & Damage (lines 1069–1071, 1232–1287)**
- All rectangles use AABB: `rectsOverlap(a, b)` checks half-widths/heights
- Enemy bullets → Player: 1 damage (if not invulnerable)
- Player bullets → Enemy: 1 damage, spawns 4-particle explosion
- Player bullets → Boss: 3 damage (when boss is done entering)
- Bomb → All enemies/boss in scene: instant kill with 8–10 particle explosion
- Laser bullets pierce (consume `pierce` counter, reusable across hits)
- Player invulnerability lasts 1.5s after hit; blinks at 4-frame intervals

### 11. **Particle & Visual Effects (lines 1073–1085, 696–730)**
- Particles: position, velocity, life timer, color
- Spawned on enemy/boss death (14 particles at full quality, scaled down on low-end devices)
- Render as circles, fade out over 0.5s
- Screen shake: magnitude and duration set per event (hit, bomb, boss defeat)
- Flash overlay: brief white flash (hit) or longer flash (bomb)
- Combo pop animation: large centered text burst at 0.6s
- Stage banner: top-left text banner for 2s (STAGE X, boss warning)

### 12. **Performance Adaptation (lines 473–483)**
- Tracks last 45 frame times; if avg > 25ms (40 FPS target), reduces quality
- **Quality 0** (full): all particles, glow effects, shadows
- **Quality 1** (medium): 60% particles, reduced glow
- **Quality 2** (low): 35% particles, no glow
- Quality bumps back up if frame times drop below 17.8ms (56 FPS)

### 13. **Accessibility (lines 520–540, 468–471, 349–358)**
- OS `prefers-reduced-motion` detected and honored (animations disabled)
- Reduced-motion toggle button (🎆/🚫) in top-right
- Screen reader live region (`#srLive`) announces game events
- Emoji-based UI icons for universal readability

### 14. **Storage & Persistence (lines 463–466, 517–527, 524–527)**
- localStorage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- Best score persists across sessions
- Audio mute, difficulty, and motion preference saved

## Key Constants & Tuning

| Constant | Value | Notes |
|----------|-------|-------|
| BASE_W × BASE_H | 480 × 720 | Logical game resolution |
| KILLS_PER_STAGE | 18 | Enemies per stage before boss spawns |
| Player speed | 260 px/s | Movement speed |
| Vulcan cooldown | 0.15s | Fire rate (~6.7 shots/sec) |
| Laser cooldown | 0.10s | Fire rate (10 shots/sec) |
| Player invuln | 1.5s | After-hit invulnerability duration |
| Boss telegraph | 22 frames | Pre-attack flash duration (~366ms at 60 FPS) |
| Bomb flash | 0.5s | Bomb explosion animation duration |
| Chain timer | 2.2s | Time window to maintain kill chain |
| Chain multiplier | 1 + floor(kills/5) | Caps at 10× |

## Boss Attack Patterns

Each boss (indexed 0–5) has a pattern set of 3 attacks that repeat:
- **VANGUARD**: radial burst → aimed fan → wide V-fan
- **BEHEMOTH**: heavy columns → slow homing spread → dense radial wall
- **SPECTER**: teleport dash + radial → tight spiral → double aimed shots
- **DREADNOUGHT**: 8-way cross → homing missiles → dense radial
- **WRAITH**: teleport barrage + cross → expanding double ring → aimed cross-spread
- **TITAN**: heavy columns → full 26-way ring → heavy homing volley

Patterns fire on a ~22-frame telegraph before each attack. Boss health scales by difficulty multiplier and per-boss hpMul.

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (w, h, hp, speedMul, color, dark, score, minStage, optional weave)
2. Update `spawnEnemy()` choice logic if needed (e.g., tier-based spawn rates)
3. Add explosion SFX to `EXPLODE_SFX` mapping if unique sound desired
4. Collision/damage already generic; no other changes needed

### Adding a New Boss
1. Add boss design to `BOSSES` array (name, hull colors, core color, hpMul)
2. Add pattern set to `BOSS_PATTERNS` array at matching index
3. Boss selection auto-cycles per stage: `BOSSES[(stage - 1) % BOSSES.length]`
4. Health formula: `70 + stage * 35` scaled by difficulty and boss.hpMul

### Adjusting Difficulty Curves
- Edit `DIFFICULTIES` object (spawn rates, speeds, lives, bombs, boss HP)
- Or modify `spawnTimer` ramp in `update()` (line 1201) to change progression speed
- Quality adaptation settings: lines 476–482

### Adding Visual Themes
1. Add theme to `THEMES` array (sky gradient colors, cloud color, sun color, name)
2. Format: `{ sky: [color1, color2, color3], cloud: 'r,g,b', sun: 'r,g,b', name: 'Theme Name' }`
3. Themes auto-cycle per stage: `THEMES[(stage - 1) % THEMES.length]`

### New Sound Effects
1. Call `beep(freq, duration, waveform, volume, {options})` or `noiseBurst(duration, volume, {options})`
2. Add to `sfx` object with a method name
3. Options: `sweepTo`, `filterFreq`, `filterType`, `pan`, `delay`
4. Pan calculated from X position via `panFor(x)` for stereo positioning

## Testing Checklist

- [ ] Game runs at 60 FPS on target device
- [ ] All enemy types spawn and behave correctly per stage
- [ ] Weapons upgrade and switch as expected
- [ ] Boss phases fire correct attack patterns with telegraph
- [ ] Collision detection works (bullets, enemies, boss, power-ups)
- [ ] Audio synth works; mute toggle works
- [ ] Reduced-motion preference respected
- [ ] Touch controls respond (joystick, buttons)
- [ ] Score/multiplier/chain system works
- [ ] localStorage persists best score, difficulty, settings
- [ ] Mobile responsive (scales to device size)
