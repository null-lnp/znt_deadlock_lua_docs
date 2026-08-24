---
description: Read typed hero, player, ability, weapon, visibility, range, and timing snapshots.
---

# Game data

The `znt.game` namespace exposes copied snapshots of live game state. Editing a returned table never changes the game. Request a fresh snapshot before each new decision.

Unless noted otherwise, game-data functions may be called from top-level code or any callback. Temporarily unavailable data returns `nil` or `false`; invalid arguments raise a script error.

## Time

### `znt.game.time_ms()`

**Signature:** `znt.game.time_ms()`

**Arguments:** none

**Valid phase:** top-level code or any callback

**Returns:** `integer` — monotonic milliseconds suitable for script timers.

**Failure:** none under normal runtime operation.

```lua
local started_at = znt.game.time_ms()

znt.events.on("update", function()
    local elapsed_ms = znt.game.time_ms() - started_at
end)
```

## Hero identity

### `znt.game.local_hero()`

**Signature:** `znt.game.local_hero()`

**Arguments:** none

**Valid phase:** top-level code or any callback

**Returns:** `LocalHero | nil`.

#### `LocalHero`

| Field | Type | Description |
| --- | --- | --- |
| `id` | `integer` | Resolved hero ID |
| `name` | `string` | Canonical hero name |

**Failure:** returns `nil` while the local hero cannot be resolved.

### `znt.game.is_hero(hero)`

**Signature:** `znt.game.is_hero(hero)`
**Valid phase:** top-level code or any callback

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `hero` | `Hero` | Yes | Known numeric hero ID or case-insensitive hero name |

**Returns:** `boolean` — whether the current local hero matches.

**Failure:** an unknown hero name or ID, or the wrong argument type, raises a script error. An unavailable local hero returns `false`.

```lua
if not znt.game.is_hero("Vindicta") then
    return
end
```

## Players

### `znt.game.local_player()`

**Signature:** `znt.game.local_player()`

**Arguments:** none

**Valid phase:** top-level code or any callback

**Returns:** `PlayerSnapshot | nil`.

**Failure:** returns `nil` while the local player cannot be resolved.

### `znt.game.player(entity_index)`

**Signature:** `znt.game.player(entity_index)`
**Valid phase:** top-level code or any callback

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `entity_index` | `integer` | Yes | Player or owned child-entity index |

**Returns:** `PlayerSnapshot | nil`. Owned child entities, such as weapons, are resolved to their player when possible.

**Failure:** returns `nil` when no player can be resolved; a non-number raises a script error.

### `znt.game.players()`

**Signature:** `znt.game.players()`

**Arguments:** none

**Valid phase:** top-level code or any callback

**Returns:** `PlayerSnapshot[]` — a one-based array containing up to 24 currently resolved players. The array may be empty.

**Failure:** unavailable entries are omitted.

### `PlayerSnapshot`

| Field | Type | Description |
| --- | --- | --- |
| `index` | `integer` | Entity index |
| `team` | `integer` | Team number |
| `health` | `integer` | Current health |
| `max_health` | `integer` | Maximum health |
| `hero_id` | `integer` | Resolved hero ID |
| `hero_name` | `string` | Canonical hero name |
| `alive` | `boolean` | Whether the player is alive |
| `local_player` | `boolean` | Whether this is the local player |
| `origin` | `Vector3` | World position |
| `velocity` | `Vector3` | Current world velocity |
| `view_angles` | `Vector3` | Client view angles |
| `eye_angles` | `Vector3` | Player eye angles |

## Abilities

### `znt.game.ability(slot)`

**Signature:** `znt.game.ability(slot)`
**Valid phase:** top-level code or any callback

| Argument | Type | Required | Accepted values |
| --- | --- | --- | --- |
| `slot` | `integer` | Yes | `1` through `4` |

**Returns:** `AbilitySnapshot | nil`.

**Failure:** returns `nil` when the ability cannot be resolved. An invalid slot or argument type raises a script error.

### `AbilitySnapshot`

| Field | Type | Description |
| --- | --- | --- |
| `present` | `boolean` | Ability entity is available |
| `learned` | `boolean` | Ability has at least one upgrade point |
| `ready` | `boolean` | Ability is learned and currently usable by cooldown state |
| `charges` | `integer` | Current charge count |
| `upgrade_level` | `integer` | Current displayed ability level |
| `game_time` | `number` | Current simulation time in seconds |
| `cooldown_remaining` | `number` | Remaining cooldown in seconds |
| `charge_recharge_remaining` | `number` | Seconds until the next charge |
| `scoped` | `boolean` | Current scoped state where supported |
| `scope_started_at` | `number` | Simulation time at which scope began |
| `projectile_speed` | `number` | Live projectile speed |
| `projectile_gravity` | `number` | Live projectile gravity |
| `projectile_inherit` | `number` | Shooter-velocity inheritance scale |

```lua
local ability = znt.game.ability(1)
if not ability or not ability.ready or ability.charges == 0 then
    return
end
```

## Primary weapon

### `znt.game.weapon()`

**Signature:** `znt.game.weapon()`

**Arguments:** none

**Valid phase:** top-level code or any callback

**Returns:** `WeaponSnapshot | nil`.

**Failure:** returns `nil` while the local weapon cannot be resolved.

### `WeaponSnapshot`

| Field | Type | Description |
| --- | --- | --- |
| `present` | `boolean` | Weapon entity is available |
| `in_reload` | `boolean` | Reload is active |
| `can_active_reload` | `boolean` | Active Reload is currently available |
| `clip` | `integer` | Current magazine count |
| `game_time` | `number` | Current simulation time in seconds |
| `reload_started_at` | `number` | Reload start time |
| `next_attack_at` | `number` | Next allowed attack time |
| `attack_delay_pause` | `number` | Pause accumulated during the current attack delay |
| `reload_available_at` | `number` | Active Reload availability time |

{% hint style="warning" %}
Ability and weapon timestamps use simulation seconds. Compare them with the same snapshot's `game_time`, not `znt.game.time_ms()`.
{% endhint %}

## Visibility

### `znt.game.is_visible(entity_index)`

**Signature:** `znt.game.is_visible(entity_index)`
**Valid phase:** top-level code or any callback

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `entity_index` | `integer` | Yes | Target player or owned entity index |

**Returns:** `boolean` — `true` when the target resolves and at least one sampled body trace is unobstructed.

The SDK starts at the active camera, falling back to standing eye height when no camera is available. It tests upper, middle, and lower body heights and excludes both player pawns from the obstruction filter. This avoids close-range false negatives caused by floor-level origins or by the target intersecting its own ray.

**Failure:** unavailable, invalid, or obstructed targets return `false`; a non-number raises a script error.

## Melee range

### `znt.game.melee_range(entity_index)`

**Signature:** `znt.game.melee_range(entity_index)`
**Valid phase:** top-level code or any callback

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `entity_index` | `integer` | Yes | Target entity index |

**Returns:** `MeleeRangeSnapshot | nil`.

| Field | Type | Description |
| --- | --- | --- |
| `valid` | `boolean` | Whether the range was resolved |
| `index` | `integer` | Resolved target index |
| `range` | `number` | Current stat-scaled `MeleeAttackLength` in world units |
| `half_angle` | `number` | Current `MeleeHalfAngle` in degrees, or `0` when this optional property is unavailable |

The SDK checks the player's ability component first, then the replicated entity list for a hold-melee ability owned by that player. This second path makes remote melee range and angle data available when the component vector omits the ability.

**Failure:** returns `nil` when the player, owned melee entity, or range data is unavailable. A non-finite or out-of-range index raises a script error. A valid range snapshot may still contain `half_angle == 0`; use an explicit conservative fallback rather than treating zero as a real cone.

## Melee state

### `znt.game.melee_state(entity_index)`

**Signature:** `znt.game.melee_state(entity_index)`

**Valid phase:** top-level code or any callback. Poll it from `pre_move` when reaction timing matters.

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `entity_index` | `integer` | Yes | Player entity index from `0` through `32766` |

**Returns:** `MeleeStateSnapshot | nil`.

### `MeleeStateSnapshot`

| Field | Type | Description |
| --- | --- | --- |
| `valid` | `boolean` | Whether all required replicated fields were resolved |
| `index` | `integer` | Resolved player index |
| `active` | `boolean` | A melee input is currently in progress |
| `charging` | `boolean` | The attack is still in its charge phase |
| `threatening` | `boolean` | The attack has entered a dash or strike phase and can threaten a target |
| `hit` | `boolean` | The current attack already registered a hit |
| `attack_type` | `string` | `none`, `light`, `heavy`, `heavy_air`, or `slide` |
| `attack_type_id` | `integer` | Replicated numeric attack type |
| `state` | `string` | `none`, `charging`, `ground_dashing`, `air_dashing`, `attacking`, or `slide_dashing` |
| `state_id` | `integer` | Replicated numeric attack state |
| `game_time` | `number` | Attacker simulation time in seconds |
| `state_started_at` | `number` | Simulation timestamp when the current state began |
| `attack_started_at` | `number` | Stable simulation timestamp identifying the current attack |
| `state_age` | `number` | Non-negative seconds spent in the current state |
| `source` | `string` | `"ability"` for a full ability snapshot, `"cooldown"` for a fresh public cooldown-start edge, or `"modifier"` for the remote-player fallback |

The full ability snapshot is resolved from either the player's component vector or an owned hold-melee entity in the replicated entity list. When the public melee-ability cooldown-start timestamp advances before owner-only detailed melee fields replicate, the SDK emits a minimal `"cooldown"` snapshot with that timestamp in `attack_started_at`. Recognized modifiers remain the fallback when no full ability is available.

**Failure:** returns `nil` when the player is unavailable or dead, or when neither a standard hold-melee ability nor a recognized replicated melee modifier can be resolved. A non-finite or out-of-range index raises a script error.

Use `active` as the earliest reaction gate for a light melee. A heavy melee can remain `charging` for a while, so wait until `threatening` becomes `true` for heavy attacks. Use `attack_started_at` to prevent repeated work for the same swing. Modifier-backed snapshots are conservative and may not carry every ability detail, so use `source` when diagnostics need to distinguish the fallback.

```lua
local handled = {}

znt.events.on("pre_move", function()
    for _, player in ipairs(znt.game.players()) do
        local melee = znt.game.melee_state(player.index)
        local actionable = melee and (melee.attack_type == "light" and melee.active or melee.threatening)
        if actionable and not melee.hit then
            if handled[player.index] ~= melee.attack_started_at then
                handled[player.index] = melee.attack_started_at
                -- Validate team, range, facing, and visibility before acting.
            end
        elseif not melee or not melee.active then
            handled[player.index] = nil
        end
    end
end)
```

## Related pages

- [Types and conventions](types-and-conventions.md)
- [Hero assistance](hero-api.md)
- [Damage](damage-api.md)
