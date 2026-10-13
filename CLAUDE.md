# Sky Fighter — Game Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 Canvas application. The game features multiple difficulty levels, dynamic boss encounters, weapon progression, and adaptive performance tuning.

## Architecture

The entire game runs in `/index.html` as a self-contained JavaScript application with no external dependencies. The structure follows a classic game loop pattern:

- **Canvas rendering**: 480x720 base resolution, responsive scaling up to 1.6x
- **Game loop**: `requestAnimationFrame` at variable delta-time step (capped at 50ms)
- **Input handling**: Keyboard, mouse, and touch (with joystick visualization on mobile)
- **Audio**: Web Audio API synthesis with procedurally generated sound effects

## Key Game Systems

### Difficulty Levels
- **Easy**: Lower spawn rates, slower enemies, weaker bosses, 4 lives
- **Normal**: Balanced experience, 3 lives (default)
- **Hard**: High spawn rates, fast enemies, strong bosses, 2 lives

### Player Mechanics
- **Weapons**: VULCAN (spread shots) and LASER (piercing shots), each with 3 upgrade levels
- **Movement**: WASD/Arrows or mouse steering (mouse takes priority if active, keyboard overrides)
- **Bombs**: Limited-use full-screen damage (keyboard: X or Shift; touch: BOMB button)
- **Invulnerability**: 1.5s after hit
- **Lives and hearts**: Display updates via HUD

### Enemy Types
1. **Grunt** (stage 1+): Standard enemy, single HP
2. **Interceptor** (stage 1+): Fast, weaving pattern, single HP
3. **Tank** (stage 2+): Heavy armor, 4 HP, slow, high score

### Boss System
- **Six boss designs** cycle through stages, each with unique attack patterns
- **Boss patterns**: Radial bursts, aimed spreads, columns, teleporting attacks, etc.
- **Telegraph phase**: Visual warning (white glow) before attack sequence
- **Health bar**: Displays at top center during boss fights
- **Stage progression**: Each defeated boss advances to the next stage

### Scoring & Multiplier
- **Chain system**: Consecutive kills without missing extend multiplier (max 10x at 50+ kills)
- **Chain timer**: 2.2 seconds between kills to maintain multiplier
- **Boss defeat bonus**: 500 base points

### Performance Adaptation
- **Quality levels**: 0 (full), 1 (medium), 2 (low)
- **Tracked metrics**: Frame times over 45 frames; adjusts if avg >16.67ms or <17.86ms
- **Effects reduced at low quality**: Particle counts, shadows, glow

## Visual System

### Themes
Six procedurally varied sky themes cycle per stage (daylight, sunset, night, storm, arctic, volcanic), each with:
- Sky gradient (3 color stops)
- Cloud layers (2 speeds)
- Sun glow

### UI Elements
- **HUD**: Score, best score, multiplier, weapon level, hearts, bomb count (top center)
- **Boss bar**: Boss name and health fill bar (top during boss fight)
- **Overlay**: Menu, stats, difficulty selector
- **Combo pop**: Large text showing 10-kill milestones
- **Stage banner**: Displays stage transitions
- **Flash effects**: White screen flash on damage/bomb; vignette pulse when critical
- **Reduced motion mode**: Accessibility toggle disables animations/shakes

### Input Methods
- **Keyboard**: WASD or arrows for movement, Space for fire, X/Shift for bomb, P/Esc for pause
- **Mouse**: Move to steer, click to fire
- **Touch**: Left zone (55% width) for movement joystick, right zone (45% width) for FIRE/BOMB buttons

## Storage & Persistence

Uses `localStorage` with four keys:
- `skyfighter.bestScore`: Best score (number)
- `skyfighter.muted`: Mute state (0 or 1)
- `skyfighter.difficulty`: Selected difficulty (string: easy/normal/hard)
- `skyfighter.reducedMotion`: Accessibility toggle (0 or 1)

## Audio Design

All audio is synthesized using Web Audio API (no asset files):
- **Basic sounds**: `beep()` for tonal effects (square/sine/sawtooth), `noiseBurst()` for impact
- **Sound effects**: `sfx` object with ~15 distinct sounds (shoot, laser, explosions, powerup, boss telegraph, etc.)
- **Volume management**: `masterGain` node with dynamics compressor; individual gains per sound
- **Panning**: Spatial audio based on screen position (weapons use `panFor()`)

## Game States

- **not running**: Initial overlay with difficulty selection and start button
- **running && !paused**: Active gameplay
- **running && paused**: Pause overlay
- **gameOver**: Final stats and replay prompt
- **boss.entering**: Boss descending into view (2s animation)

## Collision Detection

Simple `rectsOverlap()` function checks AABB collisions for:
- Player bullets ↔ enemies
- Player bullets ↔ boss
- Enemy bullets ↔ player
- Enemies ↔ player
- Power-ups ↔ player
- Boss ↔ player

## Important Implementation Notes

1. **Boss patterns** are stored in `BOSS_PATTERNS` array (6 sets, one per boss design)
2. **Spawn rates** decrease dynamically as frame count increases (difficulty ramp every 300 frames)
3. **Stage progression**: After 18 kills (KILLS_PER_STAGE), boss warning triggers, boss spawns after 1.2s
4. **Particle lifetimes**: All particles fade over 0.5s
5. **Teleporting enemies**: Interceptors weave using sine wave offset based on frame
6. **Weapon cooldown**: Vulcan 0.15s, laser 0.1s (faster fire rate)
7. **Screen shake**: Triggered on damage, bomb use, boss defeat; reduced if motion accessibility is on

## Performance Considerations

- Particle count scales with quality setting (full quality: ~14 per explosion, low quality: ~5)
- Shadow effects only render at full quality
- Canvas shadow/blur filters disabled at lower quality
- Glow effects (`shadowBlur`) only on full quality
- Enemy projectiles use optional `radial` properties for parametric movement (calculated per frame)

## Development Tips

- **Testing difficult scenarios**: Adjust `DIFFICULTIES` to quickly test, use browser console to set `lives`, `bombCount`, etc.
- **Balancing boss difficulty**: Each boss has `hpMul` factor; tweak `baseHp` calculation
- **New weapon ideas**: Add to `fireWeapon()` and weapon cycle logic in `applyPowerUp()`
- **Custom themes**: Push new object to `THEMES` array
- **Boss patterns**: Add pattern functions and new array to `BOSS_PATTERNS`

## Browser Compatibility

- Requires: Canvas 2D context, Web Audio API, `localStorage`, `requestAnimationFrame`
- Optional: Touch API for mobile, `matchMedia` for reduced-motion detection
- Tested: Modern browsers (Chrome, Firefox, Safari, Edge)

## Accessibility Features

- Screen reader support via `#srLive` (ARIA live region)
- Reduced motion option (disables shake, flash, animations)
- Clear keyboard controls without reliance on mouse
- Audio mute toggle
- High contrast UI (light text on dark background)
