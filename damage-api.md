---
description: Predict reviewed ability damage and evaluate typed raw-damage formulas against live modifiers.
---

# Damage

The `znt.damage` namespace evaluates damage against copied live source and target data.

Every damage function follows a fail-closed result convention. Success returns one complete `DamagePrediction`; its `status` field is `"complete"`. Failure returns `nil, status`. A failure never returns a partial prediction.

{% hint style="warning" %}
Unavailable damage is unknown damage. Do not substitute zero or continue a lethal-only action when the prediction is `nil`.
{% endhint %}

## Status type

`DamageStatus` is one of:

| Value | Meaning |
| --- | --- |
| `"complete"` | Prediction completed |
| `"invalid_request"` | Request cannot be evaluated |
| `"source_unavailable"` | Local damage source is unavailable |
| `"target_unavailable"` | Target cannot be resolved |
| `"ability_unavailable"` | Required ability state is unavailable |
| `"property_unavailable"` | Required live property is unavailable |
| `"runtime_unavailable"` | Required runtime state is unavailable |
| `"unsupported_ability"` | No reviewed high-level formula exists |
| `"missing_factor"` | A required formula factor is missing |

## `znt.damage.ability(target_index, slot, options)`

Predicts a supported local ability using a reviewed high-level formula and live modifiers.

**Signature:** `znt.damage.ability(target_index, slot, options)`

**Valid phase:** top-level code or any callback

| Argument | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `target_index` | `integer` | Yes | — | Target player entity index |
| `slot` | `integer` | Yes | — | Ability slot from `1` through `4` |
| `options` | `AbilityDamageOptions` or `nil` | No | `nil` | Charge, hit variant, and impact timing |

### `AbilityDamageOptions`

| Field | Type | Required | Default | Accepted values |
| --- | --- | --- | --- | --- |
| `charge` | `number` or `string` | No | Live current charge | Number from `0.0` through `1.0`, `"current"`, or `"full"` |
| `hit` | `string` | No | `"head"` | `"head"` or `"body"` |
| `impact_delay` | `number` or `string` | No | `0.0` | `0.0` through `60.0` seconds, or `"auto"` to use the reviewed ability's live cast/travel timing |

**Returns:** `DamagePrediction` on success; `nil, DamageStatus` on failure. Capturing two values is recommended so the same call site handles both cases; the second value is `nil` on success.

**Failure:** invalid argument types or option ranges raise a script error. Valid but unavailable or unsupported data returns `nil, status`.

The reviewed high-level models currently support:

- Vindicta slot 4, Assassinate
- Shiv slot 2, Slice and Dice
- Shiv slot 4, Killing Blow

Shiv's automatic timing reads the current source-to-target distance and the ability's live cast delay, movement speed, and minimum travel time. Its current and maximum Rage are read directly from the live ability resource. Slice and Dice includes the delayed second hit only when that resource is full. That branch projects regeneration between hits and applies the first hit's live Spirit-resistance reduction before evaluating the echo. Killing Blow evaluates both its normal live Spirit damage and its upgraded maximum-health execute threshold.

At full Rage, `full_rage_damage_bonus` exposes the live `BuffDamage` percentage shown by the game. The same outgoing bonus is already represented by `source_global_scale`, so consumers must not multiply it into the formula again.

```lua
local prediction, status = znt.damage.ability(target_index, 4, {
    hit = "body",
    impact_delay = "auto",
})

if not prediction then
    return -- required live data is unavailable; do not guess
end

if prediction.executes then
    znt.log("Killing Blow will execute at impact")
elseif prediction.lethal then
    znt.log("Killing Blow's normal Spirit damage is lethal")
end
```

Assassinate may still provide an explicit timing horizon:

```lua
local prediction, status = znt.damage.ability(target_index, 4, {
    charge = "current",
    hit = "head",
    impact_delay = 0.25,
})

if not prediction then
    znt.log("Prediction unavailable: " .. status)
    return
end

if prediction.lethal then
    znt.log("Target is executable")
end
```

## `znt.damage.typed(target_index, damage_type, amount, options)`

Applies live source and target modifiers to an already reviewed raw-damage formula.

**Signature:** `znt.damage.typed(target_index, damage_type, amount, options)`

**Valid phase:** top-level code or any callback

| Argument | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `target_index` | `integer` | Yes | — | Target player entity index |
| `damage_type` | `DamageType` | Yes | — | `"bullet"`, `"spirit"`, `"melee"`, or `"pure"` |
| `amount` | `number` | Yes | — | Reviewed non-negative raw amount, up to `10,000,000` |
| `options` | `TypedDamageOptions` or `nil` | No | `nil` | Critical, shield, and impact timing options |

### `TypedDamageOptions`

| Field | Type | Required | Default | Accepted values |
| --- | --- | --- | --- | --- |
| `critical_bonus` | `number` | No | `0.0` | Percentage from `-100.0` through `1000.0` |
| `bypass_shield` | `boolean` | No | `false` | Ignore the matching shield when `true` |
| `impact_delay` | `number` | No | `0.0` | `0.0` through `60.0` seconds |

The `amount` must already include the attack's base value, upgrades, attribute coefficients, charge curve, and conditional mechanics. This function applies shared outgoing, resistance, shield, regeneration, and lethality stages.

**Returns:** `DamagePrediction` on success; `nil, DamageStatus` on failure. The second captured value is `nil` on success.

**Failure:** invalid argument types or ranges raise a script error. Unavailable live data returns `nil, status`.

```lua
local prediction, status = znt.damage.typed(target_index, "spirit", 250.0, {
    impact_delay = 0.4,
})

if prediction then
    znt.log("Expected health damage: " .. tostring(prediction.health_damage))
end
```

## `DamagePrediction`

All fields below exist on a completed prediction.

| Field | Type | Description |
| --- | --- | --- |
| `ok` | `boolean` | `true` for a complete result |
| `status` | `string` | `"complete"` |
| `type` | `DamageType` | Resolved damage type |
| `source_index` | `integer` | Local source entity index |
| `target_index` | `integer` | Target entity index |
| `ability_rank` | `integer` | Resolved ability rank, or model default for typed damage |
| `applied_factors` | `integer` | Internal applied-factor bit set for diagnostics |
| `missing_factors` | `integer` | Internal missing-factor bit set; zero on a complete result |
| `source_power` | `number` | Source attribute used by the formula |
| `charge` | `number` | Charge progress from `0.0` through `1.0` |
| `charge_scale` | `number` | Applied charge multiplier |
| `minimum_charge_scale` | `number` | Minimum charge multiplier |
| `full_charge_time` | `number` | Seconds required for full charge |
| `formula_damage` | `number` | Ability-specific or provided formula result |
| `critical_bonus` | `number` | Selected critical bonus percentage |
| `source_critical_scale` | `number` | Source-side critical multiplier |
| `target_critical_scale` | `number` | Target-side critical multiplier |
| `critical_scale` | `number` | Combined critical multiplier |
| `source_global_scale` | `number` | Global outgoing multiplier |
| `source_type_scale` | `number` | Damage-type outgoing multiplier |
| `after_source` | `number` | Damage after outgoing modifiers |
| `resistance` | `number` | Target resistance percentage |
| `mitigated_damage` | `number` | Damage after resistance |
| `shield` | `number` | Matching target shield before damage |
| `shield_damage` | `number` | Damage absorbed by shield |
| `health_damage` | `number` | Damage reaching health |
| `target_health` | `number` | Current target health |
| `target_max_health` | `number` | Maximum target health |
| `target_health_regen` | `number` | Target health regeneration per second |
| `impact_delay` | `number` | Projected seconds until impact |
| `projected_target_health` | `number` | Health after regeneration at impact time |
| `health_after` | `number` | Projected health after damage |
| `lethal_shortfall` | `number` | Additional health damage needed to be lethal |
| `lethal_health` | `number` | Lethality threshold used by the model |
| `secondary_health_damage` | `number` | Health damage from an included conditional follow-up hit; otherwise `0.0` |
| `secondary_impact_delay` | `number` | Seconds from now to that follow-up impact; otherwise `0.0` |
| `execute_threshold_health` | `number` | Maximum-health execute threshold for the resolved ability; otherwise `0.0` |
| `rage_current` | `number` | Shiv's current live Rage resource; otherwise `0.0` |
| `rage_max` | `number` | Shiv's live maximum Rage resource; otherwise `0.0` |
| `rage_fraction` | `number` | Normalized Rage progress from `0.0` through `1.0`; otherwise `0.0` |
| `full_rage_damage_bonus` | `number` | Active live full-Rage outgoing bonus percentage; `0.0` when Rage is not full |
| `conditional_followup` | `boolean` | Whether the prediction includes a currently active conditional follow-up hit |
| `executes` | `boolean` | Whether projected impact health satisfies the ability's execute condition |
| `full_rage` | `boolean` | Whether Shiv's live Rage resource is full |
| `lethal` | `boolean` | Whether the completed prediction is lethal |

For a multi-hit result, `formula_damage`, `after_source`, `mitigated_damage`, and `health_damage` include every modeled hit. `projected_target_health` remains the health immediately before the first hit; `health_after` includes regeneration and damage through the last modeled hit.

The model is exact for the deterministic state captured by the call. A future movement change can still move a target out of Slice and Dice's echo path, so scripts should refresh the prediction immediately before submitting a cast.

## Draw a completed result

Refresh predictions during state or command logic, then pass the copied result to `znt.draw.damage_preview()` during `render`:

```lua
znt.events.on("render", function()
    if latest_prediction then
        znt.draw.damage_preview(target_index, latest_prediction, nil, "EXECUTE")
    end
end)
```

## Related pages

- [Drawing](drawing-api.md)
- [Game data](game-api.md)
- [Troubleshooting](troubleshooting.md)
