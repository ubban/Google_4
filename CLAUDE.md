# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely as a single HTML5 file (`index.html`). The game features:
- Three difficulty levels (Easy, Normal, Hard) with progressive challenge scaling
- Six distinct enemy types and boss encounters across stages
- Procedural 2D graphics rendered via canvas
- Synthesized sound effects using Web Audio API (no external audio assets)
- Touch-based controls for mobile devices
- Persistent score tracking via localStorage
- Accessibility features: screen reader announcements, reduced motion preferences

## Architecture & Key Systems

All game logic is contained in `index.html` with three main sections:

### 1. **CSS (lines 10–420)** 
- Game canvas styling with responsive sizing via `clamp()`
- HUD elements: score, health (hearts), bombs, weapon display
- Overlay screens: start menu, game over, pause
- Visual effects: flashes, vignettes, combo popups, stage banners
- Animations for UI elements (button presses, screen transitions)
- Responsive layout with `env(safe-area-inset-*)` for mobile notches

### 2. **JavaScript Game Engine (lines 422–1669)**

#### **Core State Management**
- `Game` object singleton holds: player, enemies, projectiles, stage, difficulty, score
- `Player` object: position, health, bombs, current weapon, velocity
- Enemy and projectile objects stored in arrays; destroyed each frame when hp ≤ 0

#### **Game Loop & Rendering**
- **Main render loop:** `window.requestAnimationFrame(gameLoop)` calls `gameLoop(now)`
- **Frame timing:** `gameLoop` calculates delta time (`dt`), updates game state, then renders
- **Adaptive performance:** Tracks frame times in `frameTimes` array; lowers quality setting (particle count, effect detail) if avg FPS drops below 40

#### **Player Mechanics**
- **Movement:** WASD or arrow keys; mouse/touch aims reticle; can move and shoot independently
- **Weapons:** Four weapon types (Basic, Spread, Laser, Homing) with upgradeable levels; each has unique fire rate and spread pattern
- **Bombs:** Smart bombs clear all on-screen projectiles and damage all enemies; limited supply per stage
- **Health:** Collision with enemy or enemy projectile costs 1 life; game over at 0 health

#### **Enemy Spawn & Behavior**
- **Difficulty curves:** `DIFFICULTIES` object defines spawn rate, enemy speed, shoot chance, and boss HP multiplier per difficulty
- **Enemy types** defined in `ENEMY_TYPES`: grunt, interceptor (fast + weave), tank (slow + tanky)
- **Spawn mechanics:** Enemies spawn from top randomly; spawn rate decreases as kills accumulate, triggering boss at `KILLS_PER_STAGE` (18) kills
- **Boss mechanics:** Each boss has unique color scheme and damage scaling; appears once per stage; boss bar shows health; telegraph phase before big attack

#### **Collision & Scoring**
- **Hit detection:** Simple AABB (bounding box) for all entities
- **Weapon multiplier:** Hit streak resets on player damage; each consecutive hit raises multiplier (visible in HUD)
- **Enemy drops:** On kill, enemies drop projectiles or weapon upgrades (randomized by weapon level)

#### **Audio (Web Audio API)**
- **Synthesized sounds:** No external audio files; all effects generated via `beep()` (oscillator + envelope) and `noiseBurst()` (pink noise)
- **Spatial audio:** `panFor()` maps x-position to stereo pan; gunfire and explosions have directional effect
- **Mute state:** Stored in localStorage; `muted` flag disables all sounds

#### **Theme System**
- `THEMES` array defines six visual themes (Daylight, Sunset, Night, Storm, Arctic, Volcanic)
- Each theme has sky gradient colors, cloud/sun color palette; randomly selected at game start

#### **Input Handling**
- **Keyboard:** `keydown`/`keyup` on WASD, arrows, space (fire), X (bomb), P (pause), M (mute)
- **Touch:** Virtual joystick and fire/bomb buttons; zones defined in CSS (left side = move, right side = fire)
- **Mouse:** Move player to cursor; click to fire (desktop); also feeds touch zone logic for hybrid devices

#### **Persistence & Settings**
- **localStorage keys:**
  - `skyfighter.bestScore`: High score
  - `skyfighter.muted`: Audio mute state
  - `skyfighter.difficulty`: Last selected difficulty
  - `skyfighter.reducedMotion`: Accessibility setting
- **Settings flow:** Loaded on init, saved when user toggles or plays

#### **UI State Machine**
- **States:** `idle` (title screen) → `playing` → `paused` or `gameover`
- **Overlay control:** Visible when not playing; shows score, best score, difficulty buttons, start button
- **HUD updates:** Each frame updates `scoreEl`, `bestEl`, `multEl`, hearts/bombs display, weapon name/level
- **Announcements:** `announce()` pushes text to screen reader via `#srLive` for a11y

## Common Development Tasks

### Running the Game
1. **Local file:** Open `index.html` directly in a browser (file:// protocol)
2. **Local server:** `python3 -m http.server 8000`, then open `http://localhost:8000/`
3. **Mobile:** Serve via `localhost` on network and open on device; supports touch joystick

### Making Changes

#### **Adjust Difficulty Curves**
Edit `DIFFICULTIES` object (line ~485). Parameters:
- `spawnBase`: Initial frame count between spawns (lower = more frequent)
- `spawnMin`: Minimum spawn rate as difficulty scales
- `enemySpeed`: Movement speed multiplier
- `enemyShootChance`: Probability enemy fires per frame
- `lives`, `bombs`: Starting player resources
- `bossHpMul`: Boss health scaling factor

#### **Add/Modify Enemy Types**
Edit `ENEMY_TYPES` (line ~502):
```javascript
myNewType: { w: 48, h: 48, hp: 2, speedMul: 1.2, color: '#ff0000', dark: '#880000', score: 25, minStage: 1 }
```
Then add spawn logic in `spawnEnemy()` to randomly select this type.

#### **Adjust Boss Stats**
Edit `BOSSES` array (line ~508). Each boss has hull colors, wing color, core color, and `hpMul` scaling.

#### **Tweak Visual/Audio Feedback**
- **Screen flash on hit:** `flashHit()` function (line ~698); modify animation duration in CSS
- **Sound effects:** Edit `sfx` object (line ~624); use `beep()` for tones, `noiseBurst()` for percussion
- **Vignette on low health:** `vignetteEl.classList.toggle('on', ...)` at line ~687

#### **Change Themes**
Edit `THEMES` array (line ~493) or randomize selection logic in `startGame()`.

#### **Modify Weapon Behavior**
Weapons are hardcoded in player fire logic. Look for `fireWeapon()` function to adjust:
- Fire rate per weapon
- Spread angle & projectile count
- Bullet speed and lifetime

### Testing & Debugging

- **Console logging:** Add `console.log()` in game loop or event handlers; check DevTools Console
- **Pause mid-game:** Press P to pause and inspect state
- **Frame rate:** Watch browser DevTools Performance tab; check adaptive quality setting in console
- **Mobile testing:** Use DevTools device emulation or real device via local network

### Performance Considerations

- **Canvas rendering:** All graphics drawn per-frame; no GPU shaders
- **Adaptive quality:** `quality` variable (0–2) lowers particle/effect detail if FPS drops
- **Object pooling:** Enemies/projectiles destroyed and recreated each cycle; no pre-allocation
- **Audio synthesis:** Real-time oscillator generation may spike CPU; mute reduces overhead

## Storage & Persistence

Game uses `localStorage` under keys prefixed with `skyfighter.*`:
- Best score, mute state, difficulty, and motion preferences persist across browser sessions
- Private browsing mode may disable localStorage; game runs without it but won't save state

## Accessibility

- **Keyboard & touch:** Full support; no mouse-only mechanics
- **Screen reader:** `announce()` function sends announcements to `#srLive` ARIA-live region
- **Reduced motion:** `prefers-reduced-motion` media query respected; toggleable via motion button
- **Color contrast:** HUD uses high-contrast colors; no color-only info (e.g., health shown as hearts)
- **Focus visible:** UI buttons styled for clear focus state

## Browser Compatibility

- Requires: `Canvas API`, `Web Audio API`, `requestAnimationFrame`
- Optional: `localStorage`, `match.Media (prefers-reduced-motion)`, `Gamepad API` (not implemented)
- Tested: Chrome, Firefox, Safari, Edge (modern versions)
- Mobile: iOS Safari, Chrome Android (touch controls enabled)
