---
name: agent-sprite-forge-2d-game-assets
description: Generate 2D sprite sheets, animated GIFs, transparent PNGs, layered maps, and engine-ready prototypes for Godot/Unity using Codex-driven asset pipelines.
triggers:
  - generate a 2d sprite sheet for my game
  - create animated character sprites with transparent background
  - make a layered rpg map with props
  - generate pixel art assets for godot
  - create sprite animations and export to gif
  - build a 2d game asset pipeline
  - generate tileset and prop pack for unity
  - create game-ready sprite sheets with cleanup
---

# Agent Sprite Forge — 2D Game Asset Generation

> Skill by [ara.so](https://ara.so) — AI Agent Skills collection.

Agent Sprite Forge is a Codex-first 2D game asset workflow where the agent decides the plan, image generation creates the raw visuals, and deterministic scripts turn those visuals into reusable game assets. This skill enables AI agents to generate sprite sheets, layered maps, transparent PNGs, animated GIFs, and engine-ready exports for Godot and Unity.

## What It Does

- **Sprite Sheets**: Characters, monsters, props, attacks, spells, projectiles, impacts, idles, walks, and reference-guided variants
- **Layered Maps**: Ground-only bases, dressed references, prop packs, transparent props, y-sort placement, collision, zones, and previews
- **Engine Handoff**: Godot scenes, editable TileMap layers, separated props, encounter grass, collision bodies, exits, and debug players
- **Local Cleanup**: Chroma-key removal, frame extraction, alignment, transparent PNG/GIF export, prop-pack slicing, and QA metadata

## Installation

```bash
# Clone the repository
git clone https://github.com/0x0funky/agent-sprite-forge.git
cd agent-sprite-forge

# Install dependencies
pip install -r requirements.txt

# Set up environment variables
export OPENAI_API_KEY="your-api-key-here"
# or create .env file
echo "OPENAI_API_KEY=your-api-key-here" > .env
```

### Requirements

- Python 3.8+
- PIL/Pillow for image processing
- OpenAI API access (for image generation)
- Optional: Godot 4.x or Unity for engine exports

## Core Skills

### 1. Generate 2D Sprite Sheets (`$generate2dsprite`)

Use this skill when you need animated units, playable characters, monsters, props, spell bundles, or projectile/impact effects.

**Basic sprite generation:**

```python
from agent_sprite_forge import SpriteGenerator

# Initialize generator
sprite_gen = SpriteGenerator()

# Generate character sprite sheet
result = sprite_gen.generate(
    prompt="samurai warrior walking animation, 4 directions, pixel art style",
    frames=4,
    directions=["down", "left", "right", "up"],
    output_dir="./output/sprites"
)

# Result includes:
# - sprite_sheet.png (full sheet)
# - frames/ (individual PNG frames)
# - animations/ (GIF animations per direction)
# - metadata.json (frame info, dimensions, cleanup settings)
```

**Attack and spell animations:**

```python
# Generate spell cast animation
spell_result = sprite_gen.generate(
    prompt="fire mage casting fireball spell, side view, 6 frames",
    frames=6,
    animation_type="cast",
    include_projectile=True,
    output_dir="./output/spells/fireball"
)

# Generates:
# - cast_animation.gif
# - projectile_animation.gif
# - impact_animation.gif (if include_impact=True)
# - transparent PNG frames for each
```

**Reference-guided variants:**

```python
# Generate sprite based on reference image
ref_result = sprite_gen.generate_from_reference(
    reference_image="./refs/crocodile.jpg",
    prompt="crocodile playing with a stone, 8 frame loop animation",
    frames=8,
    preserve_style=True,
    output_dir="./output/croc_animation"
)
```

### 2. Generate 2D Maps (`$generate2dmap`)

Use this skill for layered RPG maps with ground, props, collision, and engine-ready exports.

**Basic layered map generation:**

```python
from agent_sprite_forge import MapGenerator

map_gen = MapGenerator()

# Generate layered map with prop extraction
map_result = map_gen.generate_layered_map(
    prompt="cyberpunk canal district, neon signs, bridges, water",
    style="hand-painted HD game map",
    include_props=True,
    output_dir="./output/maps/cyber_canal"
)

# Generates:
# - ground_base.png (clean ground layer)
# - dressed_reference.png (fully decorated reference)
# - prop_pack_3x3.png (extracted props grid)
# - props/ (individual transparent prop PNGs)
# - layered_preview.png (flattened preview)
# - metadata.json (layer info, prop positions)
```

**Godot TileMap export:**

```python
# Generate Godot-ready map project
godot_map = map_gen.generate_godot_map(
    prompt="forest village with paths, houses, trees, pond",
    tileset_size=16,  # 16x16 tiles
    map_size=(40, 30),  # tiles
    include_collision=True,
    include_zones=True,  # encounter grass, exits
    output_dir="./output/godot_maps/forest_village"
)

# Generates:
# - tileset.png (autotile-ready)
# - prop_sheet_3x3.png
# - scenes/ForestVillage.tscn (editable scene)
# - resources/tileset.tres
# - resources/prop_atlas.tres
# - scripts/debug_player.gd
# - collision and area metadata
```

### 3. Post-Processing and Cleanup

**Chroma-key background removal:**

```python
from agent_sprite_forge.processors import BackgroundRemover

remover = BackgroundRemover()

# Remove background and export transparent PNG
clean_sprite = remover.remove_background(
    input_path="./raw/sprite_sheet.png",
    chroma_color="#00FF00",  # green screen
    tolerance=30,
    output_path="./output/sprite_sheet_clean.png"
)
```

**Frame extraction and GIF export:**

```python
from agent_sprite_forge.processors import FrameExtractor, GIFExporter

# Extract frames from sprite sheet
extractor = FrameExtractor()
frames = extractor.extract_frames(
    sprite_sheet="./output/sprite_sheet_clean.png",
    frame_width=64,
    frame_height=64,
    output_dir="./output/frames"
)

# Create animated GIF
gif_exporter = GIFExporter()
gif_exporter.create_animation(
    frames_dir="./output/frames",
    output_path="./output/animation.gif",
    frame_duration=100,  # milliseconds
    loop=0  # infinite loop
)
```

**Prop pack slicing:**

```python
from agent_sprite_forge.processors import PropPackSlicer

slicer = PropPackSlicer()

# Slice 3x3 prop pack into individual props
props = slicer.slice_prop_pack(
    prop_pack="./output/maps/cyber_canal/prop_pack_3x3.png",
    grid_size=(3, 3),
    output_dir="./output/maps/cyber_canal/props",
    remove_background=True,
    chroma_color="#FFFFFF"
)

# Returns list of prop files with bounding box metadata
```

## Configuration

Create a `config.yaml` file to set defaults:

```yaml
# Image generation settings
image_generation:
  model: "dall-e-3"
  size: "1024x1024"
  quality: "hd"
  style: "natural"  # or "vivid"

# Sprite settings
sprites:
  default_frame_count: 4
  default_frame_size: [64, 64]
  background_color: "#00FF00"  # chroma key
  export_formats: ["png", "gif"]

# Map settings
maps:
  default_tile_size: 16
  prop_pack_grid: [3, 3]
  layer_order: ["ground", "props", "collision", "zones"]

# Processing
processing:
  chroma_tolerance: 30
  gif_frame_duration: 100
  png_compression: 9

# Engine exports
godot:
  version: "4.5"
  tilemap_layer_enabled: true
  include_debug_player: true

unity:
  version: "2022.3"
  sprite_pixels_per_unit: 16
  include_prefabs: true
```

Load config in Python:

```python
from agent_sprite_forge import load_config

config = load_config("config.yaml")
sprite_gen = SpriteGenerator(config=config)
```

## Common Patterns

### Full Character Asset Pipeline

```python
from agent_sprite_forge import SpriteGenerator, BackgroundRemover, FrameExtractor, GIFExporter

def create_character_assets(character_name, prompt_base):
    """Generate complete character asset set"""
    sprite_gen = SpriteGenerator()
    
    # 1. Generate directional walk animations
    walk_result = sprite_gen.generate(
        prompt=f"{prompt_base}, walking animation, 4 directions",
        frames=4,
        directions=["down", "left", "right", "up"],
        output_dir=f"./output/characters/{character_name}/walk"
    )
    
    # 2. Generate attack animation
    attack_result = sprite_gen.generate(
        prompt=f"{prompt_base}, sword attack, side view",
        frames=6,
        animation_type="attack",
        output_dir=f"./output/characters/{character_name}/attack"
    )
    
    # 3. Generate idle animation
    idle_result = sprite_gen.generate(
        prompt=f"{prompt_base}, idle breathing animation",
        frames=2,
        animation_type="idle",
        output_dir=f"./output/characters/{character_name}/idle"
    )
    
    return {
        "walk": walk_result,
        "attack": attack_result,
        "idle": idle_result
    }

# Usage
samurai_assets = create_character_assets(
    "samurai_warrior",
    "pixel art samurai in blue armor"
)
```

### RPG Map with Godot Export

```python
from agent_sprite_forge import MapGenerator

def create_rpg_zone(zone_name, description):
    """Generate RPG map zone with Godot export"""
    map_gen = MapGenerator()
    
    # Generate layered map
    map_result = map_gen.generate_godot_map(
        prompt=description,
        tileset_size=16,
        map_size=(40, 30),
        include_collision=True,
        include_zones=True,
        zone_types=["encounter_grass", "exit", "trigger"],
        output_dir=f"./output/maps/{zone_name}"
    )
    
    # Add custom metadata
    map_result.add_metadata({
        "zone_name": zone_name,
        "encounters": ["slime", "bat", "wolf"],
        "exits": [
            {"name": "north_exit", "to": "zone_02"},
            {"name": "south_exit", "to": "zone_00"}
        ]
    })
    
    return map_result

# Usage
forest_zone = create_rpg_zone(
    "forest_01",
    "dense forest with winding path, old shrine, mushroom clusters"
)
```

### Spell Bundle Generation

```python
def create_spell_bundle(spell_name, element, color_scheme):
    """Generate complete spell asset bundle"""
    sprite_gen = SpriteGenerator()
    
    spell_dir = f"./output/spells/{spell_name}"
    
    # Cast animation
    cast = sprite_gen.generate(
        prompt=f"mage casting {element} spell, {color_scheme} colors, side view",
        frames=6,
        output_dir=f"{spell_dir}/cast"
    )
    
    # Projectile
    projectile = sprite_gen.generate(
        prompt=f"{element} projectile, {color_scheme}, spinning animation",
        frames=4,
        animation_type="projectile",
        output_dir=f"{spell_dir}/projectile"
    )
    
    # Impact
    impact = sprite_gen.generate(
        prompt=f"{element} explosion impact, {color_scheme}, burst effect",
        frames=8,
        animation_type="impact",
        output_dir=f"{spell_dir}/impact"
    )
    
    return {
        "cast": cast,
        "projectile": projectile,
        "impact": impact
    }

# Usage
fireball = create_spell_bundle("fireball", "fire", "red and orange")
ice_lance = create_spell_bundle("ice_lance", "ice", "blue and white")
```

### Unity Survivors-like Setup

```python
from agent_sprite_forge import SpriteGenerator, UnityExporter

def setup_survivors_game(game_name):
    """Generate assets for survivors-like game"""
    sprite_gen = SpriteGenerator()
    unity_export = UnityExporter()
    
    base_dir = f"./output/unity_projects/{game_name}"
    
    # Hero sheets (4 directions)
    hero = sprite_gen.generate(
        prompt="anime hero character, walking animation, 4 directions",
        frames=4,
        directions=["down", "left", "right", "up"],
        output_dir=f"{base_dir}/Assets/Heroes/Hero01"
    )
    
    # Summon/minion sheets
    summons = []
    for i, summon_type in enumerate(["wolf", "eagle", "bear"]):
        summon = sprite_gen.generate(
            prompt=f"spirit {summon_type} companion, idle and attack",
            frames=6,
            output_dir=f"{base_dir}/Assets/Summons/{summon_type}"
        )
        summons.append(summon)
    
    # Enemy sheets
    enemies = []
    for enemy_type in ["slime", "skeleton", "demon"]:
        enemy = sprite_gen.generate(
            prompt=f"{enemy_type} enemy, walk and attack animation",
            frames=4,
            output_dir=f"{base_dir}/Assets/Enemies/{enemy_type}"
        )
        enemies.append(enemy)
    
    # Export Unity-ready structure
    unity_export.export_project(
        assets={
            "hero": hero,
            "summons": summons,
            "enemies": enemies
        },
        output_dir=base_dir,
        include_prefabs=True,
        include_animator_controllers=True
    )
    
    return base_dir

# Usage
game_path = setup_survivors_game("SummonSurvivors")
```

## API Reference

### SpriteGenerator

```python
class SpriteGenerator:
    def generate(
        self,
        prompt: str,
        frames: int = 4,
        directions: list = None,
        animation_type: str = "walk",
        include_projectile: bool = False,
        include_impact: bool = False,
        output_dir: str = "./output"
    ) -> SpriteResult
    
    def generate_from_reference(
        self,
        reference_image: str,
        prompt: str,
        frames: int,
        preserve_style: bool = True,
        output_dir: str = "./output"
    ) -> SpriteResult
```

### MapGenerator

```python
class MapGenerator:
    def generate_layered_map(
        self,
        prompt: str,
        style: str = "hand-painted HD game map",
        include_props: bool = True,
        output_dir: str = "./output"
    ) -> MapResult
    
    def generate_godot_map(
        self,
        prompt: str,
        tileset_size: int = 16,
        map_size: tuple = (40, 30),
        include_collision: bool = True,
        include_zones: bool = True,
        zone_types: list = None,
        output_dir: str = "./output"
    ) -> GodotMapResult
```

### BackgroundRemover

```python
class BackgroundRemover:
    def remove_background(
        self,
        input_path: str,
        chroma_color: str = "#00FF00",
        tolerance: int = 30,
        output_path: str = None
    ) -> str
```

### GIFExporter

```python
class GIFExporter:
    def create_animation(
        self,
        frames_dir: str,
        output_path: str,
        frame_duration: int = 100,
        loop: int = 0
    ) -> str
```

## Troubleshooting

### Background removal leaves artifacts

```python
# Increase tolerance for better removal
remover.remove_background(
    input_path="sprite.png",
    chroma_color="#00FF00",
    tolerance=50,  # Increase from default 30
    output_path="sprite_clean.png"
)

# Or use edge smoothing
remover.remove_background(
    input_path="sprite.png",
    chroma_color="#00FF00",
    tolerance=30,
    smooth_edges=True,
    edge_blur=2,
    output_path="sprite_clean.png"
)
```

### Generated sprites don't align properly

```python
# Use frame alignment processor
from agent_sprite_forge.processors import FrameAligner

aligner = FrameAligner()
aligned_frames = aligner.align_frames(
    frames_dir="./output/frames",
    alignment="center",  # or "bottom", "top"
    reference_point="auto",  # detect common ground point
    output_dir="./output/frames_aligned"
)
```

### Godot export scene not loading

```python
# Verify Godot version compatibility
godot_map = map_gen.generate_godot_map(
    prompt="village map",
    godot_version="4.5",  # Explicit version
    output_dir="./output/godot_maps/village"
)

# Check generated scene file
import os
assert os.path.exists("./output/godot_maps/village/scenes/Village.tscn")
```

### Image generation quality issues

```python
# Use HD quality and specific style guidance
sprite_gen = SpriteGenerator(
    image_model="dall-e-3",
    quality="hd",
    style="vivid"  # More saturated colors
)

result = sprite_gen.generate(
    prompt="detailed pixel art character, sharp edges, vibrant colors, 64x64 resolution",
    frames=4,
    output_dir="./output/sprites"
)
```

### GIF animations too large

```python
# Optimize GIF export
gif_exporter = GIFExporter()
gif_exporter.create_animation(
    frames_dir="./output/frames",
    output_path="./output/animation.gif",
    frame_duration=100,
    loop=0,
    optimize=True,  # Enable optimization
    colors=64,  # Reduce color palette (max 256)
    dither=False  # Disable dithering for smaller size
)
```

### Prop extraction misses objects

```python
# Adjust detection sensitivity
slicer = PropPackSlicer()
props = slicer.slice_prop_pack(
    prop_pack="./props.png",
    grid_size=(3, 3),
    output_dir="./output/props",
    min_object_size=10,  # Smaller minimum
    detection_threshold=128,  # Adjust alpha threshold
    remove_background=True
)
```

## Environment Variables

```bash
# Required
export OPENAI_API_KEY="sk-..."

# Optional
export AGENT_SPRITE_FORGE_OUTPUT_DIR="./my_assets"
export AGENT_SPRITE_FORGE_CACHE_DIR="./.cache"
export AGENT_SPRITE_FORGE_LOG_LEVEL="INFO"  # DEBUG, INFO, WARNING, ERROR

# Godot export
export GODOT_EXECUTABLE_PATH="/usr/local/bin/godot"

# Unity export
export UNITY_EXECUTABLE_PATH="/Applications/Unity/Hub/Editor/2022.3.0f1/Unity.app/Contents/MacOS/Unity"
```

## CLI Usage

```bash
# Generate sprite sheet
python -m agent_sprite_forge sprite \
  --prompt "knight walking animation" \
  --frames 4 \
  --directions down,left,right,up \
  --output ./output/knight

# Generate layered map
python -m agent_sprite_forge map \
  --prompt "desert oasis with palm trees" \
  --include-props \
  --output ./output/maps/oasis

# Generate Godot map
python -m agent_sprite_forge godot-map \
  --prompt "cave system with crystals" \
  --tileset-size 16 \
  --map-size 40,30 \
  --output ./output/godot/cave

# Remove background
python -m agent_sprite_forge remove-bg \
  --input ./raw/sprite.png \
  --output ./clean/sprite.png \
  --chroma "#00FF00" \
  --tolerance 30

# Create GIF
python -m agent_sprite_forge make-gif \
  --frames ./output/frames \
  --output ./animation.gif \
  --duration 100 \
  --loop 0
```
