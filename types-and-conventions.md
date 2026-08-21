---
description: Understand SDK types, copied tables, coordinates, slots, heroes, actions, and failure conventions.
---

# Types and conventions

The SDK uses Lua 5.1/LuaJIT values with strict native argument validation. Passing the wrong type raises a script error unless a function explicitly documents another result.

## Primitive types

| Documentation type | Lua value |
| --- | --- |
| `boolean` | `true` or `false` |
| `integer` | A whole Lua number accepted as an integer by the function |
| `number` | A finite Lua number |
| `string` | A Lua string within the documented length limit |
| `function` | A Lua callback function |
| `table` | A Lua table with the documented fields |
| `nil` | Data is unavailable or no valid result exists |

Optional does not mean that `nil` is accepted as a placeholder. Unless a function says otherwise, omit trailing optional arguments or provide every earlier positional argument.

## Shared table types

### `Vector3`

```lua
local position = {
    x = 0.0, -- number
    y = 0.0, -- number
    z = 0.0, -- number
}
```

### `ScreenPoint`

```lua
local point = {
    x = 960.0, -- number, pixels from the left edge
    y = 540.0, -- number, pixels from the top edge
}
```

### `ScreenSize`

```lua
local size = {
    width = 1920,  -- integer
    height = 1080, -- integer
}
```

### `Hero`

Arguments documented as `Hero` accept either:

- `integer`: a known hero ID
- `string`: a known case-insensitive hero name

```lua
znt.game.is_hero("Haze")
znt.game.is_hero(13)
```

## Common numeric conventions

| Value | Accepted form |
| --- | --- |
| Ability slot | `integer` from `1` through `4` |
| Aim style | `0` for Regular, `1` for pSilent |
| Virtual key | `integer` from `0` through `254` |
| RGBA channel | `number`, clamped from `0` through `255` |
| Script timer | `znt.game.time_ms()` in monotonic milliseconds |
| Ability/weapon timer | Snapshot `game_time` in simulation seconds |

## Named input actions

Input functions accept these case-insensitive strings:

```text
attack, reload, ability1, ability2, ability3, ability4, parry
```

The lowercase forms are recommended for consistency.

## Snapshots are copies

Player, ability, weapon, projectile, and damage tables are copied into Lua. Editing a returned table does not change game state.

```lua
local player = znt.game.local_player()
if player then
    player.health = 999999 -- Changes only this Lua table.
end
```

Never retain a snapshot as permanent truth. Request fresh data when making a new gameplay decision.

## Failure behavior

The SDK fails closed:

- Temporarily unavailable game data normally returns `nil` or `false`.
- Invalid argument types, values, or callback phases raise a script error.
- A callback error unloads only the affected script.
- Damage functions return `nil, status` instead of an incomplete prediction.

Every API page documents its specific return and failure contract.
