---
description: Add native tabs, icon-backed item sections, and typed persistent controls.
---

# Menu

The `znt.menu` namespace lets a script add native controls under **Scripts**. Register tabs in top-level code. Call widgets only from the callback assigned to that tab.

Widget values persist between sessions. Keys are automatically namespaced by script filename, so different scripts may use the same local key.

## `znt.menu.add_tab(name, callback)`

Adds a generic script tab.

**Signature:** `znt.menu.add_tab(name, callback)`

**Valid phase:** top-level registration is recommended

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | `string` | Yes | Non-empty tab name, up to 63 characters |
| `callback` | `function()` | Yes | Function that submits this tab's widgets |

**Returns:** `boolean` — `true` after successful registration. Registering the same name replaces that script's existing tab callback.

**Failure:** invalid arguments or exceeding tab limits raises a script error.

```lua
znt.menu.add_tab("Utilities", function()
    local enabled = znt.menu.checkbox("enabled", "Enabled", true)
    znt.menu.text(enabled and "Ready" or "Disabled")
end)
```

## `znt.menu.add_hero_tab(hero, name, callback)`

Adds a script tab associated with a hero portrait.

**Signature:** `znt.menu.add_hero_tab(hero, name, callback)`

**Valid phase:** top-level registration is recommended

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `hero` | `Hero` | Yes | Known hero ID or case-insensitive hero name |
| `name` | `string` | Yes | Non-empty tab name, up to 63 characters |
| `callback` | `function()` | Yes | Function that submits this tab's widgets |

**Returns:** `boolean` — `true` after successful registration.

**Failure:** unknown hero data, invalid arguments, or exceeding tab limits raises a script error.

{% hint style="warning" %}
A hero tab changes presentation only. It does not stop callbacks on other heroes. Guard logic with `znt.game.is_hero()`.
{% endhint %}

## `znt.menu.add_item_tab(designer_name, name, callback)`

Adds a script tab associated with an item icon.

**Signature:** `znt.menu.add_item_tab(designer_name, name, callback)`

**Valid phase:** top-level registration is recommended

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `designer_name` | `string` | Yes | Stable item identifier, up to 127 characters |
| `name` | `string` | Yes | Non-empty tab name, up to 63 characters |
| `callback` | `function()` | Yes | Function that submits this tab's widgets |

**Returns:** `boolean` — `true` after successful registration. Registering the same name replaces that script's existing tab callback.

**Failure:** invalid arguments or exceeding tab limits raises a script error. Known items use their real item-atlas icon; unsupported designer names fall back to the generic script icon.

```lua
znt.menu.add_item_tab("upgrade_containment", "Slowing Hex", function()
    local enabled = znt.menu.checkbox("enabled", "Enabled", true)
end)
```

{% hint style="warning" %}
An item tab changes presentation only. Search `znt.game.active_item(1)` through `znt.game.active_item(4)` for the live inventory slot before requesting an item action.
{% endhint %}

## `znt.menu.item_section(designer_name, label)`

Ends the current settings card and begins a separate item card inside the same tab. Its compact, non-interactive header uses the real item-atlas icon when known; unsupported designer names use the generic script icon. This keeps multi-item hosts on one sidebar page while preserving a clear visual boundary for each rule.

**Signature:** `znt.menu.item_section(designer_name, label)`

**Valid phase:** registered menu-tab callback only

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `designer_name` | `string` | Yes | Stable item identifier containing 1 through 127 characters |
| `label` | `string` | Yes | Visible section title containing 1 through 63 characters |

**Returns:** nothing.

**Failure:** an empty or oversized string, or calling outside a registered menu callback, raises a script error.

```lua
znt.menu.add_tab("Activator", function()
    local enabled = znt.menu.checkbox("enabled", "Enabled", true)

    znt.menu.item_section("upgrade_ability_power_shard", "Echo Shard")
    local echo_enabled = znt.menu.checkbox("echo_enabled", "Enabled", true)

    znt.menu.item_section("upgrade_containment", "Slowing Hex")
    local hex_enabled = znt.menu.checkbox("hex_enabled", "Enabled", true)
end)
```

Every call creates a new visually separated card for the controls that follow it. `item_section()` otherwise affects presentation and search grouping only; it does not locate, equip, activate, or validate the item.

## `znt.menu.checkbox(key, label, default)`

**Signature:** `znt.menu.checkbox(key, label, default)`

**Valid phase:** registered menu-tab callback only

| Argument | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `key` | `string` | Yes | — | Stable setting key unique within the script |
| `label` | `string` | Yes | — | Visible control label |
| `default` | `boolean` | No | `false` | Initial value before a saved value exists |

**Returns:** `boolean` — current persisted value.

**Failure:** invalid strings, an overlong composed key, or wrong phase raises a script error.

## `znt.menu.slider_float(...)`

**Signature:** `znt.menu.slider_float(key, label, default, minimum, maximum)`

**Valid phase:** registered menu-tab callback only

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `key` | `string` | Yes | Stable setting key |
| `label` | `string` | Yes | Visible control label |
| `default` | `number` | Yes | Initial value |
| `minimum` | `number` | Yes | Lowest selectable value |
| `maximum` | `number` | Yes | Highest selectable value; must be at least `minimum` |

**Returns:** `number` — current persisted value.

**Failure:** invalid numeric bounds, strings, composed key, or phase raises a script error.

## `znt.menu.slider_int(...)`

**Signature:** `znt.menu.slider_int(key, label, default, minimum, maximum)`

**Valid phase:** registered menu-tab callback only

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `key` | `string` | Yes | Stable setting key |
| `label` | `string` | Yes | Visible control label |
| `default` | `integer` | Yes | Initial integer value |
| `minimum` | `integer` | Yes | Lowest selectable integer |
| `maximum` | `integer` | Yes | Highest selectable integer; must be at least `minimum` |

**Returns:** `integer` — current persisted value.

**Failure:** invalid bounds, strings, composed key, or phase raises a script error.

## `znt.menu.combo(key, label, default, options)`

**Signature:** `znt.menu.combo(key, label, default, options)`

**Valid phase:** registered menu-tab callback only

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `key` | `string` | Yes | Stable setting key |
| `label` | `string` | Yes | Visible control label |
| `default` | `integer` | Yes | Zero-based default option index |
| `options` | `string[]` | Yes | One-based Lua array containing 1 through 16 non-empty labels; each label is at most 63 bytes |

**Returns:** `integer` — current zero-based selected index.

**Failure:** an empty or oversized option list, invalid default, invalid strings, composed key, or phase raises a script error.

```lua
local style = znt.menu.combo("aim_style", "Aim style", 0, {
    "Regular",
    "pSilent",
})
```

## `znt.menu.multiselect(key, label, default_mask, options)`

**Signature:** `znt.menu.multiselect(key, label, default_mask, options)`

**Valid phase:** registered menu-tab callback only

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `key` | `string` | Yes | Stable setting key unique within the script |
| `label` | `string` | Yes | Visible control label |
| `default_mask` | `integer` | Yes | Non-negative initial selection bitmask |
| `options` | `string[]` | Yes | One-based Lua array containing 1 through 16 non-empty labels; each label is at most 63 bytes |

**Returns:** `integer` — current persisted selection mask. Bit `0` controls `options[1]`, bit `1` controls `options[2]`, and so on.

**Failure:** an empty or oversized option list, a default mask containing a bit without a corresponding option, invalid strings, an overlong composed key, or the wrong callback phase raises a script error.

```lua
local MELEE_HEAVY = 1
local MELEE_LIGHT = 2
local selected = znt.menu.multiselect(
    "melee_types",
    "Melee types",
    MELEE_HEAVY + MELEE_LIGHT,
    {"Heavy", "Light"})

local heavy_enabled = selected % (MELEE_HEAVY * 2) >= MELEE_HEAVY
local light_enabled = selected % (MELEE_LIGHT * 2) >= MELEE_LIGHT
```

## `znt.menu.keybind(key, label, default_key)`

**Signature:** `znt.menu.keybind(key, label, default_key)`

**Valid phase:** registered menu-tab callback only

| Argument | Type | Required | Accepted values |
| --- | --- | --- | --- |
| `key` | `string` | Yes | Stable setting key |
| `label` | `string` | Yes | Visible control label |
| `default_key` | `integer` | Yes | Virtual-key code from `0` through `254` |

**Returns:** `integer` — current persisted virtual-key code.

**Failure:** an invalid key code, strings, composed key, or phase raises a script error.

## `znt.menu.text(text)`

**Signature:** `znt.menu.text(text)`

**Valid phase:** registered menu-tab callback only

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `text` | `string` | Yes | Unformatted text to display |

**Returns:** nothing.

**Failure:** a non-string or wrong phase raises a script error.

## Complete tab example

```lua
local enabled = true
local aim_style = 0
local target_types = 3
local fov = 120.0
local activation_key = 0x45

znt.menu.add_hero_tab("Haze", "Sleep Dagger", function()
    enabled = znt.menu.checkbox("enabled", "Enabled", true)
    aim_style = znt.menu.combo("aim_style", "Aim style", 0, {"Regular", "pSilent"})
    target_types = znt.menu.multiselect("target_types", "Targets", 3, {"Players", "Souls"})
    fov = znt.menu.slider_float("fov", "FOV", 120.0, 20.0, 360.0)
    activation_key = znt.menu.keybind("key", "Activation key", 0x45)
end)
```

## Limits

| Limit | Value |
| --- | --- |
| Custom tabs per script | 4 |
| Custom script tabs in the menu | 31 |
| Combo options | 16 |
| Multiselect options | 16 |

## Related pages

- [Getting started](getting-started.md)
- [Hero scripting guide](hero-scripting-guide.md)
- [Types and conventions](types-and-conventions.md)
