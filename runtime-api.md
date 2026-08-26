---
description: Inspect the SDK runtime, coordinate isolated scripts, and write script-scoped log messages.
---

# Runtime

The root `znt` table is available while the script's top-level code is running and from every callback.

## `znt.sdk_version`

| Type | Value |
| --- | --- |
| `integer` | `20` |

This is the public SDK contract version. Test it when a script depends on a recently added API:

```lua
local REQUIRED_SDK = 20

if znt.sdk_version < REQUIRED_SDK then
    error("This script requires SDK version " .. REQUIRED_SDK)
end
```

Do not compare `znt.runtime` to detect API support.

## `znt.runtime`

| Type | Description |
| --- | --- |
| `string` | Human-readable name and version of the bundled Lua runtime |

This value is informational and may change without changing the SDK contract.

## Shared activity modes

Scripts keep isolated Lua states. `znt.shared` provides boolean activity modes for the narrow cases where independent scripts need to coordinate without sharing globals or arbitrary values.

A mode is active while at least one loaded script publishes it. Every published mode is owned by its calling script; unloading or failing that script removes its modes automatically.

### `znt.shared.set_mode(name, active)`

Publishes or clears a mode for the calling script.

**Signature:** `znt.shared.set_mode(name, active)`

**Valid phase:** top-level code or any callback

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | `string` | Yes | Non-empty mode name containing at most 63 bytes |
| `active` | `boolean` | Yes | `true` to publish; `false` to clear |

**Returns:** `boolean` — `true` when the mode was published or cleared.

**Failure:** invalid arguments or publishing more than eight modes from one script raises a script error.

```lua
local held = znt.input.key_down(combo_key)
znt.shared.set_mode("combo", held or combo_in_progress)
```

### `znt.shared.mode_active(name)`

Checks whether any loaded script currently publishes a mode.

**Signature:** `znt.shared.mode_active(name)`

**Valid phase:** top-level code or any callback

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | `string` | Yes | Non-empty mode name containing at most 63 bytes |

**Returns:** `boolean`.

**Failure:** an invalid argument raises a script error.

```lua
if znt.shared.mode_active("combo") then
    -- Cooperate with the active combo.
end
```

Use modes for current activity, not persistent storage. Clear a published mode when the activity ends; automatic unload cleanup is the failure-safe path.

## `znt.log(message)`

Writes one script-scoped message to the detached **Lua Console**. Debug builds also mirror it to `C:\dev\deadlock\logs.txt`.

**Signature:** `znt.log(message)`

**Valid phase:** top-level code or any callback

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `message` | `string` | Yes | Message to show in the Lua console and Debug log |

**Returns:** nothing.

**Failure:** a non-string argument raises a script error. Convert other values explicitly with `tostring()`.

```lua
znt.log("Current SDK: " .. tostring(znt.sdk_version))
```

## Related pages

- [Callbacks](callbacks-api.md)
- [Lifecycle and safety](lifecycle-and-safety.md)
- [Troubleshooting](troubleshooting.md)
