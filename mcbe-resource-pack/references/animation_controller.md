# Animation Controller

Animation controllers are state machines usable in both Resource Packs (RP) and Behavior Packs (BP). Located in `animation_controllers/`.

> **RP Animation Controllers**: Play bone animations, can also trigger sounds and particles.
> **BP Animation Controllers**: Execute commands, send entity events, and run Molang.

## RP vs BP Key Differences

| Feature | RP (Resource Pack) | BP (Behavior Pack) |
|---------|-------------------|-------------------|
| Play animations | Yes | Yes (via "animation" commands) |
| `on_entry` / `on_exit` | Molang statements only | Commands + Molang + entity events |
| `sound_effects` | Yes (pre-register required) | — |
| `particle_effects` | Yes (pre-register required) | — |

### on_entry / on_exit in RP

**Available in RP, but only Molang statements — no slash commands.** Example:
```json
"on_entry": [
  "v.tickets += 1;",
  "v.some_flag = 1;"
]
```

### on_entry / on_exit in BP

BP supports three content types:
- **Slash commands**: `"/say Hello!"`
- **Entity events**: `"@s wiki:transform_into_plane"`
- **Molang expressions**: `"v.tickets += 1;"`

## Basic Structure

```json
{
  "format_version": "1.10.0",
  "animation_controllers": {
    "controller.animation.name": {
      "initial_state": "default",
      "states": {
        "default": {
          "animations": ["idle"],
          "transitions": [{ "running": "query.is_moving" }]
        },
        "running": {
          "animations": ["run"],
          "transitions": [{ "default": "!query.is_moving" }]
        }
      }
    }
  }
}
```

## State Properties

Each state can contain:
- `animations` — Animation list to play
- `transitions` — State transition conditions (checked in order)
- `on_entry` — Runs on entering state (RP: Molang only, BP: commands+Molang+events)
- `on_exit` — Runs on exiting state (same RP/BP rules)
- `sound_effects` — Trigger sounds (must be pre-registered in entity JSON)
- `particle_effects` — Trigger particles (must be pre-registered in entity JSON)
- `variables` — Variable remapping (1.17.30+)
- `blend_transition` — Transition blend time in seconds

### Attaching to Entity

In entity JSON:
```json
"description": {
  "animations": { "my_ctrl": "controller.animation.my.name" },
  "scripts": { "animate": ["my_ctrl"] }
}
```

Conditional (only runs when Molang is truthy):
```json
"scripts": { "animate": [{ "my_ctrl": "q.has_rider" }] }
```

### Lifecycle

On entity load: enters `initial_state` (defaults to `"default"`). Resets on entity reload (player respawn, chunk reload).

Per tick: 1) play current state animations, 2) check transitions, execute first valid one.

## Sounds & Particles (RP)

Pre-register in entity JSON `description`:
```json
"sound_effects": { "explosion": "wiki.custom_tnt.explosion" },
"particle_effects": { "fuse_lit": "wiki:tnt_fuse_lit_particle" }
```

Trigger in controller state:
```json
"explode_state": {
  "sound_effects": [{ "effect": "explosion" }],
  "particle_effects": [{ "effect": "fuse_lit" }]
}
```
Particles can specify a locator: `{ "effect": "fuse_lit", "locator": "bone_name" }`

## Animation List

```json
"animations": [
  "idle",                                    // Direct play, weight 1.0
  { "walk": "variable.walk_blend" },         // key=anim name, value=Molang blend weight
  { "sprint": "query.is_sprinting ? 1.0 : 0.0" }
]
```

## State Transitions

```json
"transitions": [
  { "next_state": "query.is_moving" },
  { "other_state": "query.health < 5" },
  { "default": "(1.0)" }                    // Fallback — prevents deadlock
]
```
Checked in order, first truthy triggers the transition.

## Advanced Features

### Variable Remapping (1.17.30+)

Map query values to custom curves:
```json
"variables": {
  "ground_speed_curve": {
    "input": "q.ground_speed",
    "remap_curve": { "0.0": 0.2, "1.0": 0.7 }
  }
},
"animations": [{ "walk": "v.ground_speed_curve" }]
```
Vanilla sheep uses this to map speed to walk animation weight.

### Multi-State Sequenced Animation

`query.all_animations_finished` waits for animations to complete:
```json
"ranging1": {
  "animations": ["ranging1", "ring_basic"],
  "transitions": [{ "ranging2": "query.all_animations_finished" }]
}
```

### Particle Trigger State Machine

Particles fire on state entry; immediate transition back enables conditional re-triggering:
```json
"default": {
  "transitions": [{ "emit": "condition && Math.random(0, 1.0) > threshold" }]
},
"emit": {
  "on_entry": ["1;"],
  "particle_effects": [{ "effect": "my_particle" }],
  "transitions": [{ "default": "1" }]
}
```

### Interactive States

```json
"sneaking": { "on_entry": ["v.amount = v.amount + 1;"] },
"attacking": { "on_entry": ["v.amount = v.amount - 1;"] }
```

## Common Pitfalls
1. Transitions checked in order — mind priority
2. Always include a fallback `"default": "(1.0)"` to prevent deadlock
3. Animation names reference entity JSON keys, not full identifiers
4. Sounds/particles require pre-registration in entity JSON
5. RP `on_entry` only accepts Molang, not `/command`
6. BP `on_entry` accepts commands, entity events, and Molang
