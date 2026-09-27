# Claude.md - Development Guide for Sky Fighter

## Project Overview

This repository contains **Sky Fighter**, a production-grade Raiden-style vertical shoot-em-up game built as a single-page HTML5 canvas application. The entire game logic, rendering, audio synthesis, and UI are contained in `index.html`.

## Project Structure

- **index.html** - Complete game application (~1700 lines)
  - Canvas-based 2D rendering engine
  - Game state management and core loop
  - Audio synthesis engine with dynamics processing
  - Touch/keyboard/mouse input handling
  - Accessibility features (reduced-motion, screen reader support)
  - Adaptive performance monitoring

## Development Guidelines

### Code Organization Within index.html

The game follows a modular structure within the single file:

1. **DOM & Canvas Setup** - Canvas initialization, full-screen support
2. **Constants & Configuration** - Game parameters (speeds, sizes, colors, thresholds)
3. **Asset Definitions** - Enemy/boss designs, weapon specs, stage themes
4. **Audio Engine** - Synthesized sound generation with Web Audio API
5. **Game State** - Player, enemies, bosses, projectiles, UI state
6. **Rendering Pipeline** - Draw functions for backgrounds, entities, HUD, overlays
7. **Physics & Collision** - Movement, hit detection, damage application
8. **Input Handling** - Keyboard, touch joystick, mouse steering
9. **Game Loop** - Update, render, frame timing
10. **Accessibility** - ARIA regions, reduced-motion preferences, screen reader support

### Adding Features

When adding new features:

1. **Keep the single-file structure** - Maintain all code in index.html for simplicity
2. **Follow the existing pattern** - Match variable naming and code organization
3. **Test with headless Playwright** - Verify changes work without console errors:
   ```bash
   npx playwright test --headed  # or headless mode
   ```
4. **Test across input methods** - Keyboard, touch, and mouse steering
5. **Verify accessibility** - Test reduced-motion toggle and screen reader announcements
6. **Check performance** - Ensure frame time monitor stays stable across quality tiers

### Game Features to Preserve

- **Three difficulty modes** - Easy, Normal, Hard with distinct enemy spawn rates
- **Multi-stage progression** - Stages with wave-based enemies + boss encounters
- **Boss variety** - 6 unique boss designs with distinct bullet patterns
- **Weapon system** - Vulcan/Laser with 3 power levels, smart bombs
- **Power-ups** - Weapon level, bombs, extra lives, score multipliers
- **Adaptive performance** - Automatic quality scaling based on frame time
- **Audio synthesis** - Layered sounds with spatial panning and dynamics
- **Accessibility** - Reduced-motion, ARIA announcements, keyboard/mouse/touch support
- **Mobile optimization** - Touch joystick, scaling for various screen sizes
- **Persistence** - High score saved to localStorage

### Performance Considerations

- Frame-time monitor tracks performance across 3 quality tiers
- Particle effects and shadows automatically reduced on low-end devices
- Audio uses Web Audio API's native processing (no large audio files)
- All graphics rendered to canvas (no large image assets)

### Input Methods (Priority Order)

1. **Keyboard** - Arrow keys for movement, Space for fire, P for pause
2. **Touch** - Virtual joystick (left side) + fire button (right side)
3. **Mouse** - Cursor-based steering with crosshair
4. Each method works independently; highest-priority active input controls the plane

### Audio Engine

- Synthesized with Web Audio API (no external audio files)
- Master gain routed through dynamics compressor
- Per-sound biquad filtering and pitch sweeps
- Stereo panning based on on-screen x position
- Layered sequences for explosions, boss attacks, UI feedback

## Commit Message Convention

Follow this format for commits:

```
[Feature/Fix] Brief description

Detailed explanation of what changed and why.

- Bullet points for specific changes
- Verification steps taken

Co-authored-by: Claude <noreply@anthropic.com>
```

## Testing Workflow

1. **Manual Testing** - Open `index.html` in a browser, test all input methods
2. **Headless Testing** - Use Playwright to verify core mechanics
3. **Accessibility Check** - Test with reduced-motion enabled and screen reader
4. **Cross-platform** - Verify on desktop and mobile browsers

## Known Technical Details

- Game runs at target 60 FPS with adaptive quality scaling
- Canvas resolution adapts to window size with high-DPI support
- LocalStorage used only for high score persistence
- No external dependencies (pure HTML5 canvas + Web Audio)
- Full TypeScript-like JSDoc comments for critical functions

## Collaboration Notes

- This is a single-developer game project
- Changes should be self-contained to avoid conflicts
- When adding new content (enemies, bosses, stages), add to the asset definition section
- Performance impact should be measured with the frame-time monitor before committing
- Preserve all existing game balance parameters unless intentionally tuning difficulty

## Future Enhancements (Not Yet Implemented)

- Consider: Additional stage themes (8+ total)
- Consider: More boss variety (8+ designs)
- Consider: Leaderboard tracking
- Consider: Achievements/statistics
- Consider: Settings persistence beyond high score

---

Last updated: 2026-09-27
Project Status: Production-ready, accepting polish improvements and feature additions
