# Materials

Material files define render mappings controlling how sky, entities, particles, etc. are rendered. Extension `.material`, in `materials/` directory. **Merges with vanilla materials.**

## File Format

```json
{
  "version": "1.0.0",
  "material_name:inherited_material": {
    "+states": ["Blending"],
    "frontFace": { "stencilFront": {...} },
    "+defines": ["ENABLE_VERTEX_SHADER"]
  }
}
```

## Key Fields

### State Control
- `+states`: Enable render state (`"Blending"`, `"DisableDepthWrite"`, `"DisableDepthTest"`, `"DisableCulling"`, `"StencilTest"`, `"InvertCulling"`)
- `-states`: Disable render state

### Defines
`+defines` / `-defines`: `ALPHA_TEST`, `ENABLE_VERTEX_SHADER`, `FOG`, `USE_SKINNING`, `USE_EMISSIVE`

### Blending
```json
"blendSrc": "SourceAlpha",
"blendDst": "OneMinusSrcAlpha"
```

### Samplers
```json
"samplerStates": [{
  "textureIndex": 0,
  "textureFilter": "Point",
  "textureWrap": "Clamp"
}]
```

## Common Material Files

| File | Purpose |
|------|---------|
| `entity.material` | Entity rendering |
| `terrain.material` | Block rendering |
| `particles.material` | Particle rendering |
| `sky.material` | Sky |
| `ui.material` | UI |

## Custom Materials

```json
{
  "version": "1.0.0",
  "ring:entity_alphatest": {
    "+states": ["DisableDepthWrite"]
  }
}
```
Inherits `entity_alphatest` and disables depth write. Reference via short name in entity JSON's `materials`.

## Common Pitfalls
1. Transparent materials need `Blending` state
2. Material merge: same-name fields override
3. Custom defines restricted under RenderDragon
4. Material names are case-sensitive
