# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 web application. The entire game (1670 lines) is self-contained in `index.html` with embedded CSS styling and vanilla JavaScript game logic, no build system or external dependencies.

Key features:
- Procedurally-generated graphics (no images) using Canvas 2D API
- Synthesized sound effects via WebAudio (no audio files)
- Three difficulty levels with dynamic spawning and progressive difficulty
- Six unique boss encounters with distinct attack patterns
- Weapon progression (Vulcan gun → Laser with pierce mechanic)
- Persistent high score and player preferences via localStorage
- Full keyboard, mouse, touch, and joystick support
- Responsive design (CSS clamp) and accessibility (screen reader announcements)

## Running the Game

There is no build process. To play:

1. **Open in browser**: Open `index.html` directly in any modern browser (Chrome, Firefox, Safari, Edge).
2. **Local server** (optional, for stricter origin policies): Run `python3 -m http.server` or similar and browse to `http://localhost:8000/index.html`.
3. **Deploy**: The single file can be deployed anywhere that serves static HTML (GitHub Pages, Netlify, S3, etc.).

**Controls**:
- **Desktop**: Mouse (move to steer), Click (fire), Space (fire), X/Shift (bomb), P/Esc (pause)
- **Mobile/Touch**: Drag left zone (move), tap FIRE/BOMB buttons

## Architecture Overview

### Game Loop & State Management

The game runs on a single `requestAnimationFrame` loop (`loop()` at line 1640) that:
1. Calculates delta time with 50ms cap
2. Tracks performance (adaptive quality reduction)
3. Calls `update(dt)` for game logic if running and not paused
4. Calls `draw()` to render each frame
5. Manages screen shake effects

**Core state variables** (line 864-869):
- `player`: Position, size, weapon level, invulnerability timer
- `enemies`, `bullets`, `enemyBullets`, `powerUps`, `boss`: Pooled entity arrays
- `score`, `lives`, `stage`, `kills`: Game progression
- `chain`, `multiplier`: Combo system for scoring bonuses

### Input Handling

**Three input methods** processed each frame in `update()`:
1. **Keyboard** (keys object, line 768): WASD/Arrows for movement, Space for fire, X/Shift for bomb
2. **Mouse** (lines 838-861): Plane glides toward pointer when over canvas; click fires
3. **Touch** (lines 814-834): Left zone (55% width) for joystick, right zone for action buttons (FIRE/BOMB)

Movement precedence: keyboard/joystick > mouse steering. This prevents fighting between input methods.

### Difficulty & Progression System

**Difficulties** (line 485):
```
easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 }
normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 }
hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
```

**Stage progression**:
- Kill 18 enemies (`KILLS_PER_STAGE`) to trigger boss warning (line 1204)
- Boss appears after 1.2s delay, enemy spawning stops
- Defeating boss advances stage, resets kill counter, increments theme/boss
- Themes and bosses cycle through their respective arrays indefinitely

### Collision & Damage System

**Overlap detection** (`rectsOverlap()`, line 1069): Simple AABB using center-based distances.

**Collision resolution order** in `update()` (lines 1232-1294):
1. Player bullets vs enemies (line 1232): Vulcan pierce=0, Laser pierce varies by level
2. Player bullets vs boss (line 1255): Always pierce reduced
3. Enemies vs player (line 1268): Triggers `loseLife()`, invulnerability frame
4. Enemy bullets vs player (line 1276): Same as above
5. Boss vs player (line 1284): Collision damage, no invulnerability (bug or intended?)
6. Powerups vs player (line 1289): Applies upgrade, removes powerup

### Entity Systems

**Enemies** (line 1022, object spawning):
- Three types: `grunt` (1 HP, fast), `interceptor` (1 HP, fastest, patterns), `tank` (4 HP, slow)
- Weaving interceptors use sine wave based on frame time
- Shoot patterns: `spread` (3-shot fan) or `aimed` (homing toward player)
- Randomly despawn powerups on death

**Bosses** (line 1043):
- One boss per stage, cycling through 6 designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Each has unique color scheme, HP multiplier, and attack pattern set
- Enter phase: animate from top to position (y=120), then enter combat
- Combat: horizontal patrolling with pattern telegraph (22 frames warning), 85-frame attack cooldown
- Six attack types spread across bosses: radial bursts, aimed fans, columns, spirals, cross-fire, homing

**Powerups** (line 1058):
- 6% spawn: +1 life (red)
- 8% spawn: +1 bomb (yellow)
- 20% spawn: weapon upgrade (green)
  - Weapon progression: Vulcan Lv1 → Lv2 (spread) → Lv3 (triple) → Laser Lv1 → Lv2 (dual lanes) → Lv3 (triple)

### Weapon System

**Vulcan** (`fireWeapon()` line 1126):
- Spread based on level: single, dual (-0.12/+0.12), triple (-0.24/0/+0.24 radians)
- All pierce=0 (destroyed on hit)
- Fire rate: 0.15s cooldown

**Laser** (line 1135):
- Lanes based on level: center, dual (±6px), triple (±6px + center)
- Pierce values: Lv1/2 pierce=1, Lv3 pierce=2
- Fire rate: 0.1s cooldown (faster)
- Visual: glow aura around player, cyan bullet color

### Scoring & Multiplier System

**Chain multiplier** (`addScore()` line 1087):
- Each kill increments chain counter
- Multiplier = 1 + min(9, floor(chain / 5))
  - Caps at 10x for 50+ consecutive kills
- Chain timer resets to 2.2s per kill
- Chain breaks on any damage taken or timer expiry

Milestone: Shows "X KILLS!" popup at 10, 20, 30+

### Audio System

**WebAudio Synth** (lines 542-676):
- No audio files; all SFX generated in real-time using oscillators and noise buffers
- `beep(freq, dur, type, vol, opts)`: Tone with optional pitch sweep, filter, panning
- `noiseBurst(dur, vol, opts)`: White noise with envelope, filtered
- Pan based on screen position for spatial audio
- Handles audio context suspension (user gesture required on some browsers)
- Mute state persisted to localStorage

### Rendering & Visual Effects

**Canvas size** (line 753): 480×720 base, scaled up to 1.6× max via CSS, responsive to window/orientation changes.

**Themes** (line 493): Six rotating sky color schemes with unique palettes affecting:
- Sky gradient (3 stops)
- Cloud color
- Sun glow color

**Visual effects**:
- Screen shake: Randomized translation offset, reduced if `reducedMotion` enabled
- Vignette pulse: Red flash edges when health critical
- Flash overlay: White flash on hit (0.25s) or bomb (0.5s)
- Combo popup: 0.6s scaling animation, center screen
- Stage banner: 2s fade animation on stage start

**Performance adaptation** (lines 474-482):
- Tracks last 45 frame times, computes average
- If avg > 25ms (40 FPS target), quality steps down: reduces particle counts, glows, shadow blur
- If avg < 17.9ms (56 FPS), quality steps back up
- Quality affects particle spawn count and shadow rendering

### UI & Persistence

**localStorage keys**:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Sound state (1 or 0)
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: User override for motion sensitivity (respects `prefers-reduced-motion` media query by default)

**Overlay states**: Menu (start), Paused (pause), Game Over (end). Managed via `overlay.hidden` and content swapping.

**Accessibility**:
- Screen reader live region (`#srLive`) receives game status announcements
- Reduced motion support: Disables all animations, reduces screen shake
- Semantic HTML buttons with titles

## Modifying the Game

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` (around line 502):
   ```javascript
   newEnemy: { w: 30, h: 30, hp: 2, speedMul: 1.5, color: '#00ff00', dark: '#00cc00', score: 25, minStage: 3 }
   ```
2. Add explosion sound to `EXPLODE_SFX` (line 678) if desired
3. Update `spawnEnemy()` selection logic (line 1026) to include it
4. Implement drawing in `draw()` loop if custom shape needed (else defaults to `drawPlane()`)

### Adding a New Boss Attack Pattern

1. Create a pattern function using helpers (`fireRadial`, `fireAimedSpread`, etc.) at line 1324+
2. Add new boss design to `BOSSES` array (line 508) with color scheme and HP multiplier
3. Add pattern set to `BOSS_PATTERNS` (line 1372) at the index corresponding to boss order
4. Pattern receives boss object `b`, fires into `enemyBullets` array

### Tweaking Game Balance

- **Enemy spawn rate**: Edit `spawnBase` and `spawnMin` in `DIFFICULTIES` object
- **Enemy damage output**: Adjust `enemyShootChance` and `speed` for their bullets in spawn logic
- **Weapon damage**: Boss HP reduction is hardcoded (3 per player bullet, line 1259); laser pierce values at line 1141
- **Boss HP**: Edit `DIFFICULTIES[diff].bossHpMul` or `baseHp` formula at line 1046
- **Bomb damage**: Edit damage value in `useBomb()` (line 1117)
- **Scoring**: Adjust `addScore()` calls and `chain` division factor (line 1090)

### Adding Sound Effects

Call `sfx.<name>()` or define a new synth function:
```javascript
sfx.custom = () => beep(400, 0.2, 'sine', 0.1, { sweepTo: 200 });
```
Then call it wherever appropriate (e.g., on power-up, in update loop events).

### Changing Game Feel

- **Player speed**: Edit `player.speed` at line 877 (default 260)
- **Screen shake**: Modify `screenShake()` calls' magnitude and duration parameters
- **Particle count**: Adjust multipliers in `spawnExplosion()` at line 1074
- **Color scheme**: Edit `THEMES` array or regenerate using CSS variables approach
- **Animation timing**: Keyframe durations in `<style>` block (e.g., `comboPulse`, `stagePulse`)

## Notes for Future Development

- **Single file limitation**: All changes go in `index.html`. For modularity, consider extracting to separate JS/CSS files and using a bundler.
- **No external assets**: Image and audio generation is computationally simple; for richer content, external files may be necessary.
- **Touch input polish**: Joystick base visibility and button sizing could be tuned per device.
- **Accessibility gaps**: Boss telegraph sounds rely on hearing; consider visual indicator.
- **Performance ceiling**: On older devices, particle effects and boss patterns may tank performance; quality adaptation helps but lower cap possible if needed.
