# Sky Fighter Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with HTML5 Canvas and vanilla JavaScript. Players control a fighter jet, switch weapons, drop smart bombs, and defeat stage bosses. The game is a single-page application delivered via `index.html`.

## Architecture & Key Files

### Main Files
- **`index.html`**: Single-file game implementation containing:
  - HTML structure with game canvas
  - Complete CSS styling (dark theme with blue accents)
  - JavaScript game engine and logic (all game code)

### Game Structure

The game engine handles:
- **Canvas rendering**: 2D game viewport with sky gradient background
- **Game state**: Player position, health, score, stage, difficulty
- **Entities**: Player, enemies, projectiles, power-ups, bosses
- **Input handling**: Keyboard and touch/mobile controls
- **Collision detection**: Bullet-to-enemy, enemy-to-player interactions
- **HUD**: Real-time display of score, health, stage, waves

## Development Guidelines

### When Modifying Game Logic

1. **Test in browser**: Always verify changes by opening `index.html` in a browser and playing through the game
2. **Preserve game feel**: Balance is crucial - weapon damage, enemy HP, spawn rates, difficulty curves
3. **Mobile-first**: Game supports touch controls on mobile devices; test responsiveness
4. **Performance**: Canvas rendering can be expensive; minimize redraws and optimize collision checks

### Code Style

- **Vanilla JavaScript**: No frameworks or build tools required
- **Single-file structure**: All code lives in `index.html` within `<script>` tags
- **Comments**: Add comments for non-obvious game mechanics or complex algorithms
- **Canvas API**: Uses standard 2D context methods for drawing

### Common Modification Areas

#### Weapon Balance
- Look for damage values, fire rates, projectile speed in entity definitions
- Test spawn patterns and cooldowns in actual gameplay

#### Enemy Spawning
- Wave definitions control enemy patterns and difficulty progression
- Adjust spawn positions, quantities, and behavior for difficulty tuning

#### Visual Tweaks
- CSS in `<style>` section controls colors, layout, shadows
- Canvas drawing calls in JavaScript handle game graphics

#### Controls
- Input handling supports both keyboard (Arrow keys, Space, Z) and touch
- Mobile viewport uses `max-width` and `max-height` constraints

## Running & Testing the Game

1. **Local Testing**: Open `index.html` directly in a web browser
2. **Play Through**: Complete multiple stages to verify changes work correctly
3. **Test Edge Cases**: 
   - Mobile/touch controls
   - Collision detection at screen edges
   - Boss fight mechanics
   - Power-up interactions

## Common Tasks

### Adding a New Enemy Type
1. Locate enemy entity definitions in the JavaScript
2. Create new entity object with position, health, and behavior properties
3. Add spawn logic to appropriate wave or encounter
4. Test collision and rendering

### Adjusting Difficulty
1. Modify enemy spawn rates, health, or damage values
2. Adjust player weapon cooldowns or damage
3. Tune stage progression and wave scaling
4. Test game pacing across all stages

### Fixing Bugs
1. Identify the affected system (rendering, physics, input, etc.)
2. Add console logging if needed for debugging
3. Test fix in browser at multiple screen sizes
4. Verify no regressions in related features

## Git Workflow

- **Branch**: Develop on `claude/inspiring-planck-gkypls`
- **Commits**: Use clear, descriptive messages (e.g., "Balance weapon damage", "Fix mobile touch input")
- **Testing**: Always test changes in-browser before committing

## Notes for Claude

- The game is self-contained in a single HTML file for easy distribution
- Canvas rendering is direct - no intermediary rendering library
- Game state is managed through JavaScript objects
- Performance considerations apply due to canvas frame-by-frame rendering
- Mobile responsiveness is important (viewport meta tag, touch events)
