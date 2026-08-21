---
description: Project world positions and compose overlays from typed text, line, circle, rectangle, and specialized drawing calls.
---

# Drawing

The `znt.draw` namespace provides reusable screen queries and drawing primitives. Screen coordinates are pixels from the top-left corner: X increases right and Y increases down.

{% hint style="warning" %}
Drawing submissions and `znt.draw.text_size()` are valid only during `render`. Calling them from another phase raises a script error. `screen_size()`, `screen_center()`, and `world_to_screen()` may be used from any callback.
{% endhint %}

RGBA arguments are numbers clamped from `0` through `255`. General primitives default to opaque white. To pass a later optional positional argument, provide all preceding color channels.

## Screen queries

### `znt.draw.screen_size()`

**Signature:** `znt.draw.screen_size()`

**Arguments:** none

**Valid phase:** top-level code or any callback

**Returns:** `ScreenSize | nil` with integer `width` and `height` fields.

**Failure:** returns `nil` before renderer dimensions are available.

### `znt.draw.screen_center()`

**Signature:** `znt.draw.screen_center()`

**Arguments:** none

**Valid phase:** top-level code or any callback

**Returns:** `ScreenPoint | nil` with numeric `x` and `y` fields.

**Failure:** returns `nil` before renderer dimensions are available.

### `znt.draw.world_to_screen(position)`

**Signature:** `znt.draw.world_to_screen(position)`

**Valid phase:** top-level code or any callback

| Argument | Type | Required | Description |
| --- | --- | --- | --- |
| `position` | `Vector3` | Yes | World position with numeric `x`, `y`, and `z` fields |

**Returns:** `ScreenPoint | nil`.

**Failure:** returns `nil` when the point is behind the camera, off screen, or cannot be projected. An invalid vector raises a script error.

```lua
local player = znt.game.local_player()
local screen = player and znt.draw.world_to_screen(player.origin)
if screen then
    -- screen.x and screen.y can be used by primitives during render.
end
```

## Text measurement

### `znt.draw.text_size(text, font_size)`

**Signature:** `znt.draw.text_size(text, font_size)`

**Valid phase:** `render` only

| Argument | Type | Required | Accepted values |
| --- | --- | --- | --- |
| `text` | `string` | Yes | Non-empty, up to 512 bytes |
| `font_size` | `number` | Yes | `6.0` through `96.0` pixels |

**Returns:** `TextSize | nil` with numeric `width` and `height` fields.

**Failure:** returns `nil` while the requested font is unavailable. Invalid arguments or phase raise a script error.

## Text

### `znt.draw.text(...)`

**Signature:** `znt.draw.text(x, y, text, font_size, red, green, blue, alpha, centered, shadow)`

**Valid phase:** `render` only

| Argument | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `x` | `number` | Yes | — | Screen X position |
| `y` | `number` | Yes | — | Screen Y position |
| `text` | `string` | Yes | — | Non-empty text, up to 512 bytes |
| `font_size` | `number` | Yes | — | `6.0` through `96.0` pixels |
| `red` | `number` | No | `255` | Red channel |
| `green` | `number` | No | `255` | Green channel |
| `blue` | `number` | No | `255` | Blue channel |
| `alpha` | `number` | No | `255` | Alpha channel |
| `centered` | `boolean` | No | `false` | Treat `(x, y)` as the text center |
| `shadow` | `boolean` | No | `false` | Draw a dark offset shadow |

**Returns:** nothing.

**Failure:** invalid coordinates, text, size, optional types, or phase raise a script error. A temporarily unavailable renderer or font produces no drawing.

```lua
znt.events.on("render", function()
    local center = znt.draw.screen_center()
    if center then
        znt.draw.text(center.x, center.y - 50, "READY", 16, 126, 190, 255, 255, true, true)
    end
end)
```

## Lines

### `znt.draw.line(...)`

**Signature:** `znt.draw.line(x1, y1, x2, y2, red, green, blue, alpha, thickness)`

**Valid phase:** `render` only

| Argument | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `x1` | `number` | Yes | — | Start X |
| `y1` | `number` | Yes | — | Start Y |
| `x2` | `number` | Yes | — | End X |
| `y2` | `number` | Yes | — | End Y |
| `red` | `number` | No | `255` | Red channel |
| `green` | `number` | No | `255` | Green channel |
| `blue` | `number` | No | `255` | Blue channel |
| `alpha` | `number` | No | `255` | Alpha channel |
| `thickness` | `number` | No | `1.0` | `0.5` through `64.0` pixels |

**Returns:** nothing.

**Failure:** invalid coordinates, thickness, color types, or phase raise a script error.

## Circles

### `znt.draw.circle(...)`

**Signature:** `znt.draw.circle(x, y, radius, red, green, blue, alpha, thickness)`

**Valid phase:** `render` only

| Argument | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `x` | `number` | Yes | — | Center X |
| `y` | `number` | Yes | — | Center Y |
| `radius` | `number` | Yes | — | Positive radius in pixels |
| `red` | `number` | No | `255` | Red channel |
| `green` | `number` | No | `255` | Green channel |
| `blue` | `number` | No | `255` | Blue channel |
| `alpha` | `number` | No | `255` | Alpha channel |
| `thickness` | `number` | No | `1.0` | `0.5` through `64.0` pixels |

**Returns:** nothing.

**Failure:** invalid geometry, optional types, or phase raise a script error.

### `znt.draw.circle_filled(...)`

**Signature:** `znt.draw.circle_filled(x, y, radius, red, green, blue, alpha)`

**Valid phase:** `render` only

| Argument | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `x` | `number` | Yes | — | Center X |
| `y` | `number` | Yes | — | Center Y |
| `radius` | `number` | Yes | — | Positive radius in pixels |
| `red` | `number` | No | `255` | Red channel |
| `green` | `number` | No | `255` | Green channel |
| `blue` | `number` | No | `255` | Blue channel |
| `alpha` | `number` | No | `255` | Alpha channel |

**Returns:** nothing.

**Failure:** invalid geometry, color types, or phase raise a script error.

## Rectangles

Rectangle X and Y specify the top-left corner.

### `znt.draw.rect(...)`

**Signature:** `znt.draw.rect(x, y, width, height, red, green, blue, alpha, rounding, thickness)`

**Valid phase:** `render` only

| Argument | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `x` | `number` | Yes | — | Left edge |
| `y` | `number` | Yes | — | Top edge |
| `width` | `number` | Yes | — | Positive width |
| `height` | `number` | Yes | — | Positive height |
| `red` | `number` | No | `255` | Red channel |
| `green` | `number` | No | `255` | Green channel |
| `blue` | `number` | No | `255` | Blue channel |
| `alpha` | `number` | No | `255` | Alpha channel |
| `rounding` | `number` | No | `0.0` | Non-negative corner radius; clamped to half the shortest side |
| `thickness` | `number` | No | `1.0` | `0.5` through `64.0` pixels |

**Returns:** nothing.

**Failure:** invalid geometry, optional types, or phase raise a script error.

### `znt.draw.rect_filled(...)`

**Signature:** `znt.draw.rect_filled(x, y, width, height, red, green, blue, alpha, rounding)`

**Valid phase:** `render` only

| Argument | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `x` | `number` | Yes | — | Left edge |
| `y` | `number` | Yes | — | Top edge |
| `width` | `number` | Yes | — | Positive width |
| `height` | `number` | Yes | — | Positive height |
| `red` | `number` | No | `255` | Red channel |
| `green` | `number` | No | `255` | Green channel |
| `blue` | `number` | No | `255` | Blue channel |
| `alpha` | `number` | No | `255` | Alpha channel |
| `rounding` | `number` | No | `0.0` | Non-negative corner radius; clamped to half the shortest side |

**Returns:** nothing.

**Failure:** invalid geometry, color types, rounding, or phase raise a script error.

## FOV helper

### `znt.draw.fov_circle(...)`

Draws a camera-scaled circle centered on the crosshair.

**Signature:** `znt.draw.fov_circle(fov, red, green, blue, alpha)`

**Valid phase:** `render` only

| Argument | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `fov` | `number` | Yes | — | Positive aim field of view |
| `red` | `number` | No | `130` | Red channel |
| `green` | `number` | No | `190` | Green channel |
| `blue` | `number` | No | `255` | Blue channel |
| `alpha` | `number` | No | `210` | Alpha channel |

**Returns:** nothing.

**Failure:** a non-positive FOV, invalid color type, or wrong phase raises a script error.

## Cursor status helper

### `znt.draw.cursor_status(...)`

Draws a compact status pill beside the crosshair.

**Signature:** `znt.draw.cursor_status(text, red, green, blue, alpha)`

**Valid phase:** `render` only

| Argument | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `text` | `string` | Yes | — | Non-empty label, up to 63 bytes |
| `red` | `number` | No | `126` | Red channel |
| `green` | `number` | No | `190` | Green channel |
| `blue` | `number` | No | `255` | Blue channel |
| `alpha` | `number` | No | `235` | Alpha channel |

**Returns:** nothing.

**Failure:** invalid text, color types, or phase raise a script error.

## Damage preview helper

### `znt.draw.damage_preview(...)`

Displays a completed damage prediction beside the target's ESP health bar.

**Signature:** `znt.draw.damage_preview(target_index, prediction, full_prediction, decision)`

**Valid phase:** `render` only

| Argument | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `target_index` | `integer` | Yes | — | Target player entity index |
| `prediction` | `DamagePrediction` | Yes | — | Complete current prediction |
| `full_prediction` | `DamagePrediction or nil` | No | `nil` | Complete comparison prediction, such as full charge |
| `decision` | `string or nil` | No | `nil` | Non-empty decision label up to 31 bytes |

**Returns:** nothing.

**Failure:** an incomplete or hand-built prediction, invalid target or label, or wrong phase raises a script error.

```lua
znt.events.on("render", function()
    if current_prediction then
        znt.draw.damage_preview(
            target_index,
            current_prediction,
            full_prediction,
            current_prediction.lethal and "EXECUTE" or "WAIT"
        )
    end
end)
```

## Complete composable example

```lua
-- znt:name Drawing Example

znt.events.on("render", function()
    local center = znt.draw.screen_center()
    if not center then
        return
    end

    local r, g, b = 126, 190, 255
    znt.draw.line(center.x - 28, center.y, center.x - 8, center.y, r, g, b, 230, 1.5)
    znt.draw.line(center.x + 8, center.y, center.x + 28, center.y, r, g, b, 230, 1.5)
    znt.draw.circle(center.x, center.y, 5, r, g, b, 230, 1.5)
    znt.draw.text(center.x, center.y + 24, "CENTER", 13, 255, 255, 255, 230, true, true)
end)
```

## Related pages

- [Types and conventions](types-and-conventions.md)
- [Callbacks](callbacks-api.md)
- [Damage](damage-api.md)
