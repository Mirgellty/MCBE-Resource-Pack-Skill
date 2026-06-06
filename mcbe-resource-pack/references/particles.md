# Particles

Particle system for visual effects (fire, smoke, magic, etc.). Particle files (.json) in `particles/` directory.

## Basic Structure

```json
{
  "format_version": "1.10.0",
  "particle_effect": {
    "description": {
      "identifier": "namespace:particle_name",
      "basic_render_parameters": {
        "material": "particles_alpha",
        "texture": "textures/particle/particles"
      }
    },
    "curves": { ... },
    "components": { ... }
  }
}
```

## Curves

```json
"curves": {
  "variable.size": {
    "type": "bezier",
    "input": "v.particle_age",
    "horizontal_range": "v.particle_lifetime",
    "nodes": [1, 1, 0.8, 0.81]
  }
}
```
Types: `"linear"`, `"bezier"`, `"catmull_rom"`. Reference in components: `"size": ["variable.size * 0.1", "variable.size * 0.1"]`

## Emitter Components

### Expression-Controlled Lifetime
```json
"minecraft:emitter_lifetime_expression": {
  "activation_expression": 1
}
```
- `activation_expression`: activates when truthy; `1` = always on
- More flexible alternative to `_once` / `_looping`

### Emission Rate
```json
"minecraft:emitter_rate_instant": { "num_particles": 1 }
```
```json
"minecraft:emitter_rate_steady": { "spawn_rate": 50, "max_particles": 200 }
```

### Emitter Shape
`point`, `sphere`, `box`, `disc`, `entity_aabb`, `custom`

## Particle Components

### Dynamic Motion
```json
"minecraft:particle_motion_dynamic": { "linear_drag_coefficient": 7 }
```

### HSL-Style Color Tinting
```json
"minecraft:particle_appearance_tinting": {
  "color": [
    "Math.clamp(math.lerp(0,1,math.sin((variable.particle_random_1-math.pi/3)*90)+0.4), 0, 1)",
    "Math.clamp(math.lerp(0,1,math.sin((variable.particle_random_1+math.pi/3)*90)+0.4), 0, 1)",
    "Math.clamp(math.lerp(0,1,math.sin(variable.particle_random_1 * 90)+0.4), 0, 1)",
    0.8
  ]
}
```
`variable.particle_random_1` gives each particle a unique phase. `±math.pi/3` creates 120° RGB phase offset for rainbow colors.

### Billboard Appearance (Flipbook)
```json
"minecraft:particle_appearance_billboard": {
  "size": ["3*(0.05 + variable.particle_random_1*0.05)*variable.size", "..."],
  "facing_camera_mode": "rotate_xyz",
  "uv": {
    "texture_width": 64, "texture_height": 8,
    "flipbook": {
      "base_UV": [0, 0], "size_UV": [8, 8],
      "step_UV": [8, 0], "frames_per_second": 3,
      "max_frame": 7, "loop": false
    }
  }
}
```

## Triggering Particles from Animation Controllers

Register in entity JSON: `"particle_effects": { "e_p": "emli:particle_1" }`

Use in controller: `"particle_effects": [{ "effect": "e_p" }]`

With probability-based transition:
`"Math.random(0, 1.0)>(1-(q.ground_speed*q.ground_speed)/200) && query.ground_speed"`

## Common Pitfalls
1. Flipbook textures must be on the same atlas
2. `rotate_xyz` faces camera but keeps rotation; `lookat_xyz` directly faces camera
3. Bezier node count affects smoothness
4. Material: `particles_blend`=translucent, `particles_add`=additive, `particles_alpha`=alpha-test
