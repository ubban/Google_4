# Sky Fighter - Developer Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file web application. Players navigate a fighter plane through stages, collect weapon power-ups, and defeat boss squadrons.

### Key Features
- **Vertical scrolling shooter** with Raiden-style gameplay
- **Three difficulty levels** (Easy, Normal, Hard) with adjustable parameters
- **Six thematic stages** with distinct visual themes and colors
- **Weapon system** with multiple gun types (Vulcan, Blaster, Spread Gun)
- **Smart bomb mechanic** for massive screen-clearing damage
- **Boss encounters** at the end of each stage with health bars
- **Scoring system** with combo multipliers
- **Full mobile support** with touch controls and joystick
- **Accessibility features** including reduced motion preferences and screen reader support

## Architecture

### Technology Stack
- **Single-file deployment**: All HTML, CSS, and JavaScript in `index.html`
- **Canvas rendering**: 2D canvas for game graphics
- **LocalStorage**: Persists high scores, mute state, difficulty, and motion preferences
- **No external dependencies**: Fully self-contained

### File Structure
```
/home/user/Google_4/
├── index.html         # Complete game application (HTML + CSS + JavaScript)
├── README.md          # Minimal readme
└── CLAUDE.md          # This file
```

## Game Mechanics

### Core Systems

#### Player
- **Movement**: WASD, Arrow keys, or mouse following
- **Weapon**: Space bar or Fire button to shoot
- **Bombs**: X, Shift, or Bomb button (limited uses per stage)
- **Lives**: 3 (Easy: 4) hearts shown at top; losing all ends the game

#### Enemies
- **Spawning**: Progressive waves with increasing difficulty
- **Behavior**: Downward movement with occasional shooting
- **Types**: Small fighters and larger enemies with different HP
- **Drops**: Weapon upgrades and health restoration

#### Weapons
1. **Vulcan** (default): Rapid-fire straight shots
2. **Blaster**: More powerful centered shots
3. **Spread Gun**: Fan pattern shots covering wider area

#### Bosses
- **One boss per stage** with unique attack patterns
- **Health bar** displayed at top during boss encounters
- **Increased enemy spawning** during boss fights
- **Difficulty scaling**: Boss HP multiplied by difficulty setting

### Scoring
- **Kill rewards**: 100-500 points per enemy
- **Combo multiplier**: Increases with consecutive hits, resets on damage
- **Best score**: Persisted in localStorage with "New Best!" notification
- **Visual feedback**: Combo counter pop-up animations

## Development Guide

### Key Constants

```javascript
// Difficulty settings (first 45 lines of game logic)
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
};

// Progression thresholds
const KILLS_PER_STAGE = 18;

// Visual themes (6 stages)
const THEMES = [
  { sky: [...], cloud: '...', sun: '...', name: 'Daylight Front' },
  // ... 5 more themes
];
```

### Modifying Game Parameters

To adjust difficulty balance:
1. Edit `DIFFICULTIES` object for spawn rates, enemy speed, and lives
2. Modify `KILLS_PER_STAGE` to change progression speed
3. Adjust weapon parameters (damage, fire rate) in weapon definitions
4. Scale boss HP using `bossHpMul` multiplier

### Adding New Themes

Add to the `THEMES` array:
```javascript
{
  sky: ['#color1', '#color2', '#color3'],  // gradient colors
  cloud: 'r,g,b',                           // cloud color
  sun: 'r,g,b',                             // sun/object color
  name: 'Theme Name'
}
```

### Touch Controls Implementation

Mobile/touch support uses:
- **Move Zone** (left 55% of screen): Drag to position player
- **Action Zone** (right 45%): Fire and Bomb buttons
- **Joystick Base**: Shows when touch-enabled, centers stick on move zone

### localStorage Keys

```javascript
const STORAGE_BEST = 'skyfighter.bestScore';
const STORAGE_MUTE = 'skyfighter.muted';
const STORAGE_DIFF = 'skyfighter.difficulty';
const STORAGE_REDUCED = 'skyfighter.reducedMotion';
```

## Performance Optimization

The game includes **adaptive quality** based on frame time:
- **Quality 0** (full): All visual effects enabled
- **Quality 1** (medium): Some effects reduced
- **Quality 2** (low): Minimal effects

The system monitors the last 45 frames and adjusts quality to maintain 40+ FPS.

### Reduced Motion Support

For accessibility, respects `prefers-reduced-motion` preference:
- Disables screen shake animations
- Removes vignette pulses
- Disables combo pop-up animations
- Preserves gameplay (no functional removal)

## Testing Checklist

Before deploying changes:

### Gameplay
- [ ] All difficulty levels have balanced enemy spawn rates
- [ ] Weapon progression feels smooth across stages
- [ ] Boss encounters are challenging but fair
- [ ] Scoring and multipliers work correctly

### UI/UX
- [ ] Stage banners display correctly
- [ ] Boss health bar shows during encounters
- [ ] HUD stats are always visible and readable
- [ ] Overlay (start/end) screens display properly

### Accessibility
- [ ] Reduced motion preferences respected
- [ ] Screen reader announcements work
- [ ] Keyboard controls fully functional
- [ ] Touch controls work on mobile

### Performance
- [ ] Maintains 50+ FPS on desktop
- [ ] Maintains 30+ FPS on mobile
- [ ] No memory leaks during long sessions
- [ ] Quality adapts smoothly with frame drops

## Deployment

1. **Single file**: Just deploy `index.html`
2. **Web server**: Can be served by any HTTP server
3. **Local testing**: Open directly in browser (file:// protocol works)
4. **Mobile**: Responsive design works on all screen sizes

## Known Limitations

- **Single-file design**: Difficult to split into modules
- **No save state**: Game restarts on page reload
- **Canvas rendering**: CPU-bound, not GPU-accelerated
- **Audio**: No sound effects implemented
- **No multiplayer**: Single-player only

## Future Enhancement Ideas

1. **Sound effects and background music** with volume control
2. **Additional weapon types** and power-up drops
3. **More boss varieties** with unique attack patterns
4. **Stage-specific obstacles** and environmental hazards
5. **Particle effects** for explosions and impacts
6. **Leaderboard integration** with cloud sync
7. **Mobile app wrapper** (Electron, React Native)
8. **Replay system** to save and replay games
9. **Customization** (plane skins, weapon colors)
10. **Story mode** with narrative between stages

## Debugging Tips

### Browser Console
- Check for JavaScript errors with F12
- Monitor performance with DevTools Performance tab
- Use `localStorage.clear()` to reset saved preferences

### Common Issues
- **Game not starting**: Check browser console for errors
- **Touch controls not working**: Verify `touchControls` element is visible
- **Performance drops**: Check DevTools Performance tab, look for frame drops

## Notes for Claude Code

This is a complete, self-contained game that:
- Requires no build process
- Has no dependencies to manage
- Can be modified directly in this single file
- Should be tested in a browser after changes
- Uses canvas API and vanilla JavaScript exclusively

When making changes:
1. Keep the game logic in the main IIFE
2. Test all difficulty levels
3. Verify mobile responsiveness
4. Check accessibility features still work
5. Monitor frame rate during testing

---

**Created**: 2026-10-18
**Game Type**: Vertical Shoot-'em-up (Raiden-style)
**Technology**: HTML5 Canvas, Vanilla JavaScript
**Status**: Active Development
