---
description: Learn which SDK patterns are demonstrated by the scripts included with Zenith.
---

# Included scripts

Zenith includes focused scripts that demonstrate complete SDK workflows. Use them as references when starting a script with similar behavior.

| Script | Demonstrates | Relevant pages |
| --- | --- | --- |
| `auto_parry.lua` | Replicated melee abilities/modifiers, live local parry readiness, independently selectable heavy/light handling, directional fallbacks, live light-melee cone geometry, optional debug-sector drawing, and accepted-request debouncing | [Game data](game-api.md), [Input](input-api.md), [Drawing](drawing-api.md), [Menu](menu-api.md) |
| `auto_reload.lua` | Weapon timing and named reload input | [Game data](game-api.md), [Input](input-api.md) |
| `activator.lua` | Item-specific activation rules, live item imbuements, confirmed cooldown handling, shared combo modes, and named item input | [Runtime](runtime-api.md), [Game data](game-api.md), [Input](input-api.md) |
| `bebop_combo.lua` | Immediate/double Bomb setup, shared combo modes, projectile prediction, Hook-entity lifecycle diagnostics, confirmed Hook state, and objective throws | [Callbacks](callbacks-api.md), [Runtime](runtime-api.md), [Hero assistance](hero-api.md), [Input](input-api.md) |
| `haze_sleep_dagger.lua` | Hero gating, projectile aim, and same-command casting | [Hero scripting guide](hero-scripting-guide.md) |
| `shiv_serrated_knives.lua` | Charge-aware projectile assistance | [Hero assistance](hero-api.md), [Game data](game-api.md) |
| `vindicta_snipe.lua` | Damage prediction, scope timing, input ownership, and damage preview drawing | [Damage](damage-api.md), [Drawing](drawing-api.md) |

## Auto Parry controls

The persistent **Melee types** multiselect enables Heavy, Light, or both independently. Disabling a type clears its pending replicated and sound-fallback state, so changing the selection cannot replay an old attack.

The SDK resolves remote hold-melee abilities from owned entity-list entries when the player component omits them, so an active light state can trigger before the later swing sound. Heavy attacks must face the local player even at point blank. Light attacks require the local player to be inside the attacker's forward sector and no farther than 200 world units. The sample uses an effective half-angle of at least 42°, while honoring wider live `MeleeHalfAngle` values. `Ability.Melee.Impact.Player` is intentionally not used for detection because it arrives after the hit.

Enable **Debug light cone** in the Auto Parry tab to draw that exact sector for 650 ms after a detected light swing. The arc and forward rays are green when the local player is inside the angle, range, and height gates, and red when outside. The label shows the active half-angle and reach in world units. This diagnostic is disabled by default.

The sample does not emit routine detection, rejection, or parry logs, keeping the Lua Console clear for errors and explicit script diagnostics.

## Activator and Bebop combo

Activator is an item-rule host rather than an Echo-only settings page. Each rule has its own labeled controls; the included **Echo Shard** rule reacts to the shared `combo` mode. It searches all four active-item slots, reads the ability binding selected when Echo Shard was purchased, and activates only while that bound ability is cooling down. There is no duplicate ability selector or combo key in Activator.

A written command bit is not treated as proof that Echo Shard activated. If the item remains ready and the bound ability remains on cooldown, the rule retries at a short bounded interval while the same combo stays active. A replicated item cooldown or restored ability readiness confirms the request. The rule then remains consumed until combo mode ends.

Bebop publishes the `combo` mode while its own key is held and through an owned sequence. Hook readiness does not gate Sticky Bomb. The script follows its locked target while the key remains held, so entering Bomb range later still plants the first Bomb even while Hook is cooling down; the opening runs at most once per key hold. It observes the cooldown edge, allows Activator to reset Bomb, and waits for reset readiness to remain stable before requesting a second Bomb. It emits a clean released command frame first, keeps the request pending, and rejects a restoration of the original cooldown as prediction rollback. Rejected command writes are retried until a genuinely new absolute Bomb cooldown cycle confirms the cast. It proceeds into the predicted Hook as soon as Hook is available. If no Bomb reset arrives within the bounded opening window, it stops waiting for the reset. A confirmed latch still owns the pull, optional Guardian/Walker turn, available Bomb, and Uppercut follow-up after the key is released.

The cursor status distinguishes **HOOK NOT READY**, **CASTING**, **HOOK OUT**, and **PULLING**. A Hook request becomes **HOOK OUT** only after replicated readiness or cooldown confirms that the game accepted the cast; the replicated victim handle confirms a successful latch. Hook visuals are not assumed to be client entities. Independently, the loaded sample records every copied `entity_created` and `entity_deleted` event. Each diagnostic includes the handle, entity index, designer name (or `<unavailable>`), and schema class (or `<unavailable>`).

The scripts remain independent. Bebop owns its customizable combo key; Activator follows the published mode and therefore needs no duplicate keybind.

## Locations

The runtime discovers loadable `.lua` files directly inside:

```text
C:\znt_dd
```

Packaged examples may also be available for reference under:

```text
C:\znt_dd\samples
```

Subdirectories are not scanned as loadable scripts. Copy a sample into `C:\znt_dd` when you want it to appear in the script manager. Give an edited copy a distinct filename and `-- znt:name` value.

## Before editing a sample

- Keep its `nil` and state checks unless replacement logic makes them unnecessary.
- Preserve callback phases when moving code.
- Give every setting a stable key unique within that script.
- Keep hero-specific callbacks gated with `znt.game.is_hero()`.
- Treat a damage status as part of the result, not optional logging.
- Reset aim state when activation, target, hero, or enabled state changes.

See [Getting started](getting-started.md) for discovery and loading instructions.
