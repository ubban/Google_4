# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built entirely in vanilla HTML5, CSS3, and JavaScript. The game runs in a single `index.html` file with no external dependencies. Players control an aircraft, blast through enemy waves, collect weapon upgrades, and survive boss encounters across multiple difficulty levels.

### Key Features
- **Gameplay Mechanics**: Vertical scrolling shooter with player aircraft, multiple enemy types, and boss battles
- **Difficulty Modes**: Easy, Normal, and Hard with different spawn rates, enemy speed, and HP multipliers
- **Weapon System**: Multiple weapons (VULCAN, SCATTER, LASER) with upgrade levels
- **Power-ups**: Weapon upgrades, health restoration, bomb capacity increases
- **Boss System**: 6 unique boss types (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- **Audio**: WebAudio API synthesis for all sound effects (no external audio files)
- **Responsive Design**: Supports desktop (mouse/keyboard) and mobile (touch) controls
- **Persistence**: LocalStorage for high scores, mute state, difficulty preference, and motion preference
- **Accessibility**: Screen reader support and reduced motion preferences

## File Structure

### Single File Architecture
- **index.html** - Complete game (CSS + HTML + JavaScript)
  - HTML structure: Canvas element, HUD overlays, game controls
  - CSS: Responsive styling with mobile-first approach, animations, themes
  - JavaScript: Core game engine (~2000+ lines)

## Code Architecture

### Main Game Loop
```javascript
function gameLoop(now) {
  const dt = Math.min((now - lastTime) / 1000, 0.05);
  lastTime = now;
  
  update(dt);
  render();
  trackPerf(dt);
  
  requestAnimationFrame(gameLoop);
}
```

### Key Game Objects and Systems

#### 1. Player System (`player` object)
- **Properties**: `x`, `y`, `w` (width), `h` (height), `hp` (health), `weapon`, `bombsLeft`
- **Methods**: Movement, firing, taking damage, healing
- **Constraints**: Canvas-bounded, collision detection with enemies/bullets

#### 2. Enemy System
**Three enemy types defined in `ENEMY_TYPES`:**
- **grunt**: Basic enemy, 34×34px, 1 HP, red (#e53935)
- **interceptor**: Fast weaving enemy, 26×26px, 1 HP, orange (#ff7043)
- **tank**: Heavy armored enemy, 46×46px, 4 HP, purple (#5e35b1), introduced in stage 2

**Spawning Logic:**
- Base spawn interval depends on difficulty
- Spawn rate tightens as difficulty increases
- All enemies removed when stage boss spawns (after 18 kills per stage)

#### 3. Weapon System (`WEAPON_INFO` object)
Each weapon has 5 upgrade levels (1-5). Properties:
- `dmg`: Damage per bullet (starts at 1, increases with level)
- `spread`: Fire spread angle
- `fireRate`: Cooldown between shots (ms)
- `bulletSpeed`: Pixels per second
- `color`: Visual bullet color

**Available Weapons:**
- VULCAN: Balanced spray weapon
- SCATTER: Wide spread, low damage
- LASER: Narrow beam, high damage, slow fire rate

#### 4. Boss System (`BOSSES` array)
Six bosses with unique visuals and stats:
- VANGUARD (stage 1), BEHEMOTH (stage 2), SPECTER (stage 3), DREADNOUGHT (stage 4), WRAITH (stage 5), TITAN (stage 6)
- Boss HP calculated: `baseHP * DIFFICULTIES[difficulty].bossHpMul * boss.hpMul`
- Boss drawn with custom canvas paths, each has unique color scheme

#### 5. Theme System (`THEMES` array)
Six atmospheric themes cycled per stage:
- Each theme has: sky gradient colors, cloud/sun colors for parallax, name
- Themes: Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge

#### 6. Difficulty System (`DIFFICULTIES` object)
Three difficulty presets with parameters:
- `spawnBase` / `spawnMin`: Enemy spawn rate (ms)
- `enemySpeed`: Multiplier for enemy movement
- `enemyShootChance`: Probability enemy fires per frame
- `lives`: Player starting health
- `bombs`: Starting bomb count
- `bossHpMul`: Multiplier for boss HP

### Core Game State Variables

**Game Flow:**
- `gameState`: 'menu', 'playing', 'paused', 'gameOver'
- `stage`: Current stage number (1-6, cycles)
- `score`: Current game score
- `mult`: Score multiplier (increases with consecutive kills)
- `kills`: Enemy count for current stage progression

**Player State:**
- `player.hp`: Current health (0 = game over)
- `player.weapon`: Currently equipped weapon
- `player.weaponLvl`: Weapon upgrade level (1-5)
- `player.bombsLeft`: Remaining bomb count

**Collections:**
- `enemies[]`: Active enemy objects
- `enemyBullets[]`: Bullets fired by enemies
- `playerBullets[]`: Bullets fired by player
- `particles[]`: Visual effects (explosions, debris)
- `pickups[]`: Dropped items (weapon upgrades, health, ammo)

### LocalStorage Keys
- `skyfighter.bestScore`: High score persistence
- `skyfighter.muted`: Audio mute state (0/1)
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Motion preference (0/1)

## Control Systems

### Keyboard Controls
- **Arrow Keys / WASD**: Move player aircraft
- **Space Bar**: Fire weapon
- **X / Shift**: Detonate smart bomb
- **P / Esc**: Pause game

### Mouse Controls
- **Move**: Position matches cursor (desktop only)
- **Click**: Fire weapon

### Touch Controls (Mobile)
- **Left 55% of screen**: Drag to move aircraft (shows joystick visualization)
- **Right 45% lower**: FIRE button (top), BOMB button (bottom left)
- **Fire Button**: Auto-fires while held
- **Bomb Button**: Detonates smart bomb

## Visual and Audio Effects

### Visual Feedback
- **Screen Shake**: On bomb detonation (disabled with reduced motion)
- **Flash Effect**: Brief white flash on player hit (reduced motion disables)
- **Vignette**: Red pulsing edge effect when player health low (critical state)
- **Combo Pop**: Floating "+X COMBO" text on screen
- **Stage Banner**: Fades in at stage start
- **Boss Health Bar**: Appears when boss spawns

### Audio Effects (WebAudio Synthesis)
All generated procedurally, no external files:
- **Fire Sound**: Square wave sweep with filtering
- **Hit/Explosion**: Noise burst with quick decay
- **Pickup**: Rising tone sweep
- **Boss Defeat**: Descending sweep sequence
- **Game Over**: Falling tone

## Performance Optimization

### Adaptive Quality System
```javascript
let quality = 0; // 0 = full, 1 = medium, 2 = low
```
Monitors frame time over 45-frame rolling average:
- If avg > 1/40s (25ms): Reduce quality (scale down rendering, reduce particles)
- If avg < 1/56s (~17.8ms): Increase quality

### Rendering Optimizations
- Particle pooling (reuse dead particles)
- Culled objects outside viewport
- Quality-based enemy detail reduction
- Canvas state caching

## Game Flow

### Menu State
1. Display overlay with:
   - Title and subtitle
   - Previous score stats (if game ended)
   - Difficulty buttons (Easy/Normal/Hard)
   - Start button
   - Control hints

### Playing State
1. Spawn initial enemies
2. Player controls aircraft, fires at enemies
3. After 18 kills, stage complete → boss spawns
4. Boss defeated → stage transitions (waves, theme change, difficulty increase)
5. Health depleted → Game Over state

### Game Over
- Final score displayed
- "New Best Score" badge if high score beaten
- Overlay shows results, allows difficulty selection, returns to menu

## Important Constants and Ratios

- **Canvas**: Fills container with max-width/max-height constraints
- **KILLS_PER_STAGE**: 18 enemy kills to progress to boss
- **Weapon Levels**: 1-5 (pickups increment level, capped at 5)
- **Starting Lives**: 3-4 depending on difficulty
- **Starting Bombs**: 2-3 depending on difficulty
- **Boss HP Base**: Calculated dynamically, scales by stage and difficulty

## Browser Compatibility

- **Audio**: Requires `AudioContext` or `webkitAudioContext`
- **Touch**: Full touch event support for mobile
- **Canvas**: 2D context required
- **LocalStorage**: Required for persistence
- **Accessibility**: ARIA attributes, screen reader announcements

## Recommended Development Practices

### When Modifying Game Balance
1. Edit `DIFFICULTIES` object for spawn rates and enemy stats
2. Edit `ENEMY_TYPES` for individual enemy properties
3. Edit `WEAPON_INFO` for weapon balance
4. Edit `BOSSES` for boss HP multipliers
5. Test across all difficulty levels

### When Adding Visual Effects
- Use `particles[]` array for temporary visual elements
- Respect `reducedMotion` flag to disable animations for accessibility
- Ensure canvas state is restored after custom drawing

### When Adding Sound
- Use `beep()` for tonal effects or `noiseBurst()` for percussion
- Ensure audio respects `muted` flag
- Call `ensureAudio()` first to initialize AudioContext (required for user interaction)

### When Modifying Controls
- Touch controls in `#touchControls` div and event listeners
- Keyboard listeners use `keydown`/`keyup` events
- Mouse/touch movement is position-based, not velocity-based

## Asset Locations
- **Game Logic**: Lines 422-2000+ (JavaScript in `<script>` tag)
- **Styling**: Lines 10-359 (CSS in `<style>` tag)
- **HTML Elements**: Lines 361-420 (game containers, overlays, buttons)

## Known Limitations
- Single-file architecture means no code splitting
- All assets/audio synthesized in real-time (no pre-cached assets)
- Mobile performance depends on device browser capabilities
- No network multiplayer (single-player only)

## Future Enhancement Ideas
- Additional boss types and stages beyond 6
- Procedural boss generation
- Leaderboard integration
- Particle effects customization
- Additional weapon types
- Campaign/story progression
- Unlockable skins or themes
