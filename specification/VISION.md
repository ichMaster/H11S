# Vision — H11S

## In one sentence

H11S is a vertical 3D shooter in the spirit of Xenon 2: Megablast for the PocketTerm35 (Raspberry Pi 5, 640x480). The player flies through a living canyon whose walls and creatures mutate on screen in real time, and every mutation demands an immediate reaction.

## What we are building

A pure arcade game in five levels plus an endless mode. The ship flies up a canyon that keeps scrolling toward it. The player has to destroy enemies and also fit through terrain that physically blocks movement. The canyon is the living organism of the H11 complex: walls grow in front of the ship, the corridor narrows or shifts, drones grow a second gun or split into three, and bosses regrow the limbs you shoot off. Between levels a shop (like Xenon 2's Crispin) sells weapons, shields, speed and satellites for credits collected in flight. The goal is high-quality real 3D graphics, not pixel retro, and every asset is generated in code inside Godot.

## For whom

The owner of a PocketTerm35 who wants a short, intense arcade session on the device's own keyboard or a USB gamepad. The game is for a single player, with no story cutscenes and no online features. A local high-score table is the only competitive element.

## Principles

- **Mutation is a visible event, not a hidden parameter.** Every mutation happens on screen, before the player's eyes, and forces a reaction within 1–2 seconds. There are no hidden genes, no statistics about the player's style, and no long evolution across sessions. If the player can't see it grow, it isn't a mutation.
- **One algorithm for world and creatures.** The same mutation system (Hypothesis 11) drives the terrain and the enemies, so every pass through the canyon and every wave is a little different.
- **Terrain is gameplay.** Walls physically block the ship. Dead ends exist, and getting out of them (heavy weapons, bombs, a mutation that opens a passage, a short reverse scroll) is part of play.
- **Punish mistakes, but not harshly.** A wall that catches the ship deals damage, not instant death. Death costs part of the upgrades, not the whole arsenal.
- **Everything is generated in code.** Meshes come from primitives, textures and normal maps from noise, and effects from particles and shaders. No Blender and no external asset packs. A glTF can replace any procedural model without changes to game logic.
- **Quality graphics on a small device.** Real 3D with a top-down view, dynamic lighting, HDR particles and glow, at a stable 60 FPS on a Raspberry Pi 5. Performance is a feature, not a late polish item.
- **Simplicity first.** Complexity is added one version at a time (ROADMAP), never all at once.

## Non-goals

- No story, cutscenes or dialogue. H11S is a pure arcade game.
- No multiplayer and no online services (leaderboards, accounts, telemetry).
- No hidden adaptation to the player: no player modelling, no difficulty tuned from statistics, no persistent evolution of the genome between sessions.
- No pixel-art or 2D sprite pipeline, and no hand-made assets from external tools.
- No code module shared with H11 or H11A. The link to the series is conceptual only.

## Connection to the H11 world

H11 is a corridor shooter in dark underground decks, where the mutation algorithm first appeared. H11A is a peaceful valley above those decks, where mutations became part of everyday life. H11S shows a third angle: a view from above onto the organic-mechanical layers that have grown through the whole structure. The H11 mark appears from time to time on panels and terrain debris, as in the other games of the series.

## Glossary

- **Canyon (тіснина)** — the level itself: a vertically scrolling heightfield corridor that the ship flies through.
- **Tile** — one 20x24 chunk of canyon terrain. Tiles stream in ahead of the ship and are freed behind it. Every tile guarantees a passable corridor at spawn time.
- **Wall** — raised terrain with a collider. Walls are the only terrain that blocks the ship.
- **Mutation** — a visible, animated real-time change to the terrain or a creature, lasting 1–2 s, that forces the player to react. Small mutations happen every few seconds, and there is one big mutation per level.
- **Telegraph** — the visible build-up that precedes a mutation's effect, so the player has time to react (e.g. a wall starting to rise, a drone swelling before it splits).
- **Biome** — a level's mutation palette: which surface types (metal, bone, crystal) and which mutations occur, plus their rhythm.
- **Genome** — the per-level set of mutation parameters (corridor width range, wall density, open/tight rhythm, allowed enemy variants). It is visible in its effects, is not tied to the player, and does not persist between sessions.
- **Variant** — one visible form of an enemy type (speed, toughness, fire pattern, group behavior). Enemies move between variants on screen through a growth animation, never through a model swap.
- **Reverse scroll** — pushing the ship against the bottom edge to scroll the canyon back a little, e.g. to collect a missed upgrade or get around a wall.
- **Shop** — the between-levels screen where the player buys and sells weapons, shields, speed, satellites and energy for credits.
- **Credits** — the currency dropped by enemies and containers during a flight.
- **Primary weapon** — laser, spread or railgun, each with three levels.
- **Side weapon** — side guns or a rear shot.
- **Satellite** — an autonomous companion that flies next to the ship and fires on its own.
- **Bomb** — an area explosion that also destroys walls.
- **Shield** — absorbs three hits. Its energy is restored in the shop or from rare containers.
- **Boss** — a large organism fused with the terrain. To beat it, the player has to find which part of the canyon it holds and open it. Each boss is a mutation of the previous one.
- **Endless mode** — unlocked after level 5: the algorithm mutates without limits, and the result goes to the high-score table.
- **PocketTerm35** — the target handheld: Raspberry Pi 5, 640x480, Wayland under Sway.
