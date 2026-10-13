# Sky Fighter - CLAUDE.md

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas and JavaScript. The game features weapon switching, smart bombs, stage bosses, and responsive mobile/desktop controls.

- **Type**: Single-page browser game
- **Tech Stack**: HTML5 Canvas, Vanilla JavaScript, CSS3
- **Entry Point**: `index.html`
- **No Build Step**: All code runs directly in the browser

## Key Architecture

### Game Loop
- Single HTML file with embedded JavaScript and CSS
- Canvas-based rendering with continuous requestAnimationFrame loop
- Game state managed through JavaScript objects and arrays

### Core Game Systems
1. **Player Entity**: Position, health, weapon, bombs, score multiplier
2. **Weapon System**: Multiple weapon types with switching capability
3. **Enemy System**: Waves of enemies with different patterns and behaviors
4. **Boss System**: Special stage-ending boss encounters
5. **Particle/Effect System**: Explosions, flashes, visual feedback (vignette, flash animations)
6. **Input Handling**: Mouse, touch, and keyboard controls with movement zones

### Key Features
- **Responsive Design**: Works on desktop, tablet, and mobile (portrait/landscape)
- **Touch Controls**: No explicit touch zone UI; player moves toward finger position
- **Weapon Switching**: Multiple weapon types available
- **Smart Bombs**: Special attack that clears the screen
- **HUD Display**: Health hearts, bomb count, weapon type, kill multiplier
- **Visual Effects**: Screen flash on hit, vignette pulse on damage, critical health heartbeat animation

## Development Guidelines

### Code Modifications
- **Minimal Changes**: Keep modifications focused and avoid over-engineering
- **No Build Tools**: Do not introduce webpack, bundlers, or transpilers
- **Vanilla JavaScript**: Maintain pure JS; avoid external libraries unless absolutely necessary
- **CSS-in-HTML**: Keep styling embedded in the `<style>` tag
- **Single File**: Keep all code in `index.html` to preserve simplicity

### Game Logic Changes
- **Weapon Balancing**: Modify weapon damage, fire rate, or patterns in the weapon configuration section
- **Enemy Waves**: Adjust enemy types, counts, spawn timing in wave definitions
- **Difficulty**: Modify enemy health, speed, or attack patterns
- **Player Stats**: Health, bomb count, starting conditions

### Performance Considerations
- Canvas rendering is the main performance bottleneck
- Use efficient collision detection (spatial partitioning not yet implemented)
- Limit particle effects count if performance degrades
- Test on lower-end devices (mobile)

### Visual/UX Changes
- **Canvas Size**: Determined by window size with max constraints; adjust in canvas sizing logic
- **Color Scheme**: Currently uses blues (#0d2f5e, #1e6fd8, #7fc1ff) for sky gradient
- **HUD Layout**: Positioned absolutely at top center; flexbox for responsive layout
- **Animations**: CSS keyframe animations (@keyframes) for heartbeat, vignette, flash effects

## Testing & Running

### How to Run
1. Open `index.html` in any modern web browser
2. Game starts automatically with player in center
3. Move mouse (desktop) or finger (touch) to move player
4. Click/tap to fire weapons
5. Press spacebar or UI gesture for smart bombs (if implemented)

### Testing Checklist
- [ ] Game initializes without errors in browser console
- [ ] Player movement responds to mouse/touch input
- [ ] Weapons fire and hit enemies
- [ ] Enemy waves spawn progressively
- [ ] Score/multiplier updates on kills
- [ ] Damage flash and vignette effects work
- [ ] Responsive on mobile devices (portrait and landscape)
- [ ] Performance acceptable on target devices

## Common Tasks

### Adding a New Weapon Type
1. Define weapon object in the weapon configuration
2. Add sprite/rendering logic in the game loop
3. Update weapon switching logic
4. Test fire rate, damage, and visual effects

### Adjusting Enemy Waves
1. Modify wave definitions in the enemy system
2. Adjust enemy spawn positions and patterns
3. Test progression difficulty

### Tweaking Game Balance
- Enemy health: Increases with stage
- Player bomb damage: Set per bomb definition
- Weapon damage: Set per weapon type
- Multiplier growth: Adjust on kill counter

### Visual Polish
- Modify gradient colors in `#gameCanvas` background style
- Adjust HUD styling (colors, sizes, opacity)
- Tune animation timings (@keyframes durations)

## Known Limitations

- Single file structure means large code as HTML file grows
- No persistent storage/save system
- No sound effects (audio not yet implemented)
- Limited enemy AI variety
- No multiplayer or level selection

## Mobile Considerations

- **Safe Area**: Uses `env(safe-area-inset-top)` for notch/cutout awareness
- **Touch Action**: `touch-action: none` on game canvas prevents scrolling
- **Max Size**: Canvas scales to fit within viewport with max-width/max-height
- **Landscape/Portrait**: CSS vw/vh units scale appropriately

## Notes for Future Work

- Consider modularizing JavaScript into separate files if complexity grows
- Add sound effects library integration when needed
- Implement level/stage progression system
- Add particle pooling for performance optimization
- Consider state persistence (localStorage for high scores)
