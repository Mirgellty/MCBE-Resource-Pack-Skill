---
name: mcbe-resource-pack
description: Minecraft Bedrock Edition resource pack development. Use when creating, editing, or troubleshooting Minecraft BE resource packs including entity client definitions, Molang expressions, animations, animation controllers, render controllers, geometry/models, textures, materials, particles, player decorations, ESP markers, health-responsive effects, walk-triggered particles, parametric rings/halos, and interactive decorations. Covers all resource pack JSON formats, multiline Molang, and the complete Molang language reference.
---

# Minecraft BE Resource Pack Development

Develop resource packs for Minecraft Bedrock Edition. Covers entity definitions, animations, render controllers, geometry, textures, materials, particles, Molang, and advanced patterns from real resource packs.

## Critical Architecture Notes

- **Entity files** (`entity/*.json`): **Overwrite** vanilla. Copy the full vanilla definition and add custom fields.
- **Other files** (animations, render_controllers, geometry, materials): **Merge** with vanilla. Only create custom files.
- **Molang in JSON can span multiple lines**: MCBE engine accepts line breaks in Molang strings. Use for readability in complex expressions.

## Resource Pack Structure

```
resource_pack/
  manifest.json / pack_icon.png
  entity/            Client entity definitions (OVERWRITES vanilla)
  models/entity/     Entity geometry (*.geo.json)
  textures/          PNG textures
  animations/        Animation files (MERGES)
  animation_controllers/  Animation controllers (MERGES)
  render_controllers/     Render controllers (MERGES)
  materials/         Material files (MERGES)
  particles/         Particle effects
```

## Quick Decision Guide

| Task | File(s) | Reference |
|------|---------|-----------|
| New/adorned entity appearance | `entity/<name>.json` | entity.md |
| Player decoration (cape/shoulder) | entity + geometry + render_controller | entity.md, geometry.md |
| Parametric ring/halo | entity + geometry + animation (25 bones) | geometry.md, animation.md |
| ESP target marker | Animation `relative_to` + 4 corners | animation.md |
| Color-cycling overlay | render_controller `overlay_color` | render_controller.md |
| Health-responsive model | Multiline Molang in rc condition | molang.md |
| Walk-triggered particles | animation_controller `particle_effects` (RP) | animation_controller.md |
| State-based animations | `animation_controllers/<name>.json` | animation_controller.md |
| Complex Molang logic | Any JSON with Molang | molang.md |

## Advanced Patterns

### Parametric Ring/Halo

Place N bones evenly around a circle, each facing the camera via `relative_to`.

**Pre-animation (configurable parameters):**
```json
"v.amount = 16;",       // number of dots
"v.angle_set = 2;",     // angle divisor
"v.scale = 25;",        // long axis (X)
"v.scale2 = 25;",       // short axis (Z) — different = ellipse
"v.height = 35;",       // Y offset
"v.rotation_speed = 100;"
```

**Animation — each bone (N from 0 to amount-1):**
```json
"N": {
  "relative_to": { "rotation": "entity" },
  "rotation": ["-query.camera_rotation(0)", "-query.body_y_rotation+query.camera_rotation(1)+180", 0],
  "position": [
    "v.scale*math.cos(360/v.amount/v.angle_set*N)",
    0,
    "v.scale2*math.sin(360/v.amount/v.angle_set*N)"
  ]
}
```

**Rotating parent bone:**
```json
"esp": { "rotation": [0, "-query.body_y_rotation+q.life_time*v.rotation_speed", 0] }
```

**Dynamic bone count via extreme offset:**
```json
"N": { "position": [0, "v.amount < N ? 100000 : 0", 0] }
```

**Per-dot textures (frame animation effect):**
Register 25 geometries, 25 textures, and 25 render controllers in entity JSON — each dot gets a unique texture.

**Interactive dot count:**
```json
"sneaking": { "on_entry": ["v.amount=v.amount+1;"] },
"attacking": { "on_entry": ["v.amount=v.amount-1;"] }
```

### Hurt Shake Ring

Radial outward shake on all ring bones:
```json
"N": { "position": {
  "0.0": [0,0,0],
  "0.0625": ["v.shake_degree*math.cos(360/v.amount*N)", 0, "v.shake_degree*math.sin(360/v.amount*N)"],
  "0.125": [0,0,0]
}}
```

### Anti-Flicker Pattern (persona_judgement)

Prevent decorations from appearing before the player leaves first person:
```json
{ "controller.render.ring": "v.is_first_person
    ? {v.persona_judgement = 1;}
    : {v.persona_judgement = v.persona_judgement;};
  return !query.is_in_ui && !variable.is_first_person && query.is_alive && v.persona_judgement;" }
```
Init: `"v.persona_judgement = 0;"`. Decoration only shows after player has entered first person at least once.

### Ring Expansion/Contraction (Range Ring)

Multi-state animation controller with `query.all_animations_finished`:
```
default -> (trigger) intorange -> ranging1 <-> ranging2 (loop) -> outofrange -> default
```

### Walk-Triggered Particles

See [animation_controller.md](references/animation_controller.md) for `particle_effects` in states.
See [particles.md](references/particles.md) for bezier curves and HSL tinting.

### Health-Responsive Model Scaling

See [molang.md](references/molang.md) for multiline Molang health lerp pattern.

### ESP Marker

See [animation.md](references/animation.md) for `relative_to` + 4 corner bones.

### Color Cycling Cape / Overlay

See [render_controller.md](references/render_controller.md) for `overlay_color` + `ignore_lighting`.

## Reference Files

- [entity.md](references/entity.md) — Client entity definition, file merge architecture, particle_effects registration
- [geometry.md](references/geometry.md) — Bones, cubes, UV, 0-thickness planes, player skeleton bones
- [textures.md](references/textures.md) — PNG format, directory structure, UV conventions
- [animation.md](references/animation.md) — Keyframes, relative_to, ESP pattern, parametric ring bones
- [animation_controller.md](references/animation_controller.md) — States, transitions, particle_effects, interactive states, RP vs BP on_entry
- [render_controller.md](references/render_controller.md) — Materials, arrays, overlay_color, texture layering, conditional rendering
- [molang.md](references/molang.md) — Full language reference, multiline Molang, query functions
- [particles.md](references/particles.md) — Emitters, bezier curves, HSL tinting, flipbook
- [materials.md](references/materials.md) — Render states, blending, defines, ignore_lighting

## Tips

1. **RP on_entry vs BP on_entry**: In RP, `on_entry`/`on_exit` accept only Molang (e.g. `v.tickets += 1;`), not commands. In BP, they accept `/command`, `@s event`, and Molang.
2. **Copy vanilla entity JSON** when targeting `minecraft:player`
3. **Multiline Molang** is engine-accepted, use for complex logic
4. **`relative_to: { rotation: "entity" }`** + `query.camera_rotation(n)` = always face camera
5. **Extreme offset hiding** (Y=100000) is the standard way to hide bones dynamically
6. **Per-bone render controllers** enable independent textures per ring dot
7. **`v.persona_judgement`** prevents first-person flicker on world join
8. **`v.amount` + interactive states** for dynamic ring dot count
9. **`v.scale` / `v.scale2`** = ellipse support for rings
10. **`query.all_animations_finished`** for sequenced multi-state transitions
11. **Bezier curves** for organic particle size, **`+-math.pi/3`** for per-particle rainbow
