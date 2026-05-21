# Spellrot Implementation Status

*Last updated: 2026-05-21 (overnight session)*

This is the running state of what's actually built versus what's still in the design.
For the full design vision, see `game-concept.md`.

## Implemented (works in PIE today)

### Corruption accumulation
- `BP_PlatformingCharacter.ReceiveAnyDamage`: `CorruptionLevel += 0.25` per hit
- Branch when `CorruptionLevel >= 1.0`: enable ragdoll physics on `Mesh`, switch to `Ragdoll` collision profile, 3 second delay, then `OpenLevel("/Game/ThirdPerson/Lvl_ThirdPerson")` to restart
- `CorruptionLevel` is a `real` member variable on the player

### Cleanse zone reset
- `BP_KOTHZone.BeginOverlap`: casts overlapping actor to `BP_PlatformingCharacter`, sets KOTHZone's `IsActive=true` (gates score accumulation), then calls `ApplyCleanse` on the player
- `BP_PlatformingCharacter.ApplyCleanse`: custom event that sets `CorruptionLevel = 0.0`
- `BP_KOTHZone.EndOverlap`: clears `IsActive`

### Fireball (single-target hitscan)
- `BP_PlatformingCharacter` `IA_Fire.Started` → `LineTraceByChannel` from player view
- Branch on hit returns true → `Cast To BP_Enemy` → `ApplyDamage(100)`
- `BP_Enemy.ReceiveAnyDamage`: casts to player, subtracts 0.1 from `CorruptionLevel`, calls `RagdollAndRecover` (2s ragdoll then restore physics + `CharacterMesh` collision)

### Wave spawning
- Three `BP_WaveSpawner` actors in `Lvl_ThirdPerson`
- Each spawner BeginPlay sets a timer calling `SpawnWave` every `SpawnInterval` seconds
- `SpawnWave` spawns `BP_Enemy` at spawner transform via `BeginDeferredActorSpawnFromClass` + `FinishSpawningActor`
- Enemies use `SimpleMoveToActor(GetPlayerPawn(0))` on Tick to chase the player

### Trail visual feedback
- `BP_PlatformingCharacter` has `Trail_L`/`Trail_R` Niagara components, activated in `BeginPlay`
- `Tick` writes `User.Color` parameter to both trails, lerping from base color toward magenta as `CorruptionLevel` rises

### HUD widget (partial)
- `WBP_HUD` widget exists with `ScoreText` (static) and `CorruptionText` (dynamic)
- `CurrentCorruption` float variable on widget; Tick formats it into `CorruptionText.Text`
- **Not yet wired**: spawn-to-viewport on game start, push player's `CorruptionLevel` into widget's `CurrentCorruption`

## In Progress / Known TODOs

| Item | Reason it's not done | Approach |
| --- | --- | --- |
| Spawn `WBP_HUD` on game start | `CreateWidget` is a K2 macro not addable via ECABridge's `add_blueprint_function_node` | Add manually in `BP_PlatformingGameMode.BeginPlay`: `CreateWidget(WBP_HUD)` → `AddToViewport`, store in `HUDWidget` variable |
| Live HUD update | Needs reference from player to active HUD instance | Player `BeginPlay`: cache HUD ref; on `AnyDamage`/`ApplyCleanse`: write to `HUDWidget.CurrentCorruption` |
| Wave scaling | `lisp_to_blueprint` repeatedly drops the actor class on `BeginDeferredActorSpawnFromClass` when used in branches | Surgical edit: insert `Get WaveCount` → `+1` → `Set WaveCount` between SpawnWave event and existing spawn chain |
| Enemy color tint variants | Needs visual playtesting | Add `EnemyType` enum on `BP_Enemy`, drive material parameter at spawn |
| Score on death | Game-over screen UMG | Add kills/time counters on game mode, display in death widget |
| Death message print | `print float` in lisp fails with float→string cast | Use `(Conv_DoubleToString)` wrapper or skip print entirely |

## Architecture Decisions Validated This Session

- **Cross-BP variable modification via custom event call** is the only reliable pattern when using `lisp_to_blueprint`. Inside a `(cast TargetClass Object ...)` body, bare variable sets resolve against the casting BP, not the cast target. The workaround is to expose a setter as a custom event on the target BP, then call it via `add_blueprint_function_node`.
- **`PrintString` debug nodes are too brittle** to script via `lisp_to_blueprint` — string concatenation with float values causes compile errors. Visual feedback through Niagara parameters works much better and is what the design called for anyway.

## Repository state

- `/Users/alex/ue/ThirdPersonClass/` — UE 5.7 project ([ibrews/spellrot-ue](https://github.com/ibrews/spellrot-ue))
- `/Users/alex/Claude-Code-Game-Studios/` — studio + GDDs ([ibrews/spellrot](https://github.com/ibrews/spellrot))
- ECABridge feature branch `feature/enhanced-input-action-node` pushed but PR not yet open ([ibrews/ECABridge](https://github.com/ibrews/ECABridge))
