# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML application. Players navigate an aircraft through incoming enemy waves, collect power-ups, and face increasingly difficult stage bosses.

- **Type**: Single-file HTML5/Canvas game
- **Technology Stack**: Vanilla JavaScript, HTML5 Canvas, Web Audio API
- **Target Platforms**: Desktop (mouse/keyboard) and mobile (touch controls)
- **No External Dependencies**: All code is self-contained in `index.html`

## File Structure

```
Google_4/
├── index.html      # Complete game (HTML, CSS, JavaScript)
├── README.md       # Project title
└── CLAUDE.md       # This file
```

## Game Mechanics & Architecture

### Game Loop
- **Frame Rate**: Adaptive performance detection (targets 60fps)
- **Update/Draw Pattern**: `update(dt)` → `draw()` running via `requestAnimationFrame`
- **Performance Tracking**: Automatic quality adjustment based on frame times (lines 473-482)

### Core Game States
```javascript
player       // { x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }
enemies      // Array of enemy objects
bullets      // Array of player projectiles
enemyBullets // Array of incoming attacks
boss         // Current boss (null between stages)
```

### Difficulty Settings (lines 485-489)
Three difficulties with different spawn rates, enemy speeds, and bullet counts:
- **Easy**: Lower spawn rate, slower enemies, fewer lives
- **Normal**: Balanced (default)
- **Hard**: Aggressive spawning, faster enemies

### Weapon System
- **Vulcan**: Multi-shot spread pattern (levels 1-3)
- **Laser**: Piercing shots with homing lanes (levels 1-3)
- Players cycle weapons and levels via power-ups

### Enemy Types (lines 502-506)
1. **Grunt**: Basic red enemies (1 HP, score: 10)
2. **Interceptor**: Fast orange enemies (1 HP, score: 15)
3. **Tank**: Slow purple tanks (4 HP, score: 35, appears stage 2+)

### Boss System (lines 508-515)
- **6 Boss Designs**: Unique visuals and attack patterns
- **Attack Patterns**: Each boss has 3 attack patterns that rotate
- **Pattern Types**: Radial bursts, aimed spreads, homing fire, etc. (lines 1372-1409)
- **Boss HP Scales**: Based on stage number and difficulty

### Scoring & Chain Multiplier (lines 1087-1101)
- Kill streak resets if player takes damage
- Multiplier increases every 5 kills (max 10x)
- Boss kills grant 500 base points

### Visual Effects
- **Explosions**: Particle effects for destroyed enemies/bosses
- **Screen Shake**: Dynamic intensity based on damage
- **Vignette**: Red pulse effect when health is critical
- **Flash**: White/bomb flash animations when hit
- **Reduced Motion**: Respects system `prefers-reduced-motion` setting

### Audio
- **WebAudio Synthesis**: All sounds generated procedurally (no external audio files)
- **Sound Effects**: Shoot, laser, explosions (by enemy type), bombs, power-ups, boss sounds
- **Mutable**: Players can mute via UI button

### Mobile/Touch Support
- **Touch Controls**: Virtual joystick (left 55% of screen) + FIRE/BOMB buttons
- **Mouse Controls**: Crosshair steering + click to fire
- **Keyboard Input**: Arrow keys/WASD + Space to fire, X/Shift for bombs, P/Esc to pause

## Storage & Persistence
Uses `localStorage` to save:
- `skyfighter.bestScore` - High score
- `skyfighter.muted` - Audio mute state
- `skyfighter.difficulty` - Last selected difficulty
- `skyfighter.reducedMotion` - Reduced motion preference

## Key Functions & Sections

### Game Initialization
- `resetGame()` (line 875): Resets all game state for new run
- `resize()` (line 754): Handles canvas scaling for window size

### Input Handling
- `joystickStart/Move/End()` (line 785): Touch joystick
- Canvas mouse event listeners (line 847): Steering target tracking
- Keyboard event listeners (line 769): Fire/bomb/pause controls

### Rendering
- `draw()` (line 1520): Main render function
- `drawPlane()` (line 913): Player and enemy aircraft
- `drawBossShip()` (line 970): Boss-specific rendering
- `drawClouds()` (line 1507): Parallax background

### Game Logic
- `update(dt)` (line 1148): Physics, collision, spawning
- `fireWeapon()` (line 1124): Weapon fire
- `updateBoss(dt)` (line 1411): Boss AI and patterns
- `useBomb()` (line 1103): Clear enemies, damage boss
- `endGame()` (line 1469): Game over sequence

### Collision Detection
- `rectsOverlap()` (line 1069): AABB collision helper
- Bullet-enemy collisions (line 1232)
- Bullet-boss collisions (line 1255)
- Enemy/bullet-player collisions (lines 1268-1287)
- Power-up pickup (line 1289)

### Audio Synthesis
- `beep(freq, dur, type, vol, opts)` (line 566): Tone generation
- `noiseBurst(dur, vol, opts)` (line 597): Noise generation
- `sfx` object (line 624): Pre-composed sound effects

### Difficulty/Theme System
- `THEMES` (line 493): Stage themes with sky gradients and cloud colors
- `ENEMY_TYPES` (line 502): Enemy definitions
- `BOSSES` (line 508): Boss designs
- `BOSS_PATTERNS` (line 1372): Attack patterns per boss

## Common Development Tasks

### Adding a New Boss Attack Pattern
1. Add new pattern function to lines 1324-1370 (e.g., `firePattern()`)
2. Add pattern to the boss's pattern array in `BOSS_PATTERNS` (line 1372)
3. Increase pattern count or create new boss variant

### Tuning Game Difficulty
- Modify `DIFFICULTIES` object (line 485)
- Adjust `KILLS_PER_STAGE` (line 491)
- Change boss HP multipliers in `BOSSES` (line 508)

### Adding New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502) with stats
2. Update enemy spawn logic (lines 1022-1041)
3. Add explosion sound effect to `sfx` (line 624)
4. Reference in `EXPLODE_SFX` map (line 678)

### Changing Visual Themes
- Modify `THEMES` array (line 493) with new sky gradients
- Add corresponding boss designs or modify existing ones
- Test parallax cloud rendering at different scroll speeds

### Adjusting Game Balance
- Weapon fire rates: `fireDelay` (line 1177)
- Player movement speed: `player.speed` (line 877)
- Enemy spawn rates: `spawnTimer` logic (line 1198)
- Boss attack intervals: `b.attackTimer` (line 1050)

## Performance Optimization

### Quality Tiers
Game automatically adjusts visual quality based on frame performance:
- `quality = 0`: Full effects (particles, shadow blur, glow)
- `quality = 1`: Reduced particle count
- `quality = 2`: Minimal particles, no shadows/glow

Key quality checks at line 1074 (particles), 1002 (shadows), 1583-1594 (glow effects)

### Performance Tips
- Particle count is adaptive: fewer at lower quality (line 1074)
- Shadow blur disabled on lower quality (line 1002)
- Bullet glow effects skip on quality > 0 (line 1579)

## Testing Instructions

### Desktop Testing
```bash
# Open in browser
open index.html
# or
firefox index.html
```

**Controls**:
- Mouse: Move to steer, click to fire
- Arrows/WASD: Move
- Space: Fire
- X/Shift: Bomb
- P/Esc: Pause

### Mobile Testing
- Use device developer tools to simulate touch
- Drag left side (55% of screen) to move
- Tap FIRE button (right side)
- Tap BOMB button (upper right)

### Testing Checklist
- [ ] All three difficulty levels playable
- [ ] Mouse steering and keyboard controls responsive
- [ ] Touch controls work on mobile
- [ ] Audio mute toggle works
- [ ] Reduced motion toggle disables animations
- [ ] High score persists after reload
- [ ] All 6 bosses reachable and defeatable
- [ ] Power-ups spawn and apply correctly
- [ ] Weapon cycling works (Vulcan → Laser → Vulcan)
- [ ] Screen shake and effects scale with difficulty
- [ ] Reduced motion setting prevents seizure risk

### Browser Compatibility
- Requires: ES6+ JavaScript, Canvas API, Web Audio API
- Tested: Chrome, Firefox, Safari, Edge
- Mobile: iOS Safari 12+, Chrome Mobile 60+

## Debugging Tips

### Common Issues

**Game Won't Start**
- Check browser console for errors
- Ensure Web Audio API is supported (requires secure context on HTTPS)
- Click to focus canvas first on some browsers

**Performance Problems**
- Open DevTools, check FPS in Performance tab
- Quality should auto-downgrade if sustained FPS < 40
- Check for infinite loops in update logic

**Sound Not Playing**
- Check mute state: `localStorage.getItem('skyfighter.muted')`
- Verify Web Audio context is initialized: `audioCtx` should not be null
- Some browsers require user gesture before audio plays

**Touch Controls Not Working**
- Ensure `touchControls.classList.contains('enabled')`
- Check `isTouchDevice` detection (line 778)
- Verify touch events on `moveZone` and button elements

### Debug Logging
Add to any function for quick debugging:
```javascript
console.log('Frame:', frame);
console.log('Enemies:', enemies.length);
console.log('Boss HP:', boss ? boss.hp : 'none');
console.log('Player:', player.x, player.y);
```

## Architecture Notes

### Single-File Design Rationale
- **Pro**: Easy deployment, no build step needed, simple to modify
- **Pro**: All game state in one scope, easy to trace logic
- **Con**: No module separation; changes need careful testing
- **Con**: Large file size (1670 lines)

### Code Organization (Top to Bottom)
1. HTML markup (lines 362-420)
2. CSS styling (lines 10-358)
3. JavaScript initialization and event setup
4. Game state and constants
5. Audio system
6. Rendering functions
7. Game logic (update, collisions, boss AI)
8. Main loop and event bindings

### Performance Characteristics
- Update logic: O(n) for enemies, O(n*m) for collision detection
- No spatial partitioning; acceptable for ~20-30 enemies
- WebAudio contexts reused; created on first user interaction

## Future Enhancement Ideas

1. **High Score Leaderboard**: Post to server backend
2. **More Boss Patterns**: Expand `BOSS_PATTERNS` array
3. **Additional Weapons**: Add sword, spread gun, etc.
4. **Upgradeable Stats**: Persistent speed/health upgrades
5. **Achievements**: UI badges for milestones
6. **Replays**: Record and playback game sessions
7. **Sound Volume Control**: Allow adjusting audio levels per SFX
8. **Accessibility**: Screen reader narration support
9. **Fullscreen Mode**: Canvas fullscreen API
10. **Progressive Web App**: Offline support via Service Worker

## Related Documentation

- **Game Design**: 3-stage difficulty ramp, increasing enemy variety per stage
- **Accessibility**: Reduced motion, screen reader announcements, keyboard + mouse + touch support
- **Storage**: All persistent data uses `localStorage` (no server needed)

---

**Last Updated**: 2026-09-21  
**Maintained By**: Claude Code Agent
