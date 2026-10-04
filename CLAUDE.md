# Sky Fighter: Raiden-style Vertical Shoot-em-up

## Project Overview

**Sky Fighter** is a single-file HTML5 canvas game inspired by classic vertical shoot-em-up games like Raiden. The game is completely self-contained in `index.html` (1,669 lines) with embedded CSS and JavaScript, making it easy to deploy and play in any modern web browser.

## Game Description

Sky Fighter is a fast-paced arcade-style shooter where players:
- Control a fighter jet moving across the bottom of the screen
- Destroy enemies descending from above
- Collect power-ups to upgrade weapons and abilities
- Face stage bosses at the end of each level
- Build combo multipliers for score bonuses
- Survive waves of enemies across multiple difficulty levels

### Core Mechanics

**Weapons System:**
- Multiple weapon types with upgrade levels
- VULCAN (default) and other weapon variants
- Weapons display current level in HUD
- Upgrades are earned through pickups

**Lives & Survival:**
- Start with multiple hearts (lives)
- Taking damage costs one heart
- Game over when all hearts are lost
- Health displayed in HUD

**Smart Bombs:**
- Limited-use special attacks
- Clearing all on-screen enemies
- Limited supply displayed in HUD
- Strategic resource requiring careful use

**Scoring & Combo:**
- Base score from defeating enemies
- Combo multiplier increases with consecutive hits
- New stage bonus achievements
- Best score tracking across sessions
- Combo counter displayed in HUD

**Difficulty Levels:**
- Easy: Reduced enemy fire, easier patterns
- Normal: Standard gameplay experience (default)
- Hard: Increased enemy density, aggressive patterns

### Game Modes

**Story Mode:** Progress through multiple stages with escalating difficulty. Each stage features:
- Enemy waves with increasing complexity
- Mid-stage patterns and formations
- Boss battles at stage completion
- Stage progression announcements

## Technical Architecture

### Single-File Structure
The entire game is contained in `index.html` with:
- **HTML:** Game canvas, UI overlays, HUD elements
- **CSS:** Styling for game interface, responsive design, dark theme
- **JavaScript:** Complete game engine with ~1,400+ lines of logic

### Key Game Systems

**Core Game Loop:**
- RequestAnimationFrame-based 60 FPS rendering
- Input handling (mouse, touch, keyboard)
- Physics simulation for all entities
- Collision detection between all game objects
- Score and combo calculations

**Entity System:**
- Player ship with movement and weapon firing
- Enemy entities with AI patterns and pathfinding
- Boss entities with multi-stage health and attack patterns
- Projectiles (player bullets and enemy fire)
- Power-ups (weapon upgrades, health, bombs)
- Effects and particles

**State Management:**
- Game states: START, PLAYING, PAUSED, STAGE_CLEAR, GAME_OVER, STAGE_TRANSITION
- Difficulty state persistence (localStorage)
- Score history and best score tracking
- Weapon and upgrade state tracking

**Accessibility:**
- Screen reader announcements via aria-live regions
- Keyboard controls support
- Touch input for mobile devices
- Reduced motion option (disables screen shake, flashes, pulses)
- High contrast UI elements

### Data Persistence
- Best score saved to localStorage
- Difficulty preference saved across sessions
- Game state reset on new game start

## File Structure

```
/home/user/Google_4/
├── index.html          # Complete game (1,669 lines)
├── README.md           # Basic project info
├── CLAUDE.md          # This file - development documentation
└── .git/              # Git repository
```

## How to Play

### Starting the Game
1. Open `index.html` in any modern web browser
2. Optionally select difficulty (Easy/Normal/Hard)
3. Click "START GAME" to begin

### Controls
- **Mouse/Touch:** Move fighter to cursor position
- **Left Click/Touch:** Fire primary weapon (continuous)
- **Spacebar:** Deploy smart bomb (limited supply)
- **P:** Pause/Resume game
- **M:** Mute/unmute sound
- **Motion:** Toggle reduced motion effects

### Objective
- Destroy all enemies in each stage
- Build combo multipliers for higher scores
- Defeat stage bosses to progress
- Survive as many stages as possible

## Development Guidelines

### Code Organization (JavaScript)
The game engine is organized into logical sections:

1. **DOM Element References** (lines ~200-230)
   - Canvas and UI element caching for performance

2. **Configuration & Constants** (lines ~230-330)
   - Game physics parameters
   - Difficulty multipliers
   - Enemy and boss patterns
   - Weapon definitions
   - Color schemes and visual settings

3. **Utility Functions** (lines ~330-430)
   - Math helpers (distance, angle calculations)
   - Random number generation
   - Collision detection
   - Audio utilities
   - Camera/screen shake effects

4. **Game State Management** (lines ~430-550)
   - State constants and transitions
   - Score and combo tracking
   - Pause/resume logic
   - Game over handling

5. **Entity Classes** (lines ~550-1000)
   - Player class (movement, firing, collision)
   - Enemy class (AI, patterns, health)
   - Boss class (multi-stage, special attacks)
   - Projectile class (bullets, explosions)
   - PowerUp class (pickups)
   - Particle class (effects)

6. **Game Loop & Rendering** (lines ~1000-1200)
   - Update phase (physics, AI, collision checks)
   - Render phase (canvas drawing)
   - Input handling

7. **Event Handlers & UI** (lines ~1200-1500)
   - Button click handlers
   - Keyboard and pointer input
   - HUD updates
   - Audio playback

8. **Initialization** (lines ~1500-1669)
   - Canvas setup and context
   - Event listener registration
   - Game start logic

### Adding New Features

**New Weapon Type:**
1. Add weapon definition to `WEAPONS` config
2. Create firing pattern in weapon's update method
3. Add visual representation in render method
4. Update HUD weapon display logic

**New Enemy Type:**
1. Create Enemy subclass or extend Enemy
2. Define AI pattern in update method
3. Set spawn rate in stage config
4. Add visual representation

**New Difficulty Level:**
1. Add to `DIFFICULTY` config with multipliers
2. Update difficulty selector buttons in HTML
3. Adjust enemy spawn patterns and health scaling
4. Test progression difficulty curve

### Performance Considerations

- **Collision Detection:** O(n²) broad-phase, optimized with spatial partitioning
- **Rendering:** Canvas 2D context used; requestAnimationFrame for smooth 60 FPS
- **Memory:** Entity pooling for projectiles and particles to reduce GC pressure
- **Mobile:** Touch-optimized with reduced effect options for lower-end devices

### Testing the Game

#### Manual Testing
1. Play through each difficulty level
2. Test all weapon types and upgrades
3. Verify enemy patterns and boss behavior
4. Check score calculation and combo mechanics
5. Verify sound on/off toggle works
6. Test reduced motion mode (screen shake disabled)
7. Test pause/resume functionality
8. Verify localStorage persistence (best score, difficulty)

#### Browser Compatibility
- Chrome/Edge: Full support
- Firefox: Full support
- Safari: Full support (iOS 14+)
- Mobile browsers: Touch input, reduced motion recommended

## Known Limitations & Future Improvements

### Current Limitations
- Single-file structure makes large changes challenging
- No explicit asset management (all drawing is code-generated)
- Audio uses Web Audio API with inline generation (no pre-recorded sounds)
- No network/multiplayer features
- Limited to single-player campaign mode

### Potential Enhancements
1. **Game Features:**
   - Additional weapon types (laser, spread shot, homing missiles)
   - Power-up variety (shield, invulnerability, rapid fire)
   - More boss types with unique attack patterns
   - Leaderboard system (if backend added)
   - Achievements/badges for milestones

2. **Accessibility:**
   - Better keyboard navigation for menu
   - Sound cues for important events
   - Visual indicators for sound/effects

3. **Performance:**
   - WebGL rendering for better performance
   - Asset spriting and texture atlases
   - Worker threads for collision detection
   - Progressive loading for mobile

4. **Code Quality:**
   - Modularization into separate files (ES6 modules)
   - TypeScript for type safety
   - Unit tests for game logic
   - Dedicated asset/sprite manager

## Building & Deployment

### Local Testing
Simply open `index.html` in a browser:
```bash
open index.html  # macOS
start index.html # Windows
xdg-open index.html # Linux
```

Or use a local server:
```bash
python -m http.server 8000
# Then visit http://localhost:8000/index.html
```

### Deployment
1. Copy `index.html` to web server
2. No build process required
3. No dependencies to install
4. Can be hosted on any static file server (GitHub Pages, AWS S3, etc.)

## Contributors

- Initial implementation: Sky Fighter game engine
- Latest updates: Stage progression and boss system enhancements

## License

Open source game - refer to repository for license details

---

*Last updated: October 4, 2026*
*Branch: claude/inspiring-planck-clfivf*
