# Geometry

Geometry files define 3D model structure, stored as `.geo.json` in `models/entity/`.

## Basic Structure

```json
{
  "format_version": "1.12.0",
  "minecraft:geometry": [{
    "description": {
      "identifier": "geometry.example.entity",
      "texture_width": 64, "texture_height": 32,
      "visible_bounds_width": 2, "visible_bounds_height": 3,
      "visible_bounds_offset": [0, 1, 0]
    },
    "bones": [{
      "name": "body", "pivot": [0, 12, 0],
      "cubes": [{ "origin": [-4, 12, -2], "size": [8, 4, 6], "uv": [0, 0] }]
    }]
  }]
}
```

## Key Components

### bones
- `name`: Bone name, `parent`: Parent bone, `pivot`: `[x, y, z]`
- `cubes`: Cube array (`origin`, `size`, `uv`)
- `locators`, `mirror`, `neverRender`

### Box UV vs Per-Face UV
Box UV: `"uv": [0, 0]` (array, auto-assigns per face)
Per-face UV: `"uv": { "north": {"uv":[0,0],"uv_size":[8,8]}, "south": {...}, ... }`

## Flat/Thin Geometry (size z=0)

```json
{ "origin": [-4, 12, 3], "size": [8, 8, 0] }
```
Must use per-face UV. For double-sided, use two planes with slightly different z offsets.

## Player Skeleton

| Bone | Pivot | Use |
|------|-------|-----|
| `body` | `[0, 24, 0]` | Capes, backpacks |
| `waist` | `[0, 12, 0]` | Belts |
| `head` | `[0, 24, 0]` | Hats |
| `leftArm` | `[5, 22, 0]` | Left shoulder |
| `rightArm` | `[-5, 22, 0]` | Right shoulder |

## Parametric Ring

N bones placed evenly around a circle. Each bone gets a 0-thickness child bone as the visual element. Animation controls position via `v.scale * math.cos(360/v.amount * N)` for X and `v.scale2 * math.sin(...)` for Z.

## Dynamic Bone Hiding (Extreme Offset)

"Hide" a bone by moving it very far away:
```json
"N": { "position": [0, "v.amount < N ? 100000 : 0", 0] }
```
Used for dynamic ring dot counts where `v.amount` controls how many dots are visible.

## Common Pitfalls
1. UV coords must stay within texture atlas bounds
2. Rotation order: X→Y→Z; pivot is the rotation center
3. 0-thickness cubes require per-face UV
4. Bone names must match vanilla exactly when parenting to player skeleton
