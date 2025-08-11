# Extended-Translucency---Community-Shaders---Auto-Patch

Automates tagging meshes with AnisotropicAlphaMaterial for use with shader mods that support anisotropic translucency (e.g., fabrics/glass).
It runs Blender + PyNifly (https://github.com/BadDogSkyrim/PyNifly) headless, scans your mod folder, and exports only changed NIFs into a mirror tree.
What it does

    Imports each .nif with PyNifly (no animations, meshes only).

    Finds candidate meshes via name & material heuristics (e.g., lace, sheer, glass), plus a transparency hint from Blender materials.

    Sets AnisotropicAlphaMaterial to:

        3 – Anisotropic fabric (recommended for most fabrics/hair-like textures)

        2 – Isotropic fabric / glass-like

        1 – Rim light (fallback/legacy)

    Skips FaceGen/CharGen and other noisy assets up front.

    No-change detection: if a mesh already has the correct value, it’s left alone and the file isn’t exported.

    Two logs:

        _patch_log.csv – compact “what changed”

        _full_run.log – detailed run with import/export messages

Requirements

    Blender 3.6 LTS or 4.5+ (tested on both)

    PyNifly io_scene_nifly 20.1 (install & enable in Blender)

    Windows (paths/commands assume Windows)

Install

    Install Blender (3.6 or 4.5).

    In Blender: Edit → Preferences → Add-ons → Install… and choose the PyNifly zip from its Releases page, then enable it.

    Put batch_patch.py and this README.md anywhere (e.g., C:\Users\<you>\Downloads\my_nif_script\).

    Edit the top of batch_patch.py:

    INPUT_DIR  = r"E:\Your\Mods"
    OUTPUT_DIR = r"E:\Your\Mods_patched"

    The script creates OUTPUT_DIR subfolders only when exporting an actual patch.

Run
Blender 3.6

"C:\Program Files\Blender Foundation\Blender 3.6\blender.exe" ^
  --background --python "C:\path\to\batch_patch.py"

Blender 4.5

"C:\Program Files\Blender Foundation\Blender 4.5\blender.exe" ^
  --background --python "C:\path\to\batch_patch.py"

Heuristics (how picks are made)

    Keywords → model (object name, material name, texture node names):

        lace, stocking, sock, sheer, veil, tulle, mesh, nylon, silk, satin, hair → 3

        glass, acrylic, plexi → 2

    Transparency hint: if a material appears to use transparency (blend_method or alpha-related node names), default to 3.

    Hard skips prevent false positives:

        Name contains: eye, mouth, teeth, tongue, brow, lash, hazard, fx, emitter, fire, smoke, magic, hairline

        Names starting with common VFX prefixes: FX, VFX, Haze, Glow, Emit, Smoke, Magic, Particle, Billboard, :

    Path skips (not even imported):

        \FaceGenData\FaceGeom\

        \SKSE\Plugins\CharGen\

        textures folders / LOD / skeletons, etc.

    You can tune these in the script:

        KEYWORD_TO_MODEL – change/add mappings

        HARD_SKIP_WORDS, IGNORE_OBJ_PATTERNS – silence known non-targets

        SKIP_PATH_PATTERNS – exclude entire subtrees

What gets written

For each selected mesh:

    Sets Blender custom property AnisotropicAlphaMaterial = {1|2|3} on the object (and mirrors to its materials).

    PyNifly writes that out as NIF extra data on export.

Exports only happen when at least one mesh changes.
Logs & Outputs

    batch_patch.py folder:

        _patch_log.csv – timestamp, nif_path, export_path, object_name, chosen_model, reason

        _full_run.log – complete run (import/export results, suppresses Blender’s mesh-validate spam in the console)

    Patched NIFs go to OUTPUT_DIR mirroring your input tree.

Troubleshooting

    Operator not found:
    Ensure the addon key is io_scene_nifly and enabled. In a console:

    blender.exe --background --python-expr "import bpy; print(bpy.context.preferences.addons.keys())"

    You should see 'io_scene_nifly'.

    FaceGen/CharGen spam:
    Already skipped by path. If you still see them, confirm your mod’s paths match the patterns (Windows backslashes) and the strings are in SKIP_PATH_PATTERNS.

    “not in View Layer” errors:
    The script uses a view-layer-safe cleanup. If you still see it, it’s harmless and only indicates Blender didn’t link an object into the active scene; export still uses selection.

    Partition/slot overlaps:
    The script detects basic BSDismember partition overlap; when found, it skips export to avoid breaking dismemberment. You can adjust thresholds in has_partition_overlap().

    No output:
    If _patch_log.csv is empty, either:

        no candidates matched your rules, or

        they already had the desired value (the script now does “no-change” skipping).

Why these defaults?

    Model 3 (Anisotropic) is a solid default for sheer/fabric/hair-like textures.

    Model 2 (Isotropic) for glass/acrylic avoids over-fuzzy edges.

    Model 1 (Rim Light) is rarely needed; keep it for niche cases or manual overrides.

If you want to try “always 3” everywhere, set KEYWORD_TO_MODEL = defaultdict(lambda: 3, {...overrides...}) and remove the transparency fallback.
Contributing

PRs welcome! Ideas:

    Per-material node graph inspection for more confident glass vs fabric separation.

    A YAML/JSON rules file instead of hard-coding dictionaries.

    A dry-run mode that only logs what would change.

License

MIT (match your repo’s preference).
