---
description: Register typed update, command, entity-lifecycle, sound, and rendering callbacks.
---

# Callbacks

Callbacks let a script react to Zenith and game events without creating a loop or thread.

## `znt.events.on(name, callback)`

Registers one callback for the current script. Registering the same event again replaces that script's previous callback for the event.

**Signature:** `znt.events.on(name, callback)`

**Valid phase:** top-level registration is recommended

| Argument | Type | Required | Accepted values |
| --- | --- | --- | --- |
| `name` | `string` | Yes | `"update"`, `"pre_move"`, `"post_move"`, `"sound"`, `"entity_created"`, `"entity_deleted"`, or `"render"` |
| `callback` | `function` | Yes | Function matching the event signature below |

**Returns:** `boolean` — always `true` after successful registration.

**Failure:** an unknown event, empty event name, or non-function callback raises a script error.

```lua
znt.events.on("update", function()
    local player = znt.game.local_player()
    if player then
        -- Read the latest copied snapshot.
    end
end)
```

## Event signatures

| Event | Callback type | Use it for |
| --- | --- | --- |
| `update` | `function()` | General state updates and read-only logic |
| `pre_move` | `function()` | Aim, look, and input that must share the current command |
| `post_move` | `function()` | Ordinary input and state-machine follow-ups |
| `sound` | `function(event: SoundEvent)` | Reactions to queued game sounds before the next command is built |
| `entity_created` | `function(event: EntityLifecycleEvent)` | Observe a copied snapshot captured at the start of the engine's add notification |
| `entity_deleted` | `function(event: EntityLifecycleEvent)` | Observe a copied snapshot captured before an entity is removed |
| `render` | `function()` | Text, geometry, and specialized overlays |

### `SoundEvent`

| Field | Type | Description |
| --- | --- | --- |
| `name` | `string` | Sound-event name |
| `entity_index` | `integer` | Source entity index |
| `time_ms` | `integer` | Monotonic event timestamp in milliseconds |

```lua
znt.events.on("sound", function(event)
    if event.name == "Player.Melee.Hold.Shared" then
        znt.log("Heavy melee from entity " .. tostring(event.entity_index))
    end
end)
```

Sound messages are captured before the game applies them. Zenith drains the queue immediately before the next `pre_move` callback, while that command's input-mutation window is active. A `sound` callback may therefore call `znt.input.tap()`, `hold()`, `clear()`, or `parry()` directly, or record state that the immediately following `pre_move` callback consumes; both approaches affect the same command. Network transit time still applies—the callback cannot run before the remote event reaches the client.

The sound queue holds 64 events. When full, the oldest queued event is discarded.

### `EntityLifecycleEvent`

Both entity-lifecycle callbacks receive the same copied table:

| Field | Type | Description |
| --- | --- | --- |
| `entity_index` | `integer` | Entity-list index; the engine may reuse it after deletion |
| `handle` | `integer` | Full entity handle, including its serial value; use this to pair create and delete events |
| `designer_name` | `string` | Designer identifier captured while the entity identity was valid; may be empty |
| `class_name` | `string` | Schema class captured while the entity was valid; may be empty |
| `time_ms` | `integer` | Monotonic lifecycle-event timestamp in milliseconds |

```lua
local observed = {}

znt.events.on("entity_created", function(event)
    if string.find(string.lower(event.class_name), "projectile", 1, true) then
        observed[event.handle] = true
        znt.log("projectile created: " .. event.designer_name)
    end
end)

znt.events.on("entity_deleted", function(event)
    if observed[event.handle] then
        observed[event.handle] = nil
        znt.log("projectile deleted: " .. event.designer_name)
    end
end)
```

For creation, Zenith reads identity directly from the entity supplied by the engine before the remaining add-notification work runs. Deletion is captured before the identity is invalidated. Lua never runs inside either engine notification: copied events are queued and drained immediately before the next `pre_move`. The entity referenced by a delete event may therefore already be gone. Treat every field as diagnostic snapshot data and never assume `znt.game.player(event.entity_index)` still resolves. The lifecycle queue holds 128 events and discards the oldest event when full.

Loading a script does not replay entities that already exist. Only lifecycle notifications received after the native hooks are active produce callbacks.

{% hint style="info" %}
Debug builds record lifecycle callback registration plus native queue and drain stages in the diagnostic log. Messages written with `znt.log()` appear in the Lua Console. These signals distinguish a missing engine event from a registered callback that script logic filtered out.
{% endhint %}

## Phase rules

- `znt.hero.aim()` and `znt.hero.look_at()` are valid only during `pre_move`.
- `znt.input.tap()`, `hold()`, and `clear()` need an active command, normally `pre_move` or `post_move`.
- Drawing submissions and `znt.draw.text_size()` are valid only during `render`.
- Screen queries and `world_to_screen()` may be called from any callback.
- Menu widgets are valid only inside their registered menu-tab callback.

An error in a callback unloads only the affected script. Each invocation has a 100,000-instruction limit.

## Same-command aim and cast

```lua
znt.events.on("pre_move", function()
    local target = znt.hero.targets(120.0, true)[1]
    if not target then
        return
    end

    local aim = znt.hero.aim(target.index, 120.0, 4.0, 4.0, 1, true, 0)
    if aim and aim.error <= 1.0 then
        znt.input.tap("ability1")
    end
end)
```

## Related pages

- [Lifecycle and safety](lifecycle-and-safety.md)
- [Input](input-api.md)
- [Drawing](drawing-api.md)
