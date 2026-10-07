# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 web application. Players control an aircraft, dodge enemy fire, collect power-ups, and defeat stage bosses across multiple levels.

- **Repository:** ubban/Google_4
- **Branch:** claude/inspiring-planck-1z9mc8
- **Tech Stack:** Vanilla HTML5, CSS3, JavaScript (Canvas API)
- **Game Engine:** Canvas 2D rendering with custom physics and collision detection

## Project Structure

```
Google_4/
├── index.html       # Complete game implementation (1669 lines)
├── README.md        # Basic project timestamp
└── CLAUDE.md        # This file
```

## Game Architecture

The entire game is implemented in a single `index.html` file with three main sections:

### 1. HTML Structure (lines 1-60)
- Meta tags for mobile-responsive design
- Safe area insets for notch/island support
- Game canvas and UI overlay elements

### 2. CSS Styling (lines 61-400)
- **Layout:** Fixed fullscreen container with centered canvas
- **HUD Elements:** 
  - Stats display (lives, bombs, weapons, multiplier)
  - Boss health bar
  - Stage banner notifications
  - Combo counter
  - Damage vignette and flash effects
- **Animations:** 
  - Heart beat (critical health)
  - Stage/combo pulse effects
  - Boss bar transitions
- **Responsive:** Uses `clamp()` for scalable UI across devices
- **Theming:** Dark color scheme (#05122b background, #0b1d3a accents)

### 3. JavaScript Game Logic (lines 400+)

#### Core Game State
```javascript
gameState = {
  score: 0,
  multiplier: 1,
  health: 3,
  bombs: 2,
  level: 1,
  stage: 1,
  waveIndex: 0
}
```

#### Player System
- **Position:** Canvas center, mouse/touch controlled
- **Health:** 3 lives (displayed as hearts)
- **Weapons:** Multiple weapon types with cooldowns
  - Basic bullets
  - Spread shots
  - Homing missiles
  - Area attacks
- **Smart Bombs:** Limited use, clears screen of projectiles

#### Enemy System
- **Enemy Classes:**
  - Standard fighters (weak, fast)
  - Medium ships (balanced)
  - Tanks (slow, durable)
  - Bosses (high HP, multiple attacks)
- **Wave System:** Progressive difficulty with stage progression
- **Spawning:** Waves spawn from top with patterns and timing

#### Collision & Physics
- **Bounding Box Collision:** Rectangle-based collision for efficient detection
- **Particle Effects:** Explosions, sparkles, hit effects
- **Projectile Management:** Auto-cleanup when off-screen
- **Multiplier System:** Score multiplier increases with consecutive hits

#### Boss Encounters
- **Stage Boss Mechanics:**
  - Health bar display
  - Multiple attack patterns
  - Dash movements
  - Pattern variations per stage
- **Boss Types:** At least 3 distinct bosses across stages

#### UI & Visual Feedback
- **HUD:** Real-time stats (health, bombs, weapon, score multiplier)
- **Notifications:** Stage banners, combo counters, damage vignette
- **Flash Effects:** Screen flash on hit/bomb detonation
- **Color Coding:** Yellow scores, green weapons, orange multiplier

## Key Features

### Gameplay Mechanics
1. **Weapon System:** Click to switch weapons, each with unique firing pattern
2. **Power-ups:** Dropped by enemies, auto-pickup when near player
3. **Score Multiplier:** Increases with consecutive hits, resets on damage
4. **Progressive Difficulty:** Each stage increases enemy count and complexity
5. **Boss Fights:** Story progression blocked behind boss defeats

### Mobile Support
- Touch/pointer controls for mobile and desktop
- Safe area padding for notch devices
- Crosshair cursor on supported browsers
- Viewport scaling prevents zoom on input

### Visual Polish
- Gradient sky background
- Particle effects for explosions
- Screen vignette on damage (red pulsing)
- Smooth animations throughout
- Text shadows for readability

## Development Notes

### Performance Optimization
- Single canvas for all rendering
- Efficient collision detection (bounding box only)
- Object pooling for particles and projectiles
- No external dependencies (vanilla JS)

### Code Organization
- Game loop: requestAnimationFrame for smooth 60fps
- State management: Centralized gameState object
- Modular functions: update(), draw(), spawnWave(), etc.
- Event listeners: Mouse/touch input handling

### Responsive Design
- CSS `clamp()` for fluid sizing
- Viewport-relative units (vw, vh)
- Safe area support for mobile notches
- Fullscreen canvas scaling

## How to Work on This Codebase

### Making Changes
1. All code is in `index.html`
2. CSS is in `<style>` block (lines ~61-400)
3. JavaScript is in `<script>` block (lines ~401+)
4. Changes take effect immediately when file is saved

### Testing Changes
1. Open `index.html` in a web browser
2. Test gameplay mechanics (movement, shooting, weapon switching)
3. Test on mobile devices for touch controls and safe area support
4. Monitor console for any errors or warnings

### Common Tasks

**Adding a New Weapon Type:**
- Define new weapon properties in weapon system
- Add firing logic in update loop
- Add UI display in HUD
- Test fire rate, damage, and spread pattern

**Adjusting Game Balance:**
- Enemy health values in enemy spawning code
- Boss stats in boss encounter section
- Player health/bomb counts in gameState initialization
- Damage values in collision detection

**Adding a New Stage:**
- Create new wave patterns in wavePatterns array
- Design new boss with unique attack patterns
- Update multiplier/difficulty scaling
- Test progression and difficulty curve

**Modifying Visuals:**
- CSS color values for theming
- Canvas drawing code for sprite/effect appearance
- Animation keyframes for timing
- HUD layout and styling

## Technical Constraints

1. **Single File:** All code in one HTML file (1669 lines)
2. **No Build Process:** Direct vanilla JS, no transpilation
3. **No External Libraries:** Pure HTML5 Canvas API
4. **Browser Requirements:** Modern browser with Canvas 2D support
5. **Mobile First:** Designed for responsive mobile play

## Known Limitations

- Single-file architecture limits code organization complexity
- No persistent save system
- No sound effects or music
- Limited to canvas rendering (no WebGL)
- No networking/multiplayer capability

## Future Enhancement Ideas

1. **Audio System:** Add sound effects and background music
2. **Persistent Progression:** Save stage progress to localStorage
3. **Leaderboard:** Track high scores
4. **Difficulty Modes:** Easy/Normal/Hard with different parameters
5. **Additional Stages:** More boss encounters and wave patterns
6. **Power-up Variety:** More diverse power-up types
7. **Visual Customization:** Selectable player/weapon skins
8. **Accessibility Features:** Colorblind modes, control remapping

## Attribution

- Created with Claude Code
- Game concept inspired by classic shmups (Raiden series)
- Built entirely in vanilla HTML5/CSS3/JavaScript

---

**Last Updated:** 2026-10-07
**Current Version:** 1.0 - Initial Sky Fighter release
