# Sky Fighter - Developer Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 canvas application. The game features player-controlled aircraft combat through multiple stages, with dynamic difficulty scaling, weapon progression, and boss encounters.

### Key Features

- **Gameplay**: Navigate a fighter jet, fire weapons, and deploy smart bombs to defeat enemies and bosses
- **Difficulty Levels**: Easy, Normal, and Hard modes with scaled enemy behavior and spawn rates
- **Weapon System**: Progressive weapon upgrade system (VULCAN, TWIN, SPREAD, FOCUS, etc.) with level progression
- **Bomb Mechanics**: Smart bombs with screen-clearing and selective damage capabilities
- **Visual Themes**: 6 different stage themes with unique color schemes and atmospheric effects
- **Controls**: 
  - Desktop: Mouse movement + click to fire, keyboard (Arrows/WASD + Space/X for bomb)
  - Mobile: Touch controls with visual joystick and action buttons
- **Audio**: Sound effects with mute toggle capability
- **Accessibility**:
  - Screen reader announcements for game state
  - Reduced motion mode for animations/effects
  - Safe area insets for notched devices
- **Performance**: Adaptive quality scaling based on frame rate monitoring

## File Structure

```
Google_4/
├── index.html          # Complete game implementation (1669 lines)
├── README.md           # Project metadata
├── CLAUDE.md          # This developer guide
└── .git/              # Version control
```

### Single-File Architecture

The entire game is contained in `index.html`:
- **HTML Structure** (lines 1-378): DOM elements for canvas, HUD, overlay, controls
- **CSS Styling** (lines 10-350): Game UI, animations, responsive design, dark theme
- **JavaScript Game Engine** (lines 379+): 
  - Game state management
  - Player/enemy/projectile mechanics
  - Collision detection
  - Boss AI
  - Input handling (keyboard, mouse, touch)
  - Score/persistence system

## Game Systems

### 1. Difficulty Settings

Located around line 1020 in the `DIFFICULTIES` object:
```javascript
{
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
}
```
- `spawnBase`: Initial spawn interval (frames)
- `spawnMin`: Minimum spawn interval at peak difficulty
- `enemySpeed`: Multiplier for enemy movement speed
- `enemyShootChance`: Probability enemies fire per frame
- `lives`: Starting player health
- `bombs`: Starting bomb count
- `bossHpMul`: Boss health multiplier

### 2. Stage/Theme System

6 visual themes (lines 1042+) with gradient skies, clouds, and atmospheric effects:
- Daylight Front (blue, classic)
- Sunset Corridor (orange/pink)
- Night Approach (dark blue)
- Storm Front (green)
- Arctic Wastes (light blue)
- Volcanic Ridge (red/orange)

Themes rotate as players progress through stages. One theme = multiple waves of enemies before a boss encounter.

### 3. Weapon Progression

Weapons upgrade through enemy drops during stages:
- VULCAN: Starting weapon
- TWIN: Dual shots
- SPREAD: Wide spread pattern
- FOCUS: Concentrated fire
- Others: Additional variant weapons

Player selects weapon type via drops, and level increases through additional drops of same type.

### 4. Enemy & Boss System

- **Regular Enemies**: Spawn in waves, move downward with horizontal patterns, fire at player
- **Bosses**: Spawn after completing 18 kills (`KILLS_PER_STAGE`), have phases, unique attack patterns
- **Collision Detection**: Rectangle-based collision for bullets, enemies, bombs, player

### 5. Input Handling

- **Keyboard**: Arrow keys / WASD for movement, Space/X for bomb, P/Esc for pause
- **Mouse**: Movement follows cursor, click to fire
- **Touch**: 
  - Left 55% of screen: movement (joystick visualization)
  - Right 45% of screen: FIRE and BOMB buttons
  - Optional joystick base for visual feedback

### 6. Score & Persistence

Stored in localStorage with keys:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Motion accessibility preference

## Development Workflow

### How to Run Locally

1. Clone the repository and checkout the feature branch:
   ```bash
   git clone <repo-url> Google_4
   cd Google_4
   git checkout claude/inspiring-planck-xtgu6x
   ```

2. Open `index.html` directly in a web browser (no server needed)
3. Or serve locally: `python -m http.server 8000` then visit `http://localhost:8000`

### Testing Changes

After modifying `index.html`:
1. Refresh browser to reload (Ctrl+R or Cmd+R)
2. Test on both desktop and mobile:
   - Desktop: Keyboard, mouse, and pause states
   - Mobile: Touch controls, portrait/landscape, safe area display
3. Test all difficulty modes
4. Verify localStorage persistence across sessions

### Key Areas to Modify

| Feature | Location | Notes |
|---------|----------|-------|
| Difficulty tuning | ~line 1020 | Adjust spawn rates, enemy speed, player lives |
| Visual themes | ~line 1042 | Add/modify stage color schemes |
| Weapon system | ~line 1300+ | Define weapon fire patterns |
| Boss behavior | ~line 1400+ | Adjust boss attack patterns and health |
| UI/HUD text | ~line 230-280 | Modify overlay text and game messages |
| Performance tuning | ~line 1010-1018 | Adjust quality scaling logic |

### Common Tasks

**Add a new difficulty level:**
1. Add entry to `DIFFICULTIES` object around line 1020
2. The game will automatically detect it via `Object.keys(DIFFICULTIES)`

**Change weapon fire pattern:**
1. Search for weapon type in the firing code (~line 1350+)
2. Modify bullet spawn positions/velocities in the weapon handler

**Adjust visual theme:**
1. Modify gradient colors in the `THEMES` array (line 1042+)
2. Each theme object has `sky` (gradient stops), `cloud`, `sun` colors

**Add/modify boss:**
1. Boss generation happens after 18 kills
2. Boss AI logic is in the game loop's enemy update section
3. Boss health is multiplied by `bossHpMul` from difficulty settings

## Code Architecture Notes

### Game State Structure
- `player`: Position, health, bomb count, weapon state
- `enemies[]`: Active enemy instances
- `projectiles[]`: Active bullet instances from player
- `playerProjectiles[]`: Bullets from player weapons
- `stage`, `stageKills`: Progress tracking
- `score`, `mult`: Scoring system

### Main Game Loop
1. Clear canvas and draw background
2. Update and render player
3. Update and render enemies (spawn new ones based on difficulty)
4. Update and render projectiles
5. Handle collisions
6. Update HUD (score, health, etc.)
7. Check win/lose conditions

### Performance Optimization
- Adaptive quality reduction (`quality` variable) based on frame time tracking
- Particle effects disabled at low quality
- Reduced particle count at low quality
- Object pooling for projectiles (optional—currently recreated)

## Accessibility Features

- **Screen Reader Support**: Announcements via `srLive` element with `aria-live="polite"`
- **Reduced Motion**: Toggle via button; disables animations when enabled
- **Safe Area Insets**: CSS `env(safe-area-inset-*)` for notched devices
- **Keyboard Navigation**: Full keyboard support, pause/resume via P/Esc
- **High Contrast**: Dark background with bright UI text and indicators

## Browser Compatibility

- Canvas 2D API (all modern browsers)
- localStorage (all modern browsers)
- Touch events (mobile devices)
- Flexbox and modern CSS (no IE support needed)
- ES6+ JavaScript (arrow functions, const/let, template literals)

## Performance Targets

- Target: 60 FPS on desktop, 30+ FPS on mobile
- Adaptive quality scaling if average frame time exceeds 1/40s
- Particles and effects culled when `quality > 0`

## Known Limitations

1. **Single-file deployment**: All code in one HTML file (1669 lines)—consider splitting if complexity grows
2. **No persistent online scores**: High scores stored locally only
3. **Fixed game resolution**: Adapted to viewport but no aspect ratio locking
4. **Limited mobile optimizations**: Touch controls are basic; could add haptic feedback

## Future Enhancement Ideas

- Multiple stages with unique boss encounters
- Leaderboard integration (online/local)
- Power-up variety (shields, speed boost, weapon slots)
- Enemy special attacks and patterns
- Screen shake and particle effects tuning
- Sound design and music
- Mobile app wrapper (Cordova/Capacitor)
- Tutorial/help overlay
- Controller gamepad support
- Replay system

## Git Workflow

Work on the designated feature branch: `claude/inspiring-planck-xtgu6x`

```bash
# Ensure on correct branch
git checkout claude/inspiring-planck-xtgu6x

# Make changes to index.html or other files

# Commit with descriptive message
git commit -m "Feature: [description]"

# Push to feature branch
git push -u origin claude/inspiring-planck-xtgu6x
```

## Contact & Attribution

- **Project**: Sky Fighter (Raiden-style vertical shoot-'em-up)
- **Last Updated**: 2026-10-05
- **Branch**: `claude/inspiring-planck-xtgu6x`
- **Created by**: Claude Code Agent

---

For questions or improvements, refer to the main README.md or contact the development team.
