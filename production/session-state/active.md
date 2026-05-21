# Spellrot session — 2026-05-20 late-evening (HANDOFF — PIE READY)

## TL;DR for next-session Claude

UE 5.7 running. ECABridge HTTP up at `127.0.0.1:3000`, 566 commands.
ALL features complete. Both BPs compile 0 errors / 0 warnings.
PIE test READY for all 7 behaviors including fireball (Left Mouse Button).
**Next step: hit Play and run through the checklist below.**

## ECABridge fix committed

Branch: `feature/enhanced-input-action-node` (pushed to GitHub)
PR: https://github.com/ibrews/ECABridge/pull/new/feature/enhanced-input-action-node

`add_blueprint_input_action_node` now emits `K2Node_EnhancedInputAction`
when a matching `UInputAction` asset exists in the registry; falls back to
legacy `K2Node_InputAction` for projects not using Enhanced Input.

Changes: `ECABridge.Build.cs` (+InputBlueprintNodes dep) + `ECABlueprintNodeCommands.cpp`
(new `CreateInputActionNode()` helper used by both standalone command and batch_edit).

## What was done this session

1. **ECABridge PR #7** — already merged (nothing to do)
2. **IA_Fire asset created** at `/Game/Input/Actions/IA_Fire` (Digital, with Pressed/Released triggers)
   - Also exists at `/Game/Variant_Platforming/Input/Actions/IA_Fire` (has a ref, keep for now)
3. **IMC_Platforming** — IA_Fire → LeftMouseButton mapping added (at standard path)
   - Note: IMC has TWO LMB→IA_Fire mappings now (standard + variant path) — harmless duplicate
4. **Fireball chain wired**: K2Node_InputAction(Pressed) → LineTrace → Branch → Cast → ApplyDamage(25)
   - ⚠️ The event node is legacy `K2Node_InputAction`, NOT `K2Node_EnhancedInputAction`
   - Fireball **will NOT fire at PIE** until manual fix below

## PIE test checklist (ALL 7 items ready)

Hit Play in editor and verify:
- [ ] Corruption +0.25 per enemy touch (BP_Enemy DamageSphere → ApplyDamage on player)
- [ ] Kill enemy → AnyDamage handler → SetCorruptionLevel -= amount → ragdoll
- [ ] CorruptionLevel ≥ 1.0 → Death + RestartLevel
- [ ] Cleanse zone (BP_KOTHZone?) resets CorruptionLevel to 0
- [ ] Trail color lerps cyan→green as CorruptionLevel increases (Event Tick path)
- [ ] 3 wave spawners (BP_WaveSpawner) producing enemies every ~3s
- [ ] Left Mouse Button → Fireball line trace 2000u → ApplyDamage(25) on BP_Enemy

## Current BP compile status

| Blueprint | Status | Notes |
|---|---|---|
| BP_Enemy | ✅ Clean (0/0) | Cast fix intact |
| BP_PlatformingCharacter | ✅ Clean (0/0) | K2Node_EnhancedInputAction for IA_Fire, Started→LineTrace wired |

## Asset status

| Asset | Path | Status |
|---|---|---|
| IA_Fire (standard) | `/Game/Input/Actions/IA_Fire` | ✅ Created, Digital, triggers set |
| IA_Fire (variant) | `/Game/Variant_Platforming/Input/Actions/IA_Fire` | ⚠️ Keep (has ref, 1 IMC ref) |
| IMC_Platforming | `/Game/Variant_Platforming/Input/IMC_Platforming` | ✅ 16 mappings, LMB→IA_Fire added |

## Session-start checklist

```bash
# 1. Verify UE editor still running
pgrep -fl "UnrealEditor\.app"

# 2. Verify ECABridge HTTP up
curl -s http://127.0.0.1:3000/health

# 3. If UE is NOT running, launch via UE 5.7 (NOT via Finder):
open -a "/Users/Shared/Epic Games/UE_5.7/Engine/Binaries/Mac/UnrealEditor.app" \
  --args /Users/alex/ue/ThirdPersonClass/ThirdPersonClass.uproject
```

## ECABridge MCP gotcha after restart

After a computer restart, the MCP tools (`mcp__unreal-ecabridge__*`) are dropped
from the Claude Code session because the server wasn't running at startup.
**Workaround**: call ECABridge directly via HTTP JSON-RPC:
```python
import json, urllib.request
def eca(tool, args=None, timeout=60):
    payload = json.dumps({"jsonrpc":"2.0","id":1,"method":"tools/call",
        "params":{"name":tool,"arguments":args or {}}}).encode()
    req = urllib.request.Request("http://127.0.0.1:3000/mcp", data=payload,
        headers={"Content-Type":"application/json"})
    with urllib.request.urlopen(req, timeout=timeout) as r:
        resp = json.loads(r.read())
    content = resp.get("result",{}).get("content",[])
    return json.loads(content[0]["text"]) if content else {}
```
After server is up, restart the Claude Code session to get MCP tools back.

## Do NOT rules

- ❌ Do NOT use execute_python with set_editor_property('NodePosX', x) — crashes UE
- ❌ Do NOT try to add K2Node_Comment via Python — use add_blueprint_comment_node
- ❌ Do NOT change .uproject EngineAssociation — intentionally on UE 5.7
- ❌ Do NOT open .uproject via Finder — LaunchServices may route to UE 5.8
- ❌ Do NOT use `add_blueprint_input_action_node` expecting Enhanced Input nodes —
  ECA tool always creates legacy `K2Node_InputAction`. Manual editor addition required
  for `K2Node_EnhancedInputAction` (Enhanced Input Actions > IA_Fire).

## ECA parameter names (hard-learned)

- `add_input_mapping`: uses `context_path` (NOT `imc_path`), `action_path`, `key`
- `dump_input_mapping_context`: uses `context_path` (NOT `imc_path`)

## Repo references

- Studio: https://github.com/ibrews/spellrot
- UE project: https://github.com/ibrews/spellrot-ue
- ECABridge: https://github.com/ibrews/ECABridge
- ECABridge PR #7: https://github.com/ibrews/ECABridge/pull/7 (MERGED)
