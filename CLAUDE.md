# Sky Fighter — Codebase Documentation

## Project Overview

**Sky Fighter** is a browser-based vertical shoot-'em-up game (Raiden-style arcade shooter) implemented as a single-file HTML5 web application. Players navigate a spaceship through waves of enemies, collect weapon power-ups, and survive encounters with stage bosses.

- **File**: `index.html` (self-contained)
- **Platform**: Web browser (desktop & mobile)
- **Type**: Single-page vanilla JavaScript game
- **No build process needed** — open `index.html` directly in a browser to play

## Game Mechanics

### Player Controls
- **Keyboard**: Arrow keys or WASD to move, Space to fire, X or Shift for bomb
- **Mouse**: Move cursor to steer, click to fire
- **Mobile**: Drag left zone to move, tap FIRE button, tap BOMB button
- **Accessibility**: Pause button (P/Esc), reduced motion toggle, mute button

### Gameplay
- **Lives**: Start with 3 (Easy: 4), lose one when hit by enemy fire or collision
- **Weapons**: VULCAN, HOMING, SPREAD — collect power-ups to level up or switch
- **Bombs**: Smart bombs deal heavy damage; limited supply (2 per game, Easy: 3)
- **Score Multiplier**: Builds from enemy kills, resets when hit
- **Stages**: Each stage requires 18 enemy kills to progress; bosses appear at stage transitions
- **Difficulty**: Easy, Normal (default), Hard — affects spawn rates, enemy speed, and boss HP

### Difficulty Settings
```
Easy:   Slower spawns, slower enemies, lower boss HP, 4 lives, 3 bombs
Normal: Balanced spawn/speed/HP, 3 lives, 2 bombs
Hard:   Fast spawns, faster enemies, higher boss HP, 3 lives, 2 bombs
```

## Architecture

### Code Structure (all in `index.html`)

**1. HTML (lines 1–421)**
- Canvas element for game rendering
- HUD elements: score, best score, weapon display, health/bombs indicators
- Boss health bar
- Overlay UI: title screen, game over screen, difficulty selector
- Touch control zone and buttons for mobile
- Accessibility elements (screen reader live region)

**2. CSS (lines 10–359)**
- Responsive design with `clamp()` for mobile/desktop scaling
- CSS animations: heart beat (critical health), combo pulse, stage banner, flash effects
- Color scheme: dark blue/navy background with bright accent colors
- Safe area insets for notched devices
- Reduced motion support (respects `prefers-reduced-motion`)

**3. JavaScript (lines 422+)**
- Game state management (mode, difficulty, lives, score)
- Canvas rendering loop with adaptive quality adjustment
- Collision detection (AABB bounding box)
- Entity management: player, enemies, bullets, effects
- Audio system (simple sine-wave synthesis, respects mute setting)
- Local storage: best score, mute state, difficulty, reduced motion preference
- Touch/mouse/keyboard input handling

### Key Constants & Configuration

**Difficulties** (line 485–489)
```javascript
DIFFICULTIES = {
  easy:   { spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul },
  normal: { ... },
  hard:   { ... }
}
```
Adjust these to balance gameplay difficulty.

**Themes** (line 493–500)
Six color schemes for sky gradients, clouds, and sun. Each stage cycles through a theme.

**Spawn & Difficulty** (line 491)
```javascript
KILLS_PER_STAGE = 18  // Enemy kills required to advance stage
```

**Entity Constants**
- Player speed: embedded in input handling
- Enemy speeds: set per-difficulty, modified by difficulty multiplier
- Weapon stats: damage, fire rate, behavior per weapon type
- Boss health: `baseHP * difficulty.bossHpMul`

### Game Loop

1. **Input handling**: Capture keyboard, mouse, touch input → update player position & firing state
2. **Update**: Move entities, handle collisions, spawn enemies, fire bullets
3. **Rendering**: Draw sky gradient, entities, effects, HUD
4. **Performance tracking**: Measure frame time, auto-adjust quality if needed
5. **Request animation frame**: ~60fps target

## File Organization

```
/home/user/Google_4/
├── index.html          # Entire game (HTML + CSS + JS)
├── README.md           # Minimal project info
└── .git/              # Git repository
```

## Testing & Running

**Local play**: Open `index.html` in any modern browser
```bash
open index.html   # macOS
xdg-open index.html  # Linux
start index.html  # Windows
```

**Mobile testing**: Open on phone/tablet browser or use browser dev tools device emulation

**Quality checks**:
- No console errors (open DevTools → Console)
- All difficulty levels playable
- Touch controls responsive on mobile
- Reduced motion setting disables animations
- Audio works and mute persists across sessions

## Common Development Tasks

### Adding a New Weapon
1. Locate weapon definitions (search for `VULCAN`, `HOMING`, `SPREAD`)
2. Add new weapon object with: `name`, `dmg`, `speed`, `fireRate`, `behavior`
3. Update weapon cycle logic to include new weapon
4. Test fire rate and damage balance

### Adjusting Difficulty
1. Modify `DIFFICULTIES` object (line 485)
2. Test at each difficulty level to ensure progression feels right
3. Watch for spawn rate creating overwhelming scenarios

### Adding a New Theme
1. Add theme object to `THEMES` array (line 493)
2. Provide: `sky` (3-color gradient array), `cloud`, `sun` (RGB strings), `name`
3. Test visual readability at different times of day if relevant

### Fixing Collision Issues
1. Collision detection uses AABB (axis-aligned bounding box)
2. Entity hitboxes defined by `x`, `y`, `w`, `h` properties
3. Check entity dimensions match visual size in rendering code

### Performance Optimization
1. Quality auto-adjusts based on frame time (line 474–483)
2. Lower quality: reduced enemy/bullet count, simpler rendering
3. Monitor frameTimes array to debug frame rate issues
4. Reduce enemy spawn rate or particle effects if needed

### Accessibility & UX
- **Screen reader**: Announcements via `srLive` element
- **Reduced motion**: CSS animation disable respected
- **Color contrast**: Ensure text remains readable over backgrounds
- **Mobile**: Touch zones must be at least 44×44 px

## Local Storage Keys

Persisted player preferences:
- `skyfighter.bestScore` — Best score achieved
- `skyfighter.muted` — Audio mute state
- `skyfighter.difficulty` — Last selected difficulty
- `skyfighter.reducedMotion` — Motion preference

## Browser Compatibility

Target: Modern browsers (Chrome, Firefox, Safari, Edge)
- Canvas 2D context
- Touch events (mobile)
- `localStorage` API
- CSS Grid & Flexbox
- CSS animations & gradients

## Development Notes

### Adding Features
- Keep game loop logic in the main `(function () { ... })()` IIFE
- Use AABB for all collision detection (no physics engine)
- Audio: Generated sine waves, no external audio files needed
- State managed through global variables in IIFE scope

### Debugging
- Use browser DevTools to inspect canvas drawing
- Log entity positions/velocities in update loop
- Verify hitboxes match visual rendering
- Check localStorage contents: `console.log(localStorage)`

### Performance Considerations
- Canvas rendering is CPU-bound on low-end devices
- Quality setting auto-downgrades particle effects and enemy count
- Avoid adding complex animations without testing on mobile
- Sprite sheets not used — all shapes drawn procedurally

## Deployment

Simply serve `index.html` over HTTP/HTTPS. No build step required.

Example:
```bash
python3 -m http.server 8000
# Then open http://localhost:8000/index.html
```

## Future Enhancement Ideas

- Leaderboard (backend integration)
- Multiple player modes (co-op, waves)
- More boss varieties with unique attack patterns
- Particle effects on boss defeat
- Weapon customization menu
- Stage select after each completion
- Sound effects library (instead of sine waves)
- Mobile app wrapper (Cordova/Capacitor)

---

**Last Updated**: 2026-09-25  
**Document Purpose**: Guide for Claude Code sessions working on Sky Fighter
