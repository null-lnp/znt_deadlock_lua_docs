---
description: Read activation keys and request supported named game actions.
---

# Input

The `znt.input` namespace exposes physical activation-key state and a constrained set of named game actions. It does not expose raw command memory.

## Supported action type

`Action` is a case-insensitive `string` with one of these values:

```text
attack, reload, ability1, ability2, ability3, ability4,
item1, item2, item3, item4, parry, alt_cast
```

Lowercase names are recommended.

`item1` through `item4` map to active-item inventory slots `1` through `4`. Resolve an item's current slot with `znt.game.active_item()` rather than assuming its position.

`alt_cast` is the game's alternate-cast modifier. For a selected friendly-target item whose intended target is the local player, keep it active with `hold()` while the item remains selected. It follows the game action rather than a physical mouse binding.

## `znt.input.key_down(virtual_key)`

Reads whether a physical key is currently held.

**Signature:** `znt.input.key_down(virtual_key)`

**Valid phase:** top-level code or any callback

| Argument | Type | Required | Accepted values |
| --- | --- | --- | --- |
| `virtual_key` | `integer` | Yes | Windows virtual-key code from `0` through `254` |

**Returns:** `boolean`.

**Failure:** an invalid type, fractional number, or value outside the accepted range raises a script error.

```lua
local activation_key = 0x45 -- E

znt.events.on("pre_move", function()
    if not znt.input.key_down(activation_key) then
        return
    end
end)
```

## `znt.input.held(action)`

Reports whether a named action is held in the active game command.

**Signature:** `znt.input.held(action)`

**Valid phase:** `pre_move` or `post_move`

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `action` | `Action` | Yes | Supported named action |

**Returns:** `boolean`; returns `false` when no command is active.

**Failure:** an unsupported action or wrong type raises a script error.

## `znt.input.tap(action)`

Requests a one-command press.

**Signature:** `znt.input.tap(action)`

**Valid phase:** `pre_move` or `post_move`

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `action` | `Action` | Yes | Supported named action |

**Returns:** `boolean` — whether the request was accepted for the active command.

**Failure:** returns `false` when no command is active; an unsupported action or wrong type raises a script error.

## `znt.input.hold(action)`

Keeps a named action active in the current command.

**Signature:** `znt.input.hold(action)`

**Valid phase:** `pre_move` or `post_move`

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `action` | `Action` | Yes | Supported named action |

**Returns:** `boolean` — whether the request was accepted.

**Failure:** returns `false` when no command is active; an unsupported action or wrong type raises a script error.

## `znt.input.clear(action)`

Removes a named action from the current command.

**Signature:** `znt.input.clear(action)`

**Valid phase:** `pre_move` or `post_move`

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `action` | `Action` | Yes | Supported named action |

**Returns:** `boolean` — whether the request was accepted.

**Failure:** returns `false` when no command is active; an unsupported action or wrong type raises a script error.

```lua
if znt.input.held("attack") and waiting_for_charge then
    znt.input.clear("attack")
end
```

## `znt.input.parry()`

Readiness-aware shortcut for requesting the named `parry` action.

**Signature:** `znt.input.parry()`

**Arguments:** none

**Valid phase:** `pre_move` or `post_move`

**Returns:** `boolean` — `true` only when the local parry ability is ready and the active command accepted the request.

**Failure:** returns `false` when `znt.game.parry_state()` is unavailable or not ready, or when no command is active.

Use `znt.game.parry_state()` when a script also needs the live cooldown value. `znt.input.tap("parry")` remains the lower-level named-action request and does not perform this readiness check.

## Self-cast item example

Select a friendly-target item through its resolved live slot, then wait for replicated selection before holding `alt_cast`:

```lua
if item.ready and not item.selected then
    znt.input.tap("item" .. item.slot)
elseif item.ready and item.selected then
    znt.input.hold("alt_cast")
end
```

## Timing guidance

Use `pre_move` when aim and input must be applied to the same command. Use `post_move` for ordinary state-machine actions that do not depend on same-command aim.

Requesting an action does not bypass readiness, range, or game rules. Validate those conditions first.

## Related pages

- [Callbacks](callbacks-api.md)
- [Hero assistance](hero-api.md)
- [Menu](menu-api.md)
