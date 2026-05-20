# ECABridge MCP — Proven Patterns for Spellrot

*Distilled from an 8-hour ECABridge-driven UE5.7 build session (2026-05-20)*
*Full gotcha catalog: `~/knowledge/intelligence/techniques/ecabridge-mcp-gotchas.md`*
*Canonical ECABridge reference: https://github.com/ibrews/ecabridge*

---

## Session-start checklist

Before any ECABridge session:
```bash
# 1. Confirm ECABridge is running
curl http://127.0.0.1:3000/health   # expect { bridge_ready: true }

# 2. Confirm Claude Code can see it
claude mcp list                      # should show unreal-ecabridge
```

After any ECABridge session, before closing the editor:
```python
# Always save — SCS changes don't survive restart without this
unreal.EditorAssetLibrary.save_dirty_assets(only_if_is_dirty=True, include_maps=True)
unreal.EditorLevelLibrary.save_current_level()
```

---

## Proven patterns for Spellrot systems

### Kill zone / Cleanse zone (overlap trigger)

**Collision profile must be `Trigger` (QueryOnly).** `OverlapAllDynamic` doesn't fire on a Pawn capsule because Pawn blocks WorldDynamic.

```
spawn_actor(class=BoxTriggerVolume, ...)
set_actor_property(CollisionProfileName="Trigger")
```

In the Blueprint: `OnActorBeginOverlap → ApplyDamage(player, 100, self)` for kill zones. For cleanse zones: `OnActorBeginOverlap → SetFloatPropertyByName("CorruptionLevel", 0)` on the wizard character.

### Zombie → Player damage

**Pawn capsule blocks Pawn — capsule overlap never fires between two Pawn actors.** Use distance check on Tick instead:

```
Event Tick
→ GetDistanceTo(GetPlayerPawn(0))
→ Branch: if < 150
→ ApplyDamage(player, 1.0, self, self, nil)
```

For the player's `AnyDamage` event to trigger corruption increase: enemy calls `ApplyDamage`, player BP has `ReceiveAnyDamage → increase CorruptionLevel float`.

### Cross-BP communication via damage relay

**Custom Events on a different BP aren't enumerable via MCP.** The reliable cross-BP pattern:

| Goal | Pattern |
|---|---|
| Trigger logic on another actor | `ApplyDamage(target, N, self)` → target has `ReceiveAnyDamage` event |
| Set a property on another actor | `SetIntPropertyByName(target, "JumpMaxCount", 2)` or `SetFloatPropertyByName` |
| Call a BP Function (not event) | BP Functions ARE enumerable; Custom Events are NOT |

### Wave spawner

Build in Blueprint, not via Spawn-on-Tick. Basic structure:
```
Event BeginPlay → Set timer by function "SpawnWave", 5.0s, looping=true

Function SpawnWave:
  → SpawnActorFromClass(BP_Zombie, RandomSpawnPoint.GetActorLocation)
  → Increment WaveCount
  → If WaveCount % 5 == 0: increase spawn count (escalation)
```

MCP can wire this via Blueprint graph commands. Use `execute_script` to chain the full wave spawner setup in one round trip.

### Corruption state machine

Simplest viable approach for the prototype (float, no enum):

```
// On character BP:
UPROPERTY(BlueprintReadWrite)
float CorruptionLevel = 0.0f;   // 0.0 = clean, 1.0 = infected, 2.0 = heavily infected

// Enemies call: SetFloatPropertyByName(wizard, "CorruptionLevel", level + 0.5)
// Kill purge: SetFloatPropertyByName(wizard, "CorruptionLevel", max(0, level - 0.1))
// Cleanse zone: SetFloatPropertyByName(wizard, "CorruptionLevel", 0)
```

Drive Niagara parameter and material scalar from the same float via Tick → SetNiagaraVariableFloat / SetScalarParameterValue.

### Niagara spell particles (create_niagara_system is broken)

`create_niagara_system` produces zero emission (100% repro in UE5.7). Always start from a duplicate:

```python
# Copy starter content Niagara asset as base
unreal.EditorAssetLibrary.duplicate_asset(
    source="/Game/StarterContent/Particles/P_Fire",
    destination="/Game/Spellrot/VFX/NS_Fireball_Clean"
)
# Then modify parameters via Python sandbox
unreal.EditorAssetLibrary.duplicate_asset(
    source="/Game/Spellrot/VFX/NS_Fireball_Clean",
    destination="/Game/Spellrot/VFX/NS_Fireball_Corrupted"
)
# Modify color on NS_Fireball_Corrupted to necrotic green/purple
```

Similarly: `set_niagara_module_input` fails on SpawnRate, color, size. Use `execute_script` + Python `FNiagaraSystemEditorLibrary` for parameter writes, or modify manually in the Niagara editor.

### AI movement (zombie chase)

**NavMeshBoundsVolume is NOT in the ThirdPerson template.** Must add before any AI can move:

```
Place > NavMeshBoundsVolume → scale to cover the arena → Build Navigation (P)
```

Then BasicAI chase on Tick:
```
Event Tick → SimpleMoveToActor(GetController(), GetPlayerPawn(0))
```

Set `MaxWalkSpeed=180` on AI character for zombie shuffle pacing.

### HUD without UMG (prototype-safe)

For wave count / score display in prototype, skip UMG widget binding entirely:

```
// Event Tick on GameMode or PlayerController:
PrintString(
    InString = "Wave: " + WaveCount,
    Key = "WaveDisplay",
    Duration = 0.05,
    TextColor = White
)
```

`Key` keeps it on one line — refreshes instead of stacking. Remove before vertical slice, replace with proper UMG then.

---

## CDO write failures — always use set_editor_property

CDO attribute snake_case writes silently no-op:

```python
# ❌ Silently fails
cdo.auto_possess_ai = unreal.AutoPossessAI.PLACED_IN_WORLD_OR_SPAWNED

# ✅ Reliable
cdo.set_editor_property('AutoPossessAI', unreal.AutoPossessAI.PLACED_IN_WORLD_OR_SPAWNED)
unreal.BlueprintEditorLibrary.compile_blueprint(bp)
unreal.EditorAssetLibrary.save_loaded_asset(bp)
```

**Always read back to verify:** `cdo.get_editor_property('AutoPossessAI')` — if it shows default, the write failed.

---

## SCS component defaults — set at BeginPlay, not Python

Writing struct properties (Vector, Rotator, AttachSocketName) to SCS component instances via Python **silently doesn't persist** — the SCS template isn't touched, only an in-memory CDO copy.

**Reliable**: set at runtime in BeginPlay. For the ponytail cable example:
```
BeginPlay → CableComponent.K2_AttachToComponent(Mesh, "head", SnapToTarget, KeepRelativeOffset)
```

For collision profile, scale, and simple scalar properties: `set_component_property` works fine. It's struct types that fail.

---

## Pin format quick reference

| Pin type | ❌ Wrong | ✅ Correct |
|---|---|---|
| FKey | `(KeyName="R")` | `R` |
| FRotator | `(Pitch=0,Yaw=-90,Roll=0)` | `0, -90, 0` |
| FVector | `(X=0,Y=0,Z=-90)` | `0, 0, -90` |

**Always read back** with `get_blueprint_node_pins` after setting — FKey silently defaults to `AnyKey` on wrong format.

---

## Diagnostic write-back

Note tags read back empty ~50% of the time in UE5.7. Use project Saved/ dir instead:

```python
import os
saved = unreal.SystemLibrary.get_project_saved_directory()
with open(os.path.join(saved, '_diag.txt'), 'w') as f:
    f.write("\n".join(diagnostic_lines))
# Read via: cat /Users/alex/ue/ThirdPersonClass/Saved/_diag.txt
```

---

## Ragdoll-and-recover pattern (if needed for wizard death)

```
ReceiveAnyDamage
→ Mesh.SetSimulatePhysics(true)
→ Mesh.SetCollisionProfileName("Ragdoll")
→ Delay(3.0)
→ Mesh.SetSimulatePhysics(false)
→ Mesh.SetCollisionProfileName("CharacterMesh")
→ Mesh.SetRelativeLocation(0, 0, -90)        // mannequin default
→ Mesh.SetRelativeRotation(0, -90, 0)        // mannequin default — BOTH resets required or character stands sideways
→ ExecuteConsoleCommand("RestartLevel")       // for death → respawn
```

Default mannequin mesh relative transform: `Location=(0,0,-90)`, `Rotation=(Pitch=0,Yaw=-90,Roll=0)`.
