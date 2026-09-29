# Claude Code Configuration for Sky Fighter

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built with vanilla JavaScript and HTML5 Canvas. Players control an aircraft, switch weapons, use smart bombs, and defeat stage bosses.

- **Repository**: ubban/google_4
- **Language**: JavaScript (HTML5 Canvas)
- **Primary File**: `index.html` (~59KB, self-contained)
- **Type**: Browser-based game

## Repository Structure

```
Google_4/
├── index.html          # Complete game implementation (HTML + CSS + JS)
├── README.md           # Project metadata
└── .git/              # Git repository
```

## Key Implementation Details

### Canvas Game Architecture
- **Canvas ID**: `gameCanvas` (dynamically sized)
- **Game Loop**: RAF-based with DeltaTime
- **Viewport**: Responsive scaling, safe-area insets for mobile
- **Touch Controls**: Cross-platform input (mouse/touch/keyboard)

### Game Systems
1. **Player Ship**: Controllable aircraft with position tracking
2. **Weapons**: Multiple weapon types (switchable during gameplay)
3. **Smart Bombs**: Limited-use screen-clear mechanics
4. **Enemies**: Spawning patterns, collision detection
5. **Bosses**: Stage-based boss encounters
6. **UI/HUD**: Score, lives, weapon status, game state display

### Styling Approach
- Dark theme with gradient sky background (#0d2f5e → #7fc1ff)
- Crosshair cursor for targeting feedback
- Text shadows and glows for visibility
- Responsive typography using clamp()
- Mobile-friendly safe-area insets

## Development Workflow

### Git Branch
All development should be on: `claude/inspiring-planck-9ppa7s`

### Before Starting Work
1. Fetch latest from remote
2. Verify branch is checked out
3. Check `git status` for uncommitted changes

### When Making Changes
1. Edit `index.html` directly (all code is in one file)
2. Test changes in a browser to verify visual/gameplay behavior
3. Use browser dev tools to debug console errors
4. Keep Canvas rendering and game logic modular within one file

### Committing Work
- Write clear, descriptive commit messages
- Reference the specific feature/fix being implemented
- Include attribution: `Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>`
- Example: `Add new weapon type: spread shot with 5-way projectiles`

### Pushing Changes
```bash
git push -u origin claude/inspiring-planck-9ppa7s
```

## Testing & Validation

### Manual Testing Checklist
- [ ] Game starts without console errors
- [ ] Canvas renders properly (gradient background visible)
- [ ] Player ship appears and is controllable (arrow keys or WASD)
- [ ] Weapons can be switched
- [ ] Smart bombs can be activated
- [ ] Enemies spawn and move
- [ ] Collision detection works
- [ ] Score/HUD updates correctly
- [ ] Responsive design works (test different viewport sizes)
- [ ] Touch controls work (on mobile/tablets)

### Browser Compatibility
- Target: Modern browsers (Chrome, Firefox, Safari, Edge)
- HTML5 Canvas and ES6+ support required
- No external dependencies

## Code Guidelines

### No External Dependencies
- Vanilla JavaScript only
- No frameworks or libraries (Canvas, DOM APIs only)
- Keep file size reasonable

### Architecture Principles
- Game loop runs continuously via `requestAnimationFrame`
- Entity-based system for ships, enemies, projectiles
- State machine for game phases (menu, playing, boss, gameover)
- Collision detection via bounding boxes or distance

### Performance Considerations
- Limit entity count to maintain 60 FPS
- Pool projectiles/enemies to avoid frequent allocations
- Use efficient canvas rendering (avoid overdraw)
- Optimize collision detection with spatial hashing if needed

### UI/UX Standards
- Clear feedback for player actions
- Visual indication of weapon/bomb status
- Boss health display during boss fights
- Game over/victory screens with replay option

## Special Instructions for Claude Code

1. **Always test gameplay** before declaring a feature complete. Changes to game logic must be verified in the browser.

2. **Keep the single-file structure** - all code stays in `index.html` unless refactoring into separate modules is explicitly requested.

3. **Preserve existing game balance** - if modifying enemy difficulty, boss patterns, or weapon values, ensure the game remains fair and fun to play.

4. **Mobile-first responsive** - test viewport changes frequently; use clamp() for scalable fonts and padding.

5. **Canvas optimization** - avoid re-drawing static elements if possible; clear canvas before each frame; batch draw calls.

6. **Accessibility** - maintain keyboard controls; provide touch alternatives; ensure readability in HUD.

## Common Tasks

### Adding a New Weapon Type
1. Define weapon properties (damage, fire rate, spread, sprite)
2. Add weapon to the weapon cycle system
3. Implement firing logic in the game loop
4. Test switching between weapons
5. Balance damage vs. fire rate

### Adding Enemy Patterns
1. Define enemy type with health, speed, score value
2. Create spawn pattern function
3. Implement movement behavior (sine wave, spiral, etc.)
4. Add collision/projectile detection
5. Test difficulty progression

### Tuning Difficulty
- Adjust enemy spawn rates in stage progression
- Modify boss health and attack patterns
- Balance weapon damage and bomb effectiveness
- Test with multiple playthroughs

## Attribution & Session Info
- Session created by Claude Code agent
- Configuration date: 2026-09-29
