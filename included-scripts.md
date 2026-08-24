---
description: Learn which SDK patterns are demonstrated by the scripts included with Zenith.
---

# Included scripts

Zenith includes focused scripts that demonstrate complete SDK workflows. Use them as references when starting a script with similar behavior.

| Script | Demonstrates | Relevant pages |
| --- | --- | --- |
| `auto_parry.lua` | Replicated melee abilities/modifiers, heavy-melee sound fallback, range, visibility, diagnostics, and debounced parry input | [Game data](game-api.md), [Input](input-api.md) |
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
| `parry: ...` | The script passed its team, range, visibility, FOV, and threat gates and requested parry input |
| `ignored heavy melee ...` | The fallback expired; the remainder of the message identifies the rejected gate |

Use **Scripts → Console** while reproducing an issue. A detection entry without a later `parry:` entry means the input request was intentionally gated rather than lost.

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
