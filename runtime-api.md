---
description: Inspect the SDK contract version and runtime, and write script-scoped log messages.
---

# Runtime

The root `znt` table is available while the script's top-level code is running and from every callback.

## `znt.sdk_version`

| Type | Value |
| --- | --- |
| `integer` | `10` |

This is the public SDK contract version. Test it when a script depends on a recently added API:

```lua
local REQUIRED_SDK = 10

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
