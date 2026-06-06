# Molang Reference

Molang is Minecraft Bedrock Edition's expression language for fast real-time value computation, powering animations, render controllers, and particles.

## Multiline Molang in JSON

**MCBE JSON files accept line breaks in Molang strings. Standard JSON parsers reject this, but the game engine handles it fine.**

Valid MCBE Molang (standard JSON would reject):
```json
"render_controllers": [
  { "controller.render.icon": "v.is_first_person
      ? {v.person_judgement = 1;}
      : {v.person_judgement = v.person_judgement;};
    return !variable.is_first_person && !variable.map_face_icon;" }
]
```

Equivalent single-line:
`v.is_first_person ? {v.person_judgement = 1;} : {v.person_judgement = v.person_judgement;};return !variable.is_first_person && !variable.map_face_icon;`

Line breaks are visual formatting only. Don't insert extra spaces that would break token boundaries. Use line breaks for readability in complex expressions.

## Basic Syntax

### Simple Expressions
`math.sin(query.anim_time * 1.23)`

### Complex Expressions
Multi-statement expressions require `;` at the end of each statement:
```
temp.moo = math.sin(query.anim_time * 1.23);
temp.baa = math.cos(query.life_time + 2.0);
return temp.moo * temp.moo + temp.baa;
```
Without `return`, the result is 0.0.

## Case Sensitivity & Aliases

Everything except strings is case-insensitive. Aliases: `q.`=`query.`, `v.`=`variable.`, `t.`=`temp.`, `c.`=`context.`

## Variables

Three lifetimes:
- `temp.` — read/write, scoped (but effectively global for performance)
- `variable.` — read/write, stored on entity, lost on world exit/reload
- `context.` — read-only, engine-defined

Public variables:
```json
"variables": { "variable.position_self_x": "public" },
"initialize": [ "variable.position_self_x = 0;" ]
```

## Values

All numbers are floats. Booleans: false=0.0, true=1.0. Errors return 0.0. Strings use single quotes: `'minecraft:pig'`, support `==` and `!=` only.

## Operators

| Operator | Description |
|----------|-------------|
| `!` | Logical NOT |
| `&&` | Logical AND |
| `||` | Logical OR |
| `<` `<=` `>=` `>` | Comparison |
| `==` `!=` | Equality |
| `?:` | Ternary conditional |
| `?` | Binary conditional (true→right value, false→0) |
| `??` | Null coalescing |

Precedence (high to low): `!` → `* /` → `+ -` → `< <= > >=` → `== !=` → `&&` → `||` → `?:`(right-to-left) → `??`

## Control Flow

- `loop(count, expr)`: max 1024 iterations
- `for_each(var, array, expr)`: iterate entity arrays
- `->` pointer: `v.pig->query.is_baby` access other entities
- `break` / `continue`: loop control
- `??`: `variable.x = (variable.x ?? 1.2) + 0.3;`
- `{ }` block scope: `(cond) ? { stmt1; stmt2; }`

### Complex condition with side effects
```
v.is_first_person ? {v.flag = 1;} : {v.flag = v.flag;};
return !variable.is_first_person && !variable.map_face_icon;
```

## Math Functions

| Function | Description |
|----------|-------------|
| `math.abs(v)` | Absolute value |
| `math.acos(v)` `math.asin(v)` `math.atan(v)` | Inverse trig |
| `math.atan2(y,x)` | atan2 |
| `math.ceil(v)` `math.floor(v)` `math.round(v)` `math.trunc(v)` | Rounding |
| `math.clamp(v,min,max)` | Clamp to range |
| `math.cos(v)` `math.sin(v)` | Trig (degrees) |
| `math.die_roll(n,low,high)` | Sum of random floats |
| `math.die_roll_integer(n,low,high)` | Sum of random ints |
| `math.exp(v)` | e^v |
| `math.lerp(s,e,t)` | Linear interp (t:0~1) |
| `math.lerprotate(s,e,t)` | Angle interp (shortest path) |
| `math.ln(v)` | Natural log |
| `math.max(a,b)` `math.min(a,b)` | Min/max |
| `math.min_angel(v)` | Normalize angle to [-180,180) |
| `math.mod(v,den)` | Modulo |
| `math.pi` | Pi constant |
| `math.pow(base,exp)` | Power |
| `math.random(low,high)` | Random (exclusive low) |
| `math.random_integer(low,high)` | Random int (inclusive) |
| `math.sqrt(v)` | Square root |

## Query Functions (Common)

### Time
`query.anim_time`, `query.delta_time`, `query.frame_alpha`, `query.life_time`

### Entity State
`query.is_baby`, `query.is_alive`, `query.is_in_water`, `query.is_on_ground`, `query.is_moving`, `query.is_sleeping`, `query.is_sheared`, `query.is_tamed`, `query.is_angry`, `query.is_charged`, `query.is_sprinting`, `query.is_swimming`, `query.is_gliding`, `query.is_eating`, `query.is_casting`, `query.is_invisible`, `query.is_ignited`, `query.is_first_person`, `query.is_sneaking`, `query.is_in_ui`

### Entity Values
`query.health`, `query.max_health`, `query.hurt_time`, `query.hurt_direction`, `query.ground_speed`

### Rotation & Direction
`query.body_x_rotation`, `query.body_y_rotation`, `query.head_x_rotation`, `query.head_y_rotation`, `query.cardinal_facing`, `query.cardinal_facing_2d`, `query.block_face`, `query.camera_rotation(0)` (pitch), `query.camera_rotation(1)` (yaw)

### Equipment
`query.is_item_equipped`, `query.get_equipped_item_name`, `query.armor_color_slot`, `query.armor_material_slot`

### Proximity
`query.get_nearby_entities`, `query.distance_from_camera`, `query.has_rider`, `query.has_target`

### Model
`query.bone_origin`, `query.bone_rotation`, `query.get_default_bone_pivot`, `query.get_locator_offset`

### UI / Render State
`query.is_in_ui`, `variable.is_first_person`, `variable.map_face_icon`

## Built-in Variables

### Particle Variables
`variable.particle_age`, `variable.particle_lifetime`, `variable.particle_random_1~4`, `variable.emitter_age`, `variable.emitter_lifetime`, `variable.emitter_random_1~4`, `variable.emitter_texture_coordinate.u/v`, `variable.emitter_texture_size.u/v`, `variable.entity_scale`, `variable.color.r/g/b/a`

## Usage Contexts

### client_entity.json pre_animation
```json
"scripts": {
  "pre_animation": [
    "variable.ZRot = !query.is_in_water ? Math.cos((query.time_stamp + global.frame_alpha) * 14.32) * 90 : 0.0;"
  ]
}
```

### Render Controller Molang (multi-statement, multiline)
```json
"render_controllers": [
  { "controller.render.icon": "(1 - q.health/q.max_health) > v.model_scale
      ? (v.model_scale = v.model_scale + (2 * math.abs(v.model_scale - 1 + q.health/q.max_health)) * 0.002;)
      : ((1 - q.health/q.max_health) < v.model_scale
        ? (v.model_scale = v.model_scale - (2 * math.abs(v.model_scale - 1 + q.health/q.max_health)) * 0.002;)
        : (v.model_scale = v.model_scale;);)
    ;return !query.is_in_ui && !variable.is_first_person;" }
]
```

### Render Controller Molang
- Array indexing: `"geometry": "array.my_geometries[query.anim_time]"`
- Conditional: `"geometry": "query.is_sheared ? geometry.sheared : geometry.woolly"`
- Material wildcards: `{ "*Saddle*": "v.is_leather ? mat.leather : mat.iron" }`

## Versioned Changes
| version | Impact |
|---------|--------|
| 1.17.30 | Fix query.item_remaining_use_duration |
| 1.17.40 | Add error reporting for invalid expressions |
| 1.18.10 | Nested ternary right-to-left |
| 1.18.20 | && before ||; comparison before equality |

## Common Pitfalls
1. Uninitialized variables → use `??`
2. Case-insensitive (except strings), but query names must be exact
3. Avoid deep structs and excessive loop nesting
4. Every statement in complex expressions must end with `;`
5. Errors silently return 0.0 — use content log for debugging
6. min_engine_version affects operator behavior
7. Molang strings in JSON can span multiple lines (engine accepts, standard JSON doesn't)
