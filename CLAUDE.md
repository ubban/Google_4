# Sky Fighter Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML/JavaScript application. Players control an aircraft, switch weapons, drop bombs, and defeat enemies and bosses across multiple stages.

- **Type**: Web-based game (single HTML file)
- **File**: `index.html` (~1,669 lines)
- **Gameplay**: Vertical scrolling shooter with weapon upgrades, combo system, and difficulty modes
- **Target**: Desktop and mobile browsers

## Game Architecture

### Core Game Loop
The main game loop is event-driven with:
- **Canvas rendering** (2D context)
- **Frame timing** (deltaTime-based updates)
- **Entity management** (player, enemies, bullets, effects)
- **State machine** (menu, gameplay, pause, gameover)

### Main Systems

#### 1. **Player System**
- Position-based movement (keyboard/touch controls)
- Three weapon types with upgrade levels (0-3)
- Smart bomb mechanics (limited use, level-based power)
- Health system (hearts/lives)
- Invulnerability frames after hit

#### 2. **Weapon System**
Located in `DIFFICULTIES` and weapon upgrade mechanics:
- **Peashooter** (basic, single bullet)
- **Spread Shot** (wide cone pattern)
- **Plasma** (rapid fire, highest DPS)
- **Bomb** (smart bomb with area effect)

Each weapon has:
- Fire rate (cooldown)
- Damage per shot
- Bullet speed
- Upgrade progression (increases power at each level)

#### 3. **Enemy System**
Defined in `ENEMY_TYPES` object:
- Multiple enemy types with different patterns
- Health/armor values
- Point values
- Wave-based spawning (18 kills per stage = stage clear)
- Formation patterns and AI

#### 4. **Boss System**
- Boss appears every N waves
- Large health pool
- Detailed boss bar UI
- Multi-phase patterns (stage-dependent)
- Special visual effects and audio

#### 5. **Scoring & Progression**
- Base score from kills (enemy type dependent)
- Score multiplier from combo chains
- Difficulty multiplier applied to score
- Best score tracked in localStorage
- Stage progression (cleared after 18 kills)

#### 6. **UI System**
- HUD (score, best, combo multiplier, weapon info, health, bombs)
- Stage banner announcements
- Boss health bar
- Start overlay with difficulty selection
- Pause state
- Game over results screen

### Input Handling
- **Desktop**: Arrow keys (movement) + spacebar (fire) + B key (bomb)
- **Mobile**: Virtual joystick + on-screen fire/bomb buttons
- **Touch**: Tap to move, hold for continuous fire
- **Settings**: Mute toggle, motion reduction, pause

### Storage & Persistence
Game uses `localStorage` with keys:
- `skyfighter.bestScore` - High score
- `skyfighter.muted` - Audio mute state
- `skyfighter.difficulty` - Last selected difficulty
- `skyfighter.reducedMotion` - Accessibility preference

### Performance Optimization
- **Dynamic quality scaling** (full/medium/low) based on frame rate
- **Frame time tracking** to detect performance issues
- Particle effect pooling
- Limited simultaneous entities

## Gameplay Mechanics

### Difficulty Modes
Three difficulty tiers (`DIFFICULTIES` object):
- Affects enemy health, weapon damage, fire rates
- Score multipliers
- Enemy spawn patterns
- Boss mechanics

### Combo System
- Consecutive hits increase multiplier
- Resets on missed attack window
- Visible combo counter in HUD
- Audio/visual feedback on combo milestones

### Weapon Upgrades
- Collected from defeated enemies
- Max level 3 per weapon
- Switching weapons mid-gameplay is instant
- Visual/stat changes at each level

### Smart Bombs
- Limited quantity (2-4 per game depending on pickup)
- Clears bullets and damages enemies
- Visual effect shows blast radius
- Sound effect unique to bombs

## Code Organization

### Game State
Main state variables controlled through events and game loop:
- Player position, health, score, stats
- Current weapon and level
- Enemies array, bullets array, effects array
- Game state (playing, paused, gameover)
- Combo counter and stage count

### Key Functions (by category)

**Setup & Init**:
- `setupAudio()` - Initialize Web Audio API
- `setupCanvas()` - Initialize canvas and resize handler
- Page lifecycle handlers

**Game Loop**:
- Main update/render loop (RAF callback)
- Entity updates (player, enemies, bullets)
- Collision detection
- Rendering pipeline

**Player Control**:
- Keyboard/touch event listeners
- Movement calculation
- Firing/bomb logic
- Invulnerability state management

**Enemy/Combat**:
- Enemy spawn logic
- Enemy AI and movement patterns
- Bullet collision detection
- Damage application and particle effects

**Audio/Visual**:
- Sound effect playback
- Screen effects (flash, vignette)
- Particle system
- Text announcements

**UI Updates**:
- HUD element updates (score, weapon, etc.)
- Overlay/menu state management
- Boss bar updates

### Styling & Layout
- Dark theme (#05122b base)
- Responsive canvas sizing
- Safe area insets for notches (mobile)
- Accessibility features (high contrast, reduced motion support)

## Development Guidelines

### Adding Features

**New Enemy Type**:
1. Add entry to `ENEMY_TYPES` object with stats
2. Set spawn wave pattern in stage definition
3. Define movement AI in update logic
4. Add sprite/rendering in draw function

**New Weapon**:
1. Add to weapon upgrade system
2. Define fire rate, damage, pattern in `DIFFICULTIES`
3. Implement bullet spawn logic
4. Add visual effect/sprite
5. Update HUD weapon display

**Boss Customization**:
1. Modify boss health/appearance per stage
2. Adjust attack pattern timing
3. Add visual phases or special attacks
4. Update boss bar and UI feedback

### Code Practices
- Keep game loop tight (no blocking operations)
- Use requestAnimationFrame for smooth 60fps
- Pool objects where possible (bullets, particles)
- Throttle sound effects to prevent overlap
- Test performance on low-end devices (quality scaling)

### Testing
- Test on both desktop and mobile (Chrome DevTools mobile emulation)
- Verify touch controls work with lag
- Check audio playback on muted devices
- Test localStorage clearing doesn't break game
- Verify frame rate stability (monitor quality scaling)

## Customization Tips

### Game Feel Tuning
- **Player speed**: Adjust movement multiplier
- **Fire rate**: Modify weapon cooldown times
- **Enemy difficulty**: Change health/damage in difficulty config
- **Combo timing**: Adjust combo reset window

### Visuals
- **Canvas gradient**: Modify `gameCanvas` background in CSS
- **Colors**: Theme colors in CSS variables
- **Particle colors**: RGB values in effect spawning
- **Enemy sprites**: Canvas drawing commands

### Progression
- **Kills per stage**: Change `KILLS_PER_STAGE` constant
- **Boss frequency**: Modify when bosses spawn
- **Difficulty balance**: Tweak weapon/enemy stats in `DIFFICULTIES`

## Known Limitations & Future Improvements
- Single HTML file (no build process needed, but large)
- Canvas-only rendering (no WebGL optimization)
- Mobile-only touch, no gamepad support
- Autosave not implemented (only high score)
- Limited visual effects on low-end devices

## Testing Checklist
- [ ] Game starts without errors
- [ ] Player can move (keyboard + touch)
- [ ] Firing works on both input methods
- [ ] Bombs drop with limited ammo
- [ ] Enemies spawn and move
- [ ] Collision detection works
- [ ] Score updates correctly
- [ ] Difficulty selection affects gameplay
- [ ] High score persists across sessions
- [ ] Audio mute state persists
- [ ] Game runs at 60fps (check console quality)
- [ ] Mobile layout responsive (test on small screens)
- [ ] Pause/resume works correctly
