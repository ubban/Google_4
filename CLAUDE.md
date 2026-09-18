# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game is self-contained in `index.html` (~1670 lines) with embedded CSS and JavaScript—no build process, no external assets, no dependencies.

**Key constraints:**
- Single HTML file (Canvas + WebAudio for all visuals and audio)
- No asset pipeline (all graphics are procedurally drawn; all sound is synthesized)
- Browser-first (desktop and mobile touch controls)
- Persistent settings via localStorage

## Game Architecture

### Core Game Loop & State
- **Main loop** (line 1640): `requestAnimationFrame` calls `loop()` at ~60 FPS
- **Update phase** (line 1148): `update(dt)` handles physics, collisions, and spawning
- **Draw phase** (line 1520): `draw()` renders all entities using Canvas 2D
- **Frame time tracking** (line 475): Adaptive performance system throttles quality if frame times exceed budget

**Game state variables** (line 864–869):
- `player`: Current ship position, weapon, level, invincibility timer
- `enemies`, `bullets`, `enemyBullets`, `particles`, `powerUps`: Active game objects
- `score`, `lives`, `stage`, `kills`, `chain`, `multiplier`: Session stats
- `boss`: Current boss entity (or `null`)
- `running`, `paused`, `gameOver`: State flags

### Difficulty & Progression
**Difficulties** (line 485–489): `DIFFICULTIES.easy/normal/hard` control:
- Enemy spawn frequency (spawnBase, spawnMin)
- Enemy movement speed multiplier
- Enemy shoot chance and accuracy
- Player starting lives and bombs
- Boss health multiplier

**Stage progression** (line 491, 1204–1207):
- After 18 kills per stage, a boss spawns
- Each stage increments and cycles through 6 boss designs
- Stages loop through 6 visual themes (different sky gradients and palettes)

### Entity Systems

#### Player Ship
- Positioned at bottom center; can move freely within canvas bounds
- Dual input modes: keyboard/joystick (priority) vs. mouse (fallback smoothing)
- Weapon system: Vulcan (spread burst) or Laser (piercing lanes)
  - Vulcan: levels 1–3 increase spread width (1 → 2 → 3 bullets)
  - Laser: levels 1–3 increase lane count (1 → 2 → 3 shots)
  - Weapon swaps reset level to 1
- Invincibility frame after damage (1.5s) with visual blink

#### Enemies
**Types** (line 502–506):
- **Grunt**: Red, slow, 1 HP, score 10
- **Interceptor**: Orange, fast (1.7x speed), weaves side-to-side, score 15
- **Tank**: Purple, slow, 4 HP, shows health bar, score 35 (stage 2+)

**Spawning** (line 1022–1041):
- Random position across screen width at y = -30
- Spawn frequency decreases over time (difficulty ramp at line 1201)
- Two shoot patterns: "spread" (3-way fan) or "aimed" (straight at player)
- Weaving motion applied to Interceptors in real-time

#### Bosses
**Six designs** (line 508–515): VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Each has unique HP multiplier, color scheme, and attack pattern set
- Boss health is calculated as: `baseHp × difficultyMul × designMul` (line 1046–1049)
- Entry animation: boss slides down from off-screen to y=120 before combat

**Attack patterns** (line 1372–1409):
- Each boss has 3 unique patterns that cycle
- Patterns use helper functions: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`
- Telegraph phase (22 frames) with visual effect before firing
- Attack cooldown (85 frames) between patterns

#### Power-ups
- **Life**: Heals one point (max to maxLives)
- **Bomb**: Grants one bomb charge (max to maxBombs)
- **Weapon**: Increments weapon level or swaps weapon if at level 3
- Spawn chance on enemy death: 6% life, 8% bomb, 20% weapon (line 1060–1066)
- Visual: rotating colored square; sound: three ascending sine tones

### Collision & Damage
- **Bullet vs. Enemy** (line 1232–1253): Laser bullets pierce up to 2 enemies; Vulcan bullets destroy on hit; enemy killed if HP ≤ 0
- **Bullet vs. Boss** (line 1255–1266): Boss takes 3 damage per hit
- **Enemy vs. Player** (line 1268–1273): Player loses life and becomes invincible
- **Enemy bullet vs. Player** (line 1276–1282): Same; player's chain combo resets
- **Boss vs. Player** (line 1284–1287): Collision damage during non-entry phase
- **Power-up vs. Player** (line 1289–1294): Consumed on overlap

**Combo/Multiplier** (line 1087–1101):
- Chain counter increments per kill; resets after 2.2 seconds without a kill
- Multiplier = 1 + floor(chain / 5), capped at 10x
- Score gained = base × multiplier

### Audio System
**WebAudio synthesizer** (line 545–595): Procedurally generated sound effects using oscillators, filters, and noise bursts (no external files).

**SFX catalog** (line 624–676):
- `shoot`: Vulcan fire (square wave, downward sweep)
- `laser`: Laser fire (sawtooth, upward sweep)
- `explodeGrunt/Interceptor/Tank`: Type-specific explosion sounds
- `hit`: Player damage (dual square + noise)
- `bomb`: Bomb detonation (long low rumble + noise)
- `powerup`: Three-tone ascending chime
- `bossHit`, `bossTelegraph`, `bossDown`: Boss-specific audio cues
- `stageClear`: Victory fanfare
- `gameover`: Descending failure tones

**Mute state** persists via localStorage (line 464, 518).

### Graphics & Rendering

**Canvas**: Fixed 480×720 base resolution, scaled to window size with aspect-ratio preservation (line 753–765)

**Rendering layers**:
1. **Background**: Gradient sky (6 stage themes) + sun glow + layered parallax clouds (line 1507–1544)
2. **Player ship**: Procedural plane drawing (line 913–968); aura glow when using laser (line 1549–1557)
3. **Bullets**: Rectangles (Vulcan) or cylinders (Laser); glow on high-quality (line 1579–1591)
4. **Enemy bullets**: Circles; red glow (line 1593–1601)
5. **Enemies**: Planes with health bars for tanks (line 1603–1613)
6. **Boss**: Procedurally drawn with animated parts (line 970–1019)
7. **Power-ups**: Rotating colored squares (line 1617–1626)
8. **Particles**: Small circles that fade out (line 1628–1636)

**Visual effects**:
- Screen shake on damage/bomb (magnitude and duration configurable; line 727–729)
- Flash overlay on hit or bomb (separate animation per type; line 698–709)
- Vignette pulse when health = 1 (red radial gradient; line 75–87)
- Stage banner and combo pop animations (line 712–724)
- Boss health bar (line 149–180)

### Performance & Quality Adaptation
**Adaptive quality** (line 474–483):
- Tracks last 45 frame times
- Quality levels: 0 (full, glow effects), 1 (medium, reduced particle count), 2 (low, no glow)
- Upgrades quality if avg frame time < 1/56s; downgrades if > 1/40s
- Used to reduce particle counts and disable shadow/glow effects at lower qualities

### Input Handling

**Keyboard/Gamepad**:
- Arrow keys or WASD: movement
- Space or mouse click: fire
- X or Shift: bomb
- P or Esc: pause

**Mouse**: Crosshair cursor follows mouse position; player glides toward cursor if no keyboard input active (line 836–861)

**Touch**: Left half of screen = joystick (virtual stick appears on touch); right half = two buttons (FIRE, BOMB)
- Joystick movement clamped to 50px radius (line 785–812)
- Buttons immediately fire/bomb on touch

**Audio context resume**: Triggered on first user interaction to work around browser autoplay restrictions (line 560, 816, 854)

### Persistent Storage
**localStorage keys** (line 463–466):
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Mute toggle (0/1)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Motion preference (0/1, overrides system preference)

## How to Modify the Game

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502):
   ```js
   newType: { w: 36, h: 36, hp: 2, speedMul: 1.2, color: '#..', dark: '#..', score: 20, minStage: 1, weave: false }
   ```
2. Optionally add explosion sound to `EXPLODE_SFX` (line 678) and define in `sfx` object (line 624)
3. Adjust `spawnEnemy()` probability table (line 1029) to include new type

### Add a New Boss or Modify Existing
- **Boss design**: Add to `BOSSES` array (line 508–515) with name, hull/wing/core colors, HP multiplier
- **Boss attack patterns**: Add 3-pattern array to `BOSS_PATTERNS` (line 1372–1409)
  - Use helper functions or create new attack function
  - Patterns fire into `enemyBullets` array
  - Pattern fires during telegraph phase; boss re-telegraphs after attack cooldown

### Adjust Difficulty Parameters
Edit `DIFFICULTIES` object (line 485–489):
- Increase `spawnBase`/lower `spawnMin` for more enemies
- Increase `enemySpeed` or `enemyShootChance` for harder enemies
- Adjust `lives`, `bombs`, or `bossHpMul`

### Change Visual Themes
**Sky gradients and colors** in `THEMES` array (line 493–500):
- `sky`: 3-stop gradient from top to bottom
- `cloud`: RGB values for parallax cloud fill
- `sun`: RGB values for sun glow
- `name`: Display name

### Tune Audio
Edit `sfx` definitions (line 624–676) to change pitch, duration, sweep direction:
- `beep(freq, dur, type, vol, opts)`: Oscillator-based tone
  - `sweepTo`: Pitch sweep target
  - `filterType`, `filterFreq`: Frequency filter
  - `pan`: Stereo pan (L-R)
- `noiseBurst(dur, vol, opts)`: White noise-based sound

## Testing & Debugging

**No build step**: Just open `index.html` in a browser. All code is inline.

**Common edits**:
- Change enemy spawn rate: Adjust `DIFFICULTIES` spawnBase/spawnMin
- Modify player weapon balance: Edit `fireWeapon()` cooldown (line 1177) or bullet properties
- Adjust screen shake: Change `screenShake()` calls' magnitude/duration parameters
- Toggle visual effects: Add `if (reducedMotion)` guards or quality checks (`if (quality === 0)`)

**Accessibility**:
- Respects `prefers-reduced-motion` media query (line 520–523)
- Screen reader announcements via `announce()` for key events (line 468–471)
- Full keyboard support; no mouse-only mechanics

## Code Organization

The 1669-line script is organized roughly as:
1. **Constants** (Difficulties, Enemy types, Bosses, Themes, Audio effects)
2. **DOM setup & event listeners** (Canvas, buttons, input handlers)
3. **Game state** (Player, enemies, bullets, etc.)
4. **Helper functions** (Drawing, collision, audio, UI updates)
5. **Update & draw loops** (Core game simulation)
6. **Startup** (Event listeners, initial render)

No external dependencies or frameworks—pure Canvas/Web Audio/DOM manipulation.
