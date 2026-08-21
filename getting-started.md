---
description: Create, discover, load, and manage your first Zenith Lua script.
---

# Getting started

Zenith discovers `.lua` files placed directly in `C:\znt_dd`. Subdirectories are not scanned as loadable scripts.

## Create a script

Create `C:\znt_dd\hello.lua`:

```lua
-- znt:name Hello World

local enabled = true

znt.events.on("update", function()
    if not enabled then
        return
    end

    local player = znt.game.local_player()
    if player then
        -- Read the copied player snapshot here.
    end
end)

znt.menu.add_tab("Hello", function()
    enabled = znt.menu.checkbox("enabled", "Enabled", true)
    znt.menu.text(enabled and "Script active" or "Script paused")
end)

znt.log("Hello World loaded")
```

## Load it

1. Open **Scripts → Scripts**.
2. Leave and reopen the page if the file was added while that page was already open.
3. Find **Hello World** and press **Load**.
4. Open the new **Hello** tab under Scripts.

The script manager provides:

| Action | Behavior |
| --- | --- |
| **Load** | Creates a clean state and executes the script's top-level code |
| **Unload** | Removes the script state, callbacks, and custom tabs |
| **Retry** | Restarts a failed script in a clean state |

Successfully loaded filenames are remembered between sessions. A failed or manually unloaded script is removed from the saved set.

## Set the display name

Add metadata anywhere on its own line:

```lua
-- znt:name My Script
```

Without metadata, Zenith uses the filename without `.lua`. Double-click the displayed name in the manager to edit this metadata without renaming the file.

## Next steps

- Learn the SDK's strict type notation in [Types and conventions](types-and-conventions.md).
- Choose the correct phase from [Callbacks](callbacks-api.md).
- Review sandbox behavior in [Lifecycle and safety](lifecycle-and-safety.md).
- Start a hero script from the [Hero scripting guide](hero-scripting-guide.md).
