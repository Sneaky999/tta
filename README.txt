# STREET CRIME — Project Structure

A mobile-first top-down open-world crime game.

## Files

```
streetcrime/
├── index.html      ← HTML shell + CSS + script loader
├── world.js        ← Map, zones, tiles, weapons, cars, entities, cop grades
├── assets.js       ← SVG sprites, tile textures, drawCar(), shadeCol(), roundRect()
├── game.js         ← Input, shop, HUD, AI, physics, render, game loop
├── island_a.js     ← [EXPANSION] Missions system (disabled by default)
├── island_b.js     ← [EXPANSION] Events & Weather system (disabled by default)
└── README.md
```

## Module Responsibilities

### `world.js`
- Canvas setup & resize
- World grid (100×100 tiles), zone definitions, special zones
- `buildWorld()`, `isW()`, `tColl()`, `mvE()`
- Weapon definitions (`WDEFS`), weapon slot system
- Car types (`CT`), police car types, `makeCar()`
- Player object (`PL`)
- NPC, gangster, cop spawning + cop grades (`COP_GRADES`)
- Bullet/particle/pickup helpers
- Wanted system (`addWanted`)

### `assets.js`
- Sprite system (`SP`, `svgImg`, `mkTile`, `initSprites`)
- All SVG sprite definitions (player, cop, gang, 6 NPC variants)
- Car overlay SVGs (sedan, sports, truck, SUV, van)
- Pickup SVGs (health, cash, gun)
- Pre-rendered tile textures (grass, road, sidewalk, park, water, building)
- `drawCar(c2, car, isPlayer)` — main car renderer
- `shadeCol(hex, amt)` — color utility
- `roundRect(c2, x, y, w, h, r)` — path utility

### `game.js`
- Input system (touch joystick + keyboard)
- Shop overlay (`openShop`, `closeShop`, `shopBuy`)
- HUD update (`updateHUD`, `buildWHUD`)
- Minimap background (`buildMM`)
- Game state management (`startGame`, `resetGame`)
- `update(dt)` — full AI/physics tick:
  - Player movement (on foot + in car)
  - Traffic AI
  - NPC wandering
  - Gangster territory AI (with `gangKills` aggro scaling)
  - Cop grade-aware AI (patrol, raid, chase)
  - Police car following
  - Bullet collision (faction-aware routing)
  - Pickup collection
  - Hospital healing (blocked near enemies)
- `render()` — full scene draw:
  - Tiles, roads, special zones
  - Police cars with siren lights
  - Cop foot officers with grade badges
  - Gangsters with danger aura
  - NPCs, pickups, player, bullets, particles
  - Night vignette, minimap with scanlines
- `loop(ts)` — requestAnimationFrame main loop

## Activation of Expansion Modules

In `index.html`, uncomment the desired module:

```html
<script src="island_a.js"></script>  <!-- Missions -->
<script src="island_b.js"></script>  <!-- Weather & Events -->
```

Then in `game.js` inside `startGame()` / `update(dt)` / `render()`:

```js
// island_a (Missions)
if (typeof initMissions  === 'function') initMissions();
if (typeof updateMissions=== 'function') updateMissions(dt);
if (typeof renderMissions=== 'function') renderMissions(CX, cam, W, H);

// island_b (Weather)
if (typeof initEvents    === 'function') initEvents();
if (typeof updateEvents  === 'function') updateEvents(dt);
if (typeof renderWeather === 'function') renderWeather(CX);
```

## Serving Locally

Because the files use ES modules-style split scripts, they **must be served
over HTTP** (not opened as `file://`). The quickest way:

```bash
cd streetcrime
python3 -m http.server 8080
# Open http://localhost:8080
```

Or use VS Code Live Server, or any static file server.

## Cop Grade Reference

| Grade | Badge | HP  | Speed | Accuracy | Car        | Min Wanted |
|-------|-------|-----|-------|----------|------------|------------|
| Patrol    | `P`    | 50  | 88  | Wide spread  | Blue cruiser | 0 |
| Sergeant  | `SGT`  | 80  | 95  | Medium       | Blue cruiser | 2 |
| Detective | `DET`  | 70  | 100 | Tight        | On foot only | — |
| SWAT      | `SWAT` | 130 | 82  | Precise+burst| Black SUV   | 4 |

## Weapon Reference

| Weapon  | DMG | Rate  | Ammo | Special         | Cost   |
|---------|-----|-------|------|-----------------|--------|
| Fists   | 22  | 0.44s | ∞    | Melee only      | Free   |
| Pistol  | 13  | 0.27s | 30   | —               | $200   |
| Shotgun | 28  | 0.62s | 15   | 5-bullet spread | $500   |
| Uzi     | 8   | 0.09s | 60   | Rapid fire      | $700   |
| Sniper  | 80  | 1.10s | 10   | Long range      | $1,200 |
| RPG     | 120 | 1.30s | 5    | Splash damage   | $2,500 |
