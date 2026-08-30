---
name: project-souls-game-findings
description: Concrete bugs and improvement targets found by running/reading the souls-like game, plus the agreed improvement priority
metadata:
  type: project
---

From launching the built game and reading engine/game source (2026-08-30):

**Critical engine bug:** `AGT_TEMPLATE/engine/src/engine/core/application.cpp` `application::run()` computes `timestep` from raw unclamped frame time (`gameLoopTimer.elapsed()`), no max-dt clamp or fixed timestep/accumulator. Confirmed in practice: after the window lost real OS focus for a few seconds (during automated testing), the next frame's huge dt caused the player to go from full HP to 0 near-instantly against the boss — classic spiral-of-death / simulation-instability bug. This affects physics, combat timing, and anything frame-rate or stall dependent. High priority fix: clamp dt (e.g. cap at ~1/15s) or move to a fixed-timestep accumulator.

**Rendering is dated for a portfolio piece:** `game/assets/shaders/mesh.glsl` is labelled "PBR" in a comment but is actually classic ambient/diffuse/specular Blinn-Phong (struct `Material{ambient,diffuse,specular,shininess}`), max 2 point lights + 2 spot lights + 1 directional light, has fog support. No shadow mapping, no post-processing pipeline (no bloom/tonemapping/SSAO/framebuffer effects found anywhere in `engine/src/engine/renderer`). This is the single highest-leverage area for making the game look "industry standard" in screenshots/video.

**Architecture:** `game/src/example_layer.cpp` is a 1100+ line god-class doing scene setup, spawning, HUD text, input, and game-state (MainMenu/InGame/PauseMenu) all in one file/class. Player/Enemy/Boss/Priest each have their own small FSM-based logic classes (~150-300 lines) which is a reasonable pattern already — the fix is extracting scene/spawn/HUD/menu responsibilities out of example_layer rather than rewriting combat logic.

**Gameplay/level design observed:** game currently drops the player into a boss fight ("BOSS: DEVIL") essentially immediately with no explored intro/level pacing before it; environment had a giant grass card clipping right through the camera view (grass billboard/quad scale or placement bug); HUD is plain coloured debug-style text (`HP:`, `Stamina:`, `Potions:`, `Soul Fragments:`) with an ASCII bracket boss health bar — functional but not stylized. Main menu is plain black background with yellow/white text list, no visual polish, no video/logo.

**Repo hygiene is already fine:** build outputs (`bin/`, `inter/`) are correctly gitignored (`AGT_TEMPLATE/.gitignore`), not committed. No action needed there.

**Improvement roadmap discussed, by category:**
1. Engineering: fix dt clamping (critical correctness bug), break up example_layer god-class, consider ECS-lite or at least a SceneManager/UIManager/SpawnManager split.
2. Graphics: shadow mapping (single directional shadow map first), post-processing framebuffer pass (bloom + tonemapping at minimum), more dynamic lights, fix grass clipping, improve material setup (normal maps if assets allow).
3. Gameplay/level design: add pacing before the boss (exploration, weaker enemies first, checkpoints), death/game-over screen and respawn flow (currently unclear if one exists — HP hit 0 with no observed death UI), lock-on targeting (common souls-like expectation), i-frames tuning on roll.
4. Polish/UX: restyle HUD (health/stamina bars instead of raw numbers), stylized main menu, screen-space damage/hit feedback, sound feedback confirmation.

**How to apply:** when resuming this work, check with the user which category to tackle first rather than assuming — this was presented as a menu of options, not a committed plan. See [[project-souls-game-overview]] for how to build/run.

**Progress (2026-08-30):** User picked order: (1) timestep fix, (2) architecture cleanup, then (3) gameplay/level design pacing. Both (1) and (2) are done:
- Timestep fix: `application.cpp::run()` now clamps dt to a max of 1/15s before constructing the `timestep` (see `max_timestep_seconds`).
- Architecture cleanup: extracted two new classes out of `example_layer.cpp` (was 1108 lines): `game/src/Hud.h/.cpp` (all 2D text rendering — main menu, in-game stat HUD, floating enemy/priest health bars now deduplicated into one `render_floating_health_bar` helper, boss bar, pause/upgrade menu) and `game/src/SpawnManager.h/.cpp` (enemy/priest spawning, per-frame AI update + player-hit detection + respawn + soul-drop logic, and their rendering). example_layer is now a thinner orchestrator. Deliberately did NOT extract the static world/environment setup (skybox/terrain/trees/grass/castle/halberd) — it's entangled with a pre-existing bug (see below) and lower value than the HUD/spawn duplication that was removed.
- Found but did NOT fix (flagged for the user, not in scope of this pass): `m_physics_manager->physical_objects.at(3)` in the constructor (damping applied assuming index 3 is the halberd rigid body) is almost certainly wrong given the actual push order into `m_game_objects` (player, 5 warriors, 2 priests, boss, terrain, halberd, ...) — index 3 is actually a warrior, not the halberd. Pre-existing bug, left untouched to avoid silently changing behavior during an "architecture only" pass.
- Build verified via a full Rebuild of the `game` target (`MSBuild engine.sln -t:game:Rebuild`) after manually adding the two new files to `game/game.vcxproj` (do NOT regenerate via `premake5.exe vs2022` — the checked-in vcxproj has a hand-adjusted assimp library path that doesn't match the current `premake5.lua`'s `LibDir["assimp"]` formula; regenerating breaks the assimp link. Add new files to the vcxproj by hand instead).
- Not yet playtested end-to-end by a human; the user was actively in Visual Studio during this session and offered to playtest themselves since automated keyboard input into the GLFW window from outside (SendKeys/keybd_event) is unreliable/laggy for precise testing.
