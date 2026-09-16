# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file web application. It features:
- Canvas-based 2D rendering at 480x720 resolution
- Multiple difficulty settings (Easy, Normal, Hard)
- 6 unique boss designs with distinct attack patterns
- 6 environmental themes with dynamic sky gradients
- WebAudio API synthesis for all sound effects (no external audio files)
- Touch and keyboard controls with mouse aiming support
- Particle effects, screen shake, and visual feedback
- Progressive difficulty through stage progression
- Local storage for persistence (best score, settings, accessibility preferences)

## Architecture & Code Structure

The entire game is contained in `index.html` (~1670 lines). The structure is:

1. **HTML Markup** (lines 1-420)
   - Canvas element for game rendering
   - HUD overlays (score, health, bombs, weapon status)
   - Boss health bar and stage banners
   - Title screen, pause screen, and game-over screen
   - Touch control buttons (for mobile)
   - Screen reader announcements area

2. **CSS Styling** (lines 10-358)
   - Responsive layout using CSS Grid and Flexbox
   - Dark theme with glassmorphic UI elements
   - Animations: screen shake, flash effects, combo popups, stage transitions
   - Mobile-first design with `clamp()` for fluid typography
   - Reduced-motion media query support for accessibility

3. **JavaScript Game Engine** (lines 422-1666)
   - Single IIFE to avoid global namespace pollution
   - Core game loop using `requestAnimationFrame`
   - Input handling (keyboard, mouse, touch with joystick)
   - Game state management (player, enemies, bullets, boss, particles)
   - Collision detection and entity lifecycle management
   - Audio synthesis and sound effect generation
   - LocalStorage persistence of settings and high scores

### Key State Objects

**Player**: `{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }`
- `weapon`: 'vulcan' (spray) or 'laser' (pierce-through)
- `level`: 1-3, upgrades affect spread pattern and fire rate
- `invuln`: invulnerability timer after taking damage

**Enemies**: Array of `{ x, y, w, h, speed, hp, type, pattern, weave, weaveSeed, baseX, ... }`
- Types: 'grunt' (1 hp), 'interceptor' (1 hp, fast), 'tank' (4 hp)
- `pattern`: 'spread' (radial bullet fire) or 'aimed' (targeted at player)
- `weave`: interceptors smoothly weave side-to-side
- Difficulty affects spawn rate and enemy behavior

**Boss**: `{ x, y, w, h, hp, maxHp, entering, phaseTimer, attackTimer, patternIndex, vx, design, telegraph }`
- Enters from top with deceleration, then moves horizontally
- `telegraph`: countdown timer before firing (22 frames)
- `patternIndex`: cycles through 3 attack patterns per boss design
- 6 boss designs with unique color schemes and HP multipliers

**Bullets**: `{ x, y, w, h, speed, vx, pierce, laser }`
- Player bullets: vulcan (4x12) or laser (5x20)
- Laser bullets pierce enemies (reduce pierce counter)
- Enemy bullets: small circles with radial movement
- Projectiles removed when off-screen

**PowerUps**: `{ x, y, w, h, speed, type }`
- Types: 'life' (6%), 'bomb' (8%), 'weapon' (20%)
- Spawn from defeated enemies, fall downward

**Particles**: Temporary visual effects (explosions, hits, pulses)

### Game Flow

1. **Initialization**: `resetGame()` sets up player at bottom-center, clears all entities
2. **Spawning Phase**: Enemies spawn in waves until `KILLS_PER_STAGE` (18) are destroyed
3. **Boss Phase**: After 18 kills, boss spawns with stage banner and warning
4. **Stage Progression**: Defeating boss increments stage, resets spawn timer, changes theme
5. **End Game**: Player dies → display final score and best score → wait for restart

### Difficulty System

```javascript
DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
};
```

Spawn timers decrease as stage progresses (frame-based ramp-up after 300 frames).

### Scoring & Multiplier

- Base scores: grunt (10), interceptor (15), tank (35), boss (500)
- Chain counter increments on each kill; breaks after 2.2 seconds
- Multiplier = 1 + floor(chain/5), capped at 10x
- Displayed as combo milestones every 10 kills

### Weapon System

**Vulcan** (spray): Multiple bullets in fan pattern
- Level 1: 1 bullet (center)
- Level 2: 2 bullets (±0.12 spread)
- Level 3: 3 bullets (±0.24 spread)
- Fire rate: 0.15s cooldown

**Laser** (pierce): Narrow penetrating beams
- Level 1: 1 beam (center), pierce=1
- Level 2: 2 beams (±6 offset), pierce=1
- Level 3: 2 beams, pierce=2
- Fire rate: 0.1s cooldown (faster)
- Visual indicator: blue aura around player when laser is active

### Boss Attack Patterns

Each of 6 bosses has 3 unique attack patterns that cycle:

- **VANGUARD**: Radial burst → Aimed fan → Wide V-spread
- **BEHEMOTH**: Heavy columns → Slow homing spread → Dense radial
- **SPECTER**: Teleport + radial → Spiral burst → Double aimed shots
- **DREADNOUGHT**: 8-way cross → Homing missiles → Dense radial wall
- **WRAITH**: Teleport + cross fire → Double expanding rings → Aimed cross-spread
- **TITAN**: Heavy columns → Full ring → Relentless homing volley

Patterns fire after 22-frame telegraph period; attack timer resets to 85 frames between attacks.

## Input Handling

### Desktop
- **Arrow keys / WASD**: Move player
- **Mouse move**: Optional steering (takes priority over keyboard, yields to keyboard/joystick input)
- **Mouse click / Space**: Fire
- **X / Shift**: Drop bomb
- **P / Esc**: Pause

### Mobile
- **Left 55% screen**: Drag to steer (joystick UI)
- **Right 45% screen**: FIRE and BOMB buttons

Input is debounced by weapon cooldown; multiple keypresses don't fire faster.

## Audio Synthesis

No external audio assets. All sounds generated via WebAudio API:

- **Player fire**: Frequency sweep (760→480 Hz), short duration, slight low-pass filter
- **Laser fire**: High-frequency sawtooth sweep (1500→2400 Hz)
- **Explosions**: Enemy-type-specific multi-part beeps + noise bursts
- **Boss hit**: Distinct tone with filtering
- **Bomb**: Heavy bass explosion + noise
- **Power-up**: Ascending 3-note chime
- **UI clicks**: Brief square wave tone

Panning is applied based on bullet/enemy horizontal position for spatial audio.

## Rendering Pipeline

1. **Screen shake**: Apply random translation if `shakeTime > 0`
2. **Background**: Sky gradient + radial sun glow (theme-dependent)
3. **Clouds**: Two layered cloud passes at different speeds
4. **Player**: Plane drawing with optional flame and laser aura
5. **Bullets**: Player bullets (with glow if quality=0)
6. **Enemy projectiles**: Circles with radial movement
7. **Enemies**: Plane drawing; tanks show HP bar above
8. **Boss**: Complex multi-part ship drawing with animated core
9. **Power-ups**: Rotating squares with theme colors
10. **Particles**: Explosion debris with life-based alpha fade

### Performance Optimization

Adaptive quality system (`quality` variable):
- Level 0 (full): All shadow blurs, smooth animations
- Level 1 (medium): 60% particle count, reduced shadow blur
- Level 2 (low): 35% particle count, no shadows

Quality adjusts automatically based on average frame time over 45 frames. Target is 60 FPS; if average > 1/40s, quality degrades.

## Accessibility

- **Reduced Motion**: Toggle via button (🎆 icon); disables animations and screen shake
- **Mute**: 🔊/🔇 button persists in localStorage
- **Screen Reader**: Arena for live region announcements (`#srLive`)
- **Pause**: Full-screen overlay for game state suspension
- **Color Contrast**: White text with black shadow for readability

LocalStorage keys for preferences:
- `skyfighter.bestScore`: Best score
- `skyfighter.muted`: Sound mute state
- `skyfighter.difficulty`: Chosen difficulty
- `skyfighter.reducedMotion`: Reduced motion preference

## Modding & Customization Points

### Easy Changes
- **Enemy spawn rates**: Adjust `DIFFICULTIES` values
- **Weapon balance**: Modify `fireWeapon()` function (spread angles, fire rate)
- **Boss patterns**: Edit `BOSS_PATTERNS` array (uses helper functions like `fireRadial()`, `fireAimedSpread()`)
- **Color schemes**: Edit `THEMES` array (sky gradient, cloud, sun colors)
- **Scoring multiplier**: Change `chainTimer`, `chain` increment rate in `addScore()`

### Medium Complexity
- **New enemy type**: Add to `ENEMY_TYPES`, update spawn logic in `spawnEnemy()`, add explosion sound in `EXPLODE_SFX`
- **New power-up type**: Add to `maybeSpawnPowerUp()`, handle in `applyPowerUp()`
- **Boss AI phases**: Add new pattern functions and attach to `BOSS_PATTERNS` entries

### High Complexity
- **New rendering**: Modify `draw()` function (may require understanding canvas context state management)
- **New input method**: Extend input handling block (keyboard, mouse, touch)
- **Persistent data beyond localStorage**: Requires backend service (game is currently client-only)

## Testing & Debugging

### Manual Testing Checklist
- [ ] Game starts, HUD displays score=0
- [ ] Enemies spawn and move downward
- [ ] Player shoots continuously when Space held
- [ ] Enemy bullets damage player; invulnerability works (blinking)
- [ ] Bomb clears all enemies and damages boss
- [ ] Boss appears after 18 kills, warning message displays
- [ ] Boss health bar updates during combat
- [ ] Difficulty buttons change spawn rate and enemy behavior
- [ ] Sound mutes/unmutes correctly
- [ ] Reduced motion toggle removes animations
- [ ] Pause/resume works (overlay appears, game state frozen)
- [ ] Game Over shows final score and best score
- [ ] Best score persists after page reload
- [ ] Touch controls work on mobile (drag left, tap FIRE/BOMB)

### Performance Testing
- Run on low-end device or throttle CPU in DevTools
- Quality should auto-degrade if FPS drops below ~25
- Particle count should reduce at lower quality levels

### Browser Compatibility
- Chrome 60+, Firefox 55+, Safari 12+ (WebAudio, Canvas, ES6)
- Mobile browsers with touch event support (iOS Safari 12+, Android Chrome)
- Fallback for absent `AudioContext`: `window.webkitAudioContext` included

## Common Issues & Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| No sound | `AudioContext` not resumed | Audio only plays after user interaction (click, touch). `ensureAudio()` resumes suspended context. |
| Enemies spawn too fast/slow | Difficulty not saved to localStorage | Clear browser storage or reset with DevTools; difficulty button should reflect `localStorage.skyfighter.difficulty`. |
| Game runs at low FPS | Quality not adapting | Check DevTools performance profile; quality ramps down after ~10 frames of avg > 1/40s. |
| Touch controls not appearing | Device detection failed | Verify `('ontouchstart' in window)` or `navigator.maxTouchPoints > 0`; add console logging if needed. |
| Boss doesn't fire pattern | Pattern index out of bounds | Ensure `BOSS_PATTERNS` has entry for all 6 bosses; check `patternIndex % patterns.length` calculation. |

## Future Enhancement Ideas

1. **Leaderboard**: Store top 10 scores with timestamps
2. **Endless Mode**: Skip boss fights, escalating difficulty
3. **Weapon Attachments**: Homing missiles, shield, time-warp
4. **Enemy Spawner Tool**: Drag-and-drop level editor for custom enemy waves
5. **Replays**: Record and playback player input sequences
6. **Procedural Boss Design**: Randomize boss visuals and attacks
7. **Soundtrack**: Replace/complement sound synthesis with music loops
8. **Mobile App**: Wrap in Electron or Capacitor for native distribution
