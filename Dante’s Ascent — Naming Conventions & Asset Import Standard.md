# Dante’s Ascent — Naming Conventions & Asset Import Standard

Project standard for file, scene, node and script naming, and for Godot 4 asset import defaults that protect the hand-drawn visual style.

| Field | Value |
| --- | --- |
| Document ID | DA-STD-001 |
| Version | 0.1 (Draft) |
| Date | 13 November 2026 |
| Owner / approver | Carl O’Brien |
| Applies to | The `DantesAscent` repository (Godot 4.x) |
| Status | Draft — pending approval of the items in section 9 |

## 1. Purpose and scope

Dante’s Ascent is a short vertical 2D platformer demo built as a portfolio piece. Anyone reviewing the repository (a hiring manager, a collaborator, or you in six months) should be able to open it and immediately understand where things are and what they are called. The visual identity — monumental, cold, oppressive, hand-drawn in the manner of Doré’s engravings — must also survive the trip from Procreate into the engine without being softened, crunched or distorted by default import settings.

This document defines:

- how files, folders, scenes, nodes, scripts and other project identifiers are named;
- the project structure those names live in;
- the Godot project settings and texture import defaults that keep all imported art consistent;
- the lightweight governance needed to keep the standard from decaying.

**In scope:** everything under `res://`, source-art handling, and Git conventions that affect assets. **Out of scope:** art direction itself (palette, line weight, subject matter), level design, and the itch.io release process.

## 2. Guiding principles

1. **One obvious name.** A thing has exactly one correct name, derivable from a rule rather than taste.
2. **Lowercase on disk, PascalCase in the tree.** Godot projects move between Windows, macOS, Linux and the web. Case-insensitive file systems cause bugs that appear only after export, so every file and folder name is lowercase snake\_case. Node and class names are PascalCase, matching Godot’s own conventions.
3. **Names describe what, not where.** `chain_long_01.png` ages better than `level2_thing3.png`.
4. **Scale lives in the art, not the node.** Sprites are drawn at the size they appear on screen. Rescaling nodes in the editor is how line weight drifts and the hand-drawn look becomes inconsistent.
5. **Defaults are set once, centrally.** Import and rendering settings are configured at project level so that a new asset is correct without anyone remembering to fix it.
6. **Follow the engine’s conventions where they exist.** Where this document is silent, the official GDScript style guide and Godot’s project-organization guidance apply.

## 3. Project structure

The project is organised by **feature first, asset type second**: everything needed to run an entity lives together, while shared art is grouped by category.

```
res://
├── actors/
│   ├── player/          player.tscn, player.gd, player_*.png, player_*.tres
│   └── enemies/
│       ├── enemy_base/  enemy_base.tscn, enemy_base.gd
│       └── <enemy_name>/
├── levels/              level_01.tscn, level_01.gd
├── art/
│   ├── backgrounds/     bg_*.png
│   ├── environment/     env_*.png (platforms, walls, chains, ledges)
│   ├── props/           prop_*.png
│   ├── fx/              fx_*.png
│   └── ui/              ui_*.png
├── audio/
│   ├── music/           mus_*.ogg
│   └── sfx/             sfx_*.wav / sfx_*.ogg
├── shaders/             *.gdshader (e.g. vignette_asymmetric.gdshader)
├── ui/                  hud.tscn, title_screen.tscn, scripts
├── autoloads/           game_state.gd, audio_manager.gd
├── resources/           shared .tres (themes, SpriteFrames, power-up data)
├── fonts/               *.ttf / *.otf
└── _source/             .procreate, .psd, .blend — contains an empty .gdignore file
```

Rules:

- Top-level folders are fixed. Adding one requires a change entry in section 8.4.
- Maximum folder depth under `res://` is four levels.
- `_source/` holds working files that Godot must never import. Placing an empty file named `.gdignore` in it makes the editor skip it entirely. Source files can still be committed (see section 7).
- No `misc/`, `temp/`, `new/` or `old/` folders. Delete or promote.

## 4. Naming conventions

### 4.1 General rules (all files and folders)

- Lowercase `snake_case` only: letters, digits and underscores.
- No spaces, hyphens, capitals or non-ASCII characters.
- English words, singular nouns (`platform`, not `platforms`) except for folders, which are plural collections.
- Numeric suffixes are zero-padded two digits (`_01`, `_02`) so they sort correctly.
- Names are at most 40 characters and describe content, not status. Never use `final`, `new`, `v2` or `copy` — Git holds the history.
- File extensions are lowercase.

### 4.2 Scenes (.tscn)

| Rule | Convention | Example |
| --- | --- | --- |
| File name | `snake_case.tscn` | `player.tscn` |
| Root node name | PascalCase version of the file name | `Player` |
| Base / reusable scenes | `<thing>_base` | `enemy_base.tscn` |
| Variant scenes | `<thing>_<variant>` | `enemy_<name>.tscn` |
| Levels | `level_<nn>` | `level_01.tscn` |
| UI scenes | Screen or widget name | `hud.tscn`, `title_screen.tscn` |
| Test / sandbox scenes | `test_` prefix, kept in `res://levels/_sandbox/` and excluded from export | `test_player_movement.tscn` |

A scene lives in the same folder as its script and its scene-specific art. A scene should do one job; if a scene’s node tree needs more than about 30 nodes, split it into sub-scenes.

### 4.3 Nodes

- **PascalCase**, no spaces, no numbers as the only distinguisher.
- Name by **role**, not by node type: `Sprite` is acceptable for the main visual, but `HurtboxShape` is better than `CollisionShape2D2`.
- Always rename nodes that Godot auto-numbers (`Sprite2D2`, `CollisionShape2D3`).
- Standard role names, used consistently across every scene:

| Role | Node name | Typical type |
| --- | --- | --- |
| Main visual | `Sprite` or `AnimatedSprite` | Sprite2D / AnimatedSprite2D |
| Animation controller | `AnimationPlayer` | AnimationPlayer |
| Body collision | `BodyShape` | CollisionShape2D |
| Damage dealt by this actor | `Hitbox` (children: `HitboxShape`) | Area2D |
| Damage received by this actor | `Hurtbox` (children: `HurtboxShape`) | Area2D |
| Ground / wall probes | `GroundRay`, `WallRayLeft`, `WallRayRight` | RayCast2D |
| Timers | Named by purpose: `CoyoteTimer`, `AttackCooldownTimer`, `InvulnerabilityTimer` | Timer |
| Camera | `Camera` | Camera2D |
| Containers | Plural noun: `Enemies`, `Platforms`, `Pickups` | Node2D |

- Nodes the script needs to reach from anywhere in the scene are marked **Access as Unique Name** and referenced as `%NodeName`. All other references use `@onready` variables.
- Groups use `snake_case`: `enemies`, `pickups`, `hazards`, `player`.

### 4.4 Scripts (GDScript)

**Files:** `snake_case.gd`, identical to the scene it is attached to (`player.tscn` → `player.gd`). Scripts with no scene (state machines, utilities, resources) are named for what they are: `state_machine.gd`, `damage_info.gd`.

**Identifiers:**

| Element | Convention | Example |
| --- | --- | --- |
| `class_name` | PascalCase | `class_name Player` |
| Functions | snake\_case | `apply_damage()` |
| Variables | snake\_case | `move_speed` |
| Private members | Leading underscore | `_velocity_buffer`, `_update_facing()` |
| Constants | CONSTANT\_CASE | `MAX_FALL_SPEED` |
| Enum names | PascalCase | `enum State { IDLE, RUN, JUMP }` |
| Enum members | CONSTANT\_CASE | `State.IDLE` |
| Signals | snake\_case, past tense | `health_changed`, `died`, `power_up_collected` |
| Signal handlers | `_on_<NodeName>_<signal>` | `_on_Hurtbox_area_entered` |
| Booleans | `is_`, `has_`, `can_` prefix | `is_grounded`, `can_double_jump` |
| Exported tuning values | Grouped with `@export_group` | `@export_group("Movement")` |

**Required practices:**

- Static typing everywhere: `var health: int = 3`, `func jump() -> void:`.
- Script member order follows the official style guide: `class_name`, `extends`, docstring, signals, enums, constants, exports, public vars, private vars, `@onready` vars, built-in callbacks, public methods, private methods.
- One tab for indentation; maximum line length 100 characters.
- Magic numbers belong in named constants or exports. Gameplay tuning values (jump height, gravity, attack time) are `@export` variables so they can be tuned without editing code.
- Comments explain *why*, not *what*.

**Autoloads:** file `snake_case.gd`, autoload name PascalCase (`game_state.gd` → `GameState`). Limit autoloads to genuinely global services; target no more than four for this demo.

### 4.5 Art assets (.png)

Pattern: `<category>_<subject>_<descriptor>_<nn>.png`

| Category prefix | Use | Example |
| --- | --- | --- |
| `char_` | The player character | `char_dante_run_01.png` |
| `enemy_` | Enemies | `enemy_<name>_attack_03.png` |
| `env_` | Platforms, walls, ledges, hanging chains | `env_ledge_stone_02.png` |
| `prop_` | Decorative or interactive objects | `prop_torch_01.png` |
| `bg_` | Parallax and backdrop layers | `bg_cavern_far_01.png` |
| `fx_` | Particles, impact marks, glows | `fx_hit_spark_01.png` |
| `ui_` | HUD and menu graphics | `ui_health_full.png` |

- **Animation frames** are individual files numbered from `01`, in playback order, with the action as the descriptor: `char_dante_idle_01.png`, `char_dante_idle_02.png`. Spritesheets, if used, end in `_sheet`: `char_dante_run_sheet.png`.
- **Variants** are alphabetic or descriptive: `env_ledge_stone_a.png`, `env_ledge_stone_cracked.png`.
- **Layers by depth** for parallax use `_far`, `_mid`, `_near`.
- Only **PNG (32-bit, with alpha)** is accepted into `res://art/`. No JPG, no WebP, and no layered files.
- Source files in `_source/` mirror the exported name with the working extension: `char_dante_run.procreate`.

### 4.6 Audio, shaders, resources, fonts

- Audio: `mus_<name>.ogg` for music (looping tracks use `_loop` as a final suffix), `sfx_<subject>_<action>_<nn>.wav` for effects (`sfx_dante_jump_01.wav`). Use `.ogg` for anything longer than about five seconds.
- Shaders: `<effect>.gdshader`, snake\_case (`vignette_asymmetric.gdshader`). Shader uniforms are snake\_case.
- Resources: `<subject>_<kind>.tres` (`dante_sprite_frames.tres`, `power_up_double_jump.tres`).
- Fonts: `<family>_<weight>.ttf`.

### 4.7 Project-level identifiers

| Identifier | Convention | Examples |
| --- | --- | --- |
| Input actions | snake\_case, verb first | `move_left`, `move_right`, `jump`, `attack`, `pause` |
| Animation names (inside AnimationPlayer / SpriteFrames) | lowercase snake\_case, no character prefix | `idle`, `run`, `jump`, `fall`, `attack_01`, `hurt`, `death` |
| Physics layers (named in Project Settings → Layer Names) | Singular noun, snake\_case | 1 `world`, 2 `player`, 3 `enemy`, 4 `player_hitbox`, 5 `enemy_hitbox`, 6 `pickup`, 7 `hazard` |
| Audio buses | PascalCase | `Master`, `Music`, `Sfx` |
| Export presets | `<platform>_<kind>` | `windows_release`, `web_debug` |

## 5. How the naming rules are checked

A rule nobody checks stops being a rule. Three levels of enforcement, in order of cost:

1. **Pre-merge self-review** using the checklist in section 8.2.
2. **Automated linting.** Run `gdlint` and `gdformat --check` from gdtoolkit on all `.gd` files, via a pre-commit hook locally and optionally a GitHub Actions workflow.
3. **File-name check.** A small script (or CI step) that fails if any path under `res://` contains uppercase letters, spaces or characters outside `[a-z0-9_./]`, with exemptions for engine-generated files.

## 6. Godot configuration for the hand-drawn look

### 6.1 Why the defaults matter

Godot’s defaults are tuned for general use, and several work against hand-drawn art. Lossy or GPU-compressed textures introduce blockiness and smear fine ink-like linework and soft gradients. Pixel snapping makes slowly moving hand-drawn sprites shimmer. Missing alpha-border fixes produce light halos around sprite edges when filtering is on. The settings below keep every asset on the same pipeline.

All values in this section are **proposed defaults** for approval. They assume the art is hand-drawn at screen resolution, not pixel art.

### 6.2 Project settings (Project → Project Settings)

| Setting (path) | Value | Rationale |
| --- | --- | --- |
| Display → Window → Size → Viewport Width / Height | 1920 × 1080 | Single design resolution; art is authored 1:1 for this size. |
| Display → Window → Stretch → Mode | `canvas_items` | Renders at native window resolution and scales the layout, so linework stays crisp instead of being upscaled from a low-res buffer. |
| Display → Window → Stretch → Aspect | `keep` | Prevents the composition of the climb from changing with window shape. |
| Rendering → Textures → Canvas Textures → Default Texture Filter | `Linear` | Smooth, painterly edges. `Nearest` is for pixel art and would make hand-drawn lines look jagged. |
| Rendering → Textures → Canvas Textures → Default Texture Repeat | `Disabled` | Tiling is opt-in per node. |
| Rendering → 2D → Snap → Snap 2D Transforms to Pixel | Off | Avoids shimmer on slow movement and subpixel camera motion. |
| Rendering → 2D → Snap → Snap 2D Vertices to Pixel | Off | Same reason. |
| Rendering → Anti Aliasing → Quality → MSAA 2D | 2× (raise to 4× if affordable) | Smooths edges of polygons and lines. It does not affect textures; texture edges rely on Linear filtering and the alpha-border fix. |
| Rendering → Environment → Default Clear Color | A near-black project colour chosen by the art lead | Guarantees any uncovered area matches the oppressive tone rather than flashing grey. |

### 6.3 Texture import defaults

These apply to every PNG imported as `Texture2D`.

| Import option | Value | Rationale |
| --- | --- | --- |
| Compress → Mode | **Lossless** | Preserves every stroke exactly. VRAM compression (S3TC/BPTC/ETC) and Lossy WebP both visibly degrade fine linework and soft shading. Revisit only if measured memory use forces it. |
| Compress → Lossy Quality | Not used (Lossless) | Left at default. |
| Compress → Channel Pack | Default | Not relevant to 2D colour art. |
| Mipmaps → Generate | **Off** | Mipmaps are only useful for textures drawn smaller than their source size. Off keeps memory low and avoids soft blur at 1:1. |
| Process → Fix Alpha Border | **On** | Bleeds edge colour into fully transparent pixels so linear filtering does not produce pale or dark halos around sprites. |
| Process → Premultiply Alpha | Off | Only enabled for specific blend-mode problems (see exceptions). |
| Process → Size Limit | 0 (none), with a hard art-side maximum of 4096 px per side | Limit is enforced by art spec, not silently by the importer. |
| Detect 3D → Compress To | **Disabled** | Prevents Godot from silently switching a texture to a VRAM-compressed format if it ever gets used in a 3D context. This is a 2D project. |

### 6.4 Per-asset exceptions

Exceptions are allowed only when the asset class genuinely needs them, and must be logged in section 8.3.

| Asset class | Override | Why |
| --- | --- | --- |
| Large backgrounds drawn at more than 100% of screen size and shown scaled down (parallax plates, up to \~2× authoring size) | Mipmaps → Generate **On**, and the node’s Texture Filter set to `Linear Mipmap` | Downscaling large art without mipmaps produces shimmer and aliasing as the camera moves. |
| Particle and glow sprites using additive blending that show dark fringes | Process → Premultiply Alpha **On** (verify against the material blend mode) | Fixes fringing for additive effects. |
| Seamless tiling textures | Node Texture Repeat set to `Enabled`; import defaults unchanged | Tiling is a node property, not an import one. |
| Full-screen vignette or lighting overlays | Handled by `vignette_asymmetric.gdshader`, not a baked texture where possible | Keeps the asymmetric, upward-emphasising vignette resolution-independent. |

If an asset needs a different compression mode, flag it for review instead of changing it quietly.

### 6.5 Applying the defaults

There are two equivalent ways; use the first and verify with the second.

1. **In the editor.** Select any PNG in the FileSystem dock, open the Import dock, set the options in section 6.3, then use the **Preset…** menu to set these as the default for the texture importer, and click **Reimport**.
2. **In `project.godot`.** The saved defaults are written to the `[importer_defaults]` section, which is committed to Git. A correct result looks like this:

```
[importer_defaults]

texture={
"compress/mode": 0,
"mipmaps/generate": false,
"process/fix_alpha_border": true,
"process/premult_alpha": false,
"detect_3d/compress_to": 0
}
```

Important behaviours to remember:

- Importer defaults apply only to files imported **after** they are set. Assets already imported keep their own `.import` settings. After changing defaults, select the existing textures and use Reimport (or delete their `.import` files and let Godot regenerate them).
- `.import` files are version-controlled; the `.godot/` cache folder is not.
- Run a spot check after any Godot version upgrade, since importer option names can change between versions.

### 6.6 Art export specification (from Procreate)

Import settings can only preserve what is exported correctly.

- Export as **PNG** with transparency (no flattened background colour), sRGB colour profile.
- Draw at the on-screen size for 1920 × 1080. Characters and enemies are authored at their final in-game pixel size; do not rely on scaling in-engine.
- Keep a consistent canvas size per asset family (all `char_dante_*` frames share the same canvas and pivot reference), so animation frames don’t jump.
- Leave 2–4 px of transparent padding around sprites to avoid edge clipping with filtering.
- Maximum 4096 px on any side. Anything larger is split into tiles.
- Keep a consistent line-weight and texture-grain target across all assets, defined by the art lead in a one-page style sheet stored in `_source/`.

## 7. Version control conventions

- **Branches:** `feature/<short-name>`, `fix/<short-name>`, `art/<short-name>`, `docs/<short-name>`. Lowercase with hyphens. `main` is always playable.
- **Commits:** imperative mood with a type prefix: `feat:`, `fix:`, `art:`, `refactor:`, `docs:`, `chore:`. Example: `art: add dante run cycle frames 01–08`.
- **One logical change per commit.** Art imports and the code that uses them can be committed together, but an unrelated bulk rename gets its own commit.
- **Renames and moves** are done inside the Godot editor’s FileSystem dock so that references update. Never rename scene or resource files in the OS file explorer.
- **`.gitignore`** must include `.godot/`, export output folders, and OS files (`.DS_Store`, `Thumbs.db`). Commit `.import` files, and (on Godot 4.4 or later) the generated `.uid` files.
- **`.gitattributes`:** `* text=auto` for consistent line endings, and Git LFS tracking for large binaries (`*.psd`, `*.procreate`, `*.blend`, `*.wav`, and optionally `*.png` over a few MB).
- **Licensing:** the repository’s licence file must be named `LICENSE` so that GitHub detects it (see section 9).

## 8. Governance

### 8.1 Roles

This is currently a solo project, so one person holds every role. The roles are written separately so that collaborators can be onboarded without changing the process.

| Role | Responsibility | Held by |
| --- | --- | --- |
| Standard owner / approver | Approves the standard and exceptions | Carl O’Brien |
| Art lead | Owns style sheet, art export spec and import defaults | Carl O’Brien |
| Technical lead | Owns code conventions, linting and CI | Carl O’Brien |
| Contributors | Follow the standard; raise exceptions | Future collaborators |

### 8.2 Pre-merge checklist

- [ ] All new files and folders are lowercase snake\_case and sit in the correct top-level folder.
- [ ] Scene root node name matches the scene file name in PascalCase; script name matches the scene.
- [ ] No auto-numbered node names (`Sprite2D2`) remain.
- [ ] GDScript is statically typed; `gdlint` and `gdformat --check` pass.
- [ ] New textures use the project defaults, or an exception is logged below.
- [ ] No sprites are scaled in the editor beyond ±10%; if they are, the art is redrawn instead.
- [ ] No debug or sandbox scenes are included in export presets.
- [ ] The game runs from a clean clone with F5.

### 8.3 Exception log

| ID | Date | Asset or scope | Deviation | Reason | Approved by |
| --- | --- | --- | --- | --- | --- |
| EX-001 | — | — | — | — | — |

### 8.4 Change control

- Changes to this document are made on a `docs/` branch and recorded below.
- Minor edits (clarifications, examples) bump the minor version; any change to a rule or default bumps the major version of the standard.
- When a rule changes, the migration plan is stated in the change entry: what gets renamed or reimported, and when.
- Review the document at three points: before art production starts in earnest, at vertical-slice complete, and before the itch.io release.

| Version | Date | Author | Change |
| --- | --- | --- | --- |
| 0.1 | 13 Nov 2026 | Carl O’Brien | Initial draft |

## 9. Open decisions and assumptions

These need confirmation before this document moves from Draft to Approved.

1. **Design resolution** of 1920 × 1080 and the authoring-at-1:1 policy.
2. **Filtering model:** Linear filtering by default, with mipmaps only for large scaled-down backgrounds.
3. **Compression:** Lossless for all textures. Reconsider after the first full level if memory use is a problem on the target platforms.
4. **Organisation:** feature-first folder layout in section 3.
5. **Target platforms:** the README names a Windows build on itch.io. If a web build is also planned, check the renderer choice and texture memory limits early.
6. **Repository housekeeping:** rename `LISCENSE` to `LICENSE`, and replace the `[itch.io link]` placeholder in the README once the page exists.
7. **Enemy names:** the three enemy types should be fixed so that `enemy_<name>` prefixes can be finalised.

## 10. Quick reference

| Thing | Rule | Example |
| --- | --- | --- |
| Folder / file | lowercase snake\_case | `enemy_base.tscn` |
| Scene root node | PascalCase of file name | `EnemyBase` |
| Node | PascalCase, named by role | `HurtboxShape` |
| Script file | same as scene | `player.gd` |
| `class_name` | PascalCase | `Player` |
| Function / variable | snake\_case | `apply_damage` |
| Constant | CONSTANT\_CASE | `MAX_FALL_SPEED` |
| Signal | snake\_case, past tense | `health_changed` |
| Art | `category_subject_descriptor_nn.png` | `char_dante_run_01.png` |
| Audio | `mus_` / `sfx_` prefix | `sfx_dante_jump_01.wav` |
| Input action | snake\_case, verb first | `move_left` |
| Texture filter | Linear | — |
| Texture compression | Lossless | — |
| Alpha border fix | On | — |
| Mipmaps | Off, except large scaled-down backgrounds | — |
| Pixel snapping | Off | — |
