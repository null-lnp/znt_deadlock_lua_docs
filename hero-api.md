---
description: Select targets, aim, predict projectiles, inspect range, and use supported hero helpers.
---

# Hero assistance

The `znt.hero` namespace provides constrained combat helpers built on validated game state. It does not expose writable memory.

{% hint style="warning" %}
`znt.hero.aim()` and `znt.hero.look_at()` are valid only during `pre_move`. Recheck readiness, target state, range, and path immediately before requesting an action.
{% endhint %}

## `znt.hero.targets(fov, body_only)`

Returns visible enemies inside a field of view, sorted from smallest to largest crosshair distance.

**Signature:** `znt.hero.targets(fov, body_only)`

**Valid phase:** top-level code or any callback; use a command callback for combat decisions

| Argument | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `fov` | `number` | Yes | — | Positive aim field of view |
| `body_only` | `boolean` | No | `false` | Excludes the head target group when `true` |

**Returns:** `TargetSnapshot[]` — a one-based array that may be empty.

### `TargetSnapshot`

| Field | Type | Description |
| --- | --- | --- |
| `index` | `integer` | Player entity index |
| `health` | `integer` | Current health |
| `max_health` | `integer` | Maximum health |
| `screen_distance` | `number` | Pixel distance from the crosshair |

**Failure:** a missing or non-positive `fov` raises a script error. No valid target produces an empty array.

```lua
local target = znt.hero.targets(120.0, true)[1]
if not target then
    znt.hero.reset_aim()
    return
end
```

## `znt.hero.aim(...)`

Revalidates a target, visibility, FOV, and optional projectile data, then prepares aim for the current command.

**Signature:** `znt.hero.aim(target_index, fov, smooth_x, smooth_y, ability_slot, body_only, aim_style)`

**Valid phase:** `pre_move` only

| Argument | Type | Required | Default | Accepted values |
| --- | --- | --- | --- | --- |
| `target_index` | `integer` | Yes | — | Player entity index |
| `fov` | `number` | Yes | — | Positive field of view |
| `smooth_x` | `number` | Yes | — | `1.0` through `30.0` |
| `smooth_y` | `number` | Yes | — | `1.0` through `30.0` |
| `ability_slot` | `integer` or `nil` | No | `nil` | `1` through `4` for live projectile data; `nil` for instant aim |
| `body_only` | `boolean` | No | `false` | Exclude the head group when `true` |
| `aim_style` | `integer` | No | `0` | `0` Regular, `1` pSilent |

**Returns:** `AimResult | nil`.

| Field | Type | Description |
| --- | --- | --- |
| `index` | `integer` | Revalidated target index |
| `error` | `number` | Remaining angular error in degrees |

**Failure:** returns `nil` outside an active `pre_move` input context or when the target cannot be validated. Invalid types or ranges raise a script error.

{% hint style="info" %}
To omit `ability_slot` while passing later positional arguments, pass `nil` explicitly.
{% endhint %}

```lua
local aim = znt.hero.aim(target.index, 120.0, 4.0, 4.0, 1, true, 0)
if aim and aim.error <= 1.0 then
    znt.input.tap("ability1")
end
```

## `znt.hero.reset_aim()`

Clears the current script's aim smoothing and target history.

**Signature:** `znt.hero.reset_aim()`

**Arguments:** none

**Valid phase:** top-level code or any callback

**Returns:** nothing.

**Failure:** safe no-op if the script record is unavailable.

Call it whenever a script is disabled, a key is released, a target is lost, or the local hero no longer matches.

## `znt.hero.projectile(...)`

Builds a live projectile intercept assessment for an ability.

**Signature:** `znt.hero.projectile(target_index, fov, ability_slot, body_only)`

**Valid phase:** top-level code or any callback; refresh it in the command that may cast

| Argument | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `target_index` | `integer` | Yes | — | Player entity index |
| `fov` | `number` | Yes | — | Positive field of view |
| `ability_slot` | `integer` | Yes | — | Ability slot from `1` through `4` |
| `body_only` | `boolean` | No | `false` | Exclude the head group when `true` |

**Returns:** `ProjectileSnapshot | nil`.

### `ProjectileSnapshot`

| Field | Type | Description |
| --- | --- | --- |
| `valid` | `boolean` | Whether the assessment resolved |
| `index` | `integer` | Resolved target index |
| `aim_position` | `Vector3` | Predicted world-space intercept |
| `distance` | `number` | Current target distance |
| `cast_range` | `number` | Live ability cast range |
| `flight_time` | `number` | Predicted travel time in seconds |
| `in_range` | `boolean` | Whether the endpoint is in cast range |
| `path_clear` | `boolean` | Whether the predicted projectile path is unobstructed |

**Failure:** returns `nil` when prediction data is unavailable. Invalid target, FOV, or slot arguments raise a script error.

```lua
local shot = znt.hero.projectile(target.index, fov, 1, true)
if not shot or not shot.valid or not shot.in_range or not shot.path_clear then
    return
end
```

## `znt.hero.range(target_index, ability_slot)`

Checks the current distance against an ability's live cast range.

**Signature:** `znt.hero.range(target_index, ability_slot)`

**Valid phase:** top-level code or any callback

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `target_index` | `integer` | Yes | Target entity index |
| `ability_slot` | `integer` | Yes | Ability slot from `1` through `4` |

**Returns:** `RangeSnapshot | nil`.

| Field | Type | Description |
| --- | --- | --- |
| `valid` | `boolean` | Whether range resolved |
| `in_range` | `boolean` | Whether the target is in range |
| `index` | `integer` | Resolved target index |
| `distance` | `number` | Current target distance |
| `cast_range` | `number` | Live ability cast range |

**Failure:** returns `nil` when required live data is unavailable. Invalid arguments raise a script error.

## `znt.hero.item_target(item_slot)`

Finds the nearest visible enemy inside an active item's live targeting cone.

**Signature:** `znt.hero.item_target(item_slot)`

**Valid phase:** top-level code or any callback; refresh it in the command that may activate or confirm the item

| Argument | Type | Required | Accepted values |
| --- | --- | --- | --- |
| `item_slot` | `integer` | Yes | Active-item inventory slot `1` through `4` |

**Returns:** `ItemTargetSnapshot | nil`.

### `ItemTargetSnapshot`

| Field | Type | Description |
| --- | --- | --- |
| `valid` | `boolean` | Whether a cone candidate resolved |
| `in_range` | `boolean` | Whether the candidate is inside the live item cast range |
| `path_clear` | `boolean` | Whether the game ray trace and Fissure-wall checks passed |
| `index` | `integer` | Candidate player entity index |
| `distance` | `number` | Current target distance |
| `cast_range` | `number` | Live item range after contextual range scaling |
| `cone_angle` | `number` | Live targeting-cone half-angle in degrees |
| `angular_error` | `number` | Candidate angle from the active camera direction |

**Failure:** returns `nil` when the item has no supported cone/range data or no visible enemy is inside the cone. An invalid slot raises a script error.

```lua
local item = znt.game.active_item(2)
local target = item and znt.hero.item_target(item.slot) or nil

if item and item.ready and target and target.valid and
    target.in_range and target.path_clear then
    znt.input.tap("item" .. item.slot)
end
```

## `znt.hero.hook(ability_slot)`

Returns Bebop's confirmed hook state.

**Signature:** `znt.hero.hook(ability_slot)`

**Valid phase:** top-level code or any callback

| Argument | Type | Required | Default | Accepted values |
| --- | --- | --- | --- | --- |
| `ability_slot` | `integer` | No | `3` | `1` through `4` |

**Returns:** `HookSnapshot | nil`.

| Field | Type | Description |
| --- | --- | --- |
| `active` | `boolean` | Whether a confirmed hook reel is active |
| `index` | `integer` | Hooked player index |
| `reel_started_at` | `number` | Hook reel start in simulation seconds |
| `cancel_at` | `number` | Hook state expiry in simulation seconds |

**Failure:** returns `nil` when hook state cannot be resolved. An invalid slot raises a script error.

## `znt.hero.throw_destination(ability_slot, target_index)`

Finds a reachable allied Guardian or Walker destination for a Bebop Uppercut throw.

**Signature:** `znt.hero.throw_destination(ability_slot, target_index)`

**Valid phase:** top-level code or any callback

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `ability_slot` | `integer` | Yes | Throw ability slot from `1` through `4` |
| `target_index` | `integer` | Yes | Player being thrown |

**Returns:** `ThrowDestination | nil`.

| Field | Type | Description |
| --- | --- | --- |
| `valid` | `boolean` | Whether a destination resolved |
| `objective_index` | `integer` | Allied objective entity index |
| `look_position` | `Vector3` | World point the camera should face |
| `landing_position` | `Vector3` | Predicted landing point |
| `distance` | `number` | Distance to the objective |
| `throw_distance` | `number` | Predicted throw travel distance |

**Failure:** returns `nil` when no reachable allied destination exists. Invalid arguments raise a script error.

## `znt.hero.look_at(position, smoothing)`

Turns the visible camera toward a copied world position.

**Signature:** `znt.hero.look_at(position, smoothing)`

**Valid phase:** `pre_move` only

| Argument | Type | Required | Default | Accepted values |
| --- | --- | --- | --- | --- |
| `position` | `Vector3` | Yes | — | Finite numeric `x`, `y`, and `z` fields |
| `smoothing` | `number` | No | `0` | `0.0` through `30.0`; `0` turns directly |

**Returns:** `LookResult | nil`.

| Field | Type | Description |
| --- | --- | --- |
| `error` | `number` | Remaining angular error in degrees |

**Failure:** an invalid phase, vector, or smoothing value raises a script error. Returns `nil` when a valid look result cannot be produced.

## Safe projectile cast

```lua
znt.events.on("pre_move", function()
    local target = znt.hero.targets(120.0, true)[1]
    if not target then
        znt.hero.reset_aim()
        return
    end

    local shot = znt.hero.projectile(target.index, 120.0, 1, true)
    if not shot or not shot.in_range or not shot.path_clear then
        znt.hero.reset_aim()
        return
    end

    local aim = znt.hero.aim(target.index, 120.0, 4.0, 4.0, 1, true, 0)
    if aim and aim.error <= 1.0 then
        znt.input.tap("ability1")
    end
end)
```

## Related pages

- [Hero scripting guide](hero-scripting-guide.md)
- [Game data](game-api.md)
- [Input](input-api.md)
