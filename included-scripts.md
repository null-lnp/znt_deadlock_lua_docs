---
description: Learn which SDK patterns are demonstrated by the scripts included with Zenith.
---

# Included scripts

Zenith includes focused scripts that demonstrate complete SDK workflows. Use them as references when starting a script with similar behavior.

| Script | Demonstrates | Relevant pages |
| --- | --- | --- |
| `auto_parry.lua` | Replicated melee abilities/modifiers, directional heavy/light fallbacks, live light-melee cone geometry, optional debug-sector drawing, diagnostics, and debounced parry input | [Game data](game-api.md), [Input](input-api.md), [Drawing](drawing-api.md) |
| `auto_reload.lua` | Weapon timing and named reload input | [Game data](game-api.md), [Input](input-api.md) |
| `bebop_combo.lua` | Projectile prediction, confirmed hook state, staged input, and objective throws | [Hero assistance](hero-api.md), [Input](input-api.md) |
| `haze_sleep_dagger.lua` | Hero gating, projectile aim, and same-command casting | [Hero scripting guide](hero-scripting-guide.md) |
| `shiv_serrated_knives.lua` | Charge-aware projectile assistance | [Hero assistance](hero-api.md), [Game data](game-api.md) |
| `vindicta_snipe.lua` | Damage prediction, scope timing, input ownership, and damage preview drawing | [Damage](damage-api.md), [Drawing](drawing-api.md) |

## Auto Parry diagnostics

`auto_parry.lua` writes one concise entry for each important decision:

| Console entry | Meaning |
| --- | --- |
| `melee state detected ... [ability]` | The full replicated melee ability supplied the phase |
| `melee state detected ... [modifier]` | A remote-player melee modifier supplied the fallback phase |
| `heavy melee detected ... [sound fallback]` | The known heavy-charge sound was received |
| `light melee detected ... [swing sound fallback: ...]` | A hero's pre-impact light-swing sound was received |
| `parry: ...` | The script passed the applicable team, range, visibility, FOV, and threat gates and requested parry input |
| `ignored heavy melee ...` | The fallback expired; the remainder of the message identifies the rejected gate |
| `ignored light melee ...` | The short light-swing window expired; the remainder identifies the last rejected gate |

Use the detached **Lua Console** while reproducing an issue. A detection entry without a later `parry:` entry means the input request was intentionally gated rather than lost. Heavy attacks must face the local player even at point blank. Light attacks require the local player to be inside the attacker's forward sector and no farther than 300 world units. The sample uses the live `MeleeHalfAngle` when available; the inspected standard melee VData reports a 30° half-angle, and unavailable remote VData falls back conservatively to 37.5°. `Ability.Melee.Impact.Player` is intentionally not used for detection because it arrives after the hit.

Enable **Debug light cone** in the Auto Parry tab to draw that exact sector for 650 ms after a detected light swing. The arc and forward rays are green when the local player is inside the angle, range, and height gates, and red when outside. The label shows the active half-angle and reach in world units. This diagnostic is disabled by default.

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
