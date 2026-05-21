# Spellrot — Overnight Session (2026-05-21) — MORNING TEST NEEDED

## Critical morning test (do this FIRST)

**Hit Play. Let ONE enemy walk into you. You should:**
1. Immediately fall over (ragdoll) on first hit
2. 3 seconds later → level restarts (second "BP_PC BeginPlay OK" in log)

**What this tells us:**
- ✅ Ragdoll + restart works → death chain is fine, issue was in the corruption math
  → Next: re-add proper accumulation logic via lisp_to_blueprint
- ❌ Still invincible → the OpenLevel/Delay chain itself is broken
  → Deeper engine issue, need different restart mechanism

## What was done overnight

### BP_PlatformingCharacter (SAVED)
- **Death**: AnyDamage → direct SetSimulatePhysics → Delay(3s) → OpenLevel
  (bypasses ALL corruption math — diagnostic to isolate root cause)
- **Trail activation**: BeginPlay now calls `Activate()` on Trail_L + Trail_R
  so Niagara emits continuously — color visible without jumping
- **Trail color**: `SetColorParameter("User.Color", ...)` — correct Niagara namespace
- **Clean graph**: all patched arithmetic/print nodes removed

### BP_KOTHZone (SAVED — cleanse zone)
- BeginOverlap cast: `BP_ThirdPersonCharacter → BP_PlatformingCharacter` ✅
- EndOverlap cast: same fix ✅
- Zone now activates when the correct player enters
- CorruptionLevel reset: not yet implemented (SetFloatPropertyByName not in Kismet)

### ECABridge (feature branch pushed)
- `add_blueprint_input_action_node` now emits `K2Node_EnhancedInputAction`
  when a UInputAction asset is found — fixes IA_Fire fireball binding

## After the morning test

### If death works (most likely path):
1. Use `lisp_to_blueprint` to rebuild proper corruption accumulation:
```
(event AnyDamage
  (set CorruptionLevel (+ (get CorruptionLevel) 0.25))
  (branch (>= (get CorruptionLevel) 1.0)
    :true (seq
      (call (component Mesh) SetSimulatePhysics true)
      (delay 2.0)
      (OpenLevel self "/Game/ThirdPerson/Lvl_ThirdPerson"))
    :false nil))
```
2. Compile + save
3. Restore BP_Enemy purge logic (it already exists — just verify it works)
4. Run PIE full-loop test: corruption trail + death + cleanse + fireball

### If still invincible (fallback):
- Try `ExecuteConsoleCommand "RestartLevel"` instead of OpenLevel
- Try checking if BP_PlatformingCharacter actually receives the event
  (the ragdoll Mesh component may not be set up on this variant)

## Remaining TODOs
- [ ] PIE full-loop verification (corruption system end-to-end)
- [ ] Test cleanse zone CorruptionLevel reset (currently missing the reset logic)
- [ ] Fireball test (LMB → LineTrace → enemy ragdoll + corruption purge)
- [ ] Doc screenshots of working gameplay

## Environment
- UE 5.7 running
- ECABridge: 566 commands at 127.0.0.1:3000

## Session-start checklist
```bash
pgrep -fl "UnrealEditor" | head -1
curl -s http://127.0.0.1:3000/health | python3 -c "import sys,json; d=json.load(sys.stdin); print(f'ECA: {d[\"commands\"]} cmds, ready={d[\"bridge_ready\"]}')"
```

## Do NOT rules
- ❌ Do NOT use execute_python with set_editor_property('NodePosX', x) — crashes UE
- ❌ Do NOT add K2Node_Comment via Python — use add_blueprint_comment_node
- ❌ Do NOT change .uproject EngineAssociation — intentionally on UE 5.7
- ❌ Do NOT open .uproject via Finder — LaunchServices may route to UE 5.8
- ❌ Do NOT use add_blueprint_function_node for KismetMath promoted operators
  (float+float, float>=float) — use lisp_to_blueprint instead
- ❌ Do NOT rely on lisp_to_blueprint for events that already exist
  in the BP — it creates duplicates (use delete first, then lisp)

## ECA param names
- add_input_mapping: `context_path` (not imc_path)
- dump_input_mapping_context: `context_path` (not imc_path)
- lisp_to_blueprint: `code` (not lisp_code)

## Repos
- Studio: https://github.com/ibrews/spellrot
- UE project: https://github.com/ibrews/spellrot-ue
- ECABridge: https://github.com/ibrews/ECABridge
