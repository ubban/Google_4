# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sky Fighter is a Raiden-style vertical shoot-'em-up game built entirely in **vanilla HTML5, CSS, and JavaScript**. The entire game is contained in a single `index.html` file with no external dependencies, build process, or package manager.

**Key characteristics:**
- Single-file architecture (58KB HTML file)
- No npm, build tools, or compilation required
- Purely client-side with WebAudio API for procedural sound
- Responsive design supporting desktop and mobile/touch

## Running and Testing

### Quick Start
Simply open `index.html` in any modern web browser:
```bash
# Option 1: Direct file
open index.html

# Option 2: Local HTTP server (for better compatibility)
python3 -m http.server 8000
# Then visit http://localhost:8000
```

### Testing
- **Desktop:** Use keyboard (Arrow keys/WASD to move, Space to fire, X/Shift for bombs, P/Esc to pause)
- **Mobile:** Tap the FIRE and BOMB buttons, drag left side to move
- **Audio:** Click the speaker icon to toggle mute, test procedural sound effects
- **Settings:** Adjust difficulty (Easy/Normal/Hard) before starting; change reduced-motion setting for accessibility
- **Performance:** Watch frame rate adapt automatically (quality variable adjusts spawn rates if FPS drops)

### Debugging
The game uses browser console for errors. Check localStorage persistence:
```javascript
localStorage.getItem('skyfighter.bestScore')    // Best score
localStorage.getItem('skyfighter.difficulty')   // Selected difficulty
localStorage.getItem('skyfighter.muted')        // Mute state
localStorage.getItem('skyfighter.reducedMotion') // Reduced motion state
```

## Architecture

### Single HTML File Structure
The `index.html` file contains three interleaved sections:

#### 1. **HTML Markup** (Lines 361–420)
- Canvas element for game rendering
- HUD elements: score, best score, multiplier, weapon level, hearts, bombs
- Boss health bar (hidden until boss appears)
- Overlay system for menus (main menu, game over, stage clear)
- Mobile touch controls (move zone, fire/bomb buttons)
- Top bar buttons (mute, motion, pause)
- Screen effects (vignette, flash) and combo pop animations

#### 2. **CSS Styling** (Lines 10–359)
- Dark space-themed color scheme with blue/gold accents
- Responsive design using viewport-relative units (clamp, vw, vh)
- Safe-area support for notched devices
- Animations: heart beat (critical health), screen shake/flash, combo text, stage banner
- Reduced-motion media query support for accessibility
- Touch button styling for mobile

#### 3. **Game Engine (JavaScript)** (Lines 422+)
The game loop and logic are organized into conceptual sections (marked with `// ----------`):

**Core Systems:**
- **Adaptive Performance:** Monitors frame time and reduces quality (spawn rates, particle density) if FPS drops
- **Difficulty Settings:** Three presets (easy/normal/hard) that control enemy spawning, speed, shooting, lives, bombs
- **Audio Engine:** WebAudio API synth with oscillators, filters, noise, and panning for all SFX
- **Canvas Rendering:** Custom drawing functions for player, enemies, bosses, projectiles, effects
- **Input Handling:** Keyboard (Arrow/WASD/Space/X), mouse, and multi-touch joystick + buttons
- **UI Persistence:** localStorage for best score, difficulty, mute, reduced motion

**Game Objects:**
- **Player:** Position, health, weapon level, bomb count, collision detection
- **Enemies:** Three types (grunt, interceptor, tank) with different speeds, HP, and scoring
- **Bosses:** Six boss types with unique appearances and attack patterns
- **Projectiles:** Player shots, enemy fire, smart bomb blast
- **Power-ups:** Weapon upgrades (Vulcan, Laser, Photon) and multiplier bonuses

**Game Flow:**
1. Main menu overlay with difficulty selection
2. Stage banner animation
3. Enemy spawning based on kill count (KILLS_PER_STAGE = 18)
4. Boss appears after stage cleared
5. Game over when lives depleted; score compared to best
6. Reset and return to menu

**Constants & Data:**
- `DIFFICULTIES` (line 485): Spawn rates, speeds, lives, bombs per difficulty
- `THEMES` (line 493): Six visual themes with sky gradients and color schemes
- `ENEMY_TYPES` (line 502): Grunt, interceptor, tank definitions
- `BOSSES` (line 508): Six boss types with color schemes and HP multipliers
- `SFX` (line 624): 11 sound effects using procedural synthesis

### Code Organization Tips

**Finding specific features:**
- Player movement: Search `keys[` to find input handling, `playerVelX/Y` for movement
- Weapon system: Search `weaponLevel` or the `WEAPONS` object definitions
- Enemy spawning: Look for `nextEnemyTime` and spawn logic in game loop
- Boss fights: Search `bossName` or `boss` variable logic
- Collision detection: Search `for (let i = 0; i < projectiles.length` patterns
- Screen effects: Search `vignetteEl`, `flashHit()`, `screenShake`
- Audio: Search `sfx.` for all sound calls; `beep()` and `noiseBurst()` for synthesis

## Development Workflow

### Making Changes

**Performance:** The game includes adaptive quality throttling (line 474–483). If adding visual effects:
- Test with DevTools throttled to check frame impact
- Quality level 0 = full, 1 = medium, 2 = low; use `if (quality < 1) { ... }` for expensive renders

**Audio:** New SFX should use the `beep()` or `noiseBurst()` functions for consistency; no external audio files are used.

**Mobile:** Always test changes on a touch device or use DevTools device emulation. The `isTouchDevice` check (line 778) controls UI visibility.

**Animations:** Reduced-motion support is built in; animations in the CSS are disabled when `body.reduced-motion` class is set. Test with `motionBtn` toggle.

### Testing New Features
- Difficulty selector (Easy/Normal/Hard) affects spawn rates, speeds, and AI
- Frame time tracking runs every 45 frames; frame drops will auto-reduce quality
- localStorage is cleared in DevTools → Storage → Clear Site Data to reset persistent state
- Screen shake/flash respect reduced-motion; test both states

### Common Edits

**Adjust difficulty balance:**
Edit `DIFFICULTIES` object (line 485). Example: Increase `spawnMin` to spawn enemies slower.

**Add a new boss:**
1. Add entry to `BOSSES` array (line 508)
2. Add boss rendering in the drawing function
3. Adjust `hpMul` to tune difficulty

**Add visual theme:**
1. Append to `THEMES` array (line 493)
2. Update sky gradient and color palette

**Modify weapon behavior:**
Search for weapon definitions and update fire rate, damage multiplier, or sprite.

## Key Variables & Constants

- `running`: Boolean; true during gameplay
- `paused`: Boolean; paused state
- `playerX, playerY`: Player position
- `playerHealth`: Hearts remaining
- `playerBombs`: Bomb count
- `score, bestScore`: Current and best scores
- `difficulty`: String ('easy'|'normal'|'hard')
- `quality`: 0–2 adaptive quality level
- `gameStage`: Current stage number
- `enemies`: Array of active enemy objects
- `projectiles`: Array of bullets/lasers
- `canvas`: 480×720 (BASE_W × BASE_H; CSS scaling handles responsiveness)

## Browser Compatibility

- Modern browsers (Chrome, Firefox, Safari, Edge)
- Requires Canvas, WebAudio, localStorage
- Gracefully falls back if AudioContext not available (game still plays silently)
- Touch support tested on iOS and Android
- Reduced-motion respects `prefers-reduced-motion` media query

## Future Enhancement Ideas

- Multiple weapon types already have framework (Vulcan/Laser/Photon)
- Boss patterns use simple movement; could add complex attack choreography
- Themes system supports easy addition of new visual modes
- Multiplier and combo system is in place for high-score mechanics
- Mobile controls use touch; could add gamepad API support
