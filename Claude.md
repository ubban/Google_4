# Claude Code Development Guidelines for Google_4

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up arcade game built as a single-file HTML5 application using Canvas and vanilla JavaScript. The game features weapon switching, smart bombs, enemy waves, and boss encounters.

**Repository:** ubban/google_4  
**Main Branch:** main  
**Development Branch:** claude/inspiring-planck-43hqdl

## Project Structure

```
Google_4/
├── index.html       # Single-file game (HTML + embedded CSS + JavaScript)
├── README.md        # Project metadata
└── .git/            # Version control
```

## Code Organization

The `index.html` file contains:
- **HTML Structure:** Minimal DOM with canvas and HUD elements
- **CSS Styles:** Responsive styling for canvas, HUD, animations, and effects
- **JavaScript:** Game engine implemented as:
  - Canvas rendering pipeline
  - Game state management (player, enemies, projectiles, bosses)
  - Input handling (keyboard, touch, mouse)
  - Collision detection
  - Particle effects and visual feedback
  - Audio/SFX integration

## Development Guidelines for Claude

### Before Making Changes

1. **Read the entire HTML file** to understand the current implementation, state machine, and architecture
2. **Identify the specific area** you're modifying (rendering, game logic, input, collision, boss AI, UI)
3. **Preserve existing patterns** — match the style and structure of similar code in the file
4. **Test in a browser** before committing to verify gameplay, responsive design, and touch/mouse controls

### Code Style

- **No comments for obvious code** — variable names and function names should be self-documenting
- **One-line comments only** for non-obvious logic, workarounds, or subtle invariants
- **Consistent spacing:** 2-space indentation, follow existing patterns
- **Variable naming:**
  - Game objects: `player`, `enemies`, `bullets`, `boss`, `powerups`
  - State: `gameState`, `isRunning`, `isPaused`, `hasWon`
  - Positions: `x`, `y`, `vx`, `vy`, `angle`, `radius`
  - Game constants: `CANVAS_WIDTH`, `PLAYER_SPEED`, `FIRE_RATE_MS`

### Game Mechanic Changes

When modifying game mechanics:
1. **Test the golden path** — normal gameplay flow with typical inputs
2. **Test edge cases:**
   - Rapid firing / button mashing
   - Boss encounters and phase transitions
   - Weapon switching during critical moments
   - Screen boundaries and collision edge cases
   - Touch controls on various device sizes
   - Smooth difficulty progression across stages

3. **Performance:** The game runs on Canvas at 60 FPS target
   - Avoid excessive object allocation per frame
   - Reuse arrays/objects where possible
   - Profile with DevTools if adding expensive calculations

### UI/Responsiveness Changes

- Test at multiple viewport sizes: mobile (320px), tablet (600px), desktop (1200px+)
- Verify touch-action and cursor styling
- Ensure HUD elements remain readable with safe-area-inset offsets (notches, rounded corners)
- Test in both portrait and landscape orientations

### Input Handling

The game supports:
- **Keyboard:** Arrow keys (movement), Space/Z (fire), X (bomb), W (weapon switch)
- **Touch:** Drag to move, tap to fire, gesture for bombs
- **Mouse:** Move to aim/position, click to fire

When modifying input:
- Maintain `touch-action: none` on canvas
- Prevent default behaviors appropriately
- Ensure touch controls don't block scroll on document
- Test with both rapid and slow input

## Commit Message Format

Format: `<action> <component>: <description>`

Examples:
- `Add enemy spawning waves with difficulty scaling`
- `Fix collision detection for rotated projectiles`
- `Improve boss AI state machine for phase transitions`
- `Optimize particle effect rendering with pooling`
- `Refactor player input handler for multi-touch support`
- `Update HUD layout for better mobile visibility`

All commits must include the attribution footer:
```
Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_01QM64uJMVZotmbZuWG56XV4
```

## Testing Checklist Before Committing

- [ ] Game launches without console errors
- [ ] Player movement works (keyboard, touch, mouse)
- [ ] Firing/weapon mechanics function correctly
- [ ] Collisions behave as expected
- [ ] Bosses and enemy waves spawn and behave properly
- [ ] HUD displays accurate information
- [ ] Game scales responsively on different screen sizes
- [ ] No visual glitches (clipping, z-order issues, flickering)
- [ ] Frame rate remains smooth (use DevTools Performance tab)
- [ ] Game state resets correctly on new game

## Known Patterns in Codebase

### Game Loop
- RequestAnimationFrame-based rendering pipeline
- Fixed time step or delta-time based physics (verify current implementation)
- State transitions: MENU → PLAYING → BOSS → GAME_OVER → MENU

### Collision Detection
- AABB (axis-aligned bounding box) or circle-based detection
- Verify what shapes are used for: player, bullets, enemies, bosses
- Order of checks matters for performance

### Enemy/Boss AI
- Wave patterns and spawn sequences
- State machine for boss phases
- Attack timing and difficulty curves

### Visual Effects
- Canvas-based particle system
- Vignette effects for hit feedback
- Screen flash for impacts
- Rotation/scaling for visual impact

## Performance Considerations

- Canvas size is constrained to max-width/max-height to prevent over-rendering on large displays
- Avoid creating objects in the main game loop
- Use canvas transforms sparingly; batch drawings when possible
- Mobile devices may drop frames under heavy load — profile and optimize

## Debugging Tips

1. Use Chrome DevTools Console to log game state
2. Performance tab to identify rendering bottlenecks
3. Network tab to verify no blocking resources
4. Sources tab to set breakpoints in the embedded script
5. Check for `console` logs left behind (remove before committing unless essential for debugging)

## When to Request Review vs. Self-Serve

**Self-serve (Claude only):**
- Bug fixes with clear reproduction steps
- Mechanic tweaks with obvious before/after behavior
- Code cleanup that preserves all existing functionality
- Performance optimizations with verified results

**Should involve user review:**
- Major gameplay design changes
- New game modes or significant features
- Changes affecting boss/difficulty balance
- Accessibility improvements that may affect gameplay feel
- Visual/UI design decisions

## Common Pitfalls to Avoid

1. **Modifying global state without resetting** — ensure new game resets all state
2. **Breaking mobile/touch controls** while fixing desktop
3. **Leaving console logs** in production code
4. **Over-engineering solutions** — prefer simple, clear code for a game
5. **Forgetting to test viewport scaling** before committing
6. **Adding features without considering performance** on lower-end devices

## Branch and Push Guidelines

- All work goes on `claude/inspiring-planck-43hqdl` branch
- Push with: `git push -u origin claude/inspiring-planck-43hqdl`
- Retry on network failure with exponential backoff (2s, 4s, 8s, 16s)
- Do NOT push to main without explicit user permission
- Do NOT create a pull request unless explicitly requested

## Tools and Environment

- **Browser testing:** Chrome/Chromium (primary), Firefox, Safari (mobile)
- **No build step:** Single-file deployment — changes go directly to index.html
- **Git workflow:** Local development on designated branch, push to origin
- **Performance profiling:** Chrome DevTools Performance and Rendering tabs

## Attribution and Session Context

This guide was created for use with Claude Code agents working on the Sky Fighter game. All commits should include proper attribution per the footer format above.

For questions about this project or guidelines, consult the commit history and this file.

---
_Last updated: 2026-09-27_
