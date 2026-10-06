# Sky Fighter — Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game implemented as a single-file HTML5 web application. Players control a fighter jet, switch between weapons, drop smart bombs, and battle through stage bosses.

- **Platform**: HTML5 Canvas + JavaScript (no build step required)
- **Format**: Single `index.html` file (~1670 lines)
- **Playable**: Open `index.html` in any modern browser
- **Repository**: `ubban/google_4` at `main` branch

## File Structure

```
/home/user/Google_4/
├── index.html          # Complete game implementation (all code, styles, markup)
├── README.md           # Project title and metadata
└── .git/               # Git history
```

The entire game is self-contained in `index.html`:
- **HTML**: Game container, UI elements, HUD (score, weapons, lives, bombs)
- **CSS**: Styling for canvas, HUD, overlay menu, touch controls
- **JavaScript**: All game logic in a single IIFE (Immediately Invoked Function Expression)

## Architecture Overview

### Core Systems

1. **Canvas Rendering**
   - 2D canvas for game drawing (~800×600 at desktop scale)
   - Responsive: scales with viewport using `max-width: 100%; max-height: 100%`
   - Background gradient: sky blue to darker blue

2. **Player Entity**
   - Positioned at bottom-center of screen
   - Current weapon and ammo system
   - Health/lives system (with visual hearts in HUD)
   - Smart bomb mechanic (limited uses per stage)

3. **Weapon System**
   - Multiple weapon types (at least "VULCAN" visible in HUD)
   - Weapon levels/upgrades
   - Weapon switching mechanic
   - Ammo/fire rate management

4. **Enemy System**
   - Spawned dynamically across stages
   - Likely includes mini-bosses and stage bosses
   - Boss health bar with visual indicator

5. **Scoring & Progression**
   - Score tracking with multiplier (visible in HUD as "x1", "x2", etc.)
   - Best score/high score persistence
   - Stage progression system
   - Difficulty levels: Easy, Normal, Hard

6. **Visual Effects**
   - Screen vignetting (darkened edges)
   - Flash overlay (for damage/explosions)
   - Combo pop-ups (score notifications)
   - Stage banners (stage intro text)
   - Pause overlay

7. **Audio**
   - Mute button for sound control
   - Game sounds managed in-code

8. **Input Handling**
   - **Desktop**: Mouse (move to steer), Click to fire, Space/X for bomb, P/Esc to pause
   - **Keyboard**: Arrow keys or WASD to move, Space to fire, X or Shift to bomb
   - **Mobile**: Touch controls with virtual joystick (drag to move, tap buttons)
   - Virtual bomb and fire buttons for mobile

## Key Game Mechanics

### Stage Progression
- Multiple stages with increasing difficulty
- Boss encounters at end of each stage
- Stage banners announce progression

### Combat
- Player shoots projectiles (default weapon: VULCAN)
- Enemies shoot back
- Collision detection for damage
- Power-ups for weapon upgrades

### Smart Bombs
- Limited-use screen-clear ability
- HUD shows remaining bomb count

### Multiplier System
- Visible in HUD (Score x)
- Likely increases with consecutive hits/enemies destroyed
- Resets under certain conditions

### Difficulty Levels
- **Easy**: Fewer enemies, lower boss HP
- **Normal**: Balanced experience
- **Hard**: More enemies, stronger bosses, faster attacks

## UI Elements (HUD)

The HUD displays at the top in a centered layout:

| Element | Purpose |
|---------|---------|
| Score | Current game score |
| Best | Highest score achieved |
| x{mult} | Score multiplier |
| {Weapon}{Level} | Current weapon and level |
| ❤️ × n | Remaining lives/health |
| 💣 × n | Remaining bombs |
| Boss health bar | Boss encounter health display |

### Menu Overlay
- Game title and description
- Difficulty selection (buttons)
- Start Game button
- Controls hint text (updates for mobile vs. desktop)
- Final score and new best score announcement

### Control Buttons (Top-right)
- **🔊**: Mute/unmute audio
- **🎆**: Toggle reduced motion (disables screen shake, flash, pulse effects)
- **⏸**: Pause game

### Touch Controls (Mobile)
- **Move Zone**: Left side of screen, drag joystick to move
- **Action Zone**: Right side with BOMB and FIRE buttons
- Joystick visual feedback (base + stick)

## Development Notes

### Browser Compatibility
- Requires modern ES6+ JavaScript support
- Canvas 2D API
- Touch Events API (for mobile)
- LocalStorage (for high score persistence)

### How to Run
1. Open `index.html` directly in a browser
2. No build step, no dependencies
3. Works offline once loaded
4. High scores stored in browser LocalStorage

### Code Organization in `index.html`
All JavaScript is in a single IIFE at the end of the HTML file:
```javascript
(function () {
  // Game state
  const canvas = document.getElementById('gameCanvas');
  const ctx = canvas.getContext('2d');
  
  // All game classes, functions, and logic
  // ...
  
  // Event listeners
  // Input handling, button events, etc.
})();
```

### Debugging Tips
- Open browser DevTools (F12)
- Check Console tab for errors
- Inspect canvas context calls to verify rendering
- Monitor event listeners in Elements tab
- Use `console.log()` statements in the IIFE to debug game state

### Performance Considerations
- Canvas rendering runs in main thread
- No WebGL or GPU acceleration currently used
- Motion effects can be toggled off for reduced motion users
- Collision detection is likely bounding-box based

## Extending the Game

### To Add New Features
1. Edit `index.html` directly
2. Add styles in the `<style>` block
3. Add UI elements in the `<body>`
4. Add logic to the main game IIFE in the `<script>` section
5. Commit changes to the `claude/inspiring-planck-u4r7a5` branch

### Common Customization Points
- **Game balance**: Enemy spawning rates, boss HP, damage values
- **Visuals**: Colors in CSS `:root` and `<style>`, canvas draw calls
- **Audio**: Mute toggle and sound effect calls in the IIFE
- **Weapons**: Add new weapon types to weapon selection logic
- **Difficulty**: Modify enemy behavior based on `difficulty` setting

## Git Workflow

- **Active branch**: `claude/inspiring-planck-u4r7a5`
- **Commit messages**: Describe changes to game mechanics or features
- **Push**: Use `git push -u origin claude/inspiring-planck-u4r7a5`

## References

- **Canvas API**: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API
- **Game loops**: https://developer.mozilla.org/en-US/docs/Games/Anatomy
- **Touch Events**: https://developer.mozilla.org/en-US/docs/Web/API/Touch_events

---

*Last updated: 2026-10-06 by Claude Code agent*
