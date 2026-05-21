# BP_PlatformingCharacter — Fireball (Line Trace Spell)

**Date:** 2026-05-20 (evening session)
**Blueprint:** `/Game/Variant_Platforming/Blueprints/BP_PlatformingCharacter`

## What this does

Left Mouse button fires an instant-cast line trace spell 2000 units forward from
the player. If it hits a `BP_Enemy`, it calls `ApplyDamage(25)` which triggers
the enemy's `Event AnyDamage` handler — which purges some player corruption and
ragdolls the enemy.

## Prerequisites — one manual step required

`IA_Fire` is referenced by the InputAction node but has no key binding yet.
Open `Content/Input/IMC_Default.uasset` or `IMC_Platforming.uasset` in the
editor and add:

```
IA_Fire → Left Mouse Button
```

(right-click on Mappings → Add → select IA_Fire → set key to Left Mouse Button)

## Implementation

### Nodes added (all at x≈7000–9050, y≈5000)

| Node | Details |
|------|---------|
| InputAction IA_Fire | Pressed → fires the trace |
| Get Actor Location | Trace start point (player position) |
| Get Actor Forward Vector | Direction of trace |
| vector × float (2000) | Scale forward 2000u |
| vector + vector | End = Start + Forward×2000 |
| Line Trace By Channel | `bIgnoreSelf=true`, `DrawDebugType=ForDuration` (visible in PIE) |
| Branch | Gate on hit (ReturnValue bool) |
| Break Hit Result | Extract HitActor from OutHit |
| Cast To BP_Enemy | Type-check — miss if not an enemy |
| Apply Damage | `BaseDamage=25.0`, `DamagedActor=AsBP Enemy` |

Comment label: red `#DC2626` wrapping all 10 nodes.

## Flow

```
InputAction IA_Fire [Pressed]
  → Line Trace By Channel (Start=ActorLocation, End=Start+Forward×2000, DrawDebug=ForDuration)
    → Branch [hit?]
      → [true] Break Hit Result → HitActor
        → Cast To BP_Enemy
          → [success] Apply Damage (DamagedActor=enemy, BaseDamage=25.0)
            → triggers BP_Enemy Event AnyDamage
              → Cast To BP_PlatformingCharacter → SetCorruptionLevel -= amount → Ragdoll
```

## Design notes

- **Line trace vs projectile:** Instant-cast (hitscan) rather than a flying ball.
  Avoids needing to duplicate BP_Projectile and is cleaner for the prototype's
  arena scale (2000u = roughly 20m, enough for the whole spawn area).
- **Forward vector vs camera:** Traces from actor forward, not camera center.
  Accurate enough for a top-down/third-person prototype. Can switch to camera
  ray (GetPlayerCameraManager → GetCameraLocation/Rotation) before vertical slice.
- **Debug draw:** `ForDuration` shows the trace ray in PIE with red=miss,
  green=hit. Remove or set to `None` before production.
- **Damage = 25:** Triggers enemy ragdoll + one kill-purge cycle on the player's
  corruption. Tune with the `Damage` pin on Apply Damage or in BP_Enemy's
  AnyDamage handler.

## Slide screenshot

![BP_PlatformingCharacter fireball line trace](05-bp-platforming-character-fireball.png)
