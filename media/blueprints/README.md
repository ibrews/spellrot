# Spellrot Blueprints — Reference & Demo Assets

Companion screenshots + markdown for each Blueprint in the Spellrot UE prototype.
These are designed to be slide-ready for the Unreal Fest Chicago talk
*"Build a Working UE5 Prototype in 50 Minutes with Claude Code + ECABridge"*.

Each Blueprint has a paired `.md` + `.png`:
- `.md` describes what the BP does, why each node is there, and the ECABridge
  gotchas that came up while wiring it
- `.png` is a screenshot of the EventGraph as authored

## Currently documented

| # | Blueprint | Status | Notes |
|---|---|---|---|
| 01 | [BP_KillZone](01-bp-killzone-cleanse.md) | ✅ Authored, screenshotted | Cleanse zone — overlap → cast → SetCorruptionLevel(0) |
| 02 | [BP_WaveSpawner](02-bp-wavespawner.md) | ✅ Authored, screenshotted | Timer-driven BP_Enemy spawner (3 instances placed in level) |

## Planned (not yet done)

- **03 — BP_Fireball** (duplicate of XRFramework `BP_Projectile` with `ApplyDamage` overlap)
- **04 — BP_PlatformingCharacter (CorruptionLevel chain)** — corruption variable + AnyDamage handler + death
- **05 — BP_Enemy (kill-purge + chase)** — enemy AnyDamage drops player corruption, Tick chase + touch damage
- **06 — Niagara trail color drive** (corruption visualization on Trail_L / Trail_R)

## ECABridge MCP — what worked + what to avoid

**Reliable patterns we proved this session:**

- `batch_edit_blueprint_nodes` with `variable_class` param for cross-BP variable
  setters — closes ECABridge issue #6, shipped as commit `1c395fa`
- `add_blueprint_function_node` for static GameplayStatics functions like
  `BeginDeferredActorSpawnFromClass` + `FinishSpawningActor` — works when you can't
  use the K2 native `Spawn Actor From Class` node
- `spawn_blueprint_actor` for placing BP instances in the level
- `save_asset` + `save_level` to persist changes
- `compile_blueprint` to verify the BP compiles clean before moving on

**Patterns to avoid:**

- ❌ **`set_editor_property('NodePosX', x)` via execute_python**: crashed UE5.7
  with no error trace. Use `auto_layout_blueprint_graph` instead and zoom out
  in the editor for screenshots.
- ❌ **`unreal.EdGraphNode_Comment`**: not exposed via Python — UE Python
  doesn't surface K2-graph special nodes. Use external markdown for
  documentation; you cannot add comment boxes inside the BP graph from MCP.
- ❌ **`SpawnActorFromClass` as a function name**: that's the BP-native K2
  special node, not a static. ECABridge can't add it. Use the BeginDeferred/
  FinishSpawning pair.

## Screenshot workflow

The Unreal Editor doesn't expose a BP-editor-screenshot tool. To capture each
EventGraph cleanly:

1. `open_blueprint_editor(path)` — opens the BP in editor
2. `auto_layout_blueprint_graph(path, strategy="horizontal", spacing_x=500, spacing_y=400)`
3. macOS: `osascript -e 'tell application "System Events" to set frontmost of process "UnrealEditor" to true'`
4. computer-use: click the **EventGraph** tab
5. computer-use: click into graph area, then `cmd+a` (select all), `f` (focus selected)
6. `screencapture -x media/blueprints/NN-bp-name.png`

Step 5 is the trick — without it the camera lands wherever the editor last left
off, often clipping the BeginPlay header.

## Why this matters for Unreal Fest

The talk thesis: an MCP server like ECABridge + a coding assistant like Claude
Code lets you build a working UE5 prototype in under an hour, with the AI
authoring Blueprints, placing actors, wiring damage chains, and explaining
its work as it goes. These artifacts are the proof.
