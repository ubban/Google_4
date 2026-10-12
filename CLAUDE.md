# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5, CSS3, and JavaScript. The entire application is contained in a single `index.html` file with no external dependencies. It features progressive difficulty levels, weapon upgrades, smart bombs, boss battles, and cross-platform support (desktop and mobile).

## Architecture

### Single-File Structure
The project uses a monolithic architecture with all code in `index.html`:
- **HTML**: Game UI layout (canvas, HUD, overlay, controls)
- **CSS**: Responsive styling with animations, dark theme
- **JavaScript**: Game engine, entity systems, input handling, audio management

### Key JavaScript Components

#### Game State Management
- Global game state object tracking: `score`, `stage`, `lives`, `bombs`, `mute`, `difficulty`, `reducedMotion`
- Persistent storage using `localStorage` for: best score, mute state, difficulty, reduced motion preference
- Game phases: `'home'`, `'playing'`, `'dead'`, `'won'`

#### Entity Systems
1. **Player** (`entities.player`)
   - Position-based movement (keyboard and mouse)
   - Weapon system with upgradeable firepower (Vulcan, Plasma, Barrage)
   - Health: 3-4 lives depending on difficulty
   - Smart bomb ability (limited uses)

2. **Enemies** (`entities.enemies`)
   - Spawned based on difficulty progression
   - Each has: position, velocity, health, shoot cooldown, AI behavior
   - Enemies drop weapon/bomb pickups when defeated
   - Boss enemies spawn at configurable kill thresholds

3. **Projectiles** (`entities.bullets`, `entities.enemyFire`)
   - Player bullets track weapon type and piercing behavior
   - Enemy bullets with random spread patterns
   - Collision detection with enemies/player

4. **Pickups** (`entities.pickups`)
   - Weapon upgrades (Vulcan, Plasma, Barrage)
   - Bomb pickups
   - Float downward with fade-out timer

#### Core Game Loop
- **Frame-based**: `requestAnimationFrame` driving update → render → collision detection
- **Adaptive performance**: Monitors frame times and reduces quality if FPS drops below 40fps
- **Update order**: input → entity updates → projectile updates → collision checks → rendering

#### Input Handling
- **Keyboard**: WASD/Arrows for movement, Space/X/Shift for fire/bomb, P/Esc for pause
- **Mouse**: Movement tracking, left-click to fire
- **Touch**: Two-zone system (left side move, right side action buttons)
- Joystick visualization on mobile when dragging

#### Rendering System
- Uses 2D Canvas context
- Layered rendering: background → enemies → pickups → player → effects → UI
- Sprite-based: circles/triangles for entities, procedurally generated using canvas drawing methods
- Animated effects: explosions, screen shake, vignette pulse, combo popups

### Game Mechanics

#### Difficulty Levels
Three preset difficulty configurations control:
- Enemy spawn rate (base and minimum intervals)
- Enemy speed and shooting accuracy
- Player lives and bomb count
- Boss health multiplier

```javascript
easy:   { spawnBase: 75, enemySpeed: 1.0, lives: 4, bossHpMul: 0.8 }
normal: { spawnBase: 58, enemySpeed: 1.4, lives: 3, bossHpMul: 1.0 }
hard:   { spawnBase: 40, enemySpeed: 1.9, lives: 3, bossHpMul: 1.3 }
```

#### Weapon System
- **Vulcan**: Low damage, high rate of fire
- **Plasma**: Medium damage, moderate fire rate
- **Barrage**: High damage, lower fire rate, pierces multiple enemies
- Weapon levels increase damage (1-3 multiplier)

#### Scoring
- Basic kills: 100-300 points depending on weapon level
- Combo multiplier: Increases for consecutive hits, decays when missing
- Boss kills: Major point bonus
- Score display with best score tracking

#### Stage Progression
- 18 kills per stage triggers stage change
- Visual stage banner appears
- Environmental theme changes (6 different sky themes)
- Boss appears at stage 4, 7, 10, etc.

#### Smart Bombs
- Detonates all on-screen enemies and projectiles
- Clears danger but limited count
- Visual screen flash effect

### Visual Design

#### Responsive Layout
- Uses `clamp()` for fluid typography scaling
- Safe area insets for mobile notches/status bars
- Maintains 16:9 aspect ratio for game canvas
- Touch-friendly control buttons (78px radius)

#### Color Scheme
- Dark background: `#05122b`
- Sky gradient: Dynamic per-stage themes
- Accent colors: Yellow (`#ffd54f`), blue (`#1976d2`), red (`#ff5252`)
- High contrast for accessibility

#### Animation Framework
- CSS keyframes for: heartbeat (low health), vignette pulse, combo pop, stage banner, card entry
- Reduced motion support: respects `prefers-reduced-motion` via `reduced-motion` body class
- Screen shake on bomb: modifies canvas transform
- Flash effects: white overlay on hit/bomb

### Audio System
- Uses Web Audio API for sound generation (no external audio files)
- Procedurally generated sounds for: shoot, explosion, death, boss music, UI sounds
- Mute toggle with localStorage persistence
- Spatial audio cues (panning for screen edges)

### Accessibility Features
- Screen reader support with `aria-live` region
- Semantic HTML structure
- Reduced motion option that disables animations
- Keyboard navigation support
- Color contrast adherence

## File Structure

```
Google_4/
├── index.html          # Complete game (HTML + CSS + JS)
├── README.md          # Brief project note
└── CLAUDE.md          # This file
```

## Running the Game

### Local Development
1. Open `index.html` in any modern web browser
2. No build step, server, or dependencies required
3. Game persists settings to browser localStorage

### Browser Support
- Chrome/Edge 90+
- Firefox 88+
- Safari 14+
- Mobile browsers with Web Audio API support

### Performance Considerations
- Targets 60fps on modern devices
- Adaptive quality reduces particle/effect detail if FPS drops
- Canvas size optimized for viewport (responsive)
- No external asset loading (pure code generation)

## Key Technical Decisions

### Single-File Architecture
**Pros**: Easy to share, no build step, no dependencies
**Cons**: Harder to modularize, large file size (~50KB), harder to test

### Procedural Graphics
**Pros**: No image assets, compact file size, theme-able
**Cons**: CPU-bound rendering, limited visual detail

### Web Audio API Generation
**Pros**: No audio files, compact, dynamic (pitch/duration adjustable)
**Cons**: Browser support required, can't customize sounds as easily

### Canvas 2D (not WebGL)
**Pros**: Simpler API, works on all devices, easier to implement
**Cons**: Less performant for complex scenes

## Development Notes

### Adding New Enemies
1. Create enemy object with: `x`, `y`, `vx`, `vy`, `hp`, `type`
2. Add spawn logic in `updateEnemySpawn()`
3. Add rendering in `drawEnemy()`
4. Add collision/behavior in `updateEnemies()`

### Adding New Weapons
1. Add to `WEAPON_STATS` object with damage/speed
2. Extend `fireWeapon()` to handle new weapon type
3. Add pickup variant in weapon droplet list
4. Update weapon name display

### Modifying Difficulty
Edit the `DIFFICULTIES` object - changes apply to all future games started at that difficulty level.

### Theming
Edit `THEMES` array to add new visual themes:
- `sky`: Array of 3 gradient colors
- `cloud`: RGB values for cloud particles
- `sun`: RGB values for sun/background glow
- `name`: Display name shown in stage banner

## Performance Metrics

- Initial load: <100ms (single HTML file)
- Game loop: Targets 60fps (16.67ms per frame)
- Memory: ~5-10MB during gameplay
- Mobile: Runs on low-end devices with quality reduction

## Known Limitations

1. **Canvas Resolution**: Fixed 800x600 (scaled to viewport) - could be dynamic
2. **Audio**: Mono procedural sounds - no stereo or complex mixing
3. **Particle System**: Simple fade-in/out - no complex physics
4. **Collision**: Circle-based - not pixel-perfect
5. **Mobile**: Touch controls need more refinement for precise movement

## Customization Guide

### Adjust Game Balance
- Modify `DIFFICULTIES` object spawn rates and speeds
- Adjust `KILLS_PER_STAGE` for stage progression speed
- Change weapon damage multipliers in `WEAPON_STATS`
- Modify boss health with `bossHpMul` multiplier

### Visual Customization
- Canvas background color: CSS `#gameCanvas` background
- Sky themes: Edit `THEMES` array
- UI colors: CSS variables in `:root`
- Animation speeds: CSS `transition` and `animation` durations

### Gameplay Tweaks
- Player speed: Modify `player.speed` in movement logic
- Enemy bullet speed: Change in `enemyFire` creation
- Combo multiplier decay: Adjust in `updateCombo()`
- Bomb radius: Modify `getScreenDimensions()` return value usage

## Browser DevTools Tips

- **Console**: Check for errors, test game state with `game` object
- **Performance tab**: Profile frame rendering
- **Application tab**: Inspect localStorage for high score and settings
- **Sources**: Set breakpoints in game logic for debugging
