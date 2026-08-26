---
description: Learn which SDK patterns are demonstrated by the scripts included with Zenith.
---

# Included scripts

Zenith includes focused scripts that demonstrate complete SDK workflows. Use them as references when starting a script with similar behavior.

Auto Parry, Auto Reload, and Activator are utility scripts, so their tabs stay visible for every hero. Bebop, Haze, Shiv, Victor, and Vindicta are champion scripts; they may all remain loaded, while the sidebar shows only the entry matching the current local hero.

| Script | Demonstrates | Relevant pages |
| --- | --- | --- |
| `auto_parry.lua` | Replicated melee abilities/modifiers, live local parry readiness, independently selectable heavy/light handling, directional fallbacks, live light-melee cone geometry, optional debug-sector drawing, and accepted-request debouncing | [Game data](game-api.md), [Input](input-api.md), [Drawing](drawing-api.md), [Menu](menu-api.md) |
| `auto_reload.lua` | Weapon timing and named reload input | [Game data](game-api.md), [Input](input-api.md) |
| `activator.lua` | Per-item Always/Combo rules, enemy-proximity gates, live inventory slots, selected-item confirmation, health conditions, and self-cast input | [Game data](game-api.md), [Hero assistance](hero-api.md), [Input](input-api.md), [Menu](menu-api.md) |
| `bebop_combo.lua` | Immediate/double Bomb setup, shared combo modes, projectile prediction, confirmed Hook state, live-range Bomb handoff during the reel, and objective throws | [Runtime](runtime-api.md), [Hero assistance](hero-api.md), [Input](input-api.md) |
| `haze_sleep_dagger.lua` | Hero gating, projectile aim, and same-command casting | [Hero scripting guide](hero-scripting-guide.md) |
| `shiv_serrated_knives.lua` | Serrated Knives aim assistance, travel-aware Dash/Killing Blow killsteal, and health-bar damage previews | [Hero assistance](hero-api.md), [Game data](game-api.md), [Damage](damage-api.md), [Drawing](drawing-api.md) |
| `victor.lua` | One-request-per-hold casting, live stat-scaled radius checks, replicated toggle confirmation, and projected world-radius diagnostics | [Game data](game-api.md), [Input](input-api.md), [Drawing](drawing-api.md) |
| `vindicta_snipe.lua` | Damage prediction, scope timing, input ownership, and damage preview drawing | [Damage](damage-api.md), [Drawing](drawing-api.md) |

## Auto Parry controls

The persistent **Melee types** multiselect enables Heavy, Light, or both independently. Disabling a type clears its pending replicated and sound-fallback state, so changing the selection cannot replay an old attack.

The SDK resolves remote hold-melee abilities from owned entity-list entries when the player component omits them. A change to the replicated melee cooldown-start timestamp is latched as the early light-attack signal even when interpolation makes that timestamp older than local time, allowing the script to react before the later swing sound or animation state. Heavy attacks must face the local player even at point blank. Light attacks require the local player to be inside the attacker's forward sector and no farther than 200 world units. The sample uses an effective half-angle of at least 42°, while honoring wider live `MeleeHalfAngle` values. `Ability.Melee.Impact.Player` is intentionally not used for detection because it arrives after the hit.

Enable **Debug light cone** in the Auto Parry tab to draw that exact sector for 650 ms after a detected light swing. The arc and forward rays are green when the local player is inside the angle, range, and height gates, and red when outside. The label shows the active half-angle and reach in world units. This diagnostic is disabled by default.

The sample does not emit routine detection, rejection, or parry logs, keeping the Lua Console clear for errors and explicit script diagnostics.

## Activator and Bebop combo

Activator is an item-rule host with one compact **Activator** page. Echo Shard and Slowing Hex occupy the left column; Scourge, Infuser, and Fleetfoot occupy the right. It owns one configurable combo key. Each item independently chooses **Always** or **Combo key** and can require an enemy inside its own configurable range, so no rule imports another script's shared mode.

The **Echo Shard** rule searches all four active-item slots and reads the ability binding selected when the item was purchased. A written command bit is not treated as proof that Echo activated. If the item remains ready and its bound ability remains on cooldown, the rule retries at a short bounded interval while its selected condition is active. A replicated item cooldown or restored ability readiness confirms the request. In **Combo key** mode, the rule remains consumed until the Activator key is released.

The **Slowing Hex** rule reads the item's live cast range and targeting cone. **Auto activate** requests the live item slot only when a visible enemy is inside both gates. **Auto cast** is independent: it waits for `active_item().selected` to confirm that Slowing Hex is actually held, then confirms the game-selected target with Attack while the item remains ready. Channel and cast-delay state prevent repeated confirmation once casting begins.

**Scourge** also uses a confirmed two-stage flow. After selection, it retries an `alt_cast` press edge at a bounded interval until selection or cooldown state confirms that the friendly-target effect reached the local player. **Infuser** is a no-target activation gated by a configurable local-health percentage. **Fleetfoot** is a no-target mobility activation that defaults to Combo-key use. All three resolve their current inventory slot and validate readiness from `active_item()` rather than assuming a purchase key.

Bebop publishes the `combo` mode while its own key is held and through an owned sequence. Hook readiness does not gate Sticky Bomb. The script follows its locked target while the key remains held, so entering Bomb range later still plants the first Bomb even while Hook is cooling down; the opening runs at most once per key hold. It observes the cooldown edge, allows Activator to reset Bomb, and waits for reset readiness to remain stable before requesting a second Bomb. It emits a clean released command frame first, keeps the request pending, and rejects a restoration of the original cooldown as prediction rollback. Rejected command writes are retried until a genuinely new absolute Bomb cooldown cycle confirms the cast. It proceeds into the predicted Hook as soon as Hook is available. If no Bomb reset arrives within the bounded opening window, it stops waiting for the reset. A confirmed latch still owns the pull, optional Guardian/Walker turn, available Bomb, and Uppercut follow-up after the key is released.

The cursor status distinguishes **HOOK NOT READY**, **CASTING**, **HOOK OUT**, and **PULLING**. A Hook request becomes **HOOK OUT** only after replicated readiness or cooldown confirms that the game accepted the cast; the replicated victim handle confirms a successful latch. Hook visuals are not assumed to be client entities. The sample intentionally leaves the global entity-lifecycle stream unsubscribed so unrelated world-prop churn does not fill the Lua Console.

The scripts remain independent. Selecting the same physical key in Activator and Bebop lets them cooperate through their own key state without sharing Lua globals.

## Victor

Victor locks a visible Shock target inside the selected FOV, validates the live projectile range and path, prepares predicted Regular or pSilent aim, and submits the cast only after that aim is ready in the same command. **Draw FOV** controls the screen-space FOV circle without changing target selection. The script retains its target through the ability's live `cast_delay` plus a short launch grace, including after the combo key is released, so pSilent remains authoritative when the delayed projectile is created. A submitted cast remains pending until the replicated cooldown edge confirms it; when that cooldown completes, a still-held combo key immediately reacquires and casts again without requiring a new key press. If no cooldown edge appears, a bounded confirmation timeout permits a safe retry. The optional max-upgrade health gate requires at least 15% missing local health only when the replicated upgrade level is `4`; lower ability levels always cast normally because they do not yet have the missing-health heal upgrade.

**Auto cast Ability 2** is independent of the combo key. When enabled, it casts as soon as Victor's health falls strictly below the configured percentage and the replicated Self Zap state is actually castable. The readiness gate includes Self Zap's pending-heal modifier, which can prevent a recast while its base cooldown still reads zero. A pending request waits for replicated confirmation, and a rejected request receives a bounded retry backoff. When held Shock has a valid target and owns the current command, Ability 1 takes priority and Ability 2 cannot steal that frame.

When **Auto cast Ability 3** is enabled and the key remains held, Pain Aura is enabled only when an alive enemy's full 3D distance is within the ability's live `Radius` plus the configured tolerance. Disabling the option stops both automatic activation and automatic deactivation. Radius upgrades and Tech Range modifiers come from the current `AbilitySnapshot`; the script does not contain a per-level range table.

The replicated Pain Aura modifier confirms whether the toggle is actually active. Once active, cleanup no longer depends on the combo key: after the last enemy leaves the effective radius, the script requests deactivation and waits for replicated confirmation before issuing another toggle request.

**Debug radius** projects two world-space rings around Victor. The stronger inner ring is the exact live game radius; the subtle outer ring is `R + tolerance`. The label reports both values in meters and the confirmed `ON` or `OFF` state.

## Shiv

The Shiv page keeps the held Serrated Knives assistance and adds independent **Dash killsteal** and **Killing Blow killsteal** controls. Both scan visible targets inside the configured FOV, require live range/readiness checks, and use the selected Regular or pSilent aim style. Killing Blow uses its canonical live `AbilityCastRange`, including its rank upgrade and declared range-stat scaling; internal slash radii cannot replace it.

Dash is evaluated first to conserve the ultimate when both abilities are lethal. Its prediction includes the Rage echo, regeneration between hits, and the first hit's Spirit-resistance reduction when the live Rage resource is full. Both predictions expose current/maximum Rage and include the game's active outgoing bonus exactly once through the native damage multiplier. Killing Blow additionally exposes its own dynamically upgraded full-Rage percentage, normal Spirit damage, and live execute threshold. Both request automatic cast/travel timing and skip the cast whenever any required damage factor is unavailable. A confirmed request is latched through the readiness transition so a delayed cooldown update cannot emit duplicate casts.

Killing Blow launches after a cast delay. The sample retains the selected target and reapplies pSilent aim on every command through the live delay plus a short grace period, without submitting another cast input. This keeps the leap directed at the predicted enemy position instead of the visible cursor.

Enemy health bars show the same live prediction used by the killsteal selector. A lethal, castable spell is displayed in cast-priority order; before a kill is available, the preview first prefers an in-range/usable spell and then the smallest lethal shortfall. Its label identifies Dash or Killing Blow and explains the current decision, including **KILLSTEAL**, **NOT LETHAL**, **COOLDOWN**, **OUT OF RANGE**, and **HIDDEN**. Cooldowns suppress casting without hiding the damage threshold.

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
