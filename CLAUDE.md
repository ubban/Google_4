# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML file with embedded CSS and JavaScript. Players control a fighter plane, manage weapons and bombs, and defeat enemies and boss battles across multiple stages.

## Running the Game

The game is a standalone HTML file that runs directly in any modern browser:

```bash
# Simply open the file
open index.html

# Or use a local server (Python 3)
python3 -m http.server 8000
# Then visit http://localhost:8000
```

The game is fully responsive and supports:
- Desktop: Mouse steering + left-click to fire + Space/B to drop bomb
- Touch: Virtual joystick on left, fire button on right, bomb button in corner
- Keyboard: Arrow keys to move, Space to fire, B to bomb

## Architecture Overview

The entire game runs in `index.html` as a single-file application with three main sections:

### 1. **Styling** (lines 10–365)
- Dark space theme with responsive units (clamp)
- Game canvas with gradient background
- HUD elements: score, weapon, health (hearts), bombs
- Visual feedback: flash animations for hits/bombs, red vignette for low health, combo popups
- Boss health bar
- Overlay for pause/game-over screens with difficulty selection
- Joystick UI (virtual on mobile)

### 2. **HTML Markup** (lines 365–391)
- Canvas container for game rendering
- HUD statistics (score, weapon, lives, bombs, multiplier)
- Screen reader announcements (`#srLive`)
- Overlay for menus (pause, game-over, difficulty select)
- Control buttons (mute, motion preference, pause)

### 3. **JavaScript Game Logic** (lines 423–1665)

#### Game State (line 864–899)
Core objects and variables:
- `player` — Fighter plane with position, speed, weapon, ammo, invulnerability timer
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss` — Entity arrays
- `score`, `lives`, `running`, `paused`, `gameOver` — Game flags
- `stage`, `stageKills`, `chain`, `multiplier` — Progression tracking

#### Input Handling (lines 724–862)
- **Touch**: Virtual joystick on left zone, fire/bomb buttons on right
- **Mouse**: Plane glides toward cursor, click to fire
- **Keyboard**: Arrow keys for movement, Space to fire, B to bomb
- Both input methods coexist—keyboard/joystick takes priority over mouse

#### Entity Spawning (lines 1022–1071)
- `spawnEnemy()` — Creates waves based on stage and difficulty; enemies follow formation patterns
- `spawnBoss()` — Triggers after 20 kills per stage; boss has distinct attack patterns

#### Core Game Loop (lines 1652–1665)
```javascript
function loop(t) {
  const dt = deltaTime calculation;
  if (running && !paused) update(dt);
  draw();
  requestAnimationFrame(loop);
}
```

#### Update Logic (lines 1148–1410)
Main game tick (`update(dt)`):
- Move player based on input
- Update player weapon cooldown and rapid-fire mode
- Spawn bullets when firing
- Update all entities: bullets, enemies, enemy bullets, particles, power-ups
- Handle collisions: player↔enemy bullets, player↔enemies, bullets↔enemies, player↔power-ups
- Maintain chain multiplier (breaks after 3 seconds without a kill)
- Update bomb flash timer
- Check win/lose conditions

#### Rendering (lines 1507–1650)
- `draw()` — Clears canvas, draws parallax clouds, all entities, HUD, debug info
- `drawPlane(x, y, w, h, color, darkColor, flame)` — Renders player plane with thrust flame
- `drawBossShip(b)` — Renders boss with health indicator and attack patterns

#### Audio System (lines 545–669)
- `ensureAudio()` — Initializes Web Audio context on first user interaction (required by browsers)
- `beep(freq, dur, type, vol, opts)` — Generates sine/square/triangle tones with optional portamento
- `noiseBurst(dur, vol, opts)` — White noise burst for impact sounds
- `panFor(x)` — Stereo panning based on entity x-position
- Mute state persists in `localStorage`

#### Difficulty Modes (lines 905–911)
Three difficulty presets control:
- Initial lives and bomb count
- Enemy spawn rates and patterns
- Enemy bullet speed and density

#### Chain & Multiplier System
- Killing enemies within 3 seconds builds a chain
- Chain reaches x2, x3, x5 multiplier at specific thresholds (5, 15, 30 kills)
- Breaks after 3 seconds without a kill

## Key Gameplay Systems

### Weapons
- **VULCAN** (default): Rapid spread shot, upgrades affect fire rate (levels 1–4)
- **LASER**: Single beam, upgrades affect damage penetration and width (levels 1–3)
- Weapon pickups drop from defeated enemies; collected weapon determines level

### Enemy Types
Enemies spawn in formations with pattern-based movement:
- Regular enemies: Single bullet per volley
- Stronger enemies (higher stage): Tighter patterns, faster movement
- Boss: Distinct attack sequences with phase transitions

### Bombs
- Limited supply (2–3 per game depending on difficulty)
- Area-of-effect damage clearing enemies and bullets
- Flash animation and audio feedback on detonation
- Refill from power-ups or stage resets

### Power-ups
Drop from defeated enemies:
- **Health**: Restores 1 life (capped at max)
- **Weapon**: Upgrades current weapon or switches weapon
- **Bomb**: Refills bomb count

## Development Notes

### Single-File Design
All game logic is in `index.html` to keep deployment trivial. For major refactoring, consider:
- Extracting entity classes (Player, Enemy, Bullet, etc.) into separate modules if file grows beyond ~1800 lines
- Separating CSS into a stylesheet once styles exceed 400 lines
- Using a bundler if Web Audio synth code expands significantly

### Performance Considerations
- Canvas rendering at 60 FPS with delta-time based updates ensures smooth gameplay across devices
- Particle effects and enemy waves are culled off-screen
- Audio is cached per frequency to reduce synthesis overhead
- Parallax clouds use a simple linear scroll (not re-rendered every frame)

### Accessibility
- Screen reader announcements for stage transitions and game-over events
- Motion preference toggle to disable screen shake, flashes, and pulse effects
- Keyboard and touch controls both fully supported
- Color contrast meets WCAG AA standards

### Extending the Game
Common modifications:
1. **Add a new weapon**: Add entry to weapons object, update `weaponLabel()`, modify bullet spawn logic in `update()`
2. **Adjust difficulty**: Modify `DIFFICULTIES` object (line 905)
3. **New enemy pattern**: Modify `spawnEnemy()` to pick new `movePattern` values
4. **Sound customization**: Tweak `beep()` and `noiseBurst()` parameters (frequencies, durations, volumes)
5. **Visual tweaks**: Modify colors and gradients in the `<style>` block

## Git Branches

Development happens on `claude/inspiring-planck-4evrsr`. The main branch contains stable releases.
