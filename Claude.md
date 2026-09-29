# Claude.md — Sky Fighter Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-page web application. The game features:
- Player-controlled aircraft with weapon switching
- Smart bomb mechanics
- Stage-based boss encounters
- Health and resource management UI

## Project Structure

```
Google_4/
├── index.html        # Complete game implementation (HTML + CSS + JavaScript)
├── README.md         # Project metadata
└── .git/            # Version control
```

**Note:** The entire game is currently contained in a single `index.html` file (~58KB). All game logic, rendering, styling, and mechanics are inline.

## Technology Stack

- **Language:** HTML5 + CSS3 + Vanilla JavaScript (ES6+)
- **Rendering:** HTML5 Canvas
- **Styling:** Inline CSS with CSS animations and filters
- **Deployment:** Static HTML — no build process required

## Key Game Mechanics

### Player State
- **Health:** Lives displayed as hearts (❤️)
- **Weapons:** Multiple weapon types with ammo/switching
- **Bombs:** Limited-use smart bombs for area damage
- **Score Multiplier:** Increases based on performance

### UI Components
- **HUD:** Top-center status display showing health, weapon, bomb count, and score
- **Vignette Effect:** Red edge darkening when player takes damage
- **Flash Effects:** Screen flash on hit or bomb use
- **Hearts/Bombs Display:** Visual indicators using emojis and opacity states

### Canvas Rendering
- Player sprite at bottom center
- Enemies spawned in waves
- Projectiles (player bullets and enemy fire)
- Boss encounters with complex patterns
- Gradient background (sky blue to light blue)

## Development Guidelines for Claude

### When Modifying Game Code

1. **Preserve Game Balance:** Changes to damage values, spawn rates, or difficulty should be tested across multiple stages. Document balance changes clearly.

2. **CSS Animations:** Inline CSS animations (heartBeat, vignettePulse, flashHit, flashBomb) are tightly coupled to game state. Validate animation names in both CSS and JavaScript.

3. **Canvas Coordinate System:** The game uses canvas 2D rendering. Remember:
   - Origin (0,0) is top-left
   - x increases rightward, y increases downward
   - Use `canvas.width` and `canvas.height` for responsive sizing

4. **Event Handling:**
   - Touch events for mobile (with `touch-action: none` on canvas)
   - Mouse events for desktop (crosshair cursor)
   - Keyboard events for input (if implemented)

5. **Performance Considerations:**
   - Minimize DOM manipulation (only HUD updates)
   - Use requestAnimationFrame for game loop
   - Keep canvas rendering efficient (batch operations where possible)

### Testing & Validation

- Test game flow: menu → gameplay → boss fight → completion
- Verify responsive design on mobile and desktop
- Check accessibility: ensure UI text is readable with emoji support
- Validate CSS animations trigger correctly with game state changes
- Test weapon switching and bomb mechanics under stress

### Code Style

- Maintain ES6+ modern JavaScript
- Use meaningful variable names for game entities and state
- Document complex game logic with inline comments
- Keep inline CSS organized by component (HUD, canvas, animations, keyframes)

### Common Tasks

**Adding a New Weapon:**
1. Define weapon properties (damage, fire rate, ammo pool)
2. Add rendering code in the game loop
3. Update weapon cycling logic
4. Add weapon name/icon to HUD display

**Adjusting Difficulty:**
1. Modify enemy spawn frequency and patterns
2. Adjust enemy health and damage values
3. Update boss movement patterns and attack patterns
4. Test progression across all stages

**Modifying UI:**
1. Update inline CSS in `<style>` block
2. Modify HUD element generation (usually in game initialization)
3. Test on multiple screen sizes and resolutions
4. Verify vignette and flash effects still display correctly

## Deployment

The game is a static HTML file requiring no build step:
1. Push changes to the designated development branch
2. Merge to main branch (if applicable)
3. Deploy `index.html` to web server
4. No additional dependencies or build tools needed

## Browser Compatibility

Target modern browsers supporting:
- HTML5 Canvas 2D
- CSS3 (animations, filters, backdrop-filter)
- ES6 JavaScript
- Touch and Pointer Events
- viewport-fit (for notch support)

## Future Considerations

- Consider code splitting if the game grows beyond single-file architecture
- Evaluate performance on low-end devices (consider canvas resolution scaling)
- Plan for save/load state if campaign mode is added
- Document any backend requirements for leaderboards or multiplayer features

---

**Last Updated:** 2026-09-29  
**Maintained by:** Claude Code Agent
