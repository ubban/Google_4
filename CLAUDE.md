# Sky Fighter: Codebase Documentation

## Overview
**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML/JavaScript application. The game runs in the browser and features classic arcade-style gameplay with weapon switching, smart bombs, boss encounters, and progressive difficulty stages.

- **Primary File**: `index.html` (1669 lines - single-file architecture)
- **Type**: Browser-based game
- **Genre**: Vertical Shoot-'em-up (Shmup)
- **Platforms**: Desktop and mobile (with touch controls)

## Key Game Features

### Core Gameplay
- **Player Control**: Vertical scrolling flight with movement and firing mechanics
- **Weapons System**: Multiple weapons (Vulcan, etc.) with progressive levels
- **Smart Bombs**: Limited-use bombs for clearing enemies
- **Scoring System**: Points with multiplier mechanics and combo tracking
- **Difficulty Modes**: Easy, Normal, and Hard difficulty settings
- **Life System**: Hearts/lives system with game over detection
- **Boss Battles**: Stage bosses with health bars and unique patterns

### UI/UX Features
- **HUD Display**: Real-time score, best score, multiplier, weapon level, lives, bombs
- **Stage Progression**: Visual stage banners and progression tracking
- **Pause System**: Game pause/resume functionality
- **Mute Control**: Audio on/off toggle
- **Reduced Motion**: Toggle for screen shake, flash, and pulse effects (accessibility)
- **Combo Pop-ups**: Visual feedback for consecutive enemy kills
- **New Best Score**: Celebration display when high score is beaten

### Mobile Support
- **Touch Controls**: Joystick-based movement control
- **Fire Button**: Touch-friendly fire control
- **Bomb Button**: Dedicated bomb trigger for touch
- **Responsive Design**: Adapts to various screen sizes and orientations

### Accessibility
- **ARIA Live Region**: Screen reader announcements for game events
- **Keyboard Navigation**: Full keyboard control support
- **Reduced Motion Preference**: Respects user motion preferences
- **Color Contrast**: Dark theme with good contrast ratios

## Code Architecture

### Single-File Structure
All game code is contained in `index.html`:
- **HTML Section** (lines ~1-400): DOM structure, layout, and styling
- **JavaScript Section** (lines ~400-end): Game logic wrapped in IIFE

### Key JavaScript Variables & Storage
```
DOM References: canvas, ctx, overlay, UI elements (score, weapons, hearts, etc.)
Touch Controls: moveZone, joystickBase, joystickStick, fireBtn, bombBtn
Local Storage Keys:
  - skyfighter.bestScore: Player's high score
  - skyfighter.muted: Audio mute state
  - skyfighter.difficulty: Selected difficulty
  - skyfighter.reducedMotion: Motion effects preference
```

### Game State Management
- **Score & Multiplier**: Current score and combo multiplier tracking
- **Player State**: Health (hearts), bombs, current weapon and level
- **Stage Progression**: Current stage number and stage-specific mechanics
- **Pause State**: Game pause/resume flag
- **Difficulty Setting**: Easy/Normal/Hard mode selection

### Graphics & Rendering
- **Canvas**: HTML5 2D canvas for all rendering
- **Animation**: RequestAnimationFrame game loop
- **Vignette Effect**: Screen edge darkening for atmosphere
- **Flash/Pulse**: Visual effects for hits and explosions
- **Screen Shake**: Motion effect on impacts

### Audio System
- **Web Audio API**: Used for game sounds and music
- **Mutable**: Global mute control affects all audio
- **Reduced Motion**: Some audio effects affected by motion preference

## Development Guidelines

### Code Style
- Pure JavaScript (no frameworks)
- Single IIFE scope to avoid global pollution
- DOM queries cached in `const` at initialization
- Event listeners for user input (keyboard, touch, mouse)

### When Modifying the Code
1. **Game Mechanics**: Adjust constants for difficulty curves, weapon balance, enemy patterns
2. **Visual Effects**: Modify canvas drawing code in the render loop
3. **UI/HUD**: Update HTML structure and corresponding JavaScript references
4. **Audio**: New sounds integrate with the mute and reduced-motion systems
5. **Touch Controls**: Adjust joystick sensitivity in touch event handlers

### Performance Considerations
- Canvas rendering optimized for 60 FPS
- Minimal DOM manipulation during gameplay (only HUD updates)
- Enemy and bullet pooling patterns for reuse
- RequestAnimationFrame for smooth animation

### Testing Guidelines
- **Manual Testing**: Run in browser across different devices
- **Input Methods**: Test keyboard, mouse, and touch controls
- **Accessibility**: Verify screen reader announcements with arrow keys
- **Mobile**: Test on various screen sizes and orientations
- **Difficulty Levels**: Verify each difficulty level difficulty curve

## Common Tasks

### Adding a New Weapon
1. Define weapon stats (name, fireRate, damage, pattern)
2. Add weapon case in fire logic
3. Update weapon display elements
4. Adjust progression/unlocking logic

### Adjusting Difficulty
- Modify enemy spawn rates, patterns, and health
- Adjust player weapon damage and fire rate
- Balance bomb effectiveness
- Tune scoring multipliers

### Changing Visual Style
- Modify CSS color scheme (currently dark blue theme)
- Update gradient backgrounds
- Adjust canvas drawing colors
- Modify UI element styling

### Adding Accessibility Features
- Use aria-live region for announcements
- Update srLive element text for game state changes
- Ensure keyboard controls map to all touch buttons
- Test with screen readers

## Browser Compatibility
- Modern browsers with HTML5 Canvas support
- Web Audio API for sound
- LocalStorage for persistence
- Touch Events API for mobile

## File Structure
```
Google_4/
├── index.html          # Main game file (1669 lines)
├── README.md           # Brief project description
└── CLAUDE.md           # This documentation
```

## Game Loop Overview
1. **Input Handling**: Process keyboard, mouse, and touch input
2. **Update Logic**: Update player position, enemies, bullets, effects
3. **Collision Detection**: Player-enemy, bullet-enemy, bomb effects
4. **Scoring**: Calculate points and multiplier changes
5. **Rendering**: Clear canvas and redraw all game elements
6. **UI Update**: Update HUD with current state

## Known Patterns
- **Canvas-based rendering**: All visuals drawn to single canvas element
- **Keyboard & Touch input**: Dual-input support for desktop and mobile
- **LocalStorage persistence**: Settings and scores saved locally
- **ARIA live region**: For accessibility announcements
- **Reduced motion respects**: User preference via system settings or toggle
