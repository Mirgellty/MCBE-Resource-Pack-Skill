# Animation

Animation files define bone transforms over time, in the `animations/` directory.

## Basic Structure

```json
{
  "format_version": "1.8.0",
  "animations": {
    "animation.name": {
      "loop": true, "animation_length": 1.0,
      "bones": { "bone": { "position": {...}, "rotation": {...}, "scale": {...} } }
    }
  }
}
```

## Bone Transforms

### position
`"position": { "0.0": [0,0,0], "0.5": [0,5,0], "1.0": [0,0,0] }`
Or Molang: `"position": [0, "query.is_baby ? 4.0 : 0.0", 0]`

### rotation
`[x, y, z]` in degrees. Or Molang: `"rotation": "math.sin(query.anim_time * 360) * 30"`

### scale
Single value or `[x, y, z]`: `"scale": 0.45` or `"scale": "v.model_scale"`

### relative_to
Rotate relative to entity instead of world:
```json
"N": {
  "relative_to": { "rotation": "entity" },
  "rotation": ["-query.camera_rotation(0)", "-query.body_y_rotation+query.camera_rotation(1)+180", 0]
}
```

## Parametric Ring/Halo

Each bone placed evenly around a circle, with a rotating parent bone:
```
"position": ["v.scale*math.cos(360/v.amount/v.angle_set*N)", 0, "v.scale2*math.sin(360/v.amount/v.angle_set*N)"]
```
Different `v.scale`/v.scale2` = ellipse.

## Dynamic Hiding
`"position": [0, "v.amount < N ? 100000 : 0", 0]`

## Hurt Shake
```json
"N": { "position": {
  "0.0": [0,0,0],
  "0.0625": ["v.shake_degree*math.cos(360/v.amount*N)", 0, "v.shake_degree*math.sin(360/v.amount*N)"],
  "0.125": [0,0,0]
}}
```

## ESP Marker
Four corner bones + `relative_to` + `query.camera_rotation` for always-facing-player markers.

## Timeline / Sound / Particle
```json
"timeline": { "1.0": ["/particle minecraft:explosion ~~~"] },
"sound_effects": { "0.5": { "effect": "step" } }
```
