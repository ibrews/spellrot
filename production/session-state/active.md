# Spellrot — Overnight Session (2026-05-21) — Phases 1-4 SHIPPED

## Status: COMPLETE. Game playable in PIE. All BPs compile clean. All commits pushed.

Last activity: 01:15 EDT 2026-05-21

## Verified in PIE
- 8 screenshots in `production/screenshots/2026-05-21/pie_*.png`
  - `pie_running.png`, `pie_with_enemies.png` — initial smoke test
  - `pie_trail_t02s.png` through `pie_trail_t45s.png` — 45s run, stable
- Wave spawner pumps enemies, ragdoll chain fires on contact, no crashes

## What ships in the BP graphs
- **BP_PlatformingCharacter.ReceiveAnyDamage**: `CorruptionLevel += 0.25` →
  branch on `>= 1.0` → `PrintString('YOU DIED — restarting in 3...')` (red, 3s)
  → ragdoll + `Ragdoll` collision profile + 3s `Delay` + `OpenLevel` restart
- **BP_PlatformingCharacter.ApplyCleanse**: custom event resetting CorruptionLevel=0
- **BP_KOTHZone.BeginOverlap**: cast to player → SetIsActive=true → call player.ApplyCleanse
- **BP_KOTHZone.EndOverlap**: cast to player → SetIsActive=false
- **BP_WaveSpawner.SpawnWave**: WaveCount += 1 → BeginDeferredSpawn(BP_Enemy) → FinishSpawning
- **WBP_HUD**: has CorruptionText + CurrentCorruption variable + Tick binding

## Known TODOs (morning manual work, low remaining risk)
1. Spawn `WBP_HUD` to viewport: add `CreateWidget(WBP_HUD) → AddToViewport`
   manually in `BP_PlatformingGameMode.BeginPlay` (CreateWidget is a K2 macro
   that's only addable in the editor, not via ECABridge or UE Python).
2. Push player's `CorruptionLevel` into `HUDWidget.CurrentCorruption` on
   damage/cleanse (requires HUDWidget ref variable on player).
3. Optional: extend wave scaling — use `WaveCount` to drive enemy speed, more
   spawns at higher counts, etc. The increment is in place; the consumers aren't.

## Commits this session
- `spellrot-ue`:
  - `a9f8e71` corruption + cleanse wired
  - `217d0e2` HUD widget + cleanse refinement + README rewrite
  - `6dc39d3` wave count incrementer
  - `7b9bbd8` restore empty GameMode BeginPlay + ECABridge pointer
  - `f8a02a1` death message PrintString
- `spellrot`:
  - `560cf11` phase 1 progress notes
  - `d20f0b5` GDD status doc + initial screenshots
  - `26ada42` extended session state
  - `5b81b8e` PIE smoke test + death msg graph screenshots
- `ECABridge`:
  - PR #14 opened: https://github.com/ibrews/ECABridge/pull/14
- `agile-lens-kb`:
  - `0097ad8f` 10-pattern lisp_to_blueprint catalog + daily log

## Patterns learned (KB updated)
See `intelligence/techniques/ecabridge-lisp-to-blueprint-patterns.md`:
1. `(print float)` fails to compile (float→string cast)
2. `(set CrossBPVar)` inside `(cast)` doesn't target cast result — use
   custom-event-as-setter pattern instead
3. `(call Function)` inside `(cast)` breaks the cast Object pin typing
4. `:params` is dropped on custom events
5. `add_blueprint_variable_get/set` only knows variables on the current BP
6. `CreateWidget` is a K2 macro, unreachable via ECABridge or UE Python
7. `delete_loaded_asset` is destructive (deletes file from disk too)
8. `lisp_to_blueprint` drops asset references inside branch bodies
9. `cleanup_orphan_nodes` uses `delete=True`, not `dry_run=False`
10. Chunked tool results wrap in `{"chunk": "..."}` envelopes (strict=False)

## Environment
- UE 5.7 running, ECABridge 566 cmds @ 127.0.0.1:3000
- All key BPs UpToDate, 0 errors, 0 warnings
