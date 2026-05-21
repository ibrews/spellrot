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

The Unreal Editor doesn't expose a BP-editor-screenshot tool. The reliable
slide-ready capture flow:

### 1. Author the layout first

```jsonc
// (a) clean up node positions — auto OR manual
auto_layout_blueprint_graph({blueprint_path, strategy: "horizontal", spacing_x: 500, spacing_y: 400})
// OR for hand-controlled layout (since 2026-05-20, ECABridge PR #7):
set_blueprint_node_position({blueprint_path, positions: [{node_id, x, y}, ...]})

// (b) optionally add labelled comment regions (since 2026-05-20, ECABridge PR #7)
add_blueprint_comment_node({
  blueprint_path,
  comment: "Setup — Register timer on BeginPlay",
  wrap_node_ids: ["<guid1>", "<guid2>", ...],
  color: "#3B82F6",
  margin: 60
})
```

### 2. Compile + save

```jsonc
compile_blueprint({blueprint_path})  // logs "Compile successful! [in NN ms]" — nice slide indicator
save_asset({asset_path: blueprint_path})
```

### 3. Open the editor and select EventGraph

```jsonc
open_blueprint_editor({blueprint_path})  // may land on Viewport tab for fresh BPs
```

Then use **computer-use** to click the EventGraph tab (read the screenshot to find its coords).

### 4. Frame all nodes

Computer-use: click into the graph canvas to give it focus, then `cmd+a` (select all) followed by `f` (frame selected).

Without step 4 the camera lands wherever the editor last left off — often clipping the BeginPlay header.

### 5. Capture only the UE window region

```bash
# Get UE window bounds (avoids capturing Claude's TUI overlay, Finder, etc.)
BOUNDS=$(osascript -e 'tell application "System Events" to tell process "UnrealEditor" \
  to get {position, size} of front window' | tr -d ',' | awk '{print $1","$2","$3","$4}')

# Region-capture
/usr/sbin/screencapture -x -R "$BOUNDS" media/blueprints/NN-bp-name.png
```

### Gotchas that ate ~30 min the first time

- **`screencapture` is unfiltered** — captures every window on screen, including
  Claude Code's TUI sitting on top of UE. Two reliable workarounds:
  1. Hide Claude before capturing: `osascript -e 'tell application "System Events" \
     to set visible of process "Claude" to false'` → screencapture → restore visibility
  2. Use `screencapture -R "x,y,w,h"` with bounds from osascript (above). Works
     unless Claude/Finder is actively overlapping the UE region.
- **computer-use's `screenshot` tool DOES filter by allowlist** (only granted apps
  visible) but its output isn't written to your media folder — handy for verification
  inside the agent loop, not for the canonical slide screenshot.
- **`take_camera_screenshot` / `take_gameplay_screenshot` MCP tools** target the
  world viewport, NOT the Slate windows of asset editors. There's no generic
  "screenshot the BP editor" MCP tool yet (see
  `Plugins/ECABridge/docs/ecabridge-improvement-roadmap-2026-05-20.md` § 8 for
  the cross-pollination idea from AIK).
- **Don't reposition nodes via `unreal.set_editor_property('NodePosX', x)`** —
  crashes UE 5.7/5.8 hard. Use `set_blueprint_node_position` instead.

## Why this matters for Unreal Fest

The talk thesis: an MCP server like ECABridge + a coding assistant like Claude
Code lets you build a working UE5 prototype in under an hour, with the AI
authoring Blueprints, placing actors, wiring damage chains, and explaining
its work as it goes. These artifacts are the proof.
