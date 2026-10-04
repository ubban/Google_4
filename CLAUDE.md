# Sky Fighter — Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 web application. The game features a player-controlled fighter aircraft that battles through waves of enemies and stage bosses.

### Core Features
- **Vertical shoot-'em-up gameplay** — Classic shmup mechanics
- **Dynamic weapon system** — Switch between different weapon types (standard, spread, laser)
- **Smart bombs** — Limited-use panic button to clear the screen
- **Stage progression** — Multiple stages with escalating difficulty
- **Boss battles** — Stage-ending encounters with challenging AI patterns
- **Touch & mouse control** — Full mobile and desktop support
- **Responsive UI** — Safe area aware HUD that adapts to any screen
- **Progressive difficulty** — Enemy spawning and pattern complexity increase per stage

## Project Structure

```
Google_4/
├── index.html          # Complete game implementation (all-in-one file)
├── README.md           # Basic project timestamp
├── CLAUDE.md           # This documentation
└── .git/               # Version control
```

The entire game is implemented in a single `index.html` file (~58KB). This monolithic approach keeps the project simple and easy to deploy as a static web page.

## Technical Architecture

### Technology Stack
- **Runtime:** Vanilla JavaScript (ES6+) — no frameworks or build tools required
- **Graphics:** Canvas 2D API for rendering
- **Input:** Mouse, touch, and keyboard event handling
- **Deployment:** Static HTML file — runs directly in any modern browser

### Game Loop & Rendering
- **Frame rate:** 60 FPS target (requestAnimationFrame)
- **Canvas size:** 480×800px (portrait orientation, 3:5 aspect ratio)
- **Scaling:** Game canvas scales responsively while maintaining aspect ratio
- **Dark theme:** `color-scheme: dark` with `#05122b` background

### Core Game Systems

#### 1. **Player Character**
- Position: center-bottom of screen, controlled via mouse/touch or arrow keys
- Health: Single hit destroys the player (lives system)
- Weapons: Cycles through 3 weapon types (standard shots, spread shots, laser)
- Smart bombs: Limited charges that clear screen and damage all on-screen enemies

#### 2. **Enemy System**
- Enemies spawn in waves from the top of the screen
- Movement patterns: Linear vertical, diagonal, and circular trajectories
- Bullet patterns: Some enemies fire at the player
- Wave progression: Difficulty increases per stage (more enemies, faster behavior)
- Rendering: Simple rectangle-based sprites with rotation

#### 3. **Weapon System**
- **Standard fire:** Single centered shot, fastest fire rate
- **Spread fire:** 3-way shot pattern, wider coverage
- **Laser:** Continuous beam weapon, highest damage per second
- Weapon switching: Press `E` or tap weapon button
- Infinite ammo — all weapons available unlimited

#### 4. **Collision Detection**
- Player-bullet collision (enemies): Hit registration and damage
- Player-enemy collision: Game over condition
- Bullet-screen edge: Bullets despawn when off-screen
- Enemy-screen edge: Enemies despawn when off-screen

#### 5. **Scoring & UI**
- **HUD display:** Score, lives, stage, current weapon, bomb count
- **Score accumulation:** Kill streak bonuses, strategic play rewards
- **Game states:** Menu → Playing → Level Complete/Game Over

## Game Mechanics

### Wave Pattern
Each stage follows this progression:
1. Enemy waves spawn at intervals
2. Player fights through increasing difficulty
3. Boss spawns when main waves complete
4. Defeat boss → next stage

### Difficulty Scaling
- Enemy count increases per stage
- Enemy fire rate and speed increase
- Boss health/attack patterns become more complex
- Stage-specific enemy compositions

### Player Resources
- **Lives:** Start with 3 lives, game over on death
- **Smart bombs:** 2-3 available per stage, recharge between levels
- **Weapon cycle:** Always available, no reload

## Code Organization Within index.html

The file is structured as follows (approximate line ranges):
1. **HTML Structure** (~20 lines) — Canvas, HUD elements, meta tags
2. **CSS Styling** (~200 lines) — Game canvas, HUD layout, responsive design
3. **JavaScript Implementation** (~1500+ lines):
   - Game state & configuration
   - Player class and mechanics
   - Enemy & bullet classes
   - Boss encounter system
   - Input handling (keyboard/touch/mouse)
   - Collision detection
   - Rendering and animation loop
   - Stage management and progression

## Development Workflow

### Making Changes
Since all code is in a single HTML file:
1. Edit `index.html` directly
2. Refresh browser to see changes immediately
3. Test game on desktop, tablet, and mobile
4. Commit when feature is complete

### Testing Recommendations
- **Desktop browsers:** Chrome, Firefox, Safari (latest versions)
- **Mobile devices:** iOS Safari, Chrome mobile
- **Testing checklist:**
  - Player movement responsiveness
  - Weapon firing patterns and collision
  - Enemy wave spawning and difficulty progression
  - Boss behavior and damage
  - Smart bomb effect and cooldown
  - Score calculation and HUD updates
  - Screen boundaries and canvas scaling
  - Touch controls on mobile (especially safe areas)
  - Game over and stage complete transitions

### Performance Considerations
- Canvas rendering optimized with minimal DOM updates
- Requestanimationframe for smooth 60 FPS
- Bullet pooling (reuse objects) to reduce garbage collection
- Enemy despawn when off-screen to maintain reasonable entity count

## Key Configuration Values

Located in the JavaScript section, these constants control game feel:
- `gameWidth: 480` / `gameHeight: 800` — Canvas dimensions
- `spawnRate` — Milliseconds between enemy waves
- `enemySpeed` — Base movement speed (increases per stage)
- `fireRate` — Milliseconds between player shots
- `bulletSpeed` — Projectile velocity
- Boss health and damage values

Adjust these values to tune difficulty and gameplay.

## Browser Compatibility
- **Required:** ES6 support, Canvas 2D, requestAnimationFrame
- **Minimum:** Chrome 51+, Firefox 54+, Safari 10+, Edge 15+
- **Best performance:** Modern Chromium browsers on desktop

## Git Workflow

### Branch Strategy
- Work on `claude/inspiring-planck-omk3n5` branch
- Commit with clear, descriptive messages
- Push changes when feature is complete

### Commit Message Convention
- Describe what changed and why
- Reference specific game mechanics or systems affected
- Example: `"Increase boss phase difficulty and adjust bullet patterns"`

## Future Enhancement Ideas
- **Difficulty modes:** Easy/Normal/Hard with balance adjustments
- **Powerups:** Temporary weapon upgrades, shield, score multipliers
- **Visual effects:** Explosions, screen shake, particle effects
- **Sound:** Background music and SFX (add audio elements)
- **Persistence:** High score saving (localStorage)
- **Achievements:** Unlockables for specific accomplishments
- **Mobile optimizations:** Haptic feedback, landscape support
- **Level editor:** Tool to design custom enemy waves and bosses

## Known Limitations
- Single-file architecture makes the codebase grow as features increase
- No audio support (framework agnostic but easily added)
- Fixed aspect ratio (480×800) — could support multiple orientations
- Limited particle/visual effects to keep file size low
- No networking or multiplayer (single-player only)

## Useful Resources
- [Canvas API MDN Documentation](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
- [Game Programming Patterns](https://gameprogrammingpatterns.com/) — Reference for game architecture
- [Shmup design](https://en.wikipedia.org/wiki/Shoot_%27em_up) — Genre conventions
- Touch events: [MDN Pointer Events](https://developer.mozilla.org/en-US/docs/Web/API/PointerEvent)

---

**Last Updated:** 2026-10-04  
**Maintainer:** Claude Code  
**Repository:** ubban/google_4
