# Spellrot — Overnight Session (2026-05-21) — IN PROGRESS

## Phase 1 progress
- ✅ BP_PlatformingCharacter: corruption gating built via lisp_to_blueprint
  - Event AnyDamage: CorruptionLevel += 0.25 → branch >= 1.0 → ragdoll + delay + OpenLevel
  - Print debug removed (was causing float→string cast error)
  - ApplyCleanse custom event added: sets CorruptionLevel = 0.0
- ✅ BP_KOTHZone: BeginOverlap now triggers cleanse
  - On overlap with BP_PlatformingCharacter: Set IsActive=true → Call ApplyCleanse on player
  - End overlap: Set IsActive=false
- ⏳ Fireball end-to-end PIE test pending
- ⏳ HUD widget pending

## Key technique learned
- `lisp_to_blueprint` `(set CorruptionLevel ...)` inside `(cast ... )` does NOT properly
  resolve to the cast target. Workaround: add a custom event on the target BP that does
  the set, then call that event from the casting BP via `add_blueprint_function_node`
  with `target_class` pointing to the target BP's _C class.
- `(call SomeFunc)` inside `(cast ...)` breaks cast typing (Object pin becomes wildcard).
  Use surgical add_blueprint_function_node + connect_blueprint_nodes for cross-BP calls.

## Screenshots
- production/screenshots/2026-05-21/bp_pc_anydamage_graph.png — corruption death chain
- production/screenshots/2026-05-21/bp_kothzone_overlap_graph.png — cleanse zone wiring

## Environment
- UE 5.7 running, ECABridge 566 cmds @ 127.0.0.1:3000
- BP_PlatformingCharacter SAVED clean compile
- BP_KOTHZone SAVED clean compile
