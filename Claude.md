# Sky Fighter Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-page HTML5 canvas application. The entire game is contained in `index.html` (~1669 lines) with no external dependencies beyond the browser's native Web Audio API.

### Key Characteristics
- **Genre:** Vertical shoot-'em-up (shmup) with progressive difficulty
- **Architecture:** Single HTML5 file with inline CSS and JavaScript
- **Platform Support:** Desktop (keyboard/mouse) and mobile (touch joystick)
- **Accessibility:** Reduced-motion toggle, screen reader support via aria-live regions
- **Performance:** Adaptive quality tiers based on frame-time monitoring

---

## Core Game Systems

### 1. Game State & Lifecycle

**Core Game Modes:**
- **Start Screen:** Title overlay with difficulty selection (Easy/Normal/Hard)
- **In-Game:** Active gameplay loop
- **Pause:** Freezes action, shows pause overlay
- **Game Over:** Shows final stats and score

**Key State Variables:**
- `gameActive`: Boolean indicating if gameplay is running
- `gamePaused`: Pause state
- `gameState.stage`: Current stage number (1-indexed)
- `gameState.score`: Player's current score
- `gameState.mult`: Score multiplier (kill-chain bonus, resets on hit)
- `gameState.health`: Player health (1-3 lives)
- `gameState.bombs`: Remaining smart bombs
- `gameState.weaponLevel`: Current weapon upgrade level (1-3)

### 2. Player Aircraft

**Movement:**
- **Keyboard:** WASD or Arrow Keys
- **Touch:** On-screen joystick (virtual D-pad)
- **Mouse:** Pointer attracts plane toward cursor (max 1.7x normal speed)
- Plane is confined to canvas bounds with smooth acceleration/deceleration

**Weapons:**
- **Vulcan Gun:** Rapid-fire bullets (default weapon)
- **Laser:** Concentrated beam (toggled via spacebar or touch UI)
- **Weapon Levels:** 3 tiers increase fire rate, spread, and damage
- Power-ups increase weapon level and reset fire-rate cooldowns

**Bombing:**
- Limited smart bombs (typically 3 per life)
- Clears all on-screen enemies and damages bosses significantly
- Visually represented as bombs in HUD

### 3. Enemy Types

**Three Archetypes:**

1. **Grunt (⭐)**
   - Basic enemy type
   - Simple straight-down movement
   - Low health, quick to destroy
   - Explosion sound: basic crackle

2. **Interceptor (🎯)**
   - Weaving movement pattern
   - Attempts to intercept player
   - Medium health
   - Distinct explosion sound

3. **Tank (⚔️)**
   - Slow, armored enemy with its own health bar
   - Multiple hits required
   - Heavy explosion sound
   - Stands out visually with multi-hit indicator

**Spawn Patterns:**
- Waves progress by stage
- Enemy composition varies by stage and difficulty
- Boss encountered at end of each wave

### 4. Boss System

**Six Boss Designs (3 core + 3 extended):**

**Core Bosses:**
1. **Vanguard**
   - Attack patterns: radial burst, aimed fan, wide V-formation
   - Color scheme: blue/cyan

2. **Behemoth**
   - Attack patterns: heavy columns, homing spread, bullet wall
   - Color scheme: red/orange

3. **Specter**
   - Attack patterns: teleport dash, spiral, twin aimed shots
   - Color scheme: purple/dark

**Extended Bosses (Stages 4-6):**
- **Dreadnought:** Heavy, aggressive pattern set
- **Wraith:** Fast teleporting patterns
- **Titan:** Complex multi-phase attacks

**Boss Features:**
- Health bar displayed during fight
- Telegraph flash animation before each attack (visual fairness cue)
- Distinct audio cues for attacks and defeat
- Drop power-ups on death
- Cycle through roster in stage order (Vanguard → Behemoth → Specter → Dreadnought → etc.)

### 5. Power-Up System

**Types:**
- **Weapon Level (W):** Increases weapon tier (1→2→3, resets fire cooldown)
- **Bomb (+):** Adds one smart bomb to inventory
- **Extra Life (❤️):** Adds one life, caps at 3

**Drop Behavior:**
- Bosses always drop power-ups
- Some enemies drop power-ups randomly
- Difficulty affects drop rates

### 6. Scoring & Multiplier

**Score Calculation:**
- Base points per kill vary by enemy type
- **Multiplier Chain:** Each consecutive kill without taking damage increases multiplier
- **Multiplier Reset:** Taking damage resets multiplier to 1x
- Final score heavily influenced by kill-chain performance

**HUD Display:**
- Current score in top-left
- Current multiplier in orange text
- Visible feedback when multiplier changes

### 7. Visuals & Aesthetics

**Background Themes (6 rotating options):**
1. Daylight (bright blue sky)
2. Sunset (orange/warm tones)
3. Night (dark blue with stars)
4. (And 3 additional variants)

**Visual Effects:**
- **Parallax clouds:** Multi-layer background scrolling at different speeds
- **Sun glow:** Large gradient background element
- **Hit flash:** White flash overlay on player or enemy hit
- **Vignette pulse:** Red vignette on low health (pulse animation)
- **Screen shake:** Dampened on reduced-motion preference
- **Particle effects:** Explosions with adaptive counts based on performance tier
- **Glow effects:** Bullets and enemies glow; effects drop on low-performance devices

**Aircraft Design:**
- Shaded with cockpit detail, flame exhaust, tail fin
- Rotates slightly based on movement direction
- Scales with zoom for mobile support

### 8. Audio System

**Synthesized Audio Engine:**
- **Master Gain:** Routed through dynamics compressor to prevent clipping
- **Per-Sound Processing:** Biquad filtering, pitch sweeps for weight
- **Stereo Panning:** Based on on-screen x-position for spatial depth
- **Mute Toggle:** Persists in localStorage

**Sound Events:**
- **Enemy Explosions:** Different cues per enemy type (grunt/interceptor/tank)
- **Boss Telegraph:** Ascending tone before boss attack fires
- **Boss Defeat:** Ascending fanfare (distinct from enemy explosions)
- **Stage Clear:** Victory stinger
- **Hit/Bomb/Powerup:** Short tonal cues
- **Game Over:** Descending stinger
- **UI Clicks:** Feedback on difficulty/pause/bomb usage

**Audio Persistence:**
- Mute state stored in localStorage under `skyFighterMute`

---

## Accessibility Features

### Reduced-Motion Support
- Detects `prefers-reduced-motion` OS setting on load
- Dampens screen shake intensity
- Disables flash and pulse animations (vignette)
- Toggle UI for manual override
- State persists in localStorage under `skyFighterReducedMotion`

### Screen Reader Support
- aria-live region announces:
  - Life loss
  - Boss defeats
  - Game over results
- HUD updates relay key game state changes

### Mobile Accessibility
- Touch-friendly UI buttons
- Large interaction targets (minimum 48px recommended)
- Viewport scaling prevents unwanted zoom on input

---

## Performance & Optimization

### Adaptive Quality Tiers
The game monitors frame time (`delta`) and automatically adjusts:

1. **High Performance (fps > 55):**
   - Full particle counts
   - All visual effects enabled (glow, shadow-blur)

2. **Medium Performance (fps 45-55):**
   - Reduced particle counts (~70% of full)
   - Glow effects still enabled

3. **Low Performance (fps < 45):**
   - Minimal particles (~30% of full)
   - Glow and shadow-blur effects disabled

**Monitoring:** Rolling frame-time calculation adjusts quality automatically without user intervention.

---

## Storage & Persistence

**localStorage Keys:**
- `skyFighterMute`: Boolean (mute state)
- `skyFighterReducedMotion`: Boolean (animation preference)
- `skyFighterDifficulty`: String ('Easy'/'Normal'/'Hard')
- Other game progress data as needed

---

## Controls Reference

### Keyboard
- **WASD** or **Arrow Keys:** Move
- **Space** or **Click:** Fire (keyboard mode) / Toggle weapon (alternative)
- **B:** Drop bomb
- **P:** Pause
- **M:** Toggle mute

### Touch
- **Joystick:** Virtual D-pad for movement
- **Fire Button:** On-screen fire button
- **Bomb Button:** Drop bomb
- **Top Control Bar:** Mute/Motion/Pause toggles

### Mouse
- **Move Pointer:** Plane follows cursor (lower priority than keyboard/touch)
- **Click & Hold:** Fire
- **Crosshair:** Reticle tracks cursor for aiming feedback

---

## File Structure

```
Google_4/
├── Claude.md          (This file - codebase documentation)
├── README.md          (Basic project info)
├── index.html         (Complete game - HTML, CSS, JavaScript)
└── .git/              (Version control)
```

All game logic, rendering, audio, and UI are contained within `index.html`.

---

## Key Code Sections (Within index.html)

### Game Loop
- `update()`: Updates game state each frame
- `draw()`: Renders all sprites and HUD
- `requestAnimationFrame()`: 60 FPS target

### Input Handling
- Keyboard event listeners for WASD/arrows/space/B/P/M
- Touch joystick detection and rendering
- Mouse move tracking and click detection
- Priority system: keyboard > touch joystick > mouse

### Collision Detection
- Player bullets vs enemies
- Enemies/bullets vs player
- Boss detection and attacks
- Bomb area-of-effect zones

### Entity Management
- Arrays: `enemies`, `bullets`, `explosions`, `powerups`, `bosses`
- Life cycles: spawn → update → draw → destroy
- Garbage collection: removing dead entities

### Difficulty & Balance

**Easy Mode:**
- Reduced enemy health
- Slower enemy fire rate
- Higher power-up drop chance
- Larger collision forgiveness

**Normal Mode:**
- Standard values
- Balanced difficulty curve

**Hard Mode:**
- Increased enemy health
- Faster patterns
- Lower power-up drop chance
- Tighter collision detection

---

## Development Notes

### Adding New Features

1. **New Enemy Type:**
   - Add spawn logic to wave generation
   - Create update/draw functions
   - Add collision response
   - Create unique explosion sound

2. **New Boss Pattern:**
   - Define attack positions and timing
   - Add telegraph visual
   - Create audio cue
   - Test damage application and health depletion

3. **New Power-Up:**
   - Add spawn logic to drop system
   - Create collision detection
   - Define visual representation
   - Add audio feedback

4. **Performance Tuning:**
   - Adjust particle counts in quality tier thresholds
   - Monitor frame times during intensive scenes (multi-boss or heavy waves)
   - Test on low-end devices (mobile, older browsers)

### Testing Checklist

- [ ] Game starts without console errors
- [ ] Player movement smooth in all directions
- [ ] Weapon firing and switching works
- [ ] Bombs clear enemies and damage bosses
- [ ] Enemy spawning matches wave patterns
- [ ] Boss patterns telegraph and fire correctly
- [ ] Power-ups drop and apply effects
- [ ] Score and multiplier calculate properly
- [ ] Audio plays without distortion on concurrent sounds
- [ ] Mobile touch controls responsive
- [ ] Reduced-motion preferences applied
- [ ] Game over state transitions correctly
- [ ] Pause/unpause preserves game state
- [ ] Difficulty selection changes enemy stats
- [ ] localStorage persistence works across sessions

### Known Limitations

- Single-file architecture makes the codebase very large (~1669 lines)
- Audio synthesis may sound tinny on low-quality speakers
- Mobile performance varies widely by device capability
- Touch joystick occupies screen space (consider on-demand rendering for tablets)

---

## Future Enhancement Ideas

- **Multi-stage visual progression:** Different environments per stage
- **Boss rush mode:** Fight all bosses in sequence
- **Leaderboard:** Local high-score persistence with names
- **Custom difficulty:** Player-adjustable sliders for balance
- **Power-up variants:** Temporary shields, time dilation, extra fire-rate boosts
- **Story elements:** Cutscenes or dialogue (text-based for simplicity)
- **Sound effects randomization:** Slight pitch variation on repeated sounds
- **Screen effects:** Bloom, chromatic aberration for dramatic moments
- **Replay system:** Record and play back input sequences

---

## References & Resources

- **Web Audio API:** MDN docs for synthesizer and audio node documentation
- **Canvas API:** MDN for drawing and animation primitives
- **Touch Events:** MDN for touch handling on mobile
- **Performance:** Use DevTools Performance panel to profile frame times

---

## Author & Attribution

Developed with Claude AI. See `.git/` history for detailed change log.

Last updated: 2026-09-16
