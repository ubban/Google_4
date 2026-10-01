# Sky Fighter — Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in vanilla HTML5, CSS, and JavaScript. It's a single-file (~1700 lines) web game playable directly in any modern browser—no build tools, dependencies, or assets needed.

The game combines classic arcade action (shooting, power-ups, bosses) with modern web APIs (WebAudio synthesis, canvas rendering, localStorage persistence, touch controls).

### Quick Links
- **Main File**: `index.html` (complete game)
- **Playing**: Open `index.html` in a web browser
- **Platform Support**: Desktop (mouse/keyboard) and mobile (touch controls)

---

## Project Structure

```
Google_4/
├── index.html          # Complete game (1669 lines: HTML + CSS + JS)
├── README.md           # Project metadata
├── CLAUDE.md          # This file
└── .git/              # Version control
```

The entire game is contained in a single `index.html` file with three sections:

1. **HTML** (lines 1-420): Canvas, UI elements, overlays, touch controls, accessibility structure
2. **CSS** (lines 10-358): Styling for game UI, animations, responsive design, dark mode support
3. **JavaScript** (lines 422-1669): Complete game engine and logic

---

## Game Features

### Core Mechanics
- **Vertical Shooting**: Player controls a fighter jet, enemies spawn from top
- **Weapon System**: Three weapon types (Vulcan, Laser, Cannon) with upgradeable levels
- **Smart Bombs**: Limited-use bombs that clear enemy waves
- **Scoring & Multiplier**: Combo system rewards rapid successive kills
- **Lives & Stages**: Progressive difficulty through multiple stages with boss encounters

### Difficulty Modes
- **Easy**: Slower spawns, forgiving enemy AI, extra lives/bombs, reduced boss HP
- **Normal**: Balanced experience (default)
- **Hard**: Aggressive spawning, challenging enemies, tougher bosses

### Visual Features
- **Dynamic Sky Themes**: 6 different parallax backgrounds (Daylight, Sunset, Night, Storm, Arctic, Volcanic)
- **Particle Effects**: Bullet impacts, explosions, visual feedback
- **Screen Effects**: Hit vignette, flash animations, combo popups
- **Boss Health Bar**: Displayed when fighting stage bosses

### Audio
- **Web Audio API Synthesis**: Real-time audio generation (no audio files needed)
- **Adaptive Sounds**: Weapon fire, explosions, boss encounters, UI feedback
- **Spatial Audio**: Panning based on screen position
- **Mute Toggle**: Persistent sound preference

### Controls

**Desktop:**
- **Move**: Mouse position or Arrow keys / WASD
- **Fire**: Left click or Space
- **Bomb**: X or Shift
- **Pause**: P or Escape

**Mobile:**
- **Move**: Drag left side of screen (virtual joystick)
- **Fire**: Tap FIRE button (right side)
- **Bomb**: Tap BOMB button (upper right)

### Accessibility
- **Reduced Motion Mode**: Disables animations/shake for motion-sensitive users
- **Screen Reader Support**: ARIA live region announces game state changes
- **Safe Area Support**: Respects device notches/safe areas on mobile

---

## Technical Architecture

### Game Loop & Rendering

```javascript
requestAnimationFrame → Update → Render → requestAnimationFrame
```

The main game loop:
1. **Update Phase**: Process input, move entities, detect collisions, update game state
2. **Render Phase**: Draw sky, parallax layers, enemies, player, bullets, effects
3. **Performance Tracking**: Adaptive quality system monitors frame times

### Entity System

The game manages several entity types:

- **Player**: Fighter jet (position, health, weapon state, bomb count)
- **Enemies**: Three types (grunt, interceptor, tank) with different stats
- **Bullets**: Player shots and enemy projectiles
- **Particles**: Explosion and impact effects
- **Bosses**: End-of-stage encounters with unique visual designs

### State Management

**Game State:**
- `gameRunning`: Boolean indicating if game is active
- `score`: Current score with multiplier
- `lives`: Remaining player lives
- `stage`: Current stage number (affects enemy spawning)
- `waves`: Killed enemies; stage advances at KILLS_PER_STAGE

**Persistence:**
- Best score: localStorage `skyfighter.bestScore`
- Sound preference: localStorage `skyfighter.muted`
- Difficulty: localStorage `skyfighter.difficulty`
- Motion preference: localStorage `skyfighter.reducedMotion`

### Audio System

- **Beep Function**: Creates oscillator-based tones with envelope and filter
- **Noise Burst**: White noise bursts for impacts/explosions
- **Dynamic Compression**: Master gain with compressor for volume consistency
- **Spatial Panning**: Sounds positioned left/right based on screen coordinates

### Input Handling

**Mouse/Keyboard:**
- Continuous polling for key states
- Mouse position tracked for aiming
- Buttons trigger one-time actions

**Touch:**
- Virtual joystick on left side (drag to move)
- Fire and bomb buttons on right side
- Supports multi-touch (move + fire/bomb simultaneously)

---

## Key Constants & Configuration

### Game Balance
```javascript
DIFFICULTIES       // Easy/Normal/Hard spawn rates, enemy speed, enemy AI accuracy, lives, bombs
KILLS_PER_STAGE    // 18 kills per stage before boss
ENEMY_TYPES        // Grunt, Interceptor, Tank (HP, speed, score, appearance)
BOSSES             // 6 unique boss designs with varying HP multipliers
THEMES             // 6 color schemes for stage backgrounds
```

### Gameplay Tweaks
Located in the JavaScript section:
- `SPAWN_TIMER`: Controls enemy spawn rate
- Weapon balance (fire rate, damage per bullet)
- Bomb effect radius
- Score multiplier decay
- Collision detection thresholds

---

## Development Notes

### File Size Optimization
- All assets are inline (no external files)
- SVG favicon embedded in data URI
- Audio synthesized in real-time (no MP3/WAV files)
- CSS and JS concatenated for single-file delivery

### Browser Compatibility
- **Requires**: ES6+ support, Canvas 2D, WebAudio API, localStorage
- **Tested on**: Chrome, Firefox, Safari, Edge (modern versions)
- **Mobile**: iOS Safari, Android Chrome with touch support

### Performance Considerations
- **Adaptive Quality**: System detects frame drops and reduces particle count
- **Canvas Size**: Scales to fit viewport while maintaining 4:3 aspect ratio
- **Object Pooling**: Bullets and particles reused to reduce GC pressure
- **Draw Optimization**: Batched rendering, early Z-depth culling

### Known Limitations
- Single-file design makes code organization challenging
- No build process (intentional design choice)
- No external dependencies (fully vanilla)
- Touch controls don't show virtual joystick visualization by default

### Future Enhancement Ideas
- Persistent leaderboard (server-based)
- Additional weapon types or power-up effects
- Procedural boss difficulty scaling
- Player customization (ship colors/patterns)
- Local multiplayer or competitive modes
- Mobile app version (Cordova/Electron wrapper)

---

## Running & Development

### Play the Game
1. Open `index.html` in a web browser
2. Select difficulty and click "Start Game"
3. Use mouse/keyboard (desktop) or touch (mobile)

### Deploy
- Static file hosting (GitHub Pages, Netlify, S3, etc.)
- No server-side requirements
- Works offline after initial load

### Making Changes
1. Edit `index.html` directly
2. Reload in browser to test
3. All changes are in JavaScript (lines 422+), CSS (lines 10-358), or HTML (lines 1-420)
4. No build step needed; changes take effect immediately

### Code Organization (within index.html)

**Lines 1-420**: HTML & Structure
- Canvas and game UI elements
- Overlay screens (title, game over, pause)
- Touch control layout
- Accessibility markup

**Lines 10-358**: CSS Styling
- Root styles (dark mode, sizing)
- HUD elements (score, health, weapons)
- Animations (combo popups, stage banners, screen shake)
- Responsive design for mobile
- Reduced motion media query support

**Lines 422-1669**: JavaScript Game Engine
- Constants & configuration (difficulty, enemies, bosses)
- Audio system initialization and playback
- Input handling (keyboard, mouse, touch)
- Game loop & state management
- Entity update & collision logic
- Canvas rendering pipeline
- UI update & game screen transitions

---

## Debugging Tips

### Console Utilities
- Open DevTools console (`F12` or right-click → Inspect)
- Game logs stage progression and boss encounters
- Audio context state visible if audio fails to initialize

### Performance Profiling
- Use Chrome DevTools **Performance** tab to measure frame time
- Watch **quality** variable in console (0=full, 1=medium, 2=low) to see adaptive performance
- Check frame times array in console if needed

### Common Issues

**Audio not playing:**
- Check browser's autoplay permissions (Safari requires user gesture first)
- Mute button may be on; check in-game UI
- Older browsers may not support WebAudio synthesis

**Touch controls not working:**
- Ensure mobile viewport meta tag is loaded (already in index.html)
- Test in DevTools mobile emulator first
- Some browsers require explicit touch-action CSS (already set)

**Game too slow:**
- Reduce particle effects or toggle reduced motion
- Adaptive quality system should kick in automatically
- Close other browser tabs to free CPU

---

## Contributing & Versioning

**Current Version**: 1.0 (shipped with initial bosses and three difficulty modes)

**Git History:**
- Initial commit: Project setup
- Latest: "Add Sky Fighter: Raiden-style vertical shoot-em-up game"

**Branch**: `claude/inspiring-planck-x2mumh` (development branch)

---

## License & Attribution

This game was created as a standalone web project. It uses only vanilla HTML5, CSS3, and JavaScript with no external libraries or assets.

**Technologies:**
- Canvas 2D API for rendering
- Web Audio API for sound synthesis
- localStorage for persistence
- CSS animations and transforms
- Touch Events API for mobile input

---

## Quick Reference: Game Constants

| Constant | Values | Purpose |
|----------|--------|---------|
| `DIFFICULTIES` | Easy, Normal, Hard | Spawn rates, difficulty tuning |
| `ENEMY_TYPES` | Grunt, Interceptor, Tank | Enemy variants with unique stats |
| `BOSSES` | 6 named bosses | End-of-stage encounters |
| `THEMES` | 6 color schemes | Background parallax themes |
| `KILLS_PER_STAGE` | 18 | Waves needed to trigger boss |
| `quality` | 0-2 | Adaptive performance level |

---

*Documentation auto-generated for Sky Fighter codebase*
