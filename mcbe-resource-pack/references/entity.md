# Client Entity Definition

Client entity definition file (client_entity.json), in the resource pack `entity/` directory.

## File Merge Architecture

- **Entity files**: Overwrite vanilla. Your `entity/player.entity.json` replaces the original. Include all vanilla fields plus your custom additions.
- **Other files** (animations, render_controllers, geometry, etc.): Merge with vanilla. Only create your custom files.
- **Materials files**: Merge with vanilla. Your material definitions append to the vanilla list.

## Version Selection

MC picks the best-matching entity definition by `min_engine_version`: the highest version ≤ engine version is selected. Multiple definitions for the same identifier can coexist (e.g. 1.8.0 base + 1.10.0 with animations).

## Basic Structure

```json
{
  "format_version": "1.10.0",
  "minecraft:client_entity": {
    "description": {
      "identifier": "namespace:name",
      "min_engine_version": "1.8.0",
      "materials": { "default": "entity_material" },
      "textures": { "default": "textures/entity/name" },
      "geometry": { "default": "geometry.entity.name" },
      "animations": { "my_anim": "animation.my.name" },
      "render_controllers": [ "controller.render.example" ],
      "scripts": {
        "initialize": [ "v.my_var = 0.0;" ],
        "pre_animation": [ "..." ],
        "animate": [ "my_anim" ],
        "scale": "0.9375"
      },
      "enable_attachables": true
    }
  }
}
```

## Key Fields

### identifier
Entity identifier, format `namespace:name`. `"minecraft:player"` for player decorations.

### render_controllers
Array of render controller identifiers. Each entry can carry a Molang condition (multi-statement, multiline):
```json
"render_controllers": [
  { "controller.render.deco": "!variable.is_first_person && !variable.map_face_icon" },
  { "controller.render.esp": "!query.is_in_ui" }
]
```

### particle_effects / sound_effects
Register at entity level for use in animation controllers:
```json
"particle_effects": { "e_p": "emli:particle_1" },
"sound_effects": { "explosion": "wiki.custom_tnt.explosion" }
```

### scripts
- `initialize`: Runs once on entity load
- `pre_animation`: Runs every frame before animations
- `animate`: Auto-played animation/controller list
  - `"my_ctrl"` — run directly
  - `{ "walk": "q.modified_move_speed" }` — with Molang blend weight
- `scale` (or `scaleX`/`scaleY`/`scaleZ`): Geometry scale, supports Molang
- `variables`: Declare public variables readable by other entities
- `should_update_bones_and_effects_offscreen`: Keep updating bones/effects even when offscreen

### spawn_egg
```json
// Solid color
"spawn_egg": { "base_color": "#db7500", "overlay_color": "#242222" }
// Custom texture
"spawn_egg": { "texture": "wiki.example", "texture_index": 0 }
```

### Special flags
- `enable_attachables`: Show attachables (tools, equipment)
- `hide_armor`: Wear armor without rendering it

## Resource Pack Directory Structure

```
resource_pack/
  manifest.json          Manifest
  entity/                Client entity definitions — OVERWRITES vanilla
  models/entity/         Entity geometry (*.geo.json)
  textures/              PNG textures
  animations/            Animations — MERGES
  animation_controllers/ Animation controllers — MERGES
  render_controllers/    Render controllers — MERGES
  materials/             Materials (*.material) — MERGES
  particles/             Particle effects
```

## Naming Conventions
- Materials: `materials/` → entity JSON short name → render controller reference
- Textures: Path under `textures/`, no extension
- Geometry: `geometry.namespace.name`
- Animations: `animation.namespace.name`
- Render controllers: `controller.render.namespace.name`
- Animation controllers: `controller.animation.namespace.name`
- Particles: `namespace:name`
