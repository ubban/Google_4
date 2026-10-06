# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game, built as a single-file HTML5 application with no external dependencies. The game features:
- Multiple difficulty levels (Easy, Normal, Hard)
- Progressive stages with themed backdrops
- Boss battles with unique attack patterns
- Weapon upgrade system (Vulcan/Laser with 3 levels)
- Dynamic score multiplier based on kill chains
- Touch/mobile support with joystick controls
- Procedurally-generated sky themes

The entire game logic (rendering, input, physics, AI) is implemented in vanilla JavaScript using the HTML5 Canvas API and WebAudio API.

## File Structure

- **index.html** — Single file containing all HTML, CSS, and JavaScript for the game

## Development & Running

**No build step required.** Open `index.html` directly in a browser to play:
- Desktop: Open in Firefox, Chrome, Safari, or Edge
- Mobile: Works on iOS and Android with touch controls

To iterate on the game:
1. Edit `index.html` in any text editor
2. Save and refresh the browser
3. Persistence uses `localStorage` for high scores, difficulty, and audio/motion preferences

## Code Architecture

The game is organized as a single IIFE (Immediately-Invoked Function Expression) with several key systems:

### 1. **Game State Management**
- Core variables: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Game flags: `running`, `paused`, `gameOver`, `score`, `lives`, `stage`
- Chain system: `chain`, `chainTimer`, `multiplier` — tracks consecutive kills for bonus multiplier

### 2. **Input Handling**
Three input methods, merged into a single movement vector:
- **Keyboard**: Arrow keys or WASD for movement, Space to fire, X/Shift/P for bomb/pause
- **Mouse**: Movement steers player toward cursor; click to fire (when hovering canvas)
- **Touch**: Left 55% is joystick zone, right 45% has FIRE and BOMB buttons

Priority: Keyboard/joystick > Mouse (mouse only active if no keyboard input)

### 3. **Game Loop**
`requestAnimationFrame` at ~60 FPS:
1. **update(dt)** — Physics, collision, spawning, state changes (runs only if not paused)
2. **draw()** — Canvas rendering with optional screen shake

Adaptive performance tracking: If average frame time drops below 1/56s, quality level increases (reduces particle counts, glow effects).

### 4. **Entity Systems**

#### Player
- Bounding rect collision with enemies and bullets
- Weapon system: Vulcan (spread pattern) vs Laser (piercing beams)
- Invulnerability timer after being hit
- Fire cooldown differs by weapon (Laser fires faster)

#### Enemies
- **Grunt**: Basic red ship, HP=1, moderate speed
- **Interceptor**: Orange ship, faster, spawns in stage 2+
- **Tank**: Purple, highest HP (4), spawns in stage 2+, displays health bar
- Enemies have patterns: "spread" fire (3-way spread) vs "aimed" (homing to player)
- Weaving interceptors oscillate left/right while moving down

#### Bosses
6 designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), each with:
- Unique hull/core colors and HP multipliers
- 3 distinct attack patterns (telegraphed, then fire)
- Slow horizontal patrol with bounds checking
- Telegraph phase (22 frames) before each attack

#### Bullets & Projectiles
- Player bullets: move upward, hit-scan vs enemies/boss
- Enemy bullets: move downward, radial or aimed patterns
- Laser bullets pierce multiple targets (pierce counter)
- Bomb clears all enemy bullets and deals AoE damage

#### Power-ups
- Life (6% spawn chance): Restore 1 HP
- Bomb (8% spawn chance): Restore 1 bomb
- Weapon (20% spawn chance): Level up weapon or switch weapons

### 5. **Audio System**
WebAudio synth-based SFX (no external audio files):
- `beep()` — Oscillator with optional frequency sweep and filter
- `noiseBurst()` — Procedural noise with filter and pan
- `sfx` object maps game events to sound functions (shoot, bomb, hit, boss patterns, etc.)
- Mute state persists in `localStorage`

### 6. **Difficulty & Progression**
**DIFFICULTIES** object defines spawn rates, enemy speed, shoot chance, and HP multipliers:
- Easy: Slower spawns, safer enemies, 4 lives, 3 bombs
- Normal: Balanced defaults
- Hard: Rapid spawns, aggressive AI, reduced resources, boss HP +30%

**Stages**: Every 18 enemy kills triggers boss battle. 6 boss designs cycle. Sky theme changes per stage.

### 7. **Collision Detection**
`rectsOverlap()` — AABB (axis-aligned bounding box) check. Used for:
- Player vs enemies / enemy bullets / boss
- Player bullets vs enemies / boss
- Power-up pickup

### 8. **Visual Effects**
- **Screen shake**: `screenShake(magnitude, duration)` — applies random offset to canvas
- **Flash**: Brief white/colored overlay fade for hits and bomb explosions
- **Vignette**: Red pulsing overlay when player has 1 HP (critical state)
- **Combo pop**: Animated text showing kill milestone (every 10 kills)
- **Stage banner**: Shows stage name/warning, fades in/out
- All effects respect `reducedMotion` preference (CSS animations disabled or throttled)

### 9. **Canvas & Responsive Design**
- Game canvas: 480×720 (portrait)
- Scales to fit window (max 1.6× or fit within viewport)
- Mobile-friendly: Safe-area insets, touch-optimized HUD

## Key Constants & Configuration

| Constant | Purpose |
|----------|---------|
| `BASE_W`, `BASE_H` | Canvas logical size (480×720) |
| `KILLS_PER_STAGE` | Kills needed to trigger boss (18) |
| `DIFFICULTIES` | Per-difficulty spawn/speed/HP settings |
| `THEMES` | 6 sky gradients and color palettes |
| `ENEMY_TYPES` | Grunt, Interceptor, Tank definitions |
| `BOSSES` | 6 boss designs with patterns |
| `BOSS_PATTERNS` | Attack pattern functions per boss |
| `STORAGE_*` | localStorage keys for persistence |

## Common Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Add spawn logic in `spawnEnemy()` (probability check)
3. Add explosion SFX in `EXPLODE_SFX` (optional)
4. Collision/death handling is automatic

### Adding a New Boss Pattern
1. Create pattern function in `BOSS_PATTERNS` array (6 functions, one per boss design)
2. Pattern function signature: `(boss) => { /* fire bullets */ }`
3. Use helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`
4. Bullets pushed to global `enemyBullets` array

### Tweaking Game Balance
- Enemy spawn rates: Adjust `DIFFICULTIES` → `spawnBase`, `spawnMin`
- Enemy aggression: Adjust `enemyShootChance` and bullet speeds in pattern functions
- Player survivability: Adjust `lives`, `bombs` per difficulty
- Boss difficulty: Adjust `bossHpMul` per difficulty or individual boss `hpMul`

### Adding Sound Effects
1. Create function in `sfx` object using `beep()` or `noiseBurst()`
2. Call `ensureAudio()` to initialize WebAudio context (on first user interaction)
3. Check `muted` flag before playing

## Performance Considerations

- **Particle count** scales with quality level (full → medium → low)
- **Shadow blur** and glow effects disabled at lower quality
- Frame time is tracked; quality auto-adjusts if FPS drops
- Enemy count grows over time (spawn rate accelerates); no explicit cap

## Accessibility

- **Keyboard support**: Full keyboard/arrow controls without mouse
- **Screen reader support**: `aria-live` region for game announcements
- **Reduced motion**: Respects `prefers-reduced-motion` media query; disables animations on toggle
- **Mute button**: Quick toggle for audio
- **HUD contrast**: Text shadows and backdrop blur for readability

## Testing & Debugging

No automated tests. Manual testing:
- Difficulty levels: Easy, Normal, Hard
- Input methods: Keyboard, mouse, touch
- Boss transitions and attack patterns
- Mobile responsiveness and touch control feel
- Audio on different browsers (iOS Safari has autoplay restrictions)

## Browser Compatibility

Requires:
- Canvas 2D context
- WebAudio API (or webkit variant)
- localStorage
- requestAnimationFrame
- Touch Events (for mobile)

Tested: Chrome, Firefox, Safari, Edge on desktop; mobile browsers on iOS/Android.
