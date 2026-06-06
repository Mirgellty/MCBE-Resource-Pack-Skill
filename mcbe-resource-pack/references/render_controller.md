# Render Controller

Render controllers take short names from entity JSON and decide how materials, textures, and geometry combine to render the entity. Located in `render_controllers/`.

## Short Name System

Render controllers reference short names defined in entity JSON:
```json
"materials": { "default": "entity_material" },
"textures": { "default": "textures/entity/name" },
"geometry": { "default": "geometry.entity.name" }
```
RCs only know short names, not actual resource paths — so **the same RC can be reused across entities**.

## Basic Structure

```json
{
  "format_version": "1.10.0",
  "render_controllers": {
    "controller.render.name": {
      "geometry": "Geometry.default",
      "materials": [{ "*": "Material.default" }],
      "textures": ["Texture.default"]
    }
  }
}
```

## Texture Layering

`textures` is an array — textures stack top-to-bottom. Opaque lower layers occlude upper ones:
```json
"textures": [
  "Texture.bottom_layer",
  "Texture.top_layer"
]
```
Classic use: fixed frame + variable image (villager biome base + profession overlay).

Dynamic variant layering with arrays:
```json
"arrays": {
  "textures": { "Array.top": ["Texture.top_1", "Texture.top_2", "Texture.top_3"] }
},
"textures": ["Texture.bottom", "Array.top[q.variant]"]
```

## Key Fields

### geometry
Single geometry identifier. **Only one geometry** (array indexing supported, but no layering).

### materials
Material mapping array. Key matches bone names (`*` wildcard, **case-sensitive**):
```json
"materials": [
  { "*": "Material.default" },
  { "TailA": "Array.hair_colors[v.hair_color]" },
  { "*Saddle*": "v.is_leather ? Material.leather : Material.iron" }
]
```
Applied in order; later entries override earlier ones.

### textures
Texture short name array, supports layering and dynamic Molang indexing.

## Arrays

Support textures, materials, and geometries:
```json
"arrays": {
  "textures": { "Array.skins": ["Texture.skin_0", "Texture.skin_1"] },
  "geometries": { "Array.geos": ["Geometry.normal", "Geometry.baby"] }
}
```
Molang indexing with auto-modulo. Pair with `minecraft:variant` component (BP side) for dynamic switching.

## Color & Lighting

### overlay_color
Multiplies with base texture. Pair with `ignore_lighting` to skip lighting:
```json
"overlay_color": {
  "r": "math.lerp(0, 1, math.cos(query.life_time * 90))",
  "g": "math.lerp(0, 1, -math.cos(query.life_time * 90))",
  "b": "math.lerp(0, 1, math.sin(query.life_time * 90))",
  "a": "0.4"
}
```

### is_hurt_color / on_fire_color
Hurt and fire color overrides. alpha=0 explicitly disables them.

### color
Full color replacement with Molang:
```json
"color": { "r": "query.is_sheared ? 0.0 : 1.0", "g": "0.0", "b": "0.0", "a": "1.0" }
```

### light_color_multiplier
Multiplies light values on geometry. Use to darken entities.

### ignore_lighting
Skip all lighting calculations. Ideal for fullbright decorative elements.

## UV Animation

Scroll textures:
```json
"uv_anim": {
  "offset": [0, "math.mod(query.life_time * 16, 256)"],
  "scale": [1, 1]
}
```

## part_visibility

Conditionally show/hide bones:
```json
"part_visibility": [
  { "*": true },
  { "head_gear": "query.is_wearing_helmet" }
]
```

## Conditional Rendering

Attach Molang conditions to each RC in entity JSON:
```json
"render_controllers": [
  { "controller.render.deco": "!variable.is_first_person && !variable.map_face_icon" },
  { "controller.render.esp": "!query.is_in_ui" }
]
```

Common conditions: `!variable.is_first_person`, `!variable.map_face_icon`, `!query.is_in_ui`, `query.is_alive`

## Defensive Transparent Overrides

Explicitly disable color overrides:
```json
"is_hurt_color": { "r": "1", "g": "1", "b": "1", "a": "0" },
"overlay_color": { "r": "1", "g": "1", "b": "1", "a": "0" }
```

## Common Pitfalls
1. Material mapping order matters — later overrides earlier
2. `*` wildcard is case-sensitive
3. `textures` can have multiple (layering), `geometry` only one
4. Array indexing uses modulo, negative → 0
5. Decorative RCs must gate with first-person/map/UI conditions
6. `ignore_lighting` skips all lighting — not suitable for shadow-needing models
7. Short names must match entity JSON definitions exactly
8. Same-named RCs from different packs merge together
