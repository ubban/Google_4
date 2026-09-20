# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Sky Fighter: Raiden-style Vertical Shoot-'em-up

A fully self-contained HTML5 Canvas game with no external dependencies. Features a classic arcade-style vertical scrolling shooter with stage progression, boss encounters, weapon upgrades, and three difficulty levels.

## Quick Start

- **Run/Test**: Open `index.html` in a browser. No build step required.
- **Browser compatibility**: Modern browsers with Canvas, AudioContext, and ES6 support (Chrome, Firefox, Safari, Edge).
- **Mobile**: Full touch control support with on-screen joystick and action buttons.

## Architecture Overview

The entire game lives in a single `index.html` file (~1500 lines) with three sections:

### 1. **HTML/DOM Structure**
- Canvas element (`#gameCanvas`) for game rendering
- HUD overlay with score, lives (hearts), bombs, weapon info, difficulty buttons
- Game state overlays (pause, game over)
- Touch control zones (joystick area, action buttons)
- Accessibility elements (screen reader announcements via `#srLive`)

### 2. **CSS Styling**
- Responsive canvas scaling (maintains 480×720 aspect ratio)
- Dark space theme with blue gradients
- UI overlays with glassmorphism effects (backdrop blur)
- Animations: screen shake, vignette pulse, combo popups, boss health bar
- Respects `prefers-reduced-motion` for accessibility

### 3. **JavaScript Game Engine**

#### **Game State & Constants**
- `DIFFICULTIES`: Easy/Normal/Hard with spawn rates, enemy speeds, lives, boss HP multipliers
- `THEMES`: 6 stage visual themes (Daylight Front, Sunset, Night, Storm, Arctic, Volcanic)
- `ENEMY_TYPES`: grunt, interceptor, tank with stats (HP, speed, score, min stage)
- `BOSSES`: 6 unique bosses with design colors and relative HP multipliers
- `BOSS_PATTERNS`: Array of attack patterns per boss (radial fire, aimed spreads, columns, spirals, etc.)

#### **Player Object**
```js
{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }
```
- Position and collision box
- Weapons: `vulcan` (spread shots) or `laser` (piercing beams)
- Weapon levels: 1-3 (increased fire rate / spread / pierce count)

#### **Game Objects**
- **Bullets** (player): `{ x, y, w, h, speed, vx, pierce, laser }`
- **Enemies**: `{ x, y, type, hp, canShoot, pattern, weave, color, dark, score }`
- **Enemy bullets**: `{ x, y, speed, vx, radial, dx, dy }` (radial for boss patterns)
- **PowerUps**: `{ x, y, type: 'life'|'bomb'|'weapon', speed }`
- **Particles**: Explosion fragments with velocity and fade-out
- **Boss**: `{ x, y, hp, maxHp, design, telegraph, patternIndex, ... }`

#### **Core Subsystems**

**Input System**
- Keyboard (arrows + WASD for movement, Space/X/Shift for fire/bomb)
- Mouse (hover to steer, click to fire)
- Touch joystick (left side) + fire/bomb buttons (right side)
- `firing` flag tracks fire button state; mouse movement updates `mouseTarget`

**Audio (WebAudio Synth)**
- No external audio files; all sounds generated via oscillators and noise buffers
- `beep(freq, dur, type, vol, opts)`: Tone with sweep, filter, pan
- `noiseBurst(dur, vol, opts)`: Noise-based sounds
- `sfx` object with named sounds: `shoot`, `laser`, `explodeGrunt`, `bomb`, `hit`, `powerup`, `bossHit`, `stageClear`, etc.
- Master gain + compressor on all output
- Mute state persisted in localStorage

**Collision System**
- AABB (axis-aligned bounding box) via `rectsOverlap(a, b)`
- Player with enemies, enemy bullets, boss
- Player bullets with enemies and boss
- PowerUps with player
- Boss never collides during entering phase

**Rendering System**
- Canvas scaling: base 480×720 pixels, scales up to max 1.6× to fit window
- Frame-by-frame drawing (no dirty rect optimization)
- Custom ship drawing: `drawPlane()` for player/small enemies, `drawBossShip()` for bosses
- Particle explosions scaled by quality level (adaptive perf)

**Game Loop (`update(dt)` → `render()`)**
1. Input polling (keyboard, joystick, mouse)
2. Player movement + collision clamping
3. Weapon cooldown + fire (adds bullets)
4. Enemy spawning + movement + weaving
5. Bullet movement (player and enemy)
6. PowerUp movement
7. Collision checks (bullets ↔ enemies, enemies ↔ player, etc.)
8. Boss update (movement, attack patterns, telegraphs)
9. Particle updates + lifetime decay
10. Canvas render: draw background gradient, all objects, HUD

**Stage & Progression**
- `KILLS_PER_STAGE = 18`: enemies killed before boss warning
- After 18 kills, boss spawns after 1.2s warning
- Boss destroyed → stage++, reset enemy spawn, show next stage theme/name
- Themes and bosses cycle through 6 options

**Scoring & Combos**
- `chain`: consecutive kills within 2.2 seconds
- `multiplier`: 1 + floor(chain/5), max 10× (at 50+ kills)
- Each kill adds `score += base * multiplier`
- Breaking chain resets multiplier to 1×

**Adaptive Performance**
- Tracks frame times (last 45 frames)
- If avg > 1/40 FPS, reduce quality (fewer particles, less shadow blur)
- Quality levels: 0 = full, 1 = medium, 2 = low
- Helps maintain 60 FPS on lower-end devices

#### **State Persistence (localStorage)**
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio on/off
- `skyfighter.difficulty`: Selected difficulty
- `skyfighter.reducedMotion`: Motion effects preference

#### **Boss Attack Patterns**
Each boss has 3 unique attack patterns (cycles through them):

- **VANGUARD**: Radial 360° burst → Aimed 5-way fan → Wide dual V-fans
- **BEHEMOTH**: Heavy downward columns → Slow homing spread → Dense 20-bullet radial wall
- **SPECTER**: Teleport + radial burst → Tight spiral → Dual aimed shots
- **DREADNOUGHT**: 8-way cross fire → Homing missile volley → Dense radial wall
- **WRAITH**: Teleport barrage + cross → Double expanding rings → Aimed cross-spreads
- **TITAN**: Heavy columns → Full radial ring (26 bullets) → Relentless homing (7 bullets)

Boss announces "telegraph" state before each attack, then fires. Telegraph can be interrupted by damage.

## Key Implementation Details

**Movement & Collision**
- Player: pixel-perfect clamping to canvas bounds
- Enemies: `weave` types sine-wave side-to-side while scrolling
- Bullets: removed when off-screen (margin of 20px)

**Weapon System**
- Vulcan (default): 1/2/3 parallel shots, slight spread
- Laser: 1/2/3 parallel beams, pierces multiple enemies
- Weapon power-ups toggle type OR upgrade level within type
- Fire rate: vulcan 0.15s, laser 0.1s (faster)

**Difficulty Scaling**
- Affects: spawn frequency, enemy speed, enemy fire chance, lives, bomb count, boss HP
- Enemy spawns grow denser during stage (every 5s of elapsed time adds difficulty)

**Accessibility**
- Keyboard + mouse + touch controls all fully functional
- Screen reader announcements for game events (boss name, stage, lives remaining, new best score)
- Reduced motion toggle disables animations (screen shake, flash, pulse) for motion-sensitive users
- High contrast UI for visibility

## Modifying the Game

### To adjust difficulty curves:
Edit `DIFFICULTIES` object (lines ~485–489). Keys: `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`.

### To add a new stage theme:
Add entry to `THEMES` array with `sky` (3-color gradient), `cloud`, `sun` RGB values, and `name`.

### To create new enemy types:
Add to `ENEMY_TYPES` with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`.

### To add boss attack patterns:
Define pattern function in `BOSS_PATTERNS[bossIndex]`. Use helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`.

### To change weapon behavior:
Modify `fireWeapon()` function (lines ~1124–1146). Adjust spread angles, pierce counts, or fire rates.

### To adjust visual feel:
- Screen shake: `screenShake(magnitude, duration)` in game event handlers
- Particles: `spawnExplosion(x, y, color, count)` scales with quality level
- HUD colors: CSS in `<style>` section (e.g., `#ffd54f` for gold text)

## Testing & Debugging

- **No build tools needed**: Just save and refresh the browser
- **Dev tools**: Use browser DevTools to inspect canvas, inspect localStorage, toggle breakpoints
- **Performance**: Check frame rate via adaptive quality output; lower `quality` variable means frame drops
- **Audio**: Browser console may show AudioContext errors if unsupported (gracefully falls back to muted)
- **Touch**: Desktop browsers can simulate touch via DevTools → Device Mode

## Browser-Specific Notes

- **Safari**: May need to resume AudioContext on user gesture (handled in code)
- **Mobile**: Full-viewport canvas with touch input; tested on iOS and Android
- **Screen readers**: ARIA live region updates for game events
- **Reduce motion**: Respects system preference; user can toggle via UI button

## File Size & Performance

- Single 59KB HTML file (compressed well)
- No external dependencies or CDNs
- No bundling or transpilation needed
- Lightweight: ~40KB minified JavaScript, ~15KB CSS
- Graphics rendered in real-time (no pre-baked assets)
- WebAudio synthesis (CPU-bound, not bandwidth)
