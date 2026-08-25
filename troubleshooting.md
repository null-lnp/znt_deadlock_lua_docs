---
description: Diagnose script discovery, runtime errors, callback timing, input, aim, drawing, damage, menu, and timer problems.
---

# Troubleshooting

## The script does not appear

- Confirm the filename ends in `.lua`.
- Place it directly in `C:\znt_dd`; subdirectories are not scanned.
- Leave and reopen the Scripts page to trigger a rescan.
- Confirm that fewer than 32 scripts are already discovered.
- Ensure the runtime can read the file and that it was fully saved.

## The script enters Error state

Hover the script name or **Retry** button for the complete message. Fix the file, then press **Retry** to create a fresh state.

A top-level or callback error unloads only the affected script. Its callbacks and custom tabs are removed until Retry succeeds.

Open the detached **Lua Console** from **Scripts → Scripts**, or use its configured shortcut, for the chronological Lua-only log. It shows script messages, runtime diagnostics, load/unload events, and the reason a failed callback was unloaded. Closing the main menu does not close the console. Debug builds also retain the entries in `C:\dev\deadlock\logs.txt`.

Common causes are:

- a wrong Lua argument type;
- an out-of-range slot, key, FOV, or option;
- calling an API from the wrong callback phase;
- indexing a snapshot without checking it for `nil`;
- exceeding the instruction limit.

## Input returns `false`

- Call `tap()`, `hold()`, or `clear()` from `pre_move` or `post_move`.
- Use `pre_move` when input depends on aim prepared in the same command.
- Use one of the supported [Action](input-api.md#supported-action-type) strings.
- Check ability or weapon readiness separately; an accepted request does not bypass game rules.

## Aim does nothing

- Call `znt.hero.aim()` and `znt.hero.look_at()` only from `pre_move`.
- Check the local hero, activation key, ability state, target, FOV, visibility, range, and projectile path.
- Pass an ability slot when projectile prediction is required; use `nil` for instant aim.
- Call `znt.hero.reset_aim()` after target loss or key release.
- Treat a `nil` result as a rejected or unavailable aim request.

## Drawing does nothing

- Submit text and geometry only from `render`.
- Call `znt.draw.text_size()` only from `render`.
- Check `screen_size()`, `screen_center()`, and `world_to_screen()` for `nil`.
- Remember that `world_to_screen()` returns `nil` for off-screen or behind-camera points.
- Provide every RGBA channel before a later optional rounding or thickness argument.

## Damage returns `nil`

Read the second return value:

```lua
local prediction, status = znt.damage.ability(target_index, 4)
if not prediction then
    znt.log("Damage unavailable: " .. status)
    return
end
```

`"unsupported_ability"` means no reviewed high-level formula exists for that ability. Other statuses identify unavailable source, target, ability, property, runtime, or formula-factor data.

Never treat an unavailable prediction as zero. Review the full [Damage](damage-api.md) contract.

## Timers behave incorrectly

Use `znt.game.time_ms()` for script timers. Use an ability, active-item, or weapon snapshot's `game_time` for its simulation timestamps. Do not subtract milliseconds from simulation seconds.

## A menu widget fails

- Call widgets only from the callback registered with `add_tab()`, `add_hero_tab()`, or `add_item_tab()`.
- Keep combo option counts from 1 through 16.
- Use a zero-based combo default within the available options.
- Keep keybind defaults from `0` through `254`.
- Use stable, non-empty widget keys.
- A script may register up to 4 tabs; Zenith supports up to 31 custom script tabs in total.

## Runtime limits

| Limit | Value |
| --- | --- |
| Discovered scripts | 32 |
| Custom tabs per script | 4 |
| Shared modes per script | 8 |
| Custom script tabs | 31 |
| Player snapshots per `players()` call | 24 |
| Queued sound events | 64 |
| Instructions per load or callback | 100,000 |
| Combo options | 16 |
| General draw text | 512 bytes |
| Cursor-status text | 63 bytes |

When the sound queue is full, the oldest event is discarded. Exceeding the instruction limit unloads the affected script.

## Reduce the problem

Start from a minimal file:

```lua
-- znt:name Diagnostic

znt.log("Top-level code loaded")

znt.events.on("update", function()
    -- Add one operation at a time.
end)
```

Restore one callback or API call at a time until the error returns. Then compare that function's argument table, phase, and failure behavior in this book.
