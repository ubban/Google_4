# Sky Fighter — Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game built as a single-file HTML/JavaScript application. Players control a fighter aircraft, dodge enemies, collect weapon power-ups, and battle stage bosses across multiple difficulty levels.

### Core Features
- **Three Difficulty Levels**: Easy, Normal, Hard with progressively faster spawning and stronger enemies
- **Dynamic Weapon System**: Vulcan, Spiral, and Laser weapons with upgrade levels
- **Boss Encounters**: Six unique boss types with distinct visual patterns and attack behaviors
- **Scoring System**: Combo multiplier and high score tracking via localStorage
- **Accessibility**: Screen reader announcements, reduced motion preferences, color contrast
- **Mobile Support**: Touch controls with virtual joystick and action buttons
- **Adaptive Performance**: Dynamic quality scaling based on frame time averages
- **Procedural Audio**: WebAudio synthesized sound effects and music (no external asset files)
- **Visual Themes**: Six sky/background color schemes that change per stage

---

## Architecture

### Single-File Structure
All code (HTML, CSS, JavaScript) is in `index.html` (~1400+ lines). The game runs entirely in the browser without external dependencies.

### JavaScript Organization
The code uses a self-contained IIFE with tightly scoped variables. Key systems are organized chronologically:

1. **Canvas & DOM Setup** (lines ~426–462)
2. **Performance Tracking** (lines ~473–483)
3. **Constants & Configuration** (lines ~485–523)
   - Difficulty presets
   - Enemy type definitions
   - Boss specifications
   - Visual themes
4. **Audio System** (lines ~542–700+)
5. **Input Handling** (keyboard, mouse, touch)
6. **Game State & Logic**
   - Player object and mechanics
   - Enemy spawning and behavior
   - Collision detection
   - Scoring and multiplier
7. **Rendering Pipeline** (canvas drawing)
8. **Main Game Loop** (requestAnimationFrame)

---

## Key Data Structures

### Game Difficulty (`DIFFICULTIES`)
```javascript
{
  easy/normal/hard: {
    spawnBase,      // frames between spawns (lower = faster)
    spawnMin,       // minimum spawn interval
    enemySpeed,     // speed multiplier for enemies
    enemyShootChance,
    lives,          // player starting health
    bombs,          // starting smart bomb count
    bossHpMul       // health multiplier for bosses
  }
}
```

### Enemy Types (`ENEMY_TYPES`)
- **grunt**: Red, slow, 1 HP, score 10
- **interceptor**: Orange, fast (1.7× speed), 1 HP, score 15, weaving behavior
- **tank**: Purple, slow, 4 HP, score 35 (appears stage 2+)

### Boss Patterns (`BOSSES`)
Six boss types: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Each has unique hull colors, wing colors, core accent, and HP multiplier
- Visual rendering uses color properties (hullTop, hullBottom, wing, core)
- Boss type selected by `stage % BOSSES.length`

### Visual Themes (`THEMES`)
Six sky gradient themes with atmospheric colors. Each theme has:
- `sky`: Array of [top, middle, bottom] gradient colors
- `cloud`: Cloud/smoke particle color (RGB string)
- `sun`: Sun/glow color (RGB string)
- `name`: Human-readable theme name

---

## Game Loop & State Management

### Player Object
Central mutable state tracking:
- Position (x, y)
- Health (lives)
- Weapon type and level
- Bomb count and cooldown
- Score and multiplier
- Animation/visual flags

### Stage Progression
- Players must eliminate `KILLS_PER_STAGE` (18) enemies to advance
- At stage end, a unique boss spawns
- Theme and enemy types scale with stage number

### Spawn System
- Enemy spawn timer decrements each frame
- When timer ≤ 0, new enemy spawns at random horizontal position
- Spawn interval varies by difficulty; minimum spawn rate prevents overwhelming
- Tank enemies only appear from stage 2+

### Collision Detection
- Simple bounding box checks for:
  - Player bullets hitting enemies
  - Enemies hitting player
  - Bomb explosions affecting enemies/bosses
- No spatial hash; direct array iteration (feasible for ~50–100 entities)

### Scoring & Multiplier
- Base enemy score varies by type (10/15/35)
- Score multiplier increases on consecutive hits, resets on miss
- Boss defeat rewards bonus points
- High score persists in localStorage

---

## Visual & Audio Systems

### Canvas Rendering
- Clears canvas each frame and redraws all entities
- Enemies/bosses drawn as geometric shapes (rectangles, lines) in 2D canvas
- Player drawn as triangle pointing upward
- No sprite sheets or images; purely procedural rendering

### Visual Effects
- **Vignette**: Red overlay pulse when player takes damage
- **Flash**: Full-screen white flash on hit or bomb detonation
- **Combo Pop**: Animated text showing multiplier milestone
- **Stage Banner**: Title text announcing new stage/boss
- **Boss Bar**: Health bar for active boss enemy

### Audio System (WebAudio)
- `beep(freq, dur, type, vol, opts)`: Synth tone with optional sweep, filter, panning
- `noiseBurst(dur, vol, opts)`: White noise with envelope for impact sounds
- All audio is synthesized; no pre-recorded files
- Master gain and dynamics compressor for audio control
- Can be muted via UI button and localStorage preference

### Touch Controls
- Left 55% of screen: Move zone (drag to steer)
- Right 45% of screen: Action zone with FIRE and BOMB buttons
- Virtual joystick base appears on first touch drag
- Touch-specific UI disabled on desktop devices

---

## Input Handling

### Keyboard Controls
- **Arrow Keys / WASD**: Move player
- **Space**: Fire weapon
- **X / Shift**: Detonate bomb
- **P / Esc**: Pause game
- **Mouse**: Move to position, click to fire

### Mouse & Touch Events
- Mouse move controls player position (aim crosshair cursor)
- Mouse click / touch fire events
- Touch drag for mobile movement (virtual joystick)
- Prevents default touch behaviors (no scroll, no context menu)

---

## Performance Considerations

### Adaptive Quality
- Tracks frame times over 45-frame window
- If average frame time > 1/40 (25ms), degrade quality (quality++)
- If average frame time < 1/56 (~17.8ms), improve quality (quality--)
- Quality level (0=full, 1=medium, 2=low) affects:
  - Particle count
  - Effect rendering frequency
  - Enemy draw detail

### Optimization Notes
- No spatial indexing; O(n²) worst-case collision checks
- Reusable canvas context (no layer compositing)
- localStorage used for persistence (best-score, mute state, etc.)
- All assets are procedurally generated (no file I/O except page load)

---

## Local Storage Keys

```javascript
'skyfighter.bestScore'    // Best score achieved
'skyfighter.muted'         // Audio mute state (0/1)
'skyfighter.difficulty'    // Last selected difficulty
'skyfighter.reducedMotion' // Accessibility: reduce motion flag
```

---

## Common Modifications & Extension Points

### Adding a New Weapon Type
1. Define new weapon in constants section with name, damage, fire rate
2. Add weapon selection logic in weapon-cycling UI handler
3. Update bullet rendering and collision for new projectile shape
4. Update weapon name/level display in HUD

### Adding a New Boss
1. Add entry to `BOSSES` array with name, colors (hullTop, hullBottom, wing, core), hpMul
2. Boss selection uses modulo (stage % BOSSES.length), so new boss appears in rotation
3. Boss drawing code reads these color properties; no new render logic needed

### Adjusting Difficulty
Modify `DIFFICULTIES` object:
- Lower `spawnBase`/`spawnMin` → faster enemy spawning
- Increase `enemySpeed` → faster enemies
- Increase `enemyShootChance` → more enemy bullets
- Adjust `lives`/`bombs` for player starting resources
- Scale `bossHpMul` for boss health

### Adding Visual Themes
1. Add new theme object to `THEMES` with sky, cloud, sun, and name
2. Theme is selected via `stage % THEMES.length`
3. Sky gradient is applied directly to canvas background

---

## Accessibility Features

- **Screen Reader Support**: Announcements via `srLive` ARIA live region
- **Reduced Motion**: CSS animations disabled when `prefers-reduced-motion` is set
- **Color Contrast**: UI text uses shadows and contrasting backgrounds
- **Keyboard-Only Play**: Fully playable with keyboard (no mouse required)
- **Title & Subtitle**: Descriptive game name and objective in UI
- **Button Labels**: Clear titles on control buttons (Mute, Motion, Pause)

---

## CSS Architecture

- **Responsive Layout**: Uses `clamp()` for scalable font and spacing
- **Safe Area Insets**: Accounts for mobile notches/bottom bars via `env(safe-area-inset-*)`
- **Dark Theme**: `:root { color-scheme: dark }` for system-wide dark mode
- **Animations**: Keyframes for vignette pulse, combo pop, stage banner, icon bob, etc.
- **Reduced Motion**: All animations respect `@media (prefers-reduced-motion: reduce)`
- **Mobile First**: Touch-optimized controls, adaptive UI sizing

---

## Testing Checklist for Claude Agents

When modifying this game, verify:
- [ ] Game starts and pauses correctly
- [ ] Player moves with keyboard/mouse and doesn't get stuck
- [ ] Enemies spawn and scroll down at expected rate for difficulty
- [ ] Collision detection triggers on enemy hit (vignette + damage)
- [ ] Weapon fire works and damages enemies
- [ ] Score updates and multiplier increases on consecutive hits
- [ ] Boss appears after stage enemies defeated; boss health bar shows
- [ ] Bomb detonation effect visible and damages/kills nearby enemies
- [ ] Audio synthesizes and mute button toggles it
- [ ] Reduced motion preference disables animations
- [ ] Touch controls work on mobile (joystick, FIRE, BOMB buttons)
- [ ] High score persists after refresh
- [ ] Difficulty selection carries over to next game
- [ ] No console errors or performance warnings
- [ ] Canvas scales correctly on different screen sizes

---

## Common Pitfalls

1. **Collision Detection**: Enemies and projectiles are tracked separately; ensure collision checks iterate both arrays
2. **Asset Dependencies**: No external images or audio files—everything is synthesized. Don't add image paths without ensuring fallbacks
3. **Touch Event Handling**: Mobile devices don't fire `mousemove`; use `touchmove` and coordinate conversion for move zone
4. **Canvas Coordinate System**: (0,0) is top-left; y increases downward. Player starts near bottom
5. **Frame Rate**: Game assumes ~60 FPS; delta time adjustments needed if frame pacing changes
6. **Boss Health Bar**: Shown only when boss is alive; position must account for safe-area insets on mobile

---

## Future Enhancement Ideas

- Particle effects for explosions (currently just vignette/flash)
- Enemy formation patterns and wave strategies
- Leaderboard or replay recording (would require new storage format)
- Power-up system (weapon upgrades, shields, slow time)
- Mobile gyroscope aiming
- Gesture shortcuts (swipe for bomb, double-tap for pause)
- Procedural enemy/boss generation based on stage seed
