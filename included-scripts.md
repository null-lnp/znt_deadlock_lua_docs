---
description: Learn which SDK patterns are demonstrated by the scripts included with Zenith.
---

# Included scripts

Zenith includes focused scripts that demonstrate complete SDK workflows. Use them as references when starting a script with similar behavior.

| Script | Demonstrates | Relevant pages |
| --- | --- | --- |
| `auto_parry.lua` | Replicated melee abilities/modifiers, independently selectable heavy/light handling, directional fallbacks, live light-melee cone geometry, optional debug-sector drawing, and debounced parry input | [Game data](game-api.md), [Input](input-api.md), [Drawing](drawing-api.md), [Menu](menu-api.md) |
| `auto_reload.lua` | Weapon timing and named reload input | [Game data](game-api.md), [Input](input-api.md) |
| `bebop_combo.lua` | Projectile prediction, confirmed hook state, staged input, and objective throws | [Hero assistance](hero-api.md), [Input](input-api.md) |
| `haze_sleep_dagger.lua` | Hero gating, projectile aim, and same-command casting | [Hero scripting guide](hero-scripting-guide.md) |
| `shiv_serrated_knives.lua` | Charge-aware projectile assistance | [Hero assistance](hero-api.md), [Game data](game-api.md) |
| `vindicta_snipe.lua` | Damage prediction, scope timing, input ownership, and damage preview drawing | [Damage](damage-api.md), [Drawing](drawing-api.md) |

## Auto Parry controls

The persistent **Melee types** multiselect enables Heavy, Light, or both independently. Disabling a type clears its pending replicated and sound-fallback state, so changing the selection cannot replay an old attack.

The SDK resolves remote hold-melee abilities from owned entity-list entries when the player component omits them, so an active light state can trigger before the later swing sound. Heavy attacks must face the local player even at point blank. Light attacks require the local player to be inside the attacker's forward sector and no farther than 200 world units. The sample uses an effective half-angle of at least 42°, while honoring wider live `MeleeHalfAngle` values. `Ability.Melee.Impact.Player` is intentionally not used for detection because it arrives after the hit.

Enable **Debug light cone** in the Auto Parry tab to draw that exact sector for 650 ms after a detected light swing. The arc and forward rays are green when the local player is inside the angle, range, and height gates, and red when outside. The label shows the active half-angle and reach in world units. This diagnostic is disabled by default.

The sample does not emit routine detection, rejection, or parry logs, keeping the Lua Console clear for errors and explicit script diagnostics.

## Locations

The runtime discovers loadable `.lua` files directly inside:

```text
C:\znt_dd
```

Packaged examples may also be available for reference under:

```text
C:\znt_dd\samples
```

Subdirectories are not scanned as loadable scripts. Copy a sample into `C:\znt_dd` when you want it to appear in the script manager. Give an edited copy a distinct filename and `-- znt:name` value.

## Before editing a sample

- Keep its `nil` and state checks unless replacement logic makes them unnecessary.
- Preserve callback phases when moving code.
- Give every setting a stable key unique within that script.
- Keep hero-specific callbacks gated with `znt.game.is_hero()`.
- Treat a damage status as part of the result, not optional logging.
- Reset aim state when activation, target, hero, or enabled state changes.

See [Getting started](getting-started.md) for discovery and loading instructions.
