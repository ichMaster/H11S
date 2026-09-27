# Architecture — H11S

## Overview

H11S is a Godot 4 project written entirely in GDScript. It has three layers:

- **Simulation** — scrolling, terrain, mutations, enemies, weapons, the player and economy. Deterministic from a seed, and runs headless.
- **Presentation** — procedural meshes, materials, particles, lighting, HUD and audio. It reads from the simulation and never drives gameplay.
- **Meta** — the game-flow state machine, the shop, saving and high scores.

The **mutation director** ties them together. It is one system that schedules visible mutations for both the terrain and the creatures, from the current level's genome (VISION §Principles). Everything the player sees is generated in code by the **asset factory**, behind a seam that allows a glTF to replace any procedural model.

## Components

- **Game (autoload).** The top-level state machine: `boot → title → level → shop → level … → endless → game_over`. It owns the run state (`RunState`), the current level index and the seed. It is the only place that changes scenes.
- **Level.** One flight: builds the canyon from the level's `LevelDef`, starts the scroll, runs the mutation director and ends with the boss. It emits `level_completed` or `player_died`.
- **Scroller / camera.** Constant upward scroll at the level's speed. A top-down camera follows the scroll position, not the ship. **Reverse scroll:** while the ship is held against the bottom edge, scroll speed eases to a small negative value, down to a bounded distance back (§Scroll and reverse).
- **Terrain streamer.** Streams 20x24 heightfield **tiles** ahead of the camera and frees them behind it. Each tile has a render mesh and **wall-only colliders**. A tile generator guarantees a passable corridor at spawn time. Tiles expose an API for animated edits used by mutations and for wall destruction (§Terrain).
- **Mutation director.** Schedules and runs `MutationEvent`s from the level genome: small ones every few seconds and one big one per level. Each event has a **telegraph** phase followed by an **apply** phase of 1–2 s, and animates the terrain or a creature in real time. It never makes a hidden change (§Mutations).
- **Enemies.** An `EnemyType` defines several visible `Variant`s (speed, toughness, fire pattern, group behavior). Waves come from `WaveDef`s. A variant change or a split is a `MutationEvent` on the creature and plays as a growth animation, never as a model swap.
- **Bosses.** Large multi-part organisms fused into the terrain. A boss holds part of the canyon closed, and the player wins by destroying the parts that hold it and opening that passage. Parts can regrow through mutations. Each level's boss is defined as a mutation of the previous boss's definition (§Bosses).
- **Player.** Ship movement (`CharacterBody3D` on the gameplay plane), collision against walls, damage when a wall catches the ship, the shield, and the loadout (§Weapons and loadout).
- **Weapons.** Primary weapons (laser, spread, railgun, each with 3 levels), side weapons (side guns, rear shot), satellites that fire on their own, and bombs. Projectiles come from pools. Heavy weapons and bombs damage walls.
- **Economy and shop.** Credits drop from enemies and containers. The shop screen between levels buys and sells items from a catalog (`ShopItem`). On death, part of the loadout is lost by a fixed rule (§Death and penalties).
- **Asset factory.** Procedural generation of meshes (from primitives), textures and normal maps (from noise), materials, and particle and shader effects. Every visual is requested by id (`AssetFactory.mesh("drone_a")`). An override table maps an id to a glTF file, which lets you swap in a glTF with no change to logic (§Procedural assets).
- **FX and lighting.** HDR particle explosions with debris, glowing lasers, and dynamic lights from shots and explosions on hulls and walls. It reads a quality tier so the Pi can drop costly effects (§Performance).
- **Audio.** A music player whose intensity layer follows mutation activity, plus sound effects: technical for the ship and organic for enemies.
- **HUD.** Shield, credits, weapon levels, bombs, score and a boss health bar. It observes the simulation and has no logic of its own.
- **Persistence.** `RunState` for continuing a run, settings, and the high-score table, stored as JSON under `user://` (§Persistence).
- **Input.** An input map (`move_*`, `fire`, `bomb`, `pause`) bound to arrows/WASD, Space, a separate bomb key, the PocketTerm35 keyboard and a USB gamepad.

## Scroll and reverse

- The scroll position `y` advances at `LevelDef.scroll_speed`. Terrain, enemies and triggers are positioned in canyon coordinates, and the camera maps them to the screen.
- **Reverse:** when the ship touches the bottom edge and the player keeps pushing down, the scroll eases to `-reverse_speed`, down to at most `reverse_max` behind the furthest point reached. Enemy spawns are keyed to the furthest point, so reversing never re-triggers waves.
- If a wall catches up with the ship (the scroll pushes a wall into it), the ship takes damage over time and is pushed along. It does not die instantly.

## Terrain

- **Tiles.** 20x24 cells of heightfield. A cell's height determines whether it is floor (passable) or wall (blocking). The render mesh comes from the heightfield (a hard-surface, angular style) with a per-tile surface type (`metal | bone | crystal`) that selects the material.
- **Colliders.** Only wall cells get collision shapes, merged per tile, so walls are the only thing that blocks the player (§Player).
- **Guaranteed corridor.** The generator carves a continuous path from each tile's entry to its exit, with at least `LevelDef.min_corridor` cells of width, before placing walls from the genome's density and rhythm. A validator asserts the path exists (§Testing).
- **Animated edits.** `tile.animate_heights(cells, target, duration)` raises or lowers vertices over time and rebuilds the mesh region and colliders as it goes. Wall growth, corridor shifts and floor cracks are all built from this primitive.
- **Destruction.** Heavy weapons and bombs lower wall cells to floor through the same edit path, with debris FX.
- **Dead ends.** Mutations may close the corridor after spawn, so dead ends can occur. The ways out are destruction, a later mutation that opens a passage, or a reverse scroll. A small mutation must never close the corridor inside the telegraph-plus-reaction window right in front of the ship (§Mutations).

## Mutations

A single, cross-cutting mechanism for both world and creatures.

- **`MutationEvent`** has a `target` (tile region, enemy or boss part), a `kind`, a `telegraph` duration, an `apply` duration (1–2 s) and parameters. Its lifecycle is `scheduled → telegraph → apply → done`. Both phases are visible on screen.
- **Terrain kinds:** `wall_grow`, `corridor_narrow`, `corridor_shift`, `floor_crack` (a new structure rises), `surface_change` (metal/bone/crystal), `passage_open`.
- **Creature kinds:** `variant_grow` (e.g. a drone grows a second gun), `split` (one enemy becomes three smaller ones), `limb_regrow` (a boss part grows back).
- **Scheduling.** The director draws events from the level's **genome** and a seeded RNG. Small events come every `genome.small_interval` seconds, and one **big** event per level at a scripted point. In endless mode the limits are lifted, and interval, intensity and allowed kinds escalate with distance.
- **Fairness rule.** An event that blocks the ship's path must start its telegraph at least `reaction_window` (≥ 1 s) before its effect reaches the ship's row. The director rejects events that would violate this.
- **No hidden state.** Mutations change only what is on screen. There is no player modelling and nothing persists across sessions.
- **Intensity signal.** The director publishes a 0–1 `mutation_intensity`, which the music uses (§Audio).

## Bosses

- A boss is a `BossDef`: a set of parts (`BossPart{ id, mesh_id, hp, weapons, regrows }`) anchored into a terrain region that it holds closed.
- **Victory:** destroy the parts that hold the passage, which opens the canyon (`passage_open`). Parts marked `regrows` can come back through `limb_regrow` mutations during the fight.
- **Chain:** `BossDef[n]` is derived from `BossDef[n-1]` with a new limb or behavior added, so tactics partly carry over (VISION §Principles). Randomness in a fight is limited to the same visible mutations.

## Weapons and loadout

- **Primary:** `laser | spread | railgun`, level 1–3. Only one is active at a time.
- **Side:** `side_guns`, `rear_shot` (optional slots).
- **Satellites:** up to N autonomous companions with their own fire pattern.
- **Bombs:** a count. Each bomb deals area damage to enemies and walls.
- **Shield:** absorbs 3 hits. It is refilled in the shop or from rare containers.
- **Speed:** ship speed upgrades from the shop.

## Death and penalties

Death costs part of the loadout, not all of it. For example, the primary weapon drops one level, the satellites and one side slot are lost, and credits are kept. The exact rule is a single function `apply_death_penalty(RunState)`, pinned by a unit test so it can be tuned in one place.

## Contracts

These are the stable seams. Changing a contract must also change its test (§Testing).

- **Tile API:** `generate(seed, genome, entry_corridor) -> Tile`, `animate_heights(cells, target, duration)`, `destroy_cells(cells)`, and the invariant *spawned tile has a passable corridor ≥ min_corridor*.
- **Mutation API:** `MutationEvent{ kind, target, telegraph, duration, params }`, `MutationDirector.schedule(event) -> bool` (false when it violates the fairness rule), and the signal `mutation_started/finished(event)`.
- **Enemy variant:** `EnemyType{ id, variants[{ id, speed, hp, fire_pattern, group_behavior, mesh_id }] }`. A transition between variants is always a `variant_grow`/`split` event.
- **Asset factory:** `mesh(id) -> Mesh`, `material(id) -> Material`, `texture(id, size) -> Texture2D`. When the override table maps an id to a glTF, the glTF is returned instead, and callers are unchanged.
- **Save:** `SaveFile{ version, settings, run?, highscores[] }` as JSON. Unknown fields are ignored and missing ones get defaults.

## Data model

- `LevelDef{ index, biome, length_s (180–300), scroll_speed, min_corridor, genome, waves[], boss }` — one level. Authored as GDScript resources or JSON in the repo.
- `Genome{ corridor_width_range, wall_density, rhythm (open/tight pattern), surfaces[], small_interval, mutation_kinds[], enemy_variants[] }` — the level's visible mutation palette. Not per-player and not persisted.
- `WaveDef{ at_y, enemy_type, variant, count, formation }` — an enemy wave keyed to canyon position.
- `BossDef{ id, parent_id?, parts[], behavior }` — a boss, derived from `parent_id`.
- `RunState{ level, credits, score, loadout{ primary, primary_level, side[], satellites, bombs, shield, speed } }` — the current run.
- `ShopItem{ id, kind, price, sell_price, requires? }` — the shop catalog.
- `HighScore{ name, score, level_reached, mode: campaign|endless, ts }`.
- `Settings{ volume_music, volume_sfx, quality_tier, bindings? }`.

## Procedural assets

- Meshes are assembled from primitives (`BoxMesh`, `CylinderMesh`, `PrismMesh`, `SurfaceTool`/`ArrayMesh` for custom geometry) in factory scripts. The style is hard-surface: angular and technical. Organic feel comes from the shapes of terrain and bosses, not from textures.
- Textures and normal maps are baked from `FastNoiseLite` at startup (or on first use) and cached in memory.
- Palette: cold dark-blue metal and blackness below, with warm accents on engines, enemy lights and living terrain tissue.
- The `H11` mark is placed on panels and debris as a procedural decal.
- The **glTF override** is a table `asset_id → res://path.glb`, empty by default.

## Audio

- Music: an original late-80s-style electronic track in stems or layers. `mutation_intensity` crossfades the intensity layers.
- SFX: synthesized in code (`AudioStreamGenerator`) or taken from free-licensed libraries. The ship sounds technical and the enemies sound organic.

## Persistence

- A single JSON file under `user://save.json` with `SaveFile`, written atomically (write a temp file, then rename).
- Written at the shop (run checkpoint), on settings change, and on a new high score. A corrupted or missing file falls back to defaults and is logged. It never crashes the game.

## Performance

- Target: a stable **60 FPS at 640x480** on a Raspberry Pi 5. The render resolution is fixed at 640x480. On the development machine (MacBook M1) it is shown upscaled in a 1280x960 window.
- Renderer: **Compatibility** by default. If the effects need it, switching to **Mobile** is a one-line change in `project.godot` (`rendering/renderer/rendering_method`).
- Budgets: object pools for projectiles, enemies and particles, and a bounded number of live tiles and dynamic lights. A `quality_tier` setting scales particle counts, shadow quality, glow and light count.
- Tile mesh/collider rebuilds during animated edits are incremental (only the edited region) and spread across frames.

## Platform and controls

- Main client: PocketTerm35 (Raspberry Pi 5, 640x480, Wayland under Sway), built through an **export preset for Linux arm64**.
- Development: macOS on Apple Silicon in a 1280x960 window.
- Controls: arrows or WASD to move, Space to fire, a separate bomb key, pause. The device keyboard and a USB gamepad are both mapped through the input map.

## Stack and repository layout

Proposed layout. Create each directory when its version begins (ROADMAP).

```
project.godot          # Godot 4 project; renderer setting, 640x480 viewport, input map
/game                  # autoloads: Game state machine, Save, Audio bus; scene switching
/level                 # Level, scroller/camera, LevelDefs, WaveDefs
/terrain               # tile generator, heightfield mesh builder, wall colliders, animated edits
/mutation              # MutationDirector, MutationEvent kinds, genomes
/actors                # player ship, enemy types + variants, bosses
/weapons               # primary/side/satellites/bombs, projectile pools
/assets_gen            # AssetFactory: procedural meshes, noise textures, materials, glTF override table
/fx                    # particles, shaders, lighting
/audio                 # music layers, synthesized SFX
/ui                    # HUD, title, shop, game-over, high scores
/tests                 # headless smoke test + unit/contract tests
/specification         # concept, VISION.md, ARCHITECTURE.md, ROADMAP.md
export_presets.cfg     # Linux arm64 (PocketTerm35), macOS (dev)
```

## Testing

- **Headless smoke test:** `godot --headless` boots the game, runs a seeded level for N seconds with scripted input, and asserts there were no errors and the state flow worked (title → level → shop). It runs on every change.
- **Deterministic simulation:** gameplay uses a seeded RNG and a fixed timestep, so a seed plus an input script reproduces a run in tests.
- **Unit/contract tests** (run headless) pin these points: the tile corridor invariant over many seeds; the animated-edit and collider rebuild; the mutation fairness rule; variant transitions happening only through events; the death penalty rule; shop buy/sell; save/load round-trip and corrupted-file fallback; and the asset factory override.
- **Performance check on device:** a scripted benchmark flight on the PocketTerm35 records frame times per quality tier. Required before each version is called done from v3 onward.
