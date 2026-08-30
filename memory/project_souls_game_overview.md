---
name: project-souls-game-overview
description: Architecture and known issues of the AGT_TEMPLATE souls-like game project, from first hands-on review
metadata:
  type: project
---

Custom C++17/OpenGL engine (premake5-generated VS2022 solution, `AGT_TEMPLATE/engine.sln`) built for the City University "Advanced Games Tech" module — not Unity/Unreal. Engine code in `AGT_TEMPLATE/engine/src/engine` (renderer, entities, core, events), game logic in `AGT_TEMPLATE/game/src` (Player, Enemy, Boss, Priest, HolyProjectile, SoulFragment, example_layer). Uses GLFW/Glad, Assimp, Bullet physics, FMOD audio, Freetype, ImGui, spdlog, glm. The user (Abhijit) graduated and wants to raise this to industry/portfolio standard for their CV.

To run it: build via `AGT_TEMPLATE/engine.sln` (VS2022 present at `C:\Program Files\Microsoft Visual Studio\2022`), or use the already-built `AGT_TEMPLATE/bin/Debug-windows-x86_64/game/game.exe` — **must be launched with working directory `AGT_TEMPLATE/game`** (asset paths are relative, e.g. `assets/audio/...`). Menu controls: W/S to move selection, Space to confirm (not Enter). Full controls shown on the in-game splash: WASD move, Shift dash, R potion, Space attack, E upgrade menu, P spawn enemy.

**Why this matters:** any future session resuming this improvement work should not re-derive the architecture or re-discover the run steps from scratch.

**How to apply:** see [[project-souls-game-findings]] for the specific issues found and the improvement roadmap discussed with the user.
