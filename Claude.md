# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single-page HTML5 canvas application. It's a browser-based arcade game featuring airplane combat, multiple weapon systems, boss battles, and stage progression.

### Key Features
- **Gameplay**: Vertical scrolling shooter with waves of enemies culminating in boss battles
- **Weapons System**: Vulcan/Laser with 3 power levels, switchable during gameplay
- **Smart Bombs**: Limited-use screen-clearing attacks that damage bosses
- **Stage System**: Multiple waves per stage with stage-specific visual themes
- **Boss Battles**: Three unique boss designs (Vanguard, Behemoth, Specter) that cycle by stage
- **Power-ups**: Weapon level upgrades, bomb pickups, extra lives
- **Mobile Support**: Responsive design with touch controls and mobile optimization
- **Persistence**: Saves game state (high score, unlocked content)
- **Audio**: In-game sound effects and background music
- **Polish**: Parallax scrolling clouds, screen shake on hits, vignette effects, combo popups, glassmorphic UI

## Project Structure

```
Google_4/
├── index.html          # Single-file HTML5 canvas game (main deliverable)
├── README.md           # Project metadata
└── Claude.md           # This file
```

The entire game is contained in a single `index.html` file (~58KB) with embedded CSS and JavaScript.

## Technical Stack

- **Platform**: HTML5, CSS3, JavaScript (Vanilla/No frameworks)
- **Graphics**: Canvas 2D API for rendering
- **Storage**: localStorage for persistence
- **Input**: Keyboard (WASD/Arrow keys) and Mouse/Touch for controls
- **Build**: No build process—direct browser delivery

## Game Architecture

### Core Systems

1. **Game State**
   - Player position, health, weapon level, bombs
   - Enemy list and bullet manager
   - Boss state (current boss, health, pattern)
   - Score, combo multiplier, stage progression
   - Audio settings

2. **Rendering Pipeline**
   - Background (parallax clouds, sun glow)
   - Game objects (player, enemies, bullets, power-ups, boss)
   - HUD (health, score, weapon, bombs, wave info)
   - Visual effects (screen shake, vignette, hit flash, combo popups)
   - Pause overlay

3. **Input Handling**
   - Keyboard controls: WASD or Arrow keys to move, Space to fire, E to use bomb, P to pause
   - Mouse/Touch crosshair-based aiming
   - Touch-based mobile controls

4. **Collision Detection**
   - Player-bullet collisions (takes damage)
   - Player-pickup collisions (gains power-up)
   - Enemy bullet-player collisions
   - Boss health tracking per hit
   - Bullet-enemy collisions (for knockback/damage)

### Enemy Types

- **Grunt**: Basic fighter, no special behavior
- **Weaver**: Interceptor with horizontal movement pattern
- **Tank**: Heavy enemy with multiple health points and visual pip tracker

### Boss Designs (Cycle by Stage)

- **Vanguard**: Fast, aggressive pattern
- **Behemoth**: Heavy, slow, multi-phase
- **Specter**: Erratic movement, pattern attacks

Each boss:
- Has a dedicated health bar
- Cycles through distinct attack patterns
- Drops quality power-ups on defeat
- Triggers stage progression when defeated

### Power-ups

- **Weapon Level**: Increases bullet damage and fire rate (max level 3)
- **Bomb**: Adds one use to smart bomb capacity
- **Extra Life**: Restores one health point (max 5)

### Audio

- Background music loop (in-game ambience)
- Sound effects for:
  - Gunfire (Vulcan/Laser)
  - Enemy death
  - Boss hit/death
  - Power-up pickup
  - Bomb explosion
  - Player hit/death
  - Pause/unpause

## Game Flow

1. **Start Screen**: Game title, instructions, high score display
2. **Stage Waves**: 3–5 waves of escalating difficulty per stage
3. **Boss Battle**: Single powerful enemy with multi-pattern attack
4. **Stage Clear**: Brief pause, then progression to next stage
5. **Game Over**: On player death (health = 0); shows score and offers retry
6. **Pause**: P key or pause button toggles in-game pause overlay

## Persistence

- **High Score**: Saved to localStorage, persists across sessions
- **Game Settings**: Audio preference saved per session
- **Stage Unlocks**: Future extension point for unlocked content

## Performance Considerations

- **Canvas Rendering**: Direct pixel-by-pixel updates (no DOM overhead)
- **Object Pooling**: Bullets and particles reused to minimize GC
- **Frame Budget**: 60 FPS target; requestAnimationFrame for smooth updates
- **Mobile Optimization**: Reduced particle effects, optimized touch handling

## Testing

The game has been verified with headless Playwright testing:
- ✓ Firing (bullets spawn and move correctly)
- ✓ Collisions (player damage, enemy death, boss health tracking)
- ✓ Weapon pickups (level system works)
- ✓ Boss spawn/attack patterns (all bosses appear and attack)
- ✓ Boss death/stage advance (stage progression works)
- ✓ No console errors in all scenarios

## Development Guidelines

### Adding New Content

**New Enemy Type**:
1. Add enemy class/type constant (e.g., ENEMY_LIGHT, ENEMY_HEAVY)
2. Implement movement/attack logic in enemy update loop
3. Add spawn weight to wave configuration
4. Test collision and rendering

**New Boss**:
1. Create boss pattern array (position waypoints, attack timings)
2. Add to boss roster (Vanguard/Behemoth/Specter cycle)
3. Assign stage(s)
4. Test pattern execution and health tracking

**New Weapon**:
1. Add weapon constant and fire rate/damage values
2. Implement bullet spread/pattern in fire method
3. Add visual variant (color, trail)
4. Test switching and level progression

**New Visual Effect**:
1. Add to particle system or screen-shake handler
2. Tie to appropriate game event (hit, combo, power-up)
3. Test on various screen sizes and browsers

### Code Style

- Vanilla JavaScript (no external libraries beyond game code)
- Single-file structure for easy deployment
- Descriptive variable/function names
- Comments for non-obvious logic (pattern state machines, collision math)
- Responsive canvas scaling (maintains aspect ratio on mobile)

## Known Limitations & Future Work

- **Single-file constraint**: Makes modularization difficult but simplifies deployment
- **No online leaderboard**: High score is local-only (localStorage)
- **Limited mobile UI**: Touch controls work but could benefit from on-screen buttons
- **No accessibility**: Could add keyboard-only navigation, audio descriptions
- **Difficulty scaling**: Currently fixed per stage; dynamic difficulty not yet implemented

## Browser Compatibility

- **Desktop**: Chrome, Firefox, Safari, Edge (all modern versions)
- **Mobile**: iOS Safari, Chrome Android (iOS 12+, Android 6+)
- **Fallback**: Game degrades gracefully; no fallback UI for no-canvas browsers

## How to Run

1. Open `index.html` in a modern web browser
2. Game starts immediately on page load
3. Use keyboard (WASD/Arrows) or mouse (click to aim) to play
4. Press P to pause, E to use bomb, Space to fire

## Deployment

- Single static file: copy `index.html` to web server
- No build process, no dependencies
- Works offline (all code embedded)
- Optimized for both desktop and mobile viewing

---

**Last Updated**: 2026-10-04  
**Repository**: ubban/google_4  
**Branch**: claude/inspiring-planck-rtb7k1
