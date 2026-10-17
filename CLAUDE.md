# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 web application. Players navigate through multiple stages, defeat enemy squadrons, face off against powerful bosses, and collect power-ups to increase their weapon effectiveness.

- **Type**: Interactive web game (Canvas-based)
- **Platform**: Browser (desktop and mobile)
- **Architecture**: Single HTML file with embedded CSS and JavaScript (~1669 lines)
- **Entry Point**: `index.html`

## Game Mechanics

### Core Gameplay
- **Player Ship**: Navigate using mouse, keyboard (WASD/Arrows), or touch controls
- **Weapon System**: Fire continuously; collect power-ups to upgrade weapons
- **Scoring**: Defeat enemies to earn points; bonus multiplier for consecutive hits
- **Bomb Mechanic**: Limited smart bombs that clear nearby enemies and projectiles
- **Health**: Three lives (configurable by difficulty); lose a life when hit by enemy fire
- **Stages**: Progress through stages by defeating a set number of enemies (18 kills per stage)
- **Bosses**: Each stage culminates in a boss battle with unique appearance and attack patterns

### Difficulty Levels
The game supports three difficulty tiers, each affecting:
- **Easy**: Lower spawn rate, slower enemies, higher survival chance (4 lives, 3 bombs)
- **Normal** (default): Balanced challenge (3 lives, 2 bombs)
- **Hard**: Aggressive enemy spawning, faster movement, reduced bomb count (3 lives, 2 bombs)

Settings are stored in `localStorage` under key `skyfighter.difficulty`

### Enemy Types
1. **Grunt** (red, 34×34 px)
   - Basic enemy type
   - 1 HP, standard speed
   - Available from stage 1
   - Worth 10 points

2. **Interceptor** (orange, 26×26 px)
   - Faster, more agile
   - 1 HP, 1.7× speed multiplier
   - Weaving movement pattern
   - Available from stage 1
   - Worth 15 points

3. **Tank** (purple, 46×46 px)
   - Heavily armored
   - 4 HP, slower movement (0.55× speed)
   - Available from stage 2+
   - Worth 35 points

### Boss Types
Six distinct bosses with unique colors and HP multipliers:
- **VANGUARD**: Purple theme, 1.0× HP
- **BEHEMOTH**: Orange theme, 1.35× HP (tankiest)
- **SPECTER**: Teal theme, 0.8× HP (fastest)
- **DREADNOUGHT**: Gray theme, 1.55× HP (strong)
- **WRAITH**: Dark purple theme, 0.95× HP
- **TITAN**: Gold theme, 1.7× HP (strongest)

Boss HP is calculated as base value × difficulty multiplier × boss HP multiplier

## Visual Design

### Themes
The game rotates through 6 visual themes that change the sky gradient, clouds, and sun color:
1. **Daylight Front**: Bright blue sky
2. **Sunset Corridor**: Orange/pink gradients
3. **Night Approach**: Deep blue, stormy
4. **Storm Front**: Greenish, turbulent
5. **Arctic Wastes**: Icy blue tones
6. **Volcanic Ridge**: Hot orange/red

Theme selection is based on the current stage number.

### UI Elements
- **HUD** (top center): Score, best score, stage, lives (hearts), bombs (explosions), current weapon, multiplier
- **Boss Health Bar** (top center, when boss active): Boss name and HP bar with gradient fill
- **Combo Pop-up** (center screen): Shows multiplier bonus when defeating consecutive enemies
- **Stage Banner** (center screen): Shows theme/stage transition
- **Touch Controls** (mobile): Left zone for movement (joystick), right side buttons for fire and bomb
- **Top Right Buttons**: Mute toggle, reduced motion toggle, pause button

### Canvas Rendering
- Base resolution: Responsive canvas that fills the viewport
- Adaptive quality: Game monitors frame time and scales particle effects/visual quality if FPS drops
  - Quality levels: 0 (full), 1 (medium), 2 (low)
  - Quality improves if avg frame time < 1/56s, degrades if > 1/40s

## Input Handling

### Desktop Controls
- **Movement**: 
  - Mouse: Move cursor to position ship
  - Keyboard: Arrow keys or WASD
- **Fire**: Left mouse click or Space bar
- **Bomb**: X key or Shift
- **Pause/Resume**: P key or Esc
- **Mute**: M key
- **Reduced Motion Toggle**: N key

### Mobile Controls
- **Movement**: Drag in left touch zone to control ship position with visual joystick
- **Fire**: FIRE button (right side)
- **Bomb**: BOMB button (right side)
- **Pause**: Pause button (top right)

### Accessibility
- Screen reader live region (`srLive`) announces game events
- Reduced motion preference detection via `prefers-reduced-motion` media query
- Preference stored in `localStorage` under key `skyfighter.reducedMotion`

## Audio System

### Web Audio Synthesis
- Uses native Web Audio API (no external audio files)
- Synthesized sound effects for:
  - Weapon fire (different per weapon)
  - Enemy explosions
  - Bomb detonation
  - Power-up collection
  - Boss appearance/defeat
  - Hit/damage feedback

- **Master Volume**: Controlled by mute toggle
- **Mute State**: Persisted in `localStorage` under key `skyfighter.muted`

## Game State & Data Persistence

### Local Storage Keys
- `skyfighter.bestScore`: High score (number)
- `skyfighter.difficulty`: Selected difficulty ('easy', 'normal', or 'hard')
- `skyfighter.muted`: Audio mute state ('1' or '0')
- `skyfighter.reducedMotion`: Reduced motion preference ('1' or '0')

### Game Objects

#### Player
- Position (x, y)
- Width: 28 px, Height: 32 px
- Health (lives remaining)
- Current weapon type and level
- Bomb count
- Score and multiplier

#### Enemies & Bullets
- Position, velocity
- HP remaining
- Type-specific rendering and behavior
- Enemy bullets track with slight prediction toward player

#### Particles & Effects
- Explosions on enemy death
- Muzzle flash on firing
- Screen vignette effect on damage
- Screen flash on hits/bombs
- Combo text pop-ups

## Adaptive Performance

The game continuously monitors frame timing to maintain smooth performance:

```javascript
// Performance tracking (45-frame rolling window)
- If avg frame time > 1/40s (40 FPS) → reduce quality
- If avg frame time < 1/56s (56 FPS) → increase quality
- Quality affects particle count and visual effect complexity
```

## Development Notes

### File Structure
- **Single file**: All code (CSS, HTML structure, JavaScript) in `index.html`
- **No external dependencies**: Uses only native Web APIs
  - Canvas 2D context for rendering
  - Web Audio API for sound
  - localStorage for persistence
  - requestAnimationFrame for game loop

### Game Loop Architecture
1. **Update Phase**: Update game state, positions, collisions
2. **Render Phase**: Clear canvas and redraw all game objects
3. **Frame Timing**: Adaptive quality adjustment based on performance
4. **Input Processing**: Keyboard, mouse, and touch event handlers

### Key Algorithms
- **Enemy Spawn Queue**: Enemies spawn based on difficulty-adjusted timers that decrease as stage progresses
- **Collision Detection**: AABB (axis-aligned bounding box) for projectiles vs enemies/player
- **Multiplier System**: Increments on hits, resets on taking damage
- **Boss Patterns**: Boss movement and attack patterns hardcoded per boss type

## Common Tasks for Future Sessions

### Adding New Content
1. **New Enemy Type**: Add to `ENEMY_TYPES` object with stats and rendering code
2. **New Boss**: Add to `BOSSES` array with color scheme and HP multiplier
3. **New Theme**: Add to `THEMES` array with gradient colors and name
4. **New Sound Effect**: Create synthesis parameters in audio section

### Modifying Game Balance
- Adjust `DIFFICULTIES` object for spawn rates, speeds, and starting lives
- Modify `KILLS_PER_STAGE` to change progression pace
- Update boss `hpMul` values in `BOSSES` array
- Change weapon multipliers in weapon definitions

### Performance Optimization
- Monitor `quality` variable (0-2 scale)
- Reduce particle count when `quality > 0`
- Profile with browser DevTools to identify bottlenecks
- Consider offscreen canvas for prerendered sprites if needed

### Visual Customization
- Theme gradients in `THEMES` array
- Colors for HUD, buttons, and effects
- Animation durations in CSS `@keyframes`
- Canvas scaling and layout in `style` section

## Testing

### Manual Testing Checklist
- [ ] Game starts successfully on both desktop and mobile
- [ ] All three difficulty levels are selectable
- [ ] Enemy spawning accelerates with stage progression
- [ ] Boss appears after 18 enemy kills
- [ ] Weapon power-ups drop and increase weapon level
- [ ] Bomb mechanic works and count updates
- [ ] Score multiplier increases/resets correctly
- [ ] Lives counter updates on damage
- [ ] Game over screen displays final score
- [ ] Best score is persisted across sessions
- [ ] Mobile touch controls are responsive
- [ ] Audio synthesizes without errors (when unmuted)
- [ ] Mute/unmute toggle works
- [ ] Reduced motion setting disables animations

## Browser Compatibility
- Requires modern browser with:
  - Canvas 2D API support
  - Web Audio API support
  - ES2015+ JavaScript support
  - localStorage support
- Tested on Chrome, Firefox, Safari, Edge
- Mobile: iOS Safari, Chrome for Android

## Future Enhancement Ideas
- Implement difficulty scaling per stage
- Add power-up variety (shield, speed boost)
- Implement combo bonuses and special attacks
- Add leaderboard system (IndexedDB)
- Create level editor for custom stages
- Add controller support for gamepads
- Implement progressive web app (PWA) features
