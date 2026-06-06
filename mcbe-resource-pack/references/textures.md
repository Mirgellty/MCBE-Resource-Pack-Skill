# Textures

PNG images applied to model surfaces, in the `textures/` directory.

## Directory Structure

```
textures/
  entity/           Entity textures
  blocks/           Block textures
  items/            Item textures
  particle/         Particle textures
  misc/             Miscellaneous (transparent placeholders, etc.)
```

## Requirements

- Format: PNG (RGBA)
- Dimensions: powers of 2 (64x32, 64x64, 128x128, etc.)
- Paths omit extension: `"textures/entity/pig"` → `textures/entity/pig.png`

## Texture Atlas

Minecraft packs multiple textures into a single atlas. UV coordinates map to atlas positions.

### UV Coordinate System
- (0, 0) = atlas top-left
- UV value = pixel offset within atlas
- Each face independently UV-mapped

## In entity JSON

```json
"textures": {
  "default": "textures/entity/pig/pig",
  "saddled": "textures/entity/pig/pig_saddle",
  "a1": "textures/misc/(1)"
}
```

## Transparent Placeholder

Use `"textures/misc/touming"` as a fully transparent texture to hide unwanted geometry.

## Common Pitfalls
1. PNG must be RGBA format
2. Dimensions must be powers of 2
3. UV coords must not exceed atlas bounds
4. Texture file paths are case-sensitive
5. Transparent textures can serve as placeholders to hide geometry
