# Sky Fighter — Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built with vanilla JavaScript and HTML5 Canvas. The game features:
- Player aircraft with weapon switching
- Smart bombs for crowd control
- Stage-based progression with boss fights
- Responsive touch and keyboard controls
- Dark blue space theme with gradient backgrounds

This is a single-file HTML game (index.html) with embedded CSS and JavaScript.

## Getting Started

### Development Environment
- **Language**: JavaScript (ES6+)
- **Frontend**: HTML5 Canvas
- **No build system**: The game runs directly from `index.html`
- **Testing**: Manual play-testing in a web browser

### Running the Game
1. Open `index.html` in a web browser (Chrome, Firefox, Safari, Edge all supported)
2. Click to focus the canvas
3. Use mouse movement or touch to control the aircraft
4. Click/tap to fire weapons
5. Press 'W' to switch weapons (or tap weapon indicator)
6. Press 'B' for smart bomb (limited uses)

## Codebase Structure

The entire game is contained in `/index.html`:
- **HTML**: Canvas element and HUD structure (lines 1-100 approximately)
- **CSS**: Game styling and responsive design (lines 100-250 approximately)
- **JavaScript**: Complete game logic (lines 250+ with sections below)

### Key JavaScript Sections

#### 1. Configuration & Constants
- Game dimensions, colors, and speeds
- Player stats (health, weapon damage, bomb count)
- Enemy configurations
- Score multipliers

#### 2. Game Objects
- **Player**: Aircraft, weapons, health, bomb system
- **Bullets**: Player projectiles with damage types
- **Enemies**: Basic enemies, armored enemies, boss enemies
- **Explosions**: Particle effects and visual feedback
- **Items**: Drops for health and weapon upgrades

#### 3. Game Engine
- **Initialization**: Canvas setup, event listeners
- **Main loop**: `animate()` function runs at 60fps via requestAnimationFrame
- **Update phase**: Entity position and state updates
- **Render phase**: Canvas drawing with layers
- **Collision detection**: Simple rectangular collision
- **Game state management**: Running, paused, game-over states

#### 4. Input Handling
- Mouse movement for aiming
- Click/tap to fire
- Keyboard: W for weapon switch, B for bomb, P for pause
- Touch events for mobile support

#### 5. Audio (Silent by Default)
- Audio object created but no actual sound files loaded
- Ready for sound effects when needed

## Common Development Tasks

### Adding a New Enemy Type
1. Locate the enemy creation section in the game engine
2. Add new enemy configuration to `enemyConfigs`
3. Implement draw method specific to that enemy
4. Update collision/health logic if needed

### Adjusting Game Difficulty
- Modify enemy spawn rate in the main game loop
- Adjust enemy speed multipliers
- Change enemy health values in `enemyConfigs`
- Modify boss attack patterns

### Adding/Modifying Weapons
- Update weapon configuration in the `weapons` array
- Modify firing logic in the player update section
- Adjust weapon damage and fire rate values

### Improving Visual Effects
- Modify explosion particle count and colors
- Adjust gradient backgrounds
- Change HUD styling and positioning
- Update entity draw methods for visual variety

### Responsive Design Notes
- Canvas uses `clamp()` CSS for dynamic sizing
- HUD uses `clamp()` for font scaling
- Safe area insets handled for notched devices
- Touch-action set to none to prevent scrolling

## Performance Considerations

- **Canvas rendering**: Cleared each frame, entities drawn in z-order
- **Entity pooling**: Not implemented (could optimize for many particles)
- **Collision detection**: O(n²) but acceptable for game scope
- **Memory**: All game state stored in JavaScript objects

### Optimization Opportunities
1. Implement object pooling for bullets and explosions
2. Use OffscreenCanvas for background gradient (if performance needed)
3. Batch canvas operations to reduce draw calls
4. Use WebWorker for collision detection (if game expands significantly)

## Code Style Guide for Claude Agents

### When Making Changes
1. **Preserve the single-file structure** unless explicitly requested to refactor
2. **Maintain consistent indentation** (2 spaces in JavaScript, as per existing code)
3. **Update both game logic and HUD displays** when modifying mechanics
4. **Test collision detection** carefully—ensure hitboxes match visuals
5. **Keep the animation loop intact**—changes should not disrupt the 60fps target

### Debugging Tips
- Use browser DevTools console: `console.log()` for debugging
- Check canvas dimensions with: `canvas.width` and `canvas.height`
- Pause game with 'P' key to inspect game state
- Test on multiple screen sizes (mobile, tablet, desktop)

### Common Pitfalls to Avoid
- Don't modify canvas size mid-game (breaks rendering)
- Avoid synchronous operations in the game loop
- Don't forget to clear canvas each frame (`fillRect` with background color)
- Ensure collision detection uses consistent coordinate systems
- Test weapon switching and bomb mechanics after changes

## Git Workflow

This repository uses a feature branch pattern:
- **Main branch**: Production-ready code
- **Development branches**: Feature/fix branches with descriptive names
- **Commit messages**: Clear, present-tense descriptions of changes

### Before Pushing Changes
1. Test the game thoroughly in browser (at least one full playthrough)
2. Check for console errors in browser DevTools
3. Verify mobile/touch controls still work
4. Test on different screen sizes if UI was modified

## Future Enhancement Ideas

- [ ] Sound effects and background music
- [ ] Multiple stages with escalating difficulty
- [ ] Power-up system (shields, rapid-fire, etc.)
- [ ] High score persistence (localStorage)
- [ ] Pause/resume functionality with menu
- [ ] Difficulty settings (easy/normal/hard)
- [ ] Sprite-based graphics (if performance allows)
- [ ] Multiplayer scoreboard (requires backend)

## Questions and Support

When working on this project:
1. Check existing game logic before adding new features
2. Run through the game loop once to verify rendering and updates
3. Test touch controls if modifying input handling
4. Ensure new features don't block the 60fps animation loop

---

*Last updated: 2026-09-27*
*Project: Sky Fighter — Raiden-style vertical shoot-em-up game*
