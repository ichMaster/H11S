# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project state

H11S is a vertical shooter in the spirit of Xenon 2: Megablast, the third game in the H11 world (after H11 and H11A). Right now the repository holds only the design concept, `specification/H11S-concept.md`, and no Godot project. The Stage 0 prototype mentioned in the spec (`megablast-prototype`) lives outside this repo. There are no build, lint or test commands yet. When a Godot project is added, update this file with the real commands (headless smoke test, Linux arm64 export).

Documentation is written in Ukrainian. Keep new spec text in Ukrainian to match. The spec is versioned: bump the version and date in its header, and add a "Зміни у версії X" section when you change it. An English translation lives in `specification/H11S-concept-en.md`; the Ukrainian original is authoritative, so update the translation whenever the original changes.

`.gitignore` is the stock Python template and has no Godot entries (`.godot/`, `*.import`, export outputs). Add them when you create the Godot project.

## Hard constraints from the spec

- **Engine:** Godot 4 with GDScript, the same as H11 and H11A. No other languages.
- **Target:** PocketTerm35 (Raspberry Pi 5, Wayland/Sway) at 640x480 and a stable 60 FPS. Development happens on a MacBook M1 in a 1280x960 window that shows a 640x480 render. Start with the Compatibility renderer; moving to Mobile should take one config line.
- **All assets are procedural and generated inside Godot.** Meshes are built from primitives in scripts, textures and normal maps come from noise, and effects are particles and shaders. No Blender and no external asset packs. Keep a seam so any procedural model can be swapped for a glTF without logic changes.
- **Visuals:** real 3D with a top-down view, not pixel art. The terrain is a heightfield. Ships use metal materials, normal maps and shadows. Lighting is dynamic (shots and explosions light up hulls and walls), with HDR particles and glowing lasers. The palette is cold (dark blue metal, black), with warm accents on engines, enemy lights and living tissue.
- **Input:** arrows or WASD, fire on Space, a separate bomb key, and USB gamepad support.
- **Persistence:** player state and high scores are saved locally as JSON.
- Single-player arcade with no story cutscenes. There is no shared code module with H11A; the link between the games is conceptual only.

## Core architecture (as specified)

- **Terrain streaming:** tiles of 20x24 with a guaranteed corridor. Only walls have colliders, so walls are the only thing that blocks the player. If a wall catches the ship, it deals damage rather than instant death. Heavy weapons and bombs can destroy walls.
- **Scrolling:** constant upward scroll, but holding the ship at the bottom edge scrolls back slightly.
- **Mutations (Hypothesis 11) are the defining system.** A mutation is always a *visible, real-time on-screen event* that forces the player to react within 1–2 seconds. Examples: a wall grows in front of the ship, the corridor narrows or shifts, a drone sprouts a second gun or splits into three, a boss regrows a limb. There are no hidden genes, no tracking of player style, and no long-term evolution across sessions. Do not add hidden stat changes between levels.
  - Environment: corridor width, wall density, open/tight rhythm and surface type (metal, bone, crystal) change mid-flight. Small changes happen every few seconds, plus one big change per level.
  - Implementation: animated edits to the heightfield and meshes. For example, a growing wall is a tile raising its vertices over 1–2 s while its colliders update.
  - Enemies: each type has several visible variants (speed, toughness, fire pattern, group behavior). A switch between variants plays as a growth animation, never as a model swap.
- **Game loop:** five levels of 3–5 minutes each, every one with its own mutation biome and a boss. Each boss is a mutation of the previous one and is fused into the terrain, so the player has to open the part of the canyon it holds. A shop between levels (like Xenon 2's Crispin) buys and sells gear for credits. After level 5 comes an endless mode with a high-score table.
- **Weapons:** primary weapons (laser, spread, railgun) have 3 levels each. There are also side weapons (side guns, rear shot), autonomous satellites, and area bombs that destroy walls. The shield absorbs 3 hits. Death costs part of the upgrades, not all of them.

## Roadmap

Stage 0 (prototype) is done. Stage 1 covers the shop, credits, bombs, wall destruction, reverse scroll and saving. Stage 2 covers mutations. Stage 3 covers the five levels, bosses and audio. Stage 4 covers endless mode, high scores and Pi performance work.
