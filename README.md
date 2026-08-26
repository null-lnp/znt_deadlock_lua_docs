# Zenith Lua SDK

Build custom Zenith gameplay logic with safe game snapshots, callbacks, named input actions, hero assistance, damage prediction, drawing primitives, and native menu controls.

{% hint style="info" %}
This book documents Lua SDK version **19**.
{% endhint %}

Scripts run in isolated LuaJIT states and cannot access arbitrary memory, files, DLLs, engine functions, or raw command masks. A failing script is unloaded without taking down other loaded scripts.

## Start here

1. Follow [Getting started](getting-started.md) to load a script.
2. Read [Types and conventions](types-and-conventions.md) for the notation used throughout the API.
3. Review [Lifecycle and safety](lifecycle-and-safety.md) before writing gameplay logic.
4. Use [Callbacks](callbacks-api.md) to choose the correct execution phase.

## API reference

| Namespace | Page | Purpose |
| --- | --- | --- |
| `znt` | [Runtime](runtime-api.md) | SDK version, runtime metadata, and logging |
| `znt.shared` | [Runtime](runtime-api.md#shared-activity-modes) | Boolean activity modes shared safely between isolated scripts |
| `znt.events` | [Callbacks](callbacks-api.md) | Update, command, entity-lifecycle, sound, and rendering callbacks |
| `znt.game` | [Game data](game-api.md) | Hero, player, ability, active-item, weapon, visibility, and timing snapshots |
| `znt.input` | [Input](input-api.md) | Activation keys and supported named game actions |
| `znt.hero` | [Hero assistance](hero-api.md) | Targets, aim, projectile prediction, ability/item range, and hero helpers |
| `znt.damage` | [Damage](damage-api.md) | Reviewed ability prediction and typed-damage evaluation |
| `znt.draw` | [Drawing](drawing-api.md) | Screen projection, text, lines, circles, rectangles, and overlays |
| `znt.menu` | [Menu](menu-api.md) | Persistent controls, script tabs, and icon-backed item sections |

## Guides

- [Hero scripting guide](hero-scripting-guide.md) builds a complete projectile-assist script.
- [Included scripts](included-scripts.md) maps each packaged script to the APIs it demonstrates.
- [Troubleshooting](troubleshooting.md) covers discovery, callback errors, unavailable data, and runtime limits.
