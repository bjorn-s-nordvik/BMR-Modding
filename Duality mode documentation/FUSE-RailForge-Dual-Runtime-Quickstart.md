# Quick Start: Legacy Source to One FUSE/RailForge Mod Folder

This setup lets one installed Railroader mod package support **either FUSE or RailForge**.

The package contains two separate graph files:

- In this pattern, `FuseDataFiles` points FUSE at the graph stored in the mod root.
- RailForge reads the graph inside `RailForge/game-graph`.
- Each runtime leaves the other runtime's graph alone.

The graph files are not interchangeable. For a new mod, build it in FUSE first, then make a RailForge version of the same content. For an existing RailLoader/Strange Customs-family mod, treat the legacy package as conversion input and audit the generated FUSE and RF branches independently.

> This setup is for data-only graph/content mods. RailForge and FUSE both implement selected legacy compatibility, but neither provides universal conversion of arbitrary DLL behavior. It was proven with the East Whittier Yard test on 4 September 2026.

## 1. Create this folder structure

```text
Your.Mod.Name/
├── Info.json
├── Definition.json
├── game.graph.fuse.json
└── RailForge/
    └── game-graph/
        └── your-mod.json
```

Place that complete folder directly inside Railroader's `Mods` folder:

```text
Railroader/
└── Mods/
    └── Your.Mod.Name/
```

What each file does:

| File | Used by |
| --- | --- |
| `Info.json` | Unity Mod Manager and FUSE |
| `game.graph.fuse.json` | FUSE |
| `Definition.json` | RailForge |
| `RailForge/game-graph/your-mod.json` | RailForge |

The filenames may be changed, but the FUSE file's exact package-relative path must also be updated in `FuseDataFiles`. Every ordinary, unconditional RF graph belongs beneath `RailForge/game-graph`. A deliberately conditional RF payload is the exception: keep it outside native discovery and reference it from `Definition.json.mixintos`.

## 2. Add a minimal `Info.json`

Put this file in the mod root:

```json
{
  "Id": "YourName.YourMod",
  "DisplayName": "Your Mod",
  "Author": "Your Name",
  "Version": "1.0.0",
  "ManagerVersion": "0.27.10",
  "Requirements": [],
  "LoadAfter": [
    "FUSE"
  ],
  "FuseDataFiles": [
    "game.graph.fuse.json"
  ]
}
```

Change the ID, name, author, version, and graph filename for your mod.

Important:

- Keep `Requirements` free of a hard `FUSE` requirement in this dual-runtime data package. A hard requirement made RailForge report FUSE as missing during the RF-only test.
- `FuseDataFiles` makes FUSE discover the data package and names the root graph it should load.
- `LoadAfter: ["FUSE"]` is an optional UMM ordering hint when FUSE is installed; it is not a hard dependency.
- Do not leave backup files ending in `.fuse.json` inside the release folder.

## 3. Add a minimal `Definition.json`

Put this file beside `Info.json`:

```json
{
  "manifestVersion": 5,
  "id": "YourName.YourMod",
  "name": "Your Mod",
  "author": "Your Name",
  "version": "1.0.0",
  "requires": [],
  "loadAfter": []
}
```

Use the same exact mod ID and version in both manifests:

```text
Info.json:       YourName.YourMod  1.0.0
Definition.json: YourName.YourMod  1.0.0
```

If the RailForge graph needs another native/RF mod or asset pack, add that dependency to `Definition.json`. Put FUSE-side data-package relationships in the FUSE manifest fields instead.

## 4. Add the two graph files

Keep the finished FUSE graph here:

```text
Your.Mod.Name/game.graph.fuse.json
```

Make sure its filename appears exactly in `FuseDataFiles`.

Put the converted RailForge graph here:

```text
Your.Mod.Name/RailForge/game-graph/your-mod.json
```

Do not put the RF graph in the mod root, and do not add it to `FuseDataFiles`.

Keep FUSE as your source of truth. Whenever the mod changes:

1. Make and test the change in FUSE.
2. Reproduce the same change in the RF graph.
3. Update the version in both manifests.

Do not simply copy or rename the FUSE graph. The formats use different paths, field names, component types, and merge rules.

## 5. How the separation works

In FUSE mode:

```text
Info.json
└── FuseDataFiles
    └── game.graph.fuse.json
```

In RailForge mode:

```text
Definition.json
└── RailForge/game-graph/
    └── your-mod.json
```

That is the key: **one package folder, two manifests, and one graph in each loader's own location.**

## 6. When the source mod is Legacy

Keep the untouched legacy source outside the distributable dual-runtime folder:

```text
WorkingFiles/
├── LegacySource/
└── Release/
    └── Your.Mod.Name/
```

Classify each legacy part before conversion:

| Legacy content | FUSE treatment | RailForge treatment |
| --- | --- | --- |
| Track/route JSON | Convert to one or more declared `.fuse.json` fragments | Translate to native `RailForge/game-graph` files, or deliberately use RF's legacy reader |
| `StrangeCustoms/` or `SC/` data | Convert recognized roots and handlers | RF `1.98` can scan these exact legacy roots; treat this as runtime compatibility |
| `AssetPacks/`, `SCAssetPacks/`, `StrangeCustomsAssetPacks/` | Install/mount supported legacy asset packs | RF can index the exact legacy asset roots |
| Horn/whistle/bell package | Convert to native FUSE audio definitions | Use the corresponding RF audio mixinto/catalog path |
| Compiled DLL or managed-object mixinto | Compatibility host or independent reimplementation | Provider/compatibility dependent; JSON translation is not code conversion |

Do not ship an active legacy graph beside native FUSE and RF copies of the same objects. RailForge's legacy logic makes supported legacy data applicable, but it can also duplicate nodes, segments, industries, or scenery if the native RF branch contains the same content.

For a conditional legacy mixinto, preserve fragment scope separately:

- FUSE: make it a separate `.fuse.json` file, list it in `FuseDataFiles`, and put `requires[]`/`conflictsWith[]` inside its top-level `mixinto` object.
- RF: reference the RF version from a conditional `Definition.json.mixintos.<target>` object.
- Keep the conditional RF file outside `RailForge/game-graph`; use a path such as `RailForge/conditional/optional-yard.json`. Putting it in `game-graph` makes native discovery unconditional.
- Put a dependency at manifest level only when the whole package must be blocked. `loadAfter` controls order and is not a requirement.

## 7. Test the same package twice

Do not enable FUSE and RailForge together. Fully close Railroader before switching files, loaders, or mod sets.

For the FUSE test, enable:

- the dual-runtime package;
- FUSE;
- FUSE editions of its required dependencies.

For the RailForge test, enable:

- the exact same dual-runtime package;
- RailForge;
- native/RF editions of its required dependencies.

Then run this smoke test:

1. Install exactly one copy of the package in `Mods`.
2. Start in the FUSE environment after a full game restart.
3. Confirm FUSE loads the root graph; inspect tracks, industries, scenery, and progression.
4. Fully close the game and switch to the RF environment.
5. Confirm RailForge loads the nested RF graph; inspect the same content.
6. Save, fully restart, reload, and check the mod again in each runtime.

If legacy compatibility remains in the release, add a third isolated test using only the intended legacy reader path. Confirm from the logs that the same payload was not also discovered through the native branch.

Use test saves or backups. Matching package IDs do not automatically guarantee that a save can move safely between runtimes.

## Finished result

Package the complete `Your.Mod.Name` folder. The player installs that folder once and chooses which runtime and matching dependency set to use.

For legacy classification, field conversion, RailForge merge behavior, progression, troubleshooting, and full release testing, continue with the [detailed technical reference](./FUSE-RailForge-Dual-Runtime-How-To.md). For a side-by-side Legacy → FUSE ↔ RailForge inventory of namespaces, arrays, fields, handlers, and compatibility paths, use the [complete translation matrix](./FUSE-RailForge-Complete-Translation-Matrix.md).
