# H11S — Game Concept

Version 0.2, 27.09.2026 (0.1 — 13.09.2026). Author: Vitalii Bondarenko. Working prototype: megablast-prototype (Godot 4).

*English translation of [H11S-concept.md](H11S-concept.md). The Ukrainian original is authoritative.*

## 1. In One Paragraph

H11S is a vertical shooter in the spirit of Xenon 2: Megablast for the handheld PocketTerm35 (Raspberry Pi 5, 640x480). The player steers a ship through a canyon that keeps growing toward them, and has to do more than destroy enemies: they must also fit through terrain that physically blocks their movement. The main difference from the classic is that the world and the enemies follow Hypothesis 11: one and the same algorithm mutates both the environment and the creatures in it. As a result, every pass through the canyon and every wave of enemies is a little different, and the game itself adapts over time to the way it is played.

## 2. Connection to the H11 World

H11 is a corridor shooter set in dark underground decks, where the mutation algorithm first appeared. H11A is a peaceful valley above those decks, where mutations have become part of everyday life. H11S shows a third angle: a view from above onto the organic-mechanical layers that have grown through the whole structure. The canyon the player flies through is not scenery. It is the living organism of the H11 complex, which keeps on mutating. The H11 mark appears from time to time on panels and terrain debris, as in the other games of the series.

The main principle of mutations: they happen in real time, on screen, before the player's eyes, and each one forces the player to do something immediately. A mutation is not a hidden change of parameters between levels. It is a visible event. A wall grows right in front of the ship and closes the passage. The corridor narrows or shifts to the side. The floor cracks and releases a new structure. A drone grows a second gun mid-flight or splits into three smaller ones. A boss grows a new limb to replace one that was shot off. The player sees it grow and has a second or two to react. The mutation system does nothing more complex than this: no hidden genes, no player-style statistics, and no long evolution across sessions.

The environment mutates like this: corridor width, wall density, the rhythm of open and tight sections, and the surface type (metal, bone, crystal) change right during the flight, with small changes every few seconds and one large one per level. Creatures mutate like this: each enemy type has several visible variants (speed, toughness, firing style, group behavior), and the transition between them happens on screen as a growth animation, not as a model swap.

## 3. Game Loop

One level is a 3–5 minute flight through the canyon with a boss at the end. During the flight the player collects credits from enemies and containers. Between levels a shop opens, like Crispin in Xenon 2: here the player can buy weapons, shields, speed and autofire, and sell what they don't need. The game has five levels, each with its own mutation biome and boss. After that an endless mode unlocks, in which the algorithm mutates without limits and the result goes to a high-score table.

## 4. Key Mechanics

The scroll constantly moves upward, but, as in the original, the player can push the ship against the bottom edge to scroll the screen back a little, to pick up a missed upgrade or get around a wall. Terrain walls physically block the ship. If a wall catches up with the ship, the ship takes damage rather than dying instantly. Dead ends do happen, and getting out of them is part of the game: walls can be destroyed with heavy weapons, and mutations sometimes open a passage on their own.

Weapons are divided into primary weapons (laser, spread, railgun) with three levels each, side weapons (side guns, rear shot), and satellites that fly alongside the ship and fire on their own. Bombs explode over an area and destroy walls. The shield absorbs three hits. Energy is restored in the shop or from rare containers. Death means losing part of the upgrades, not the whole arsenal, so that one bad run is not punished too harshly.

Bosses are large organisms fused with the terrain: the player can't simply shoot them down, but has to find which part of the canyon they are holding and open it. The boss of each level is a mutation of the previous one, so tactics partly carry over, but each time there is a new limb or behavior.

## 5. Visual Style

The goal is the highest possible graphics quality, not pixel retro. The whole game is true 3D with a top-down view: the terrain is a heightfield, ships and enemies are volumetric meshes with metal, normal maps and shadows, explosions are HDR particles with debris, and lasers have glow. Lighting is dynamic: shots and explosions light up hulls and walls. The palette is cold, with dark blue metal and blackness below, and warm accents on engines, enemy lights and the living tissue of the terrain. The surface style is hard-surface, angular and technical. Organic feel comes from the shapes of the terrain and bosses, not from textures.

All assets are generated inside Godot: meshes are assembled from primitives in scripts, textures and normal maps come from noise, and effects come from particles and shaders. Blender and external packages are not used, but any glTF can be substituted for a procedural model without changes to the logic.

## 6. Sound

The music is an original track in the style of late-80s electronica with a live beat whose intensity changes as mutations grow. Weapon and explosion sounds are synthesized or taken from free libraries. Enemies have organic sounds that set them apart from the ship's technical sounds.

## 7. Platform and Controls

The main client is the PocketTerm35: Pi 5, 640x480, Wayland under Sway. Development is on a MacBook M1 in a 1280x960 window that shows the 640x480 render. Controls: the device keyboard, a USB gamepad, arrows or WASD, fire on Space, and bomb on a separate key. The target is a stable 60 frames per second. If the Compatibility renderer can't handle the effects, switching to Mobile takes one line in the configuration.

## 8. Technical Decisions

The engine is Godot 4 with GDScript, as in H11 and H11A. The terrain streams in 20x24 tiles with a guaranteed corridor. Only walls have colliders, so they are the only thing that blocks the player. Mutations are implemented as animated real-time changes to the heightfield and meshes (a growing wall is a tile that raises its vertices over a second or two, with its colliders updated). Player state and results are saved locally in JSON. There is a headless smoke test and an export preset for Linux arm64.

## 9. Scope and Stages

Stage 0 is done: a prototype with a procedural ship, terrain, enemies, three-level weapons, explosions, a HUD and restart. Stage 1 is the game loop: shop, credits, bombs, wall destruction, reverse scroll, saving. Stage 2 is mutations: the genome of the terrain and enemies, inheritance between waves, a big jump per level. Stage 3 is five levels, bosses, music and sound. Stage 4 is endless mode, the high-score table, and performance polish on the Pi.

## 10. Decisions on Open Questions

Mutations are only what is visible on screen in real time and forces the player to react. A code module shared with H11A is not a requirement; the connection to the series is conceptual. H11S stays a pure arcade game with no story cutscenes, for a single player. Boss randomness is limited to the same visible mutations: a boss grows or loses parts before the player's eyes, and it is this, not hidden parameters, that makes every fight different.

## Changes in Version 0.2

Section 2: mutations were redefined as visible real-time events that force a reaction; hidden genes and adaptation to player style were removed. Section 8: the requirement for a mutation module shared with H11A was removed. Section 10: open questions were replaced with decisions.
