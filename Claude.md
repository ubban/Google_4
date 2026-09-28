# Claude.md — Sky Fighter Development Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas and JavaScript. The game features:
- Player-controlled aircraft with position tracking
- Dynamic weapon system with weapon switching
- Enemy ships with varied attack patterns
- Smart bomb mechanics with screen-clearing effects
- Boss encounters with health bars
- Score tracking and multiplier system
- Responsive UI that adapts to screen size and safe areas (mobile support)

## File Structure

```
Google_4/
├── index.html          # Complete single-file game (HTML + inline CSS + JavaScript)
├── README.md           # Project metadata
├── Claude.md           # This file — development guidelines
```

## Game Architecture

### Core Components (all in index.html)

1. **Canvas & Rendering**
   - `#gameCanvas`: Main game drawing surface
   - Responsive sizing with `max-width: 100%` and `max-height: 100%`
   - Linear gradient sky background from dark blue to light blue

2. **HUD (Heads-Up Display)**
   - Lives display with heart emoji
   - Bomb inventory display
   - Current weapon indicator
   - Multiplier tracker
   - Responsive font sizing using `clamp()`

3. **Game Loop**
   - requestAnimationFrame-driven update/render cycle
   - Player input handling (keyboard, touch, mouse)
   - Collision detection
   - Entity management (player, enemies, projectiles, explosions)

4. **Player Mechanics**
   - Crosshair-based aiming
   - Weapon switching system
   - Health/lives system (3 lives default)
   - Smart bomb ability (limited use)
   - Movement constraints to canvas bounds

5. **Enemy System**
   - Varied enemy types with different behaviors
   - Enemy projectile patterns
   - Boss encounters with health bars
   - Spawn waves and difficulty progression

6. **Visual Effects**
   - Flash effects on player damage
   - Red vignette effect during critical health
   - Explosion animations
   - Heart beat animation for critical health

## Development Guidelines

### Code Style & Conventions

- **Single file structure**: Keep the game in `index.html` to maintain portability and ease of deployment
- **Inline CSS**: Use `<style>` tags for all styles (keeps everything in one file)
- **Inline JavaScript**: Use `<script>` tags for game logic
- **Comments**: Add comments only for non-obvious game mechanics, especially:
  - Collision detection algorithms
  - Enemy AI patterns
  - Canvas coordinate system notes
  - Mobile-specific touch handling

### Canvas & Coordinate System

- Canvas coordinate origin is top-left (0,0)
- X increases rightward, Y increases downward
- All game entities use this coordinate system
- Responsive sizing: ensure entities scale appropriately with viewport

### Mobile Considerations

- Use `touch-action: none` to prevent browser defaults
- Handle touch events for both aiming and menu interaction
- Test on various device sizes using responsive viewport
- Use `max-width` and `max-height` for canvas scaling
- Use `clamp()` for font sizes to adapt to screen size
- Consider `env(safe-area-inset-*)` for notch-aware positioning

### Performance Notes

- Use requestAnimationFrame for smooth 60fps rendering
- Minimize canvas context saves/restores
- Batch similar drawing operations
- Consider canvas size vs. logical game space (avoid unnecessary upscaling)

### UI/UX

- HUD uses glassmorphism design (backdrop blur with semi-transparent background)
- Color scheme: dark blue sky background with contrasting accent colors:
  - Health/lives: white/red
  - Weapons: lime green (#7CFC00)
  - Multiplier: orange (#ff8a65)
  - Score: gold (#ffd54f)
- Text shadows ensure readability over dynamic backgrounds
- Use emoji for quick visual understanding (♥ for health, 💣 for bombs, etc.)

## Git Workflow

### Branch Strategy

- Develop on the designated feature branch: `claude/inspiring-planck-45hxmc`
- Keep commits focused and descriptive
- Squash related changes before pushing

### Commit Messages

- Start with a verb: "Add", "Fix", "Update", "Refactor"
- Be specific about what changed and why
- Include issue/feature references if applicable
- Example: `Add boss health bar and stage progression system`

### Attribution

All commits include the Claude Code session attribution footer:
```
Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_01T7bwUFW2Ci1hpZo2HSus88
```

## Testing & Validation

### Manual Testing Checklist

- [ ] Canvas renders correctly at different viewport sizes
- [ ] Player can move and shoot
- [ ] Weapon switching works
- [ ] Collision detection is accurate
- [ ] Enemies spawn and behave correctly
- [ ] Smart bombs clear the screen
- [ ] Game over/win conditions trigger properly
- [ ] HUD updates reflect game state
- [ ] Touch input works on mobile devices
- [ ] Audio cues (if added) play correctly
- [ ] Performance stays smooth at 60fps

### Browser Compatibility

- Target modern browsers (Chrome, Firefox, Safari, Edge)
- Test touch handling on actual mobile devices
- Verify canvas scaling on high-DPI displays
- Test fullscreen mode on mobile

## Common Tasks

### Adding a New Enemy Type

1. Define enemy properties (position, speed, health, behavior)
2. Implement movement pattern (update position each frame)
3. Implement attack pattern (projectile generation)
4. Add to spawn wave configuration
5. Update collision detection if needed
6. Test interaction with player and bombs

### Modifying Weapon System

1. Define weapon stats (damage, fire rate, spread, etc.)
2. Update weapon switching logic
3. Modify projectile rendering (color, size, trail)
4. Adjust HUD weapon indicator display
5. Test damage calculations against various enemies

### Balancing Difficulty

1. Adjust enemy spawn patterns and timing
2. Modify enemy health and damage values
3. Tune weapon damage and fire rates
4. Adjust multiplier progression
5. Play test multiple times to verify progression feels right

## Known Limitations & Future Considerations

- Single-file structure limits code organization as the game grows
- Consider multi-file architecture if game becomes significantly larger
- Current asset generation is procedural; consider sprite sheets for better performance
- Audio should use Web Audio API or HTML5 audio elements
- Persistence could use localStorage for high scores

## Quick Reference: Key Constants

Look for these in the JavaScript section:
- `CANVAS_WIDTH`, `CANVAS_HEIGHT`: Game area dimensions
- Player stats: speed, health, weapon
- Enemy spawn intervals and patterns
- Weapon damage and fire rates
- Multiplier growth rates

## Resources & Tools

- **MDN Canvas API**: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API
- **Browser DevTools**: Use for performance profiling and debugging
- **Touch Events**: https://developer.mozilla.org/en-US/docs/Web/API/Touch_events
- **requestAnimationFrame**: https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame

## Questions & Decisions

For architectural decisions or significant feature additions:
- Discuss trade-offs between performance and features
- Consider mobile-first design approach
- Evaluate visual complexity vs. performance impact
- Plan for scalability if features grow significantly
