---
description: Add native tabs and typed persistent checkboxes, sliders, selectors, keybinds, and text.
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
local fov = 120.0
local activation_key = 0x45

znt.menu.add_hero_tab("Haze", "Sleep Dagger", function()
    enabled = znt.menu.checkbox("enabled", "Enabled", true)
    aim_style = znt.menu.combo("aim_style", "Aim style", 0, {"Regular", "pSilent"})
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

## Related pages

- [Getting started](getting-started.md)
- [Hero scripting guide](hero-scripting-guide.md)
- [Types and conventions](types-and-conventions.md)
