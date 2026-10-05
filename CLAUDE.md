# Sky Fighter - CLAUDE.md

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single HTML5 file with integrated JavaScript and CSS. The game features a player-controlled aircraft that can switch weapons, deploy smart bombs, and engage stage bosses.

- **Tech Stack**: HTML5, Canvas API, Vanilla JavaScript, CSS3
- **Browser Target**: Modern browsers (Chrome, Firefox, Safari, Edge)
- **Responsive Design**: Mobile-friendly with touch controls and viewport scaling
- **Theme**: Dark sci-fi/military aesthetic with dynamic lighting effects

## Game Architecture

### Core Mechanics

1. **Player Aircraft**
   - Vertical scroll (auto-scrolling environment)
   - Weapon switching system
   - Health/lives system
   - Smart bomb deployment (limited charges)
   - Collision detection with enemies and hazards

2. **Weaponry**
   - Multiple weapon types switchable during gameplay
   - Weapon display in HUD (top center)
   - Ammo/charge tracking

3. **Combat System**
   - Enemy spawning and movement patterns
   - Boss encounters at stage progression
   - Scoring system with multiplier tracking
   - Visual feedback (screen flash on hit, vignette on damage)

4. **HUD Elements**
   - Health indicator (heart symbols)
   - Bomb count display
   - Current weapon indicator
   - Score and multiplier tracking
   - Responsive typography (using `clamp()` for scaling)

### Visual Feedback Systems

- **Screen Flash**: White flash on hit (`#flash.hit` animation - 0.25s)
- **Damage Vignette**: Red radial vignette on player damage (pulsing animation)
- **Heartbeat Animation**: HUD hearts animate when health is critical
- **Opacity Feedback**: Bombs and hearts show obtained vs. unavailable state
- **Color Coding**: Gold for score/multiplier, Lime for weapon, Orange for multiplier

## File Structure

```
.
├── README.md           # Basic project info with timestamp
├── index.html         # Complete game implementation (single file)
│   ├── HTML Structure
│   ├── CSS Styling & Animations
│   └── JavaScript Game Logic
└── CLAUDE.md          # This documentation
```

## Canvas & Rendering

- **Canvas ID**: `#gameCanvas`
- **Container ID**: `#container` (flex layout for centering)
- **Gradient Background**: Linear gradient from dark blue to sky blue
- **Resolution**: Responsive (scales to fit viewport)
- **Cursor**: Crosshair for targeting
- **Safe Area**: Respects mobile viewport-fit and safe-area-inset

## Mobile Considerations

- Touch controls support (touch-action: none)
- Viewport scaling with clamp() for responsive sizing
- Safe area inset support for notched devices
- Maximum scale constraints (user-scalable=no)
- Flexible layout using flexbox and percentage-based sizing

## Development Guidelines

### Adding New Features

1. **UI Changes**: Update CSS in `<style>` section
2. **Gameplay Logic**: Modify JavaScript game loop
3. **Animations**: Add to existing `@keyframes` rules or create new ones
4. **HUD Updates**: Modify `#hud` structure and corresponding JS updates

### Color Palette Reference

- **Primary Dark**: `#05122b`, `#0b1d3a` (backgrounds)
- **Sky Gradient**: `#0d2f5e` → `#1e6fd8` → `#7fc1ff`
- **UI Accent**: `#ffd54f` (gold for stats)
- **Weapon Color**: `#7CFC00` (lime green)
- **Multiplier Color**: `#ff8a65` (orange)
- **Text Shadow**: `1px 1px 3px rgba(0,0,0,0.8)`
- **Border/Glow**: `rgba(255,255,255,0.25)`
- **Damage Vignette**: Red radial gradient with `rgba(255,0,0,0.55)`

### Performance Notes

- Single-file architecture simplifies deployment
- Canvas rendering for smooth 60fps gameplay
- Backdrop-filter blur on HUD elements (check browser support)
- Touch-action: none prevents browser gestures during gameplay
- User-select: none prevents text selection during interaction

### Testing in Browser

1. **Desktop**: Open `index.html` directly in browser or serve via HTTP
2. **Mobile**: Use device testing or browser DevTools device emulation
3. **Controls**: Mouse movement or touch input for aiming
4. **Game Loop**: Real-time canvas rendering with requestAnimationFrame

## Deployment

- **Simple Hosting**: Can be hosted on any static file server
- **File Size**: Single HTML file (~60KB range based on content)
- **No Dependencies**: Vanilla JavaScript, no external libraries
- **CORS**: Not required (no external asset loading)

## Git Workflow

- **Current Branch**: `claude/inspiring-planck-sgrz50`
- **Commit Messages**: Should reference specific game mechanics or UI improvements
- **Attribution**: Include Co-Authored-By footer in commits

## Future Enhancement Ideas

- Level progression system
- Enemy type variety
- Particle effects for explosions
- Sound effects (AudioContext API)
- Leaderboard/score persistence
- Additional boss patterns
- Power-up system
- Difficulty modes
