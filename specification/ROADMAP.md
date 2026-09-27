# Roadmap — H11S

Five self-contained versions, built in order, one for each stage of the concept (§9):

- **v0** — foundation (bring the Stage 0 prototype into this repo)
- **v1** — game loop (shop, credits, bombs, wall destruction, reverse scroll, saving)
- **v2** — mutations (the visible real-time mutation system for terrain and enemies)
- **v3** — content (five levels, bosses, music, sound)
- **v4** — endless mode, high scores, and performance polish on the Pi

Versions are numbered from 0. Phases inside a version are numbered `vA.B` (A = version, B = phase), e.g. `v2.3`. Each phase lists a **Goal**, a short description, a **Tasks** list, and a **Definition of Done (DoD)**. Each ships with the automated tests that encode its DoD (see [ARCHITECTURE.md](ARCHITECTURE.md) §Testing).

The game grows along two axes. Gameplay grows from flight and shooting (v0), to a full run economy (v1), to a living canyon (v2), to a complete campaign (v3), to an endless mode (v4). The target grows from the dev machine (v0), to running on the PocketTerm35 (v0.1 export), to a device-verified 60 FPS (v4). The deterministic, headless-testable simulation is built first. Complexity is added one version at a time, never all at once.

**Versioning (`A.B.C`).** `A` is the roadmap version, `B` is the phase within it, and `C` is a post-release fix on that phase. Roadmap phase `vA.B` becomes version `A.B.0`, and a fix after it bumps `C`. Never bump the version without explicit confirmation.

---

## v0 — Foundation: project, ship, canyon, enemies, weapons

Bring the Stage 0 prototype (`megablast-prototype`) into this repository as a clean project structured along ARCHITECTURE's components. Covered here: the procedural ship, streaming terrain with wall-only colliders, enemies, primary weapons with three levels, explosions, the HUD and restart. Also set up the tooling that every later phase depends on: the headless smoke test, the Linux arm64 export, and the 640x480 render setup. Depends on: nothing. This is the foundation.

### v0.1 — Project skeleton, render setup, export

**Goal:** an empty Godot 4 project that renders at 640x480, runs headless in tests, and exports to the PocketTerm35.

**Tasks:**
- Create `project.godot`: a 640x480 viewport and a 1280x960 window on the dev machine (integer upscale), the Compatibility renderer, and the input map (`move_*`, `fire`, `bomb`, `pause`) with keyboard and gamepad bindings.
- The `Game` autoload with a minimal state machine (`boot → title → level → game_over`).
- A seeded RNG service and a fixed-timestep simulation tick.
- A headless smoke test runner (`godot --headless`) that boots and exits cleanly.
- An export preset for Linux arm64. Add Godot entries to `.gitignore`.

**DoD:** the project opens on the dev machine at 640x480 in a 1280x960 window. The smoke test passes headless. An arm64 build launches on the PocketTerm35 under Sway.

**Tests:** smoke — headless boot and state flow; unit — seeded RNG determinism.

### v0.2 — Asset factory and the player ship

**Goal:** a procedural ship that the player can fly within the screen bounds.

**Tasks:**
- `AssetFactory` with `mesh/material/texture(id)`, noise-based textures and normal maps, and an empty glTF override table.
- A procedural hard-surface ship mesh with metal material, engine glow and shadows.
- Player movement on the gameplay plane, clamped to the screen, with a top-down camera.

**DoD:** the ship is generated entirely in code and flies smoothly with keyboard and gamepad. A glTF placed in the override table replaces it without code changes.

**Tests:** unit — factory override returns the glTF when mapped; smoke — the ship spawns and moves under scripted input.

### v0.3 — Streaming canyon terrain

**Goal:** an endless scrolling canyon made of heightfield tiles, with walls that physically block the ship.

**Tasks:**
- The scroller and camera with constant upward scroll.
- A 20x24 tile generator that carves a guaranteed corridor, then places walls. Heightfield mesh builder and per-tile surface material.
- Wall-only colliders. The ship collides with walls; a wall that catches the ship deals damage and pushes it (no instant death).
- Tile streaming ahead and freeing behind, with a bounded number of live tiles.

**DoD:** the ship flies an endless canyon, walls block it, and every spawned tile is passable.

**Tests:** unit — the corridor invariant over thousands of seeds, colliders only on wall cells; smoke — a 60 s seeded flight without errors.

### v0.4 — Enemies and primary weapons

**Goal:** enemies to shoot, with three primary weapons at three levels each.

**Tasks:**
- `EnemyType` with a single variant for each of a few basic types (e.g. drone, turret), and `WaveDef` spawns keyed to canyon position.
- Primary weapons: laser, spread and railgun, levels 1–3, with pooled projectiles.
- Enemy fire, and player damage from projectiles.

**DoD:** waves spawn by position, all three weapons work at every level, and enemies and the player take and deal damage.

**Tests:** unit — waves are keyed to position (no double spawn), weapon level stats; smoke — a scripted fight.

### v0.5 — Explosions, lighting, HUD, restart

**Goal:** the prototype's feel. HDR explosions, dynamic light, a HUD and a quick restart.

**Tasks:**
- HDR particle explosions with debris, glowing lasers, and dynamic lights from shots and explosions.
- A `quality_tier` setting that scales the effects.
- A HUD (shield, score, weapon) and game over with restart.

**DoD:** v0 matches or exceeds the Stage 0 prototype on the dev machine and runs on the PocketTerm35. Frame rate on the device is measured and recorded.

**Tests:** smoke — death → game over → restart; unit — HUD reads state without logic of its own.

---

## v1 — Game loop: credits, shop, bombs, destruction, reverse, saving

Turn a flight into a run. Credits, a shield with energy, bombs that destroy walls, a reverse scroll, side weapons and satellites, a shop between levels, a death penalty, and JSON saving. Depends on: v0.

### v1.1 — Credits, containers, shield

**Goal:** a run economy and survivability.

**Tasks:**
- Credits dropped by enemies and containers, and pickups.
- A shield that absorbs 3 hits. Energy containers (rare).
- `RunState` holds credits, score and the loadout.

**DoD:** credits and energy can be collected, and the shield absorbs exactly 3 hits.

**Tests:** unit — shield hit counting, credit accounting.

### v1.2 — Bombs and destructible walls

**Goal:** a way out of dead ends.

**Tasks:**
- Bombs (a limited count, bound to the bomb key): area damage to enemies and walls.
- Heavy weapons (e.g. railgun level 3) damage walls.
- `destroy_cells` on tiles: walls sink to floor through the animated-edit path, with collider rebuild and debris.

**DoD:** a bomb clears enemies and opens a passage through walls. Destroyed cells stop blocking the ship.

**Tests:** unit — destroy_cells updates colliders, bomb radius; smoke — a bomb through a scripted wall.

### v1.3 — Reverse scroll

**Goal:** the player can scroll back a little, as in the original.

**Tasks:**
- Holding the ship against the bottom edge eases the scroll to `-reverse_speed`, up to `reverse_max` behind the furthest point.
- Spawns are keyed to the furthest point reached (no re-triggered waves).

**DoD:** the player can go back to pick up a missed item or get around a wall. Waves never repeat.

**Tests:** unit — reverse bounds, spawn keying.

### v1.4 — Side weapons and satellites

**Goal:** the rest of the arsenal.

**Tasks:**
- Side guns and a rear shot as loadout slots.
- Satellites that fly next to the ship and fire on their own.
- Speed upgrades.

**DoD:** all loadout items can be equipped (via debug for now) and work together.

**Tests:** unit — loadout composition; smoke — a fight with the full loadout.

### v1.5 — Levels, the shop, and the death penalty

**Goal:** the between-levels loop.

**Tasks:**
- `LevelDef` with a length (3–5 min) and an end trigger (a placeholder boss until v3).
- The shop screen: buy and sell from the `ShopItem` catalog, with shield energy refills.
- `apply_death_penalty(RunState)`: part of the loadout is lost, not all of it.
- The state flow `level → shop → next level`.

**DoD:** you can play level → shop → level, spend and sell credits, and die with only a partial loss of the loadout.

**Tests:** unit — shop buy/sell rules, the death penalty rule; smoke — level → shop → level.

### v1.6 — Saving

**Goal:** progress and settings survive a restart.

**Tasks:**
- `SaveFile` JSON under `user://`, written atomically. A run checkpoint at the shop, plus settings.
- Fallback to defaults on a corrupted or missing file.

**DoD:** quitting at the shop and relaunching resumes the run, and a corrupted save never crashes the game.

**Tests:** unit — save/load round-trip, corrupted-file fallback, unknown/missing fields.

---

## v2 — Mutations: the living canyon

The defining system. Every mutation is a visible event in real time that forces the player to react within 1–2 s. It is driven by the level's genome. There are no hidden parameters, no player modelling, and nothing persists between sessions (VISION §Principles). Depends on: v1 (tiles with animated edits, enemies, levels).

### v2.1 — Mutation director and the event contract

**Goal:** one scheduler for all mutations, with telegraph and fairness built in.

**Tasks:**
- `MutationEvent` (`kind, target, telegraph, duration, params`) and its lifecycle `scheduled → telegraph → apply → done`.
- `MutationDirector` driven by `Genome` and the seeded RNG: small events every `small_interval`.
- The fairness rule: reject any blocking event whose telegraph can't start at least `reaction_window` before its effect reaches the ship.
- The `mutation_intensity` signal.

**DoD:** the director schedules and runs placeholder events on a seed, and unfair events are rejected.

**Tests:** unit — the lifecycle, determinism from a seed, the fairness rule; contract — the `MutationEvent` shape.

### v2.2 — Terrain mutations

**Goal:** the canyon changes shape before the player's eyes.

**Tasks:**
- `wall_grow` (a wall rises in front of the ship over 1–2 s), `corridor_narrow`, `corridor_shift`, `floor_crack` (a new structure rises), `passage_open`.
- Incremental mesh and collider rebuilds during an edit, spread across frames.
- Readable telegraphs: cracks, glow, a slow start of the rise.

**DoD:** each terrain kind plays visibly in flight, colliders follow the animation, and the player always has the reaction window.

**Tests:** unit — each kind's edit result and colliders; smoke — a seeded flight with a high mutation rate.

### v2.3 — Surface mutations and biome rhythm

**Goal:** the canyon's material and rhythm change mid-flight.

**Tasks:**
- `surface_change` between metal, bone and crystal, animated across a region.
- The genome's corridor width range, wall density and open/tight rhythm applied to newly generated tiles, with small drifts every few seconds.

**DoD:** a single flight visibly passes through changes in surface, width and rhythm.

**Tests:** unit — genome parameters reach tile generation.

### v2.4 — Enemy variants and growth animation

**Goal:** creatures mutate on screen.

**Tasks:**
- Several visible `Variant`s per enemy type (speed, toughness, fire pattern, group behavior).
- `variant_grow`: the mesh grows (e.g. a second gun) as an animation, and stats switch when it completes.
- `split`: one enemy breaks into three smaller ones.
- Waves can inherit a variant from the previous wave (a visible lineage within the level only).

**DoD:** a drone grows a second gun or splits into three mid-flight. There is never a hard model swap.

**Tests:** contract — variant changes only through events; unit — split spawns and inheritance within a level.

### v2.5 — The big mutation of the level

**Goal:** one large, memorable mutation per level.

**Tasks:**
- A scripted big event per `LevelDef` (e.g. the whole canyon shifts, a new structure takes over the corridor).
- The music intensity hook (placeholder until v3.4).

**DoD:** each level contains a big mutation that noticeably reshapes play and stays fair.

**Tests:** smoke — a level with its big mutation; unit — the fairness rule still holds.

---

## v3 — Content: five levels, bosses, music, sound

The full campaign. Depends on: v2.

### v3.1 — Five levels and biomes

**Goal:** five distinct levels, each with its own mutation biome.

**Tasks:**
- Five `LevelDef`s with their own genome, surfaces, waves and big mutation.
- Difficulty curve across the five levels.
- The `H11` mark as a procedural decal on panels and debris.

**DoD:** all five levels are playable end to end with the shop in between.

**Tests:** smoke — each level runs headless on a seed.

### v3.2 — Boss framework

**Goal:** bosses fused into the terrain.

**Tasks:**
- `BossDef` with parts anchored into the terrain region they hold closed.
- Victory by destroying the holding parts, which triggers `passage_open`.
- `limb_regrow` mutations on regrowable parts during the fight. A boss health bar in the HUD.

**DoD:** the level 1 boss can be beaten by finding and opening what it holds, and its limbs visibly regrow.

**Tests:** unit — the win condition, regrowth only through events.

### v3.3 — The boss chain

**Goal:** each boss is a mutation of the previous one.

**Tasks:**
- `BossDef[n]` derived from `BossDef[n-1]` with a new limb or behavior.
- Bosses for levels 2–5.

**DoD:** five bosses, each recognizably grown from the previous one, with tactics that partly carry over.

**Tests:** unit — derivation keeps the parent's parts and adds new ones.

### v3.4 — Music

**Goal:** an original late-80s-style electronic track that reacts to mutations.

**Tasks:**
- The track in intensity layers or stems.
- A crossfade driven by `mutation_intensity`.

**DoD:** the music audibly intensifies as mutations grow.

**Tests:** unit — the intensity → layer mapping.

### v3.5 — Sound effects

**Goal:** a distinct audio identity for the ship and the enemies.

**Tasks:**
- Synthesized or free-licensed weapon, explosion and pickup sounds.
- Organic enemy sounds and technical ship sounds. Mutation telegraph sounds.

**DoD:** every gameplay event has a sound, and mutations can be heard before they land.

**Tests:** smoke — no missing-stream errors across a full run.

---

## v4 — Endless mode, high scores, performance

Depends on: v3.

### v4.1 — Endless mode

**Goal:** after level 5, the algorithm mutates without limits.

**Tasks:**
- Endless mode unlocks after the campaign.
- Escalation with distance: shorter intervals, more intense and more kinds of mutations, and harder variants. The fairness rule still applies.

**DoD:** endless mode keeps escalating and stays fair.

**Tests:** unit — the escalation curve, fairness at high intensity.

### v4.2 — High-score table

**Goal:** a local table of records.

**Tasks:**
- `HighScore` entries for campaign and endless mode, name entry, and a table screen.
- Stored in `SaveFile`.

**DoD:** scores persist across launches and are shown sorted.

**Tests:** unit — insertion, ordering, caps.

### v4.3 — Performance on the Pi

**Goal:** a stable 60 FPS on the PocketTerm35.

**Tasks:**
- A scripted benchmark flight that records frame times on the device.
- Tune pools, the live tile count, light and particle budgets, and the quality tiers.
- If Compatibility can't carry the effects, switch to the Mobile renderer (one line) and re-verify.

**DoD:** the benchmark holds 60 FPS on the PocketTerm35 at the default quality tier.

**Tests:** device benchmark — frame-time report per tier.

### v4.4 — Release polish

**Goal:** a finished, shippable game on the device.

**Tasks:**
- A title screen, pause, settings (volume, quality, bindings).
- Final gamepad and PocketTerm35 keyboard checks. The release build from the arm64 preset.

**DoD:** a clean arm64 release plays the full campaign and endless mode on the PocketTerm35.

**Tests:** smoke — the full state flow; device — a manual playthrough.

---

## Contract mapping

- Seeded RNG + fixed timestep, headless smoke test, arm64 export — **v0.1**.
- `AssetFactory` + glTF override — **v0.2**.
- Tile API (generate + corridor invariant, wall-only colliders) — **v0.3**; `destroy_cells` — **v1.2**; `animate_heights` used by mutations — **v2.2**.
- `EnemyType`/`WaveDef` — **v0.4**; visible `Variant`s + `variant_grow`/`split` — **v2.4**.
- `RunState` + loadout — **v1.1**, completed in **v1.4**; `apply_death_penalty` + `ShopItem` — **v1.5**.
- `SaveFile` JSON — **v1.6**; `HighScore` — **v4.2**.
- `MutationEvent` + `MutationDirector` + fairness rule + `mutation_intensity` — **v2.1**.
- `Genome` in tile generation — **v2.3**; big mutation per level — **v2.5**.
- `LevelDef` — **v1.5**; five levels — **v3.1**.
- `BossDef` + `limb_regrow` + `passage_open` win — **v3.2**; boss chain — **v3.3**.
- Music intensity layers — **v3.4**.
- Renderer switch and quality tiers verified on device — **v4.3**.

## Deferred

Beyond v0–v4: new game modes, online leaderboards, story content, multiplayer, and platforms other than the PocketTerm35 and the dev machine.
