# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a browser-based vertical shoot-em-up game inspired by the classic Raiden arcade series. It's a single-file HTML5 Canvas game featuring dynamic weapon systems, boss encounters, and progression through stages.

**Repository**: ubban/google_4
**Main File**: `index.html` (1,669 lines)
**Status**: Active development (single-engine, JavaScript-based)

---

## Architecture

### Single-File Structure
The entire game is contained in `index.html`, combining:
- **HTML Structure**: Game container and UI overlay
- **CSS Styling**: In-game HUD, effects animations, modal dialogs
- **JavaScript Logic**: Core game engine, entity management, input handling (lines 423+)

This monolithic approach works well for a game scope, keeping all systems tightly integrated.

### Core Game Engine (IIFE Pattern)
- **Starting Point**: Self-invoking function at line 423
- **Canvas Setup**: 2D canvas context for rendering
- **Game Loop**: requestAnimationFrame-based update/render cycle
- **State Management**: Global game variables (score, health, stage, etc.)

---

## Key Game Systems

### 1. **Player System**
- **Health**: Lives system (displayed as hearts ❤️)
- **Position**: Constrained to canvas bounds; smooth movement
- **Collision**: Circular hitbox for enemy bullets and enemies
- **Weapon Switching**: Multiple weapon types that upgrade with pickups
  - Vulcan (spread bullet pattern)
  - Laser (focused beam)
  - Others (TBD based on pickup drops)
- **Smart Bombs**: Limited bombs that clear screen when activated

### 2. **Weapon System**
- **Weapon Types**: Name/level displayed in HUD
- **Weapon Levels**: Upgrade through pickup collection (level 1-5+ progression)
- **Weapon Colors**: Each weapon type has distinct firing pattern/appearance
- **Fire Rate**: Automatic firing; rate increases with level
- **Visual Feedback**: HUD shows current weapon name and level (#weaponStat)

### 3. **Enemy System**
- **Enemy Types**: Mix of basic enemies and stage bosses
- **Enemy Spawning**: Waves or continuous spawning based on stage progression
- **Projectiles**: Enemies fire bullets with collision detection
- **Drops**: Defeated enemies drop weapons and utility items
- **Boss Encounters**: Special multi-stage boss battles per stage

### 4. **Scoring & Progression**
- **Score**: Real-time score tracking (visible in HUD #score)
- **Best Score**: Persistent high score storage (in HUD #best)
- **Score Multiplier**: Combo multiplier (#mult stat) for consecutive kills
- **Stage System**: Game progresses through numbered stages (displayed on entry)
- **Boss Progression**: Each stage concludes with a boss battle

### 5. **Difficulty Levels**
- **Easy**: Reduced enemy fire, slower progression
- **Normal**: Balanced difficulty (default)
- **Hard**: Increased enemy fire, faster enemy waves
- **Selection**: Chosen before game start via #diffRow buttons

### 6. **Visual Effects**
- **Animations**:
  - `heartBeat`: Critical health pulse
  - `comboPulse`: Combo notification popup (#comboPop)
  - `stagePulse`: Stage banner entrance (#stageBanner)
  - `vignettePulse`: Damage vignette red pulse
  - `flashHit`: Screen flash on player hit
  - `flashBomb`: Larger flash on bomb explosion
- **Vignette Effect**: Red screen edge pulsing when taking damage (damage indicator)
- **Screen Flash**: White flash on impacts; larger flash on bomb

### 7. **Audio System**
- **Mute Toggle**: #muteBtn controls audio playback
- **Expected Sounds**: Weapon fire, enemy death, power-ups, boss phases (implementation TBD)

### 8. **Input Systems**
- **Keyboard**: WASD or Arrow keys for movement; Space for fire; Shift for smart bomb
- **Mouse/Touch**: 
  - Desktop: Mouse aim at cursor position
  - Mobile: Two-finger touch zones (#moveZone, #fireZone)
  - Touch controls enabled on mobile devices (#touchControls.enabled)
- **Pause**: #pauseBtn pauses game loop

### 9. **Accessibility**
- **Screen Reader Support**: #srLive aria-live region for game announcements
- **Reduced Motion Toggle**: #motionBtn respects prefers-reduced-motion
- **Semantic HTML**: Proper heading hierarchy and labeling
- **Safe Area Insets**: Mobile notch support via env(safe-area-inset-*)

---

## UI/UX Components

### In-Game HUD (#hud)
Located top-center during gameplay, displays:
- Score (real-time)
- Best (high score)
- Multiplier (combo bonus)
- Weapon name + level
- Hearts (health visualization)
- Bombs (smart bomb availability)

### Overlay System (#overlay)
Modal dialog for:
- **Start Screen**: Title, difficulty selection, controls hint
- **Game Over Screen**: Final score, stage reached, leaderboard
- **Animations**: Card entrance animation (cardIn)

### Boss Health Bar (#bossBarWrap)
- Shows boss name and health percentage
- Only visible during boss encounters
- Top-center positioning with gradient fill

### Control Layout
- **Desktop**: WASD/Arrows + Space + Shift keys
- **Mobile**: Touch zones at bottom corners; visual feedback on tap

---

## Data Persistence

### High Score
- Stored in `localStorage` (key TBD, likely 'skyFighterBest')
- Compared on game over to determine "New Best" message
- Synced to #bestEl and #finalBestEl

### Game Difficulty
- Selected before each game run
- Affects enemy spawn rates, damage, and projectile count

---

## Development Notes

### Code Organization
- **Single Function Scope**: All code lives within IIFE to avoid global pollution
- **Constants Section**: At top of script (after variable declarations)
- **Helper Functions**: Collision detection, entity updates, rendering
- **Main Loop**: `update()` → `render()` cycle at requestAnimationFrame
- **Event Listeners**: Keyboard, mouse, touch, button click handlers

### Performance Considerations
- **Canvas Rendering**: Cleared and redrawn each frame
- **Entity Arrays**: Enemies, bullets, explosions managed in lists
- **Collision Checks**: Likely grid-based or simple circle-based
- **Mobile Optimization**: Touch zones, reduced motion support

### Known Patterns
- **Entity Manager**: Enemies, bullets, pickups likely stored in arrays
- **Collision System**: Player circle vs. enemy bullets/enemies
- **Particle Effects**: Explosions and hits may use particle system
- **State Machine**: Game states (start, playing, paused, gameOver)

---

## How to Contribute

### Adding New Features
1. **New Weapon Type**: Add weapon object to weapon array, define firing pattern
2. **New Enemy Type**: Create enemy class/object, define AI behavior and projectile pattern
3. **New Boss**: Implement boss entity with phase transitions and attack patterns
4. **Visual Effects**: Add CSS animations and wire into game update cycle

### Debugging
- Open browser DevTools (F12)
- Check console for errors
- Use `canvas.width/height` to verify render surface
- Inspect element to check HUD updates in real-time

### Testing
- **Keyboard**: Test WASD, Space, Shift on desktop
- **Touch**: Test swipe on mobile devices
- **Difficulty**: Verify each difficulty level affects game
- **Audio**: Toggle mute on/off; verify sound plays
- **Effects**: Test reduced-motion toggle; verify animations respect it

---

## Future Enhancements (Potential Roadmap)

- [ ] Multiple weapon types with distinct fire patterns
- [ ] Power-up variety (multi-shot, shields, temporary invulnerability)
- [ ] Stage variety with themed backgrounds and music
- [ ] Boss AI with multiple attack phases
- [ ] Leaderboard/scoring system refinement
- [ ] Mobile app wrapper (Cordova/Capacitor)
- [ ] Sound effects and background music
- [ ] Particle effects system for impacts and explosions
- [ ] Screen shake and intensity-based feedback

---

## Browser Compatibility

- **Modern Browsers**: Chrome, Firefox, Safari, Edge (all supporting Canvas 2D)
- **Mobile**: iOS Safari, Chrome Android
- **Viewport**: Responsive design with clamp() for scaling
- **Fallback**: Basic Canvas support required (no WebGL)

---

## File Structure

```
/
├── index.html          # Entire game (HTML + CSS + JavaScript)
├── README.md           # Basic project info
└── Claude.md           # This documentation file
```

---

## Key Variables & Elements

| Element ID | Purpose | Type |
|-----------|---------|------|
| `gameCanvas` | 2D rendering surface | Canvas element |
| `overlay` | Game state dialogs | Modal container |
| `hud` | In-game statistics | Display div |
| `bossBarWrap` | Boss health display | Conditional display |
| `vignette` | Damage effect overlay | Visual effect |
| `touchControls` | Mobile input zones | Touch targets |
| `srLive` | Screen reader announcements | Accessible output |

---

## Contact & Attribution

- **Original Author**: Claude Haiku 4.5 AI
- **Session**: https://claude.ai/code/session_01DZEQvqxcWE71FCAbion6BL
- **Last Updated**: 2026-09-23
