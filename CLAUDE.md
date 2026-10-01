# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a browser-based Raiden-style vertical shoot-'em-up game built as a single-file HTML application. Players control a fighter aircraft, switch between weapons, drop smart bombs, and defeat stage bosses while managing health and score multipliers.

**Key Technology**: Vanilla JavaScript (ES6+), HTML5 Canvas 2D rendering, Web Audio API for procedural sound synthesis. No external dependencies.

## Architecture

### File Structure
```
Google_4/
├── index.html          # Complete game (1669 lines)
├── README.md           # Project info
└── CLAUDE.md           # This file
```

### Codebase Organization (index.html)

**HTML Structure** (Lines 1–421)
- Canvas element for game rendering
- HUD overlay with score, health (hearts), bombs, weapon status
- Game overlay for menus (start, pause, end screens)
- Stage banner and combo notification elements
- Touch controls for mobile (joystick, fire button, bomb button)
- Boss health bar
- Difficulty buttons (Easy, Normal, Hard)
- Mute and motion-reduction toggle buttons
- Screen reader announcements (sr-live)

**CSS Styling** (Embedded in `<style>` tags)
- Dark theme (`#05122b` background)
- Responsive layout using `clamp()` for font sizes and spacing
- Canvas gradient background (sky blue to light blue)
- HUD stats badges with semi-transparent glass-morphism effect
- Animation keyframes: `heartBeat`, `flashHit`, `flashBomb`, `vignettePulse`, `screenShake`
- Reduced-motion support (respects `prefers-reduced-motion` media query)
- Touch-friendly controls sized for mobile

**JavaScript Game Logic** (Lines 422–1669)
- Wrapped in IIFE with strict mode
- Modular structure with logical sections

## Game Systems

### Core Configuration (Lines 463–516)

**Difficulty Levels**
```javascript
DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, ... },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, ... },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, ... }
}
```
- Spawn rates, enemy speed, shooting chance, lives, bombs, boss HP multiplier

**Visual Themes** (6 themes: Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge)
- Sky gradient colors, cloud color, sun color, theme name

**Enemy Types**
- `grunt`: Basic enemy (34×34px, 1 HP)
- `interceptor`: Fast enemy (26×26px, 1 HP, 1.7× speed)
- `tank`: Heavy armor (46×46px, 4 HP, 0.55× speed)

**Bosses** (6 bosses with unique artwork colors and HP multipliers)
- VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN

### Audio System (Lines 542–650)

**Web Audio Synthesis** (no external audio files)
- `beep(freq, dur, type, vol, opts)`: Tone-based sounds with frequency sweep and filters
- `noiseBurst(dur, vol, opts)`: Noise-based sounds (explosions, hits)
- Features: frequency modulation, filtering, panning, envelope control
- Master gain and compressor for dynamic range

**Sound Cues**
- Weapon fire, hits, explosions, boss appear/defeat, combo rewards
- Mute toggle stored in localStorage

### Input Handling

**Keyboard**
- Arrow keys / WASD for movement
- Space for primary weapon
- X or Z for bomb

**Mouse**
- Click to position player
- Crosshair cursor

**Touch** (Lines 1240–1320)
- Virtual joystick (deadzone-based)
- Separate fire and bomb buttons
- Responsive to device motion (gyro aiming optional)

### Game State & Constants

**Game Loop**
- 60 FPS target (adaptive quality scaling for performance)
- Delta-time based updates
- Three-stage rendering: background, entities, foreground effects

**Difficulty & Progression**
- 18 kills per stage (`KILLS_PER_STAGE`)
- Difficulty affects spawn rates, enemy behavior, boss HP
- Best score persisted in localStorage

**Performance Optimization**
- Adaptive quality levels (full / medium / low) based on frame time
- Pooling/reuse of projectiles and effects
- Canvas clipping for off-screen culling

## Key Game Systems

### Combat & Scoring (Lines 1350+)

**Weapons**
- Primary weapon (increases fire rate with kills)
- Smart bombs (area effect, limited uses)
- Score multiplier increases per kill (resets on death)

**Health & Lives**
- Heart system (3–4 lives depending on difficulty)
- Visual heart display (grayed out when lost)
- Critical animation when near death

**Scoring**
- Base score per enemy type (grunt: 10, interceptor: 15, tank: 35)
- Multiplier bonus (increases after consecutive kills)
- Best score tracking (localStorage)
- Stage completion bonus

### Player Ship

**Movement**
- Collision detection with canvas boundaries
- Speed capped to prevent out-of-bounds
- Hitbox for collision detection with enemy projectiles

**Weapon System**
- Firing state management
- Damage per shot (progressive with weapon level)
- Fire rate control

### Enemy System

**Spawning**
- Wave-based with varying spawn intervals
- Difficulty-based spawn rates
- Random enemy type selection (based on minimum stage requirement)

**Behavior**
- Linear downward movement
- Weaving path for interceptors
- Shooting pattern (chance-based)
- Death with visual feedback (flash, vignette, score pop)

### Boss Encounters

**Boss Mechanics**
- Unique artwork rendering (hull, wings, core)
- Large hitbox, multiple hit points
- Boss-specific attack patterns
- Health bar display
- Difficulty-scaled HP

**Boss Battle Transition**
- Stage banner announcement
- Boss spawn animation
- Health bar appears
- Changed music (procedural tone)

### Visual Effects

**Canvas Rendering**
- Parallax scrolling background (clouds)
- Sprite rendering for player, enemies, projectiles
- Particle effects (explosions, hits)
- Screen shake on damage (reduced-motion respects setting)
- Color flash on damage (white vignette on critical)
- Combo pop-ups (floating "+X" text)

**UI Overlays**
- HUD: Score, best score, health, bombs, weapon status
- Stage banner
- Pause screen
- Game over screen (final score, best score, stage reached, new best indicator)

## Development Guidelines

### Adding New Content

**New Enemy Type**
1. Add to `ENEMY_TYPES` with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Add rendering in enemy draw section
3. Add to enemy spawn pool with minimum stage requirement

**New Boss**
1. Add to `BOSSES` array with name and color scheme
2. Implement in boss rendering section
3. Boss selection uses round-robin from cleared bosses

**New Theme**
1. Add to `THEMES` with sky gradient, cloud color, sun color, name
2. Theme selection cycles through available themes

**New Difficulty**
1. Add to `DIFFICULTIES` with spawn and balance parameters
2. Update difficulty buttons in HTML
3. Add to difficulty selection logic

### Performance Considerations

- Adaptive quality system reduces particle effects/animations under load
- Canvas dirty-rect tracking avoids redrawing entire canvas each frame
- Enemy/projectile pooling to reduce GC pressure
- Event delegation for touch controls

### Accessibility

- Screen reader announcements for game state changes
- High contrast mode support (dark theme)
- Reduced motion option (respects system preference and user toggle)
- Keyboard-only controls supported
- Touch controls with large buttons for mobile

### Browser Compatibility

- Modern browser required (Canvas 2D, Web Audio, LocalStorage)
- Mobile-first responsive design
- Touch handling with fallback for mouse
- CSS grid/flex for layout

## Common Tasks

### Tweaking Game Balance
- Modify `DIFFICULTIES` object spawn rates and speed multipliers
- Adjust `KILLS_PER_STAGE` for progression pacing
- Change enemy HP and scoring in `ENEMY_TYPES` and `BOSSES`

### Adjusting Visual Theme
- Edit `THEMES` color values or add new themes
- Modify CSS background gradients in `#gameCanvas` style
- Adjust particle colors in effect-rendering sections

### Adding Sound Effects
- Call `beep()` with frequency, duration, waveform, volume, and options
- Use `noiseBurst()` for impact/explosion sounds
- Adjust envelope timing and filtering for desired effect

### Modifying Controls
- Update keyboard event handlers for new keys
- Add to touch control button mapping
- Adjust movement/firing rates in update loop

## Debugging

### Console Logs
- Game state variables available in global scope (within IIFE context)
- Use browser DevTools to inspect canvas, frame time, entity counts

### Common Issues
- **Framerate drops**: Check adaptive quality system; adjust enemy spawn rates or particle effect density
- **Audio not playing**: Ensure `audioCtx` is initialized and user has interacted with page
- **Touch controls not working**: Check device orientation and touch event handling in mobile section
- **Collision detection off**: Verify hitbox sizes and player position calculations

## Storage & Persistence

- `skyfighter.bestScore`: Best score (number)
- `skyfighter.muted`: Mute state (0 or 1)
- `skyfighter.difficulty`: Last selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Reduced motion preference (0 or 1)

All stored in browser localStorage, cleared only by user action.

## Future Enhancement Ideas

- Leaderboard (with backend integration)
- Power-up system (health restore, weapon unlock, temporary invincibility)
- Varied boss attack patterns per stage
- Sound track with rhythm mechanics
- Gameplay modifiers/challenge modes
- Difficulty curve refinement based on player skill
- Tutorial/practice mode
- Mobile app wrapper (Electron/Cordova)
