# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML5 application. Players control a fighter plane, switch between weapons, drop smart bombs, and battle stage bosses. The entire game is contained in `index.html` (1,669 lines) with inline CSS and JavaScript.

## Project Structure

```
Google_4/
├── index.html           # Main game file (HTML, CSS, JS all-in-one)
├── README.md            # Minimal readme
└── .git/                # Git repository
```

**Architecture Note**: This is a single-file web application. All game logic, rendering, styling, and UI are contained within `index.html`. No external dependencies or build process required.

## Game Mechanics

### Core Systems

1. **Player Mechanics**
   - Controllable fighter plane with mouse/touch crosshair targeting
   - Three lives system (hearts)
   - Weapon switching (different fire patterns and damage)
   - Smart bombs (limited resources) for crowd control

2. **Enemy System**
   - Stage-based enemy waves
   - Boss encounters at stage completion
   - Collision detection with bullets and bombs
   - Health system for each enemy type

3. **Scoring & Progression**
   - Score tracking displayed in HUD
   - Kill combo multiplier system
   - Stage progression with banner displays
   - Boss health bar UI

4. **Visual Feedback**
   - Health bar display (HUD)
   - Bomb counter
   - Weapon stat indicator
   - Hit flash effects
   - Vignette effect on damage
   - Combo pop animation
   - Stage transition banners

### HUD Elements (IDs/Classes)

- `#hud` - Main heads-up display
- `#hearts` - Lives display (3 hearts max)
- `#bombs` - Smart bomb counter
- `#weaponStat` - Current weapon indicator
- `#multStat` - Combo multiplier
- `#bossBarWrap/#bossBarFill` - Boss health bar
- `#comboPop` - Combo announcement
- `#stageBanner` - Stage transition display
- `#vignette` - Red damage vignette effect
- `#flash` - Hit/bomb flash effect
- `#topBar` - Top-right control buttons

## Code Organization

### CSS Styling (Lines ~10-500)

- **Layout**: Fixed canvas container with responsive sizing
- **Animations**: 
  - `heartBeat` - Critical health pulse
  - `flashHit` - Damage flash
  - `flashBomb` - Bomb explosion flash
  - `comboPulse` - Combo number pop
  - `stagePulse` - Stage banner
  - `vignettePulse` - Damage vignette
- **Responsive Design**: Uses `clamp()` for fluid sizing across devices
- **Theme**: Dark color scheme (#05122b background, neon accents)
- **Mobile Support**: `safe-area-inset` for notches, touch-action handling

### HTML Structure (Lines ~500-700)

- `#container` - Main wrapper
- `#gameCanvas` - Canvas element for game rendering
- `#hud` - Heads-up display container with stats
- Various effect/animation containers (`#flash`, `#vignette`, `#comboPop`, etc.)
- `#overlay` - Modal overlay for menus/pause

### JavaScript Game Logic (Lines ~700+)

**Key Global Objects:**

- `gameState` - Current game state (playing, paused, menu, etc.)
- `player` - Player object (position, health, weapon, bombs)
- `enemies[]` - Active enemies
- `bullets[]` - Active bullets
- `bosses[]` - Boss entities
- `explosions[]` - Explosion particles
- `score` - Current game score
- `stage` - Current stage number

**Key Functions to Implement/Maintain:**

- `update()` - Main game loop update
- `render()` - Canvas rendering (clear, draw player, enemies, effects)
- `handleInput()` - Mouse/touch control handling
- `fireWeapon()` - Player weapon firing
- `dropBomb()` - Smart bomb deployment
- `checkCollisions()` - Bullet-enemy and bomb-enemy collision detection
- `updateHUD()` - Update display elements
- `nextStage()` - Stage progression
- `gameOver()` - Handle game end state

**Input Handling:**

- Mouse: Position tracked via `mousemove` for crosshair aiming
- Touch: `touchstart`, `touchmove`, `touchend` for mobile play
- Buttons: Click handlers for pause, controls, restart

## Development Guidelines

### Code Style

- **No external frameworks** - Vanilla JavaScript only
- **Canvas-based rendering** - Use Canvas 2D API for all drawing
- **Single file structure** - Keep all code in `index.html`
- **Responsive CSS** - Use `clamp()` and viewport units for mobile compatibility

### When Adding Features

1. **New enemy types**: Add to enemy spawning logic, define sprite/collision properties
2. **New weapons**: Implement firing pattern, register in weapon switch system
3. **New visual effects**: Add CSS animations in `<style>` block, trigger with classes
4. **UI changes**: Modify HUD HTML structure and update `updateHUD()` function
5. **Game states**: Add to `gameState` object and handle in input/render loops

### Performance Considerations

- **Collision detection**: Currently checked every frame - optimize with spatial partitioning if frame rate drops
- **Canvas clearing**: Full clear per frame - consider dirty rectangle invalidation for optimization
- **Particle effects**: Manage explosion particle count to prevent memory issues
- **Event listeners**: All setup in initialization - clean up on game over

### Testing

- Open `index.html` in a browser to test locally
- Test on mobile devices for touch responsiveness
- Verify HUD displays correctly at different viewport sizes
- Check collision detection with varied enemy patterns

## Key Numbers (Game Balance)

- **Max health**: 3 lives
- **Max bombs**: Limited per stage (configurable)
- **Boss stages**: Appear after wave completion
- **Score multiplier**: Increases with kill combos
- **Enemy spawn rate**: Varies by stage difficulty

## Future Maintenance

- Keep `index.html` under 2000 lines for maintainability
- Consider separating into multiple files if exceeds 3000 lines
- Document any new game mechanics in this file
- Update HUD element list if adding new displays
- Test on various devices (desktop, tablet, mobile)

---

**Last Updated**: 2026-10-07  
**Branch**: `claude/inspiring-planck-y6iwnc`  
**Maintained By**: Claude Code Agent
