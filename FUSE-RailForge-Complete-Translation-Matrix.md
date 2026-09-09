# Legacy → FUSE ↔ RailForge Complete Translation Matrix

## Data-only Railroader graph mods

> **Status:** Complete against the legacy conversion/compatibility documentation and source supplied with FUSE, the complete FUSE schema, and the supplied RailForge Full Guide plus inspected RailForge `1.98.BRAVO99` runtime (`RailForge.dll` assembly `0.12.99.0`). This is a translation ledger, not a promise that every feature has a native equivalent.

Use this reference when converting a legacy RailLoader/Strange Customs-family data mod and maintaining one package with a FUSE graph in the mod root and a RailForge graph under `RailForge/game-graph`. It aligns legacy source shapes and every documented FUSE namespace with the closest RailForge representation, including arrays, nested objects, component types, removals, compatibility readers, and RF-only formats.

Legacy is an **input dialect and compatibility path**, not a third native branch that should be loaded beside equivalent FUSE and RF files. A release should normally contain the two native payloads. Retain legacy data only when an explicitly documented compatibility reader is the intended implementation, and prove that it is not also being applied through a native branch.

The short installation recipe is in the [dual-runtime quickstart](./FUSE-RailForge-Dual-Runtime-Quickstart.md). The complete workflow and test procedure are in the [dual-runtime technical guide](./FUSE-RailForge-Dual-Runtime-How-To.md).

## Evidence and status legend

The supplied RailForge folder contains the Full Guide, `Info.json`, and `RailForge.dll`. Several detailed documents linked by the Full Guide are not present locally: the graph-patch reference, company-start schema/example, AMM contracts, supplement guide, and several graph/manifest examples. This matrix therefore distinguishes public guidance from current-runtime compatibility.

| Mark | Meaning |
| --- | --- |
| **Direct** | Same concept and usable shape, subject to ordinary validation. |
| **Rename/move** | Same concept, but the path or field name changes. |
| **Reshape** | Same intent, but nesting, reference shape, or array representation changes. |
| **Handler/manual** | Requires an RF handler, provider, editor export, or explicit custom design. |
| **FUSE-only** | No corresponding RF authoring contract was found in the supplied material. |
| **RF-only** | No corresponding native FUSE graph contract was found. |
| **Runtime-only** | Confirmed in RF `1.98` code, but not fully documented as a stable public authoring contract. |
| **Legacy-direct** | The current runtime can ingest the legacy package or payload without first rewriting it to its native schema. |
| **Converted** | A supplied converter/runtime adapter reshapes the legacy source into native data. |
| **Preserved-only** | The legacy data is retained for diagnostics or future work, but does not have complete native behavior. |
| **Provider/code** | Behavior depends on a compiled plugin, external provider, or ABI compatibility host; JSON conversion alone is insufficient. |

Collection notation used below:

- **scalar**: string, number, integer, boolean, or null;
- **array**: ordered JSON array;
- **dictionary**: JSON object keyed by stable object IDs;
- **object**: a fixed record with named fields;
- **patch union**: more than one accepted JSON representation.

## 1. Package discovery and manifests

### Package files

| Purpose | FUSE | RailForge | Translation status |
| --- | --- | --- | --- |
| Package manifest | `Info.json` | `Definition.json` with canonical `manifestVersion: 5` | **Reshape**; keep identity/version synchronized. |
| Main graph | File named in `FuseDataFiles`, normally in the mod root | `RailForge/game-graph/**/*.json` | Separate native files; never copy one dialect unchanged into the other location. |
| Runtime dependencies | `FuseRequires[]`, plus generic UMM fields where appropriate | `requires[]` / `dependencies[]` | **Reshape**; dependencies must name the edition used by that runtime. |
| Load order | `FuseLoadAfter[]`, `FuseLoadBefore[]`, `FuseLoadPriority` | `loadAfter[]`, `loadBefore[]` | **Rename/move**; do not use ordering as a substitute for a dependency. |
| Conflicts | `FuseConflictsWith[]` | `conflictsWith[]` | **Rename/move**. |
| FUSE asset packs | `FuseAssetPacks` string or string array | `RailForgeAssetPacks/**/Catalog.json` and `Definitions.json` | **Relocate/reference and validate**; compatible catalog/definitions payloads may be reusable, but discovery roots differ. |
| Definition overrides | `FuseDefinitionOverrides` union | RF `Definitions.json` or provider-specific definition flow | **Manual package conversion**; RF containers are not automatic definition overrides. |
| Settings file | `Settings` | No equivalent manifest field established by the supplied RF guide | **FUSE-only** unless an RF provider implements settings. |
| Graph mixinto declaration | FUSE graph `mixinto` and/or manifest data-file list | `mixintos` plus native RF folders | **Manual**; do not copy target names blindly. |
| Compiled entry point | `AssemblyName`, `EntryMethod` | RF `assemblies` support is mentioned, but the detailed local manifest schema is missing | Separate code-mod design; outside this data-only pattern. |

### Complete FUSE `Info.json` field inventory

| FUSE field | Closest RF manifest concept | Action |
| --- | --- | --- |
| `$schema` | RF manifest schema URI, if the target manifest schema defines one | FUSE tooling metadata; do not reuse the FUSE schema URI in RF. |
| `Id` | `id` | **Direct**, preserve exact case and value where possible. |
| `DisplayName` | `name` | **Rename**. |
| `Author` | Possible RF metadata field `author` | Copy only if accepted by the target RF manifest schema; the supplied Full Guide/current manifest interface does not establish it as canonical. |
| `Version` | `version` | **Rename** and keep synchronized. |
| `ManagerVersion` | No RF graph equivalent | UMM/FUSE-side metadata. |
| `GameVersion` | RF compatibility metadata, exact supplied schema unavailable | **Manual**. |
| `AssemblyName`, `EntryMethod` | RF assemblies/entry mechanism | Code-mod concern; do not infer equivalence. |
| `HomePage`, `Repository` | RF package metadata, if supported by the target manifest version | Copy only after validating the current RF manifest schema. |
| `Source` | No established RF equivalent | FUSE provenance enum: `github`, `nexus`, or `local`. |
| `Requirements[]` | `requires[]` | Generic UMM requirements can be audited by RF; never place a hard `FUSE` requirement in the tested dual data package. |
| `LoadAfter[]` | `loadAfter[]` | Generic optional ordering only. In the tested dual package, `LoadAfter: ["FUSE"]` was an empirical safe/no-op boundary while a hard requirement was not; do not imply RF resolves a FUSE provider edge. |
| `FuseLoadPriority` | No direct RF scalar | Express real dependencies/order with RF edges. |
| `FuseRequires[]` | `requires[]` | **Reshape** requirement objects/versions as needed. |
| `FuseLoadAfter[]` | `loadAfter[]` | Copy semantic ordering using RF package IDs. |
| `FuseLoadBefore[]` | `loadBefore[]` | Copy semantic ordering using RF package IDs. |
| `FuseConflictsWith[]` | `conflictsWith[]` | Copy only when the conflict also applies to the RF edition. |
| `FuseDataFile` | Native RF discovery folder | No RF filename field is required for ordinary graph discovery. |
| `FuseDataFiles[]` | `RailForge/game-graph/**/*.json` | **Move** files; keep the RF graph out of this FUSE array. |
| `FuseAssetPacks` | `RailForgeAssetPacks` | **Manual layout conversion**. |
| `FuseDefinitionOverrides` | RF `Definitions.json`/provider flow | **Manual**; not an RF container patch by default. |
| `Settings` | Provider-specific | No supplied RF data-only equivalent. |

Every `Requirements[]`, `LoadAfter[]`, `FuseRequires[]`, `FuseLoadAfter[]`, `FuseLoadBefore[]`, and `FuseConflictsWith[]` entry may be either a package-ID string or an object with `Id` plus optional `NotBefore` and `NotAfter`. Translate the resolved constraint, including version bounds, into the RF manifest's accepted dependency form.

`FuseAssetPacks` may be one package-relative folder string or an array of folder strings. `FuseDefinitionOverrides` may be one string, one object, or an array containing both forms. Each object contains required `Path` and optional `StoreIdentifier`.

For RF rolling stock, a selectable locomotive additionally needs a valid Catalog/Definitions pair, a supported locomotive archetype, `visibleInPlacer` not false, a unique exact ID, and a live PrefabStore resolution after map load. This is an RF asset-pack contract, not a FUSE graph-field translation.

FUSE manifest `Settings` currently documents these booleans; none has a general RF manifest translation:

| FUSE `Settings` field | Purpose |
| --- | --- |
| `EnableExperimentalEarlyScenePathSuppression` | Gate experimental early suppression of base scene paths. |
| `MirrorAssetPacksToLocalLow` | Copy compatible asset packs to LocalLow before loading. |
| `VerboseApplyReportDetails` | Emit per-object FUSE apply details. |
| `BlockNonHostMultiplayerClientWorldApply` | Prevent non-host clients from applying world mutations. |
| `ShowAdvancedHealthDetails` | Show advanced FUSE diagnostics. |
| `EnableUpdateCheck` | Enable the FUSE startup update check. |
| `DirectStoreNativeDeserialize` | Route directly mounted asset packs through native store deserialization. |

`Id`, `DisplayName`, `Author`, `Version`, and `ManagerVersion` are required in the FUSE manifest. `AssemblyName` and `EntryMethod` are co-required: if either is present, the other must also be present.

FUSE explicit graph discovery reads `FuseDataFile` first and then `FuseDataFiles` in declared order. Without either field, the current runtime selects exactly one top-level fallback: the first alphabetic BSON, otherwise the highest-ranked eligible JSON with `*.fuse.json` preferred. It does not load every top-level `.fuse.json` in fallback mode, despite conflicting wording in the supplied package guide. Always declare `FuseDataFiles` explicitly.

During the ecosystem rename window, FUSE dependency/mixinto matching may treat a `.RAIL`-suffixed ID as matching the suffixless package, and a converted `.FUSE` ID as satisfying its original legacy ID. Treat this as transition compatibility, not a naming pattern for new packages.

RF manifest compatibility aliases found in `1.98` include requirements under `requires`, `dependencies`, `required`, or `requirements`; conflicts under `conflictsWith`, `conflicts`, `incompatibleWith`, or `incompatibilities`; and ordering under `loadAfter`/`after` and `loadBefore`/`before`. Mixinto paths accept scalar, array, or conditional object forms and `file()`, `dir()`, `directory()`, or `folder()` references. Prefer the canonical names shown in this matrix.

RF dependency/reference objects accept ID keys `id`, `Id`, `modId`, `ModId`, `name`, or `Name`; version metadata keys are `notBefore`, `minVersion`, `version`, `notAfter`, and `maxVersion`. Canonical new authoring is `{id, notBefore, notAfter}`. Conditional mixinto objects recognize metadata keys `mixinto`, `file`, `path`, `mixintos`, `files`, `paths`, `requires`, `dependencies`, `required`, `conflictsWith`, `conflicts`, `incompatibleWith`, `incompatibilities`, `loadAfter`, `loadBefore`, `after`, and `before`. All resolved paths must remain confined to the package and may not escape through links/reparse points.

RF primarily uses `Definition.json`, but its scanner also recognizes `Info.json`; exact legacy content roots `StrangeCustoms` and `SC`; and exact legacy asset roots `AssetPacks`, `SCAssetPacks`, and `StrangeCustomsAssetPacks`. Those are migration compatibility surfaces; new RF authoring should use the canonical folders.

RF `1.98` overlays manifest identity/version in this order: `Info.json`, then `Definition.json`, then a selected redirect definition; a later nonblank case-insensitive `Id`/`id` or `Version`/`version` wins, while ordering edges accumulate. `Definition.json.redirects[]` accepts case-insensitive `definition`, `path`, or `file` aliases plus requirements. Redirect targets must remain confined to the package. This is runtime compatibility; prefer one unambiguous canonical definition for new packages.

## Legacy input: source dialect and compatibility paths

RailForge `1.98` contains a substantial legacy compatibility layer: it implements RailLoader-facing mod/mixinto interfaces, scans recognized legacy content and asset roots, reads conditional file/directory mixintos, imports several legacy sidecars, and embeds adapters for selected AlinasMapMod and ConfusingSupplements contracts. That makes many **data-only** legacy packages applicable, but it is not blanket compatibility with every legacy DLL, managed-object mixinto, handler, or save mutation.

FUSE likewise has two legacy paths: the supported converter produces native `.fuse.json` files and reports lossy/unsupported concepts, while the runtime compatibility host can load selected legacy packages and plugins. For a maintained dual-runtime release, prefer an audited conversion over relying silently on either runtime's compatibility layer.

### Package-family routing

| Legacy package or source | FUSE path | RailForge `1.98` path | Recommended release treatment |
| --- | --- | --- | --- |
| RailLoader data package with `Definition.json` and file/directory `mixintos` | Convert each recognized JSON concern to a declared `.fuse.json` fragment; preserve fragment requirements/conflicts | **Legacy-direct/runtime-only** manifest and mixinto reader | Use native FUSE and RF payloads for new maintenance; do not also ship an equivalent active legacy graph. |
| Strange Customs graph content under `StrangeCustoms/` or `SC/` | **Converted** by the FUSE converter/runtime legacy adapter | **Legacy-direct/runtime-only** exact-root scan and legacy mixinto construction | Suitable as migration input; native sidecars remain easier to validate and future-proof. |
| Legacy asset roots `AssetPacks/`, `SCAssetPacks/`, `StrangeCustomsAssetPacks/` | FUSE can discover/mount supported legacy packs; the installer routes asset-pack-only inputs instead of wrapping them | RF indexes the exact legacy roots and compatibility aliases | Reuse only after catalog IDs, Definition types, and asset references resolve in both runtimes. |
| AlinasMapMod graph content | Convert tracks, operations, scenery, labels, masks, features, loaders, stations, turntables, pole edits, and recognized splineys | Selected AMM handlers and sidecars are embedded or adapted | Audit every handler; AMM support is contract-specific, not a promise for arbitrary namespaces. |
| Alina map-tile package | Install through the FUSE-supported tile/package path; not an ordinary route-JSON conversion | RF can index the AMM-compatible `Maps/<map>/tile_x_y.data` layout | Treat tiles as package assets, not game-graph records. |
| ConfusingSupplements / For Your Convenience serialized data | Recognized component and Definition families convert or use FUSE compatibility implementations | RF embeds selected component/Definition compatibility contracts | Verify the exact discriminator and fields; unknown script-driven behavior remains provider-dependent. |
| Legacy horn, whistle, or bell pack | Convert to native FUSE `audio.whistles`, `audio.horns`, or `audio.bells` and copy contained audio | RF `1.98` can read the corresponding manifest mixinto targets through its runtime audio catalogs | Keep paths package-relative; validate profiles/keyframes and audio assets in both runtimes. |
| Legacy `game-migrations` payload | FUSE carries `waybillDestinations` and `properties` into `extensions.gameMigrations` for its migration applier | RF has a native/runtime game-migrations path | Save migration is high risk; test copied old saves and exact old/new IDs. |
| Legacy Company/start-option payload | FUSE converts `spawnPoint` to `world.spawnPoints[]` and preserves the remaining start data under `extensions.legacyStartOption` | RF imports `railforgeLegacyCompanyStarts` through an internal adapter, or uses native Company starts v1 | FUSE conversion is partial; do not advertise equivalent Company starts without live testing. |
| Compiled RailLoader/Strange Customs/other legacy DLL | No code conversion; selected plugins may run through the partial compatibility host | Selected assemblies/plugins may load through RF compatibility APIs | **Provider/code**. Never describe a successfully converted JSON file as proof that its DLL behavior was replaced. |
| Legacy loader runtime DLL itself | FUSE replaces the runtime and warns about/remediates old installations | RF provides its own compatibility surface | Do not bundle or run competing loader runtimes merely to support a data-only package. |

The FUSE converter treats these legacy runtime requirements as core capabilities supplied or replaced by FUSE and removes them instead of inventing `.FUSE` dependency IDs: `railroader`, `railloader`, `rail-loader`, `railloader.injector`, `railloader.interchange`, `assetloader`, `alinanova21.mapeditor`, `mapeditor`, `mmapeditor`, `zamu.strangecustoms`, `strangecustoms`, `zamu.confusingsupplements`, `confusingsupplements`, `zamu.foryourconvenience`, `foryourconvenience`, `alinanova21.alinasmapmod`, `alinasmapmod`, `alinamapmod`, and `fuse`. This is a FUSE conversion rule, not proof that RF satisfies every API exposed by those original runtimes. For RF, retain only the actual content/provider dependency that the selected RF implementation still needs, and verify it against the current RF compatibility report.

### Legacy graph-root translation ledger

This is the source-root index used before the detailed native field tables later in this document.

| Legacy root or shape | Native FUSE destination | Native RF destination or compatibility path | Status / warning |
| --- | --- | --- | --- |
| top-level `nodes` or `tracks.nodes` | `tracks.nodes` | `tracks.nodes`; RF also promotes the legacy top-level alias | **Converted/direct** after field validation. |
| top-level `segments` or `tracks.segments` | `tracks.segments` | `tracks.segments`; RF also promotes the legacy top-level alias | Endpoint names, style enums, groups, and patch semantics still require review. |
| top-level `spans` or `tracks.spans` | `tracks.spans` | `tracks.spans`; RF also promotes the legacy top-level alias | Validate segment identity and A/B distances rather than visual overlap alone. |
| `areas` | `tracks.areas` | top-level `areas` | Move/nest as shown in the detailed tables. |
| `loads` | `operations.loads` | top-level `loads` | Field aliases and custom fields require normalization. |
| `industries` | `operations.industries` | `areas.<areaId>.industries` | RF requires the real area nesting; component arrays may need explicit replacement. |
| `turntables` | `operations.turntables` | handler-backed `splineys` | **Converted/handler**; RF's authored record is a smaller specialized contract. |
| `scenery` | `world.scenery` | top-level `scenery` | Asset identity and scene-path semantics require validation. |
| `splineys` | recognized FUSE spliney, loader/station/turntable/label/mask/pole destination, or preserved extension | top-level `splineys` through the registered handler | Handler namespace decides whether translation is direct, reshaped, or provider-required. |
| `mandelas` (legacy spelling) | `world.sceneClones` plus scene-clone removals | `mandelas`/runtime-normalized `mandelas` | Null means removal; RF rejects a collision between both legacy spellings rather than merging them. |
| `texts` | `world.mapLabels` plus map-label removals | RF `texts` legacy/import path or promoted label handling | FUSE conversion is best effort; manually verify speed-label parsing, transforms, size, and color. |
| `progression`, `progressions` | `progression.progressionId`, `progression.sections[]`, and/or `progression.progressions` | `progressions` | Flat arrays and compact references usually need reshaping. |
| `mapFeatures` | `progression.mapFeatures` | `mapFeatures` | Normalize legacy `defaultEnableInSandbox` and `prerequisites` aliases. |
| `simpleGraphs` | `extensions.simpleGraphs` | No confirmed native materializer | **Preserved-only** in FUSE; do not assume runtime behavior. |
| `spawnPoint` plus start metadata | `world.spawnPoints[]` plus `extensions.legacyStartOption` | legacy Company-start adapter or native Company starts v1 | Partial translation; money, tutorial, feature, car-placement, and save semantics are not ordinary spawn-point fields. |
| `waybillDestinations` / `WaybillDestinations` | `extensions.gameMigrations.waybillDestinations` | RF game-migrations path | Save-key rewrite data, not graph content. |
| `properties` / `Properties` in a migration payload | `extensions.gameMigrations.properties` | RF game-migrations path | Save property-bag rewrite data. |
| legacy audio profile roots | `audio.whistles`, `audio.horns`, `audio.bells` dictionaries | `whistles`, `horns`, `bells` mixinto targets using array catalogs | Reshape keyed FUSE profiles versus RF catalog arrays. |

### Conditional legacy mixintos

A legacy conditional mixinto is applicable to both runtimes for supported **file/directory data** contracts, but it must not be allowed to bypass the native-branch isolation.

| Intent | FUSE representation | RF/legacy representation |
| --- | --- | --- |
| Conditional file | Separate declared `.fuse.json` fragment with top-level `mixinto.target`, optional `sourceFile`, `requires[]`, and `conflictsWith[]` | `Definition.json.mixintos.<target>` conditional object containing `mixinto`, `file`, or `path` |
| Several conditional paths under one condition | Several FUSE fragments, each declared in `FuseDataFiles`; repeat the condition where fragment scope matters | One object may use `mixintos`, `files`, or `paths` containing a scalar or array |
| Required package/version absent | Skip only that FUSE fragment | Skip only that conditional RF/legacy mixinto |
| Matching conflict present | Skip only that FUSE fragment | Skip only that conditional RF/legacy mixinto |
| Package-wide hard dependency/conflict | `FuseRequires[]` / `FuseConflictsWith[]` in `Info.json` | top-level `requires[]` / `conflictsWith[]` in `Definition.json` |
| Ordering only | `FuseLoadAfter[]` / `FuseLoadBefore[]` | top-level `loadAfter[]` / `loadBefore[]`; do not turn order into a requirement |

Keep a conditional RF file outside an auto-discovered native root such as `RailForge/game-graph/`; otherwise RF can load it unconditionally through native discovery and again through the legacy mixinto. A safe pattern is `RailForge/conditional/<file>.json`, referenced with `file(RailForge/conditional/<file>.json)`. All paths must remain inside the declaring package.

RF `1.98` accepts conditional metadata aliases for requirements (`requires`, `dependencies`, `required`), conflicts (`conflictsWith`, `conflicts`, `incompatibleWith`, `incompatibilities`), path values (`mixinto`, `file`, `path`, `mixintos`, `files`, `paths`), and ordering labels (`loadAfter`, `loadBefore`, `after`, `before`). Use the canonical spellings in new files. FUSE's native `mixinto` schema intentionally exposes only `target`, `sourceFile`, `requires[]`, and `conflictsWith[]`.

### Duplicate-application rule

For each logical object, choose exactly one active path per runtime:

- native FUSE fragment **or** FUSE legacy runtime conversion;
- native RF file/handler **or** RF legacy compatibility mixinto;
- never both representations of the same nodes, segments, spans, industries, scenery, or migrations in one runtime test.

Legacy compatibility is useful for migration and mixed ecosystems. It is not a license to ship three independently discoverable copies of the same graph.

## 2. Complete graph-root namespace union

This table is the master index. Later sections enumerate the fields below each root.

The FUSE graph requires `schemaVersion`, `id`, `name`, and `author` (the graph schema permits a blank author); `modVersion` is recommended but not schema-required and defaults to `1.0.0`.

FUSE uses `schemaVersion` for format migrations separately from `modVersion`. The public schema rejects unknown future schema versions, although the current runtime may attempt a best-effort load; never treat that fallback as authoring compatibility.

FUSE dictionary IDs follow `^[A-Za-z0-9][A-Za-z0-9._:-]*$`. Preserve exact stable IDs when RF allows them; never substitute display names.

| FUSE path | Type | RailForge path | Type | Status and rule |
| --- | --- | --- | --- | --- |
| `$schema` | scalar | None in ordinary graph patch | — | FUSE authoring metadata; omit from RF unless a specific RF schema requests it. |
| `schemaVersion` | scalar | No general graph-root version field | — | Required FUSE metadata: canonical string `"1.0"`; deprecated integer `1` is accepted. Individual RF formats/handlers can have their own `schemaVersion`. |
| `id` | scalar | `Definition.json.id` / patch provenance | scalar | Move package identity to the RF manifest. |
| `name` | scalar | `Definition.json.name` | scalar | Move. |
| `author` | scalar | Possible `Definition.json.author` metadata | scalar | Copy only if accepted by the target RF manifest schema; not established by the supplied minimal RF contract. |
| `modVersion` | scalar | `Definition.json.version` | scalar | Rename/move. |
| `railroaderVersion` | scalar | RF compatibility metadata | scalar | Manual; exact local RF manifest field contract is unavailable. |
| `description` | scalar | Possible manifest/package metadata | scalar | Copy only if the target RF manifest schema supports it; not an ordinary RF graph root. |
| `tags[]` | array | No supplied graph equivalent | — | Package metadata only. |
| `coordinateSpace` | scalar | Implicit RF world coordinates | — | FUSE accepts only `world`; omit in RF. |
| `map` | object | No supplied graph equivalent | — | **FUSE-only** selectable-map declaration. |
| `mixinto` | object | `Definition.json.mixintos` and/or native content folders | object/arrays | **Reshape/manual**. |
| `tracks.nodes` | dictionary | `tracks.nodes` | dictionary | Direct collection; see field renames. |
| `tracks.segments` | dictionary | `tracks.segments` | dictionary | Direct collection; see field/enums. |
| `tracks.spans` | dictionary | `tracks.spans` | dictionary | Direct collection; location representations differ. |
| `tracks.areas` | dictionary | `areas` | dictionary | Move to RF root. |
| `tracks.removals` | object of arrays | Null records under supported RF keyed paths | tombstones | Reshape. RF has no area-removal array; `areas.<id>: null` is supported. |
| `operations.loads` | dictionary | `loads` | dictionary | Move. |
| `operations.industries` | dictionary | `areas.<areaId>.industries` | dictionary | Reshape/nest by area. |
| `operations.loaders` | dictionary | `loaders` convenience root, promoted to `splineys` | dictionary | Rename some fields; handler-backed at runtime. |
| `operations.turntables` | dictionary | `splineys` with a turntable handler | dictionary | Handler/manual; RF fields are a smaller flattened subset. |
| `operations.stations` | dictionary | `splineys` with station-agent handler | dictionary | Handler/reshape. |
| `operations.removals.industries[]` | array | `areas.<area>.industries.<id>: null` | tombstone | Must know the RF area. |
| `world.scenery` | dictionary | `scenery` | dictionary | Move and rename asset identity. |
| `world.spawnPoints[]` | array | Package-root `spawn-points.json` | object with `spawnPoints[]` | **Runtime-only partial reshape**; not an RF game-graph root. RF Company `playerSpawn` remains a different, per-start concept. |
| `world.splineys` | dictionary | `splineys` | dictionary | Handler/manual; families do not share one universal schema. |
| `world.waterSurfaces` | dictionary | No supplied RF contract | — | FUSE-only/native-only. |
| `world.telegraphPoles` | dictionary | `splineys` with `RailForge.MapEditor.TelegraphPoles.TelegraphPoleBuilder` or compatibility handler | dictionary | Handler/manual expansion; RF native handler creates authored poles, while FUSE describes a line. |
| `world.telegraphPoleMovements[]` | array | `splineys.<id>` with `RailForge.MapEditor.TelegraphPoleEdits` | dictionary record | Reshape parallel arrays. |
| `world.mapLabels` | dictionary | `mapLabels` convenience root, promoted to `splineys` | dictionary | Partial mapping; several FUSE style fields lack an RF pair. |
| `world.mapMasks` | dictionary | `splineys` with AMM map-mask handler | dictionary | Handler/geometry reshape. |
| `world.mapTiles` | dictionary | No RF graph root; AMM-compatible package `Maps` tile root exists | package files | Runtime-only/manual package reshape. |
| `world.sceneClones` | dictionary | `mandelas` | dictionary | Reshape; close correspondence for path/source/transforms. |
| `world.suppressBaseScenePaths[]` | array | `mandelas.<path>.enabled: false` | keyed records | Reshape each path. |
| `world.suppressBaseTrackGroups[]` | array | Progression/visibility design | — | No deletion equivalent; manual gating. |
| `world.suppressBaseAreas[]` | array | Progression/visibility design | — | Manual gating, not `areas.<id>: null` unless true deletion is intended. |
| Deprecated `suppressScenePaths[]` | array | Same as scene-path suppression | keyed records | Normalize to the canonical FUSE name before translating. |
| Deprecated `suppressGroups[]` | array | Manual progression/visibility | — | Normalize first. |
| Deprecated `suppressAreas[]` | array | Manual progression/visibility | — | Normalize first. |
| `world.removals` | object of arrays | Null tombstones or handler-specific removal | keyed records | Per-namespace mapping; see removal ledger. |
| `audio.whistles` | dictionary | Manifest mixinto target `whistles`; source root is an array | array | **Runtime-only reshape**; built into RF 1.98 but omitted from the supplied Full Guide. |
| `audio.horns` | dictionary | Manifest mixinto target `horns`; source root is an array | array | **Runtime-only reshape**. |
| `audio.bells` | dictionary | Manifest mixinto target `bells`; source root is an array | array | **Runtime-only reshape**. |
| `progression.progressionId` | scalar | Selected keyed progression / Company-start `progressionId` | scalar | Context-dependent reshape. |
| `progression.sections[]` | array | `progressions.<id>.sections` | dictionary | Convert flat entries to IDs/records under their progression. |
| `progression.progressions` | dictionary | `progressions` | dictionary | Move; normalize arrays/references. |
| `progression.mapFeatures` | dictionary | `mapFeatures` | dictionary | Move; rename some fields/references. |
| `settings` | dictionary | No supplied RF data-only root | — | FUSE-only unless a provider supplies a settings system. |
| `featureRules` | dictionary | No supplied RF data-only root | — | FUSE-only; translate the resulting feature design manually if needed. |
| `editor` | object | No RF runtime graph equivalent | — | Editor state; never translate into release content. |
| `extensions` | arbitrary object | Provider-specific data/handlers | arbitrary | Manual; namespace owner defines meaning. |
| Legacy source `texts` | object/scalar/null dictionary | RF `texts` | dictionary | RF schema is undocumented locally; supplied FUSE converter performs a best-effort conversion to `world.mapLabels`/removals. |
| — | — | `mandelas` | dictionary | Closest FUSE root is `world.sceneClones` plus scene suppression. |
| — | — | Top-level `nodes`, `segments`, `spans` | dictionaries | RF legacy aliases normalized into `tracks`; do not author new files this way. |
| — | — | `features`, nested `progression.*`, exact `MapFeatures`/`Progressions` aliases | dictionaries/wrapper | RF compatibility aliases; prefer lowercase canonical roots. Do not assume arbitrary PascalCase roots work. |

RF also normalizes legacy `mandalas` to `mandelas`; supplying both spellings is rejected as a collision rather than merged. Top-level legacy `nodes`, `segments`, and `spans` are promoted beneath `tracks`. Normalize these compatibility forms before comparing namespace sets.

### FUSE `map` fields

| FUSE field | RF translation |
| --- | --- |
| `displayName` | No ordinary RF graph equivalent. |
| `description` | No ordinary RF graph equivalent. |
| `mapFolder` | No supplied RF selectable-map declaration equivalent. FUSE constrains it to a package-relative map folder. |
| `suppressBaseWorld` | No one-switch RF equivalent; rebuilding or suppressing every required namespace is a separate RF project. FUSE defaults this to `true`. |

### FUSE `mixinto` fields and targets

| FUSE field | RF translation |
| --- | --- |
| `target` | Choose the correct RF native folder/root or manifest mixinto; target strings are not universally shared. |
| `sourceFile` | RF native discovery normally derives source paths from the folder tree. |
| `requires[]`, `conflictsWith[]` | These are fragment-scoped in FUSE: a mismatch skips only that fragment. Preserve the scope with an RF conditional mixinto or separate package where possible. Promote a constraint to the RF package manifest only when blocking the whole package is intended. Each entry has `id`, optional `notBefore`, and optional `notAfter`. |

FUSE target names observed in supplied documentation/runtime are `game-graph`, `progressions`, `scenery`, `splineys`, `horns`, `whistles`, `bells`, plus runtime compatibility targets `hellsbells` and `game-migrations`. RF native discovery commonly replaces the need for a `game-graph` mixinto.

## 3. Merge, arrays, patches, and identity

### Core behavior

| Intent | FUSE authoring | RailForge authoring |
| --- | --- | --- |
| Leave an existing scalar/object unchanged | Omit it or use an explicit FUSE partial form | Omit it. |
| Set/overwrite a scalar | Supply the value | Supply the value. |
| Merge an object | Native object/partial semantics depend on namespace | Ordinary RF objects recursively merge. |
| Replace an array | Supply the complete array, except where an explicit FUSE partial contract says otherwise | Supply the complete ordinary array. |
| Clear an array | Supply `[]` where supported | Supply `[]`, or an RF array instruction with `$remove: true`. |
| Append IDs/elements | FUSE partial helpers such as `trackSpanPatch` | RF `$add` or `$append`, normally in an array program. |
| Remove an element from an array | FUSE namespace-specific patch helper | RF `$remove` by deep-equal value, or `$find` then `$remove`. |
| Replace a keyed subtree | FUSE replacement flag such as `replaceComponents` | RF `{ "$replace": { ...complete value... } }`. |
| Remove keyed record | FUSE removal array or `remove: true` | `null` only at explicitly supported RF paths. |

An ordinary RF array always replaces the existing array. It does not append merely because it appears in a patch. An empty RF object is an empty merge, not deletion.

Important exception: on a FUSE industry component with `partial: true`, a plain `trackSpanIds[]` appends distinct IDs. Translate that intent to an RF `$add`/`$append` program or calculate the final array; copying it as an ordinary RF array would replace the bindings.

### Complete RF array/operator inventory

| Operator | Meaning and constraints | Closest FUSE concept |
| --- | --- | --- |
| `$replace` | Replace whole target value/array. An array-wide `$replace` returns immediately; inside a successful `$find` command, later patch members/`$moveTo` can still run. Root payload must be an object. Array-wide scalar payload produces a one-item array; array payload replaces all entries. | Complete FUSE value or replacement flag. |
| `$remove: true` | Clear an array; after `$find`, remove the matched entry; on a nested object value, remove that JSON property. Root removal is forbidden. Live keyed tombstones remain a separate materializer feature. | Removal array / patch remove. |
| `$remove: value` or array | Remove deep-equal array matches. | Namespace-specific remove list. |
| `$add` | Append one or more entries. | `add`/append intent. |
| `$append` | Append one or more entries; current runtime treats it as append semantics. | `append`. |
| `$find` | Find the first array entry by scalar deep equality or selector `{path,value,comp}`. Multiple selector conditions may be supplied as an array; blank `path` compares the candidate itself. | No universal FUSE equivalent; calculate or express a namespace patch. |
| `$optional: true` | Make a missing `$find` a no-op. It does not make arbitrary properties optional. | No direct equivalent. |
| `$clone` | After `$find`, clone the matched entry to the end and optionally patch the clone. | No general native FUSE operator. |
| `$moveTo` | After `$find`, move the matched entry to an integer index. | No general native FUSE operator. |

`$find.comp` accepts numeric values 0–4 as well as `Equals`/`Equal`, `NotEquals`/`Unequal`, `StartsWith`/`Prefix`, `EndsWith`/`Suffix`, and `Contains`/`Includes` in the inspected runtime. If `$find` misses, `$add`/`$append` on the same instruction can act as a fallback append; without a fallback it errors unless `$optional: true`. A standalone array `$optional: true` is a runtime no-op. These details are **runtime-confirmed**, not fully covered by a supplied standalone graph-patch document.

If any object in an incoming JSON array is an operator command, RF treats the whole array as a program; non-command elements then append rather than causing ordinary full replacement. On a successful `$find`, `$add`/`$append` recursively patches the selected entry or clone; on a miss it is fallback append. `$add: true`/`$append: true` applies the command's ordinary members. Operator and selector names are case-insensitive, but case-alias collisions are rejected. Matched-command order is remove, clone, replace, add/append patch, ordinary members, then move.

## 4. Tracks

### Collection paths

| FUSE | RailForge | Status |
| --- | --- | --- |
| `tracks.nodes.<id>` | `tracks.nodes.<id>` | Direct collection. |
| `tracks.segments.<id>` | `tracks.segments.<id>` | Direct collection. |
| `tracks.spans.<id>` | `tracks.spans.<id>` | Direct collection. |
| `tracks.areas.<id>` | `areas.<id>` | Move to RF root. |

### Node fields

| FUSE field | RailForge field | Status |
| --- | --- | --- |
| `position` | `position` | Same coordinates, but FUSE accepts only `{x,y,z}` while RF accepts `{x,y,z}` or `[x,y,z]`. An RF array must become an object when translating back to FUSE. |
| `rotation` | `rotation` | Same one-way Vector3 shape rule. |
| `flipSwitchStand` | `flipSwitchStand` | Direct. |
| `isDiamond` | No public RF serialized-node field found | FUSE-only/manual. |
| `groupId` | No public RF serialized-node field found | FUSE-only/manual. Do not confuse with segment group. |
| `tags[]` | No public RF serialized-node field found | FUSE-only/manual. |

### Segment fields and values

| FUSE field/value | RailForge field/value | Status |
| --- | --- | --- |
| `startNodeId` | `startId` | Rename. |
| `endNodeId` | `endId` | Rename. |
| `style: standard` | `style: Standard` | Enum conversion. |
| `style: bridge` | `style: Bridge` | Enum conversion. |
| `style: tunnel` | `style: Tunnel` | Enum conversion. |
| `style: yard` | `style: Yard` | Enum conversion. |
| `trackClass: main` | `trackClass: Mainline` | Enum conversion. |
| `trackClass: branch` | `trackClass: Branch` | Enum conversion. |
| `trackClass: industrial` | `trackClass: Industrial` | Enum conversion. |
| `speedLimit` | `speedLimit` | Direct integer; FUSE allows 0–80. |
| `priority` | `priority` | Direct integer. |
| `groupId` | `groupId` | Direct only if an equivalent RF controller exists. Omission preserves a vanilla value; `""` explicitly clears it. |
| `tags[]` | No RF serialized-segment field found | FUSE-only/manual. |
| `gauge` | No RF serialized-segment field found | FUSE-only/manual. |
| `bridgeSupportsSteel` | No RF serialized-segment field found | FUSE-only/manual. |
| `yard` | No separate RF serialized-segment field found | Encode intended result through supported RF style/class or handler; verify visually. |
| `partial` | RF omission/merge semantics | Reshape; remove this control flag. |
| `preserveStyle` | Omit RF `style` | Reshape. |
| `preserveBridgeSupportsSteel` | No corresponding RF field | No translation. |
| `preserveYard` | Omit affected supported RF properties | Manual. |
| `preserveTrackClass` | Omit RF `trackClass` | Reshape. |
| `preserveSpeedLimit` | Omit RF `speedLimit` | Reshape. |
| `preservePriority` | Omit RF `priority` | Reshape. |
| `preserveGroupId` | Omit RF `groupId` | Reshape. |

FUSE gauge values are `Standard`, `Narrow`, `DualGauge`, `DualGauge_L`, `DualGauge_R`, and `DualGauge_T`; legacy aliases include `3ft`, `3 ft`, `ThreeFoot`, `Three Foot`, `Dual`, `Mixed`, and `MixedGauge`. None has a general serialized RF segment field in the supplied material.

The FUSE public schema requires both `startNodeId` and `endNodeId`. Its runtime nevertheless permits a `partial: true` segment with only one endpoint and hydrates the other from the existing segment; omitting both is invalid. RF expresses the same patch intent by supplying only the endpoint that changes. This is a FUSE schema/runtime discrepancy, so schema-valid source authoring should normally keep complete endpoints.

### Span and location fields

| FUSE field/value | RailForge field/value | Status |
| --- | --- | --- |
| `upper` | `upper` | Direct object after location conversion. |
| `lower` | `lower` | Direct object after location conversion. |
| `normalize` | `normalize` | Direct; write it explicitly when preserving FUSE's default `true`. |
| span `groupId` | No RF serialized-span field found | FUSE-only/manual. |
| location `segmentId` | `segmentId` | Direct. |
| location `distance` | `distance` | Direct physical distance. |
| location `normalized` | Calculate `distance` from the final RF segment length | Reshape. |
| `end: A` or `Start` | `end: A` recommended | Rename/normalize. Ordinary RF span runtime accepts `Start`; Company starts require exact `A`. |
| `end: B` or `End` | `end: B` recommended | Rename/normalize. Ordinary RF span runtime accepts `End`; Company starts require exact `B`. |
| location `offset` | No RF core location field | Handler/manual. Do not confuse it with RF runtime aliases for distance. |

A FUSE location requires `segmentId` plus exactly one of `normalized` (0–1) or non-negative `distance`; `end` defaults to A/Start. The upper and lower endpoint arrows must face each other, and zero-length, crossed, same-direction, and out-of-range spans are invalid.

RF `1.98` accepts runtime location aliases beyond the canonical form:

- segment: `segmentId`, `segmentID`, `segment_id`, `segment`, `trackSegmentId`, `trackSegment`, `spanSegmentId`, `id`;
- distance: `distance`, `offset`, `trackDistance`, `position`, `location`, `meters`;
- end: `end`, `terminal`, `side`, `trackEnd`.

Prefer `segmentId`, `distance`, and `end`. FUSE location `offset` is an additive offset and must never be copied into RF's compatibility `offset` distance alias.

### Area fields

| FUSE `tracks.areas.<id>` | RF `areas.<id>` | Status |
| --- | --- | --- |
| `name` | `name` | Direct. |
| `position` | `localPosition` canonical in RF authoring; runtime also accepts `position` | Rename/alias. |
| `radius` | `radius` | Direct. |
| `tagColor[]` | `tagColor[]` | Direct numeric array. Recommend 3–4 normalized components; RF only requires at least three numbers, defaults alpha to 1, and ignores extras. |
| `order` | `order` | Direct signed integer. |
| `spanIds[]` | No RF serialized-area field | FUSE-only/manual. |
| `groupId` | No RF serialized-area field | FUSE-only/manual. |

## 5. Loads

| FUSE `operations.loads.<id>` | RF `loads.<id>` | Status |
| --- | --- | --- |
| `name` | `description` | Rename. This is the dependable native RF `Load` display field. |
| — | Compatibility inputs `displayName`, `name`, `label`, or `title` | RF runtime recognizes these for provider compatibility, but the current native `Load` has no matching member; do not use them instead of `description`. |
| — | Runtime alias `tooltip` for `description` | Compatibility alias; prefer `description`. |
| — | Compatibility inputs `plural` / `pluralName` | RF runtime attempts provider-compatible reflective setters, but the current native `Load` has no plural member; not dependable native authoring. |
| `units` | `units` | Direct: `Pounds`, `Gallons`, or `Quantity`. |
| `density` | `density` | Direct. |
| `unitWeightInPounds` | `unitWeightInPounds` | Direct; RF runtime also accepts `unitWeightInQuantity`. |
| `importable` | `importable` | Direct. |
| `payPerQuantity` | `payPerQuantity` | Direct. |
| `costPerUnit` | `costPerUnit` | Direct. |
| `carTypeFilter` | No documented RF load-catalog field | Move the filter to relevant components/deliveries if that preserves intent; otherwise manual. |
| `emptyCarType` | No documented RF load-catalog field | Manual/provider-specific. |
| `loadedCarType` | No documented RF load-catalog field | Manual/provider-specific. |
| `icon` | No documented RF load-catalog field | Manual/provider-specific. |
| `fields` | RF load materializer rejects/warns on unknown fields | Do not copy blindly; provider or RF change required. |

RF does not support `loads.<id>: null` as a real load deletion in the supplied implementation.

RF requires finite non-negative `density` and `unitWeightInPounds`; `payPerQuantity` and `costPerUnit` must be finite (the runtime technically accepts negative values). `importable` accepts boolean or 1/0. `units` accepts `Pounds`, `Gallons`, `Quantity`, or a valid numeric enum value. Unsupported load members warn and are ignored while valid fields continue.

## 6. Industries and components

### Industry fields and nesting

| FUSE `operations.industries.<industryId>` | RF `areas.<areaId>.industries.<industryId>` | Status |
| --- | --- | --- |
| `name` | `name` | Direct. |
| `areaId` | Becomes parent dictionary key | Reshape. Find the actual RF area for vanilla patches. |
| `order` | No RF serialized-industry field found | FUSE-only/manual. |
| `position` | `localPosition` in public examples; runtime also accepts `position` | Rename/alias. |
| `rotation` | No RF serialized-industry field found | FUSE-only/manual. |
| `usesContract` | `usesContract` | Direct. |
| `mergeComponents: true` | Ordinary RF `components` object merge | Reshape; omit the flag. |
| `replaceComponents: true` | `components: { "$replace": { ...complete set... } }` | Reshape. |
| `components` | `components` | Direct dictionary after every component is translated. |

RF component `$replace` cleanup is transactional: stale runtime components are removed only after complete successful materialization. If the replacement is incomplete or invalid, omitted live components may be retained rather than partially destroyed. Treat any warning as a failed replacement and inspect the final live set.

### Canonical component-type translation

| FUSE canonical type | Accepted FUSE legacy/runtime aliases | RailForge type | Status |
| --- | --- | --- | --- |
| `loader` | `industryloader`, `model.ops.industryloader`, `model.opsnew.industryloader`; legacy `load` | `Model.Ops.IndustryLoader` | Direct type mapping. |
| `unloader` | `industryunloader`, `model.ops.industryunloader`, `model.opsnew.industryunloader`; legacy `unload` | `Model.Ops.IndustryUnloader` | Direct type mapping. |
| `formulaic` | `formulaicindustrycomponent`, `model.ops.formulaicindustrycomponent`, `model.opsnew.formulaicindustrycomponent`; legacy `formula` | `Model.Ops.FormulaicIndustryComponent` | Direct type mapping; terms may reshape. |
| `repairTrack` | `repair-track`, `model.ops.repairtrack`, `model.opsnew.repairtrack`; legacy `repair` | `Model.Ops.RepairTrack` | Direct type mapping. |
| `teamTrack` | `team-track`, `model.ops.teamtrack`, `model.opsnew.teamtrack`; legacy `team_track` | `Model.Ops.TeamTrack` | Direct type mapping; RF accepts profiles as a dictionary or array. |
| `interchange` | `model.ops.interchange`, `model.opsnew.interchange`, `interchangereloader.ops.interchangereloader` | `Model.Ops.Interchange` | Direct type mapping; verify external compatibility aliases. |
| `interchangedLoader` | `interchanged-loader`, `model.ops.interchangedindustryloader`, `model.opsnew.interchangedindustryloader` | `Model.Ops.InterchangedIndustryLoader` | Direct type mapping. |
| `interchangedUnloader` | `interchanged-unloader`, `model.ops.interchangedindustryunloader`, `model.opsnew.interchangedindustryunloader` | External `Model.Ops.InterchangedIndustryUnloader` or `Model.OpsNew.InterchangedIndustryUnloader` if loaded | Provider required; no native installed concrete class was established. |
| `teleportLoading` | `teleport-loading`, `teleportloadingindustry`, `model.ops.teleportloadingindustry`, `model.opsnew.teleportloadingindustry` | `Model.Ops.TeleportLoadingIndustry` | Direct type mapping. |
| `progression` | `progressionindustry`, `progression-industry`, `progressionindustrycomponent`, `model.ops.progressionindustrycomponent`, `model.opsnew.progressionindustrycomponent` | `Model.Ops.ProgressionIndustryComponent` | Runtime-confirmed mapping. |
| `passengerStop` | `passenger-stop`, `paxstationcomponent`, `alinasmapmod.paxstationcomponent`, `alinasmapmod.stations.paxstationcomponent`; legacy `passenger_stop` | `RailForge.PassengerStation` recommended; historical `AlinasMapMod.PaxStationComponent` and `AlinasMapMod.Stations.PaxStationComponent` accepted | RF special proxy/compatibility contract. |
| Fully qualified custom type | Type owner decides aliases | Same loaded concrete RF-compatible type, if available | Provider/manual; test in both runtimes. |

FUSE custom component aliases are case-insensitive:

| FUSE normalized custom type | Accepted inputs |
| --- | --- |
| `ConfusingSupplements.IndustryComponents.CaptiveConversionLoader` | `captiveconversionloader`, `captive-conversion-loader`, `confusingsupplements.captiveconversionloader`, full normalized name |
| `ConfusingSupplements.IndustryComponents.CaptiveConversionUnloader` | `captiveconversionunloader`, `captive-conversion-unloader`, `confusingsupplements.captiveconversionunloader`, full normalized name |
| `ConfusingSupplements.IndustryComponents.Pay4Resource` | `pay4resource`, `pay-for-resource`, `confusingsupplements.pay4resource`, full normalized name, `adrfdr.pay4resource` |
| `ConfusingSupplements.IndustryComponents.Empty` | only `confusingsupplements.empty` or the full normalized name |

RF `1.98` embeds concrete compatibility implementations for all four normalized Confusing Supplements types. Treat that support as **runtime-only** and re-check it on RF updates; other custom types still require their owning provider.

| Embedded RF compatibility type | Runtime-confirmed fields |
| --- | --- |
| `ConfusingSupplements.IndustryComponents.CaptiveConversionLoader` | `loadId`, `convertedLoadId`, `carTransferRate`, optional `title` |
| `ConfusingSupplements.IndustryComponents.CaptiveConversionUnloader` | Loader fields plus `maxStorage` |
| `ConfusingSupplements.IndustryComponents.Pay4Resource` | `loadId`, `carTransferRate`, `hours: [start,end]`, `costPerUnit`, `fillPercentage`, `bookReasons[]`, optional `title`; exactly two hour values, each 0–24 |
| `ConfusingSupplements.IndustryComponents.Empty` | Empty compatibility component |

Captive Loader requires non-empty resolvable `loadId` and `convertedLoadId` with equal LoadUnits plus positive `carTransferRate`; Captive Unloader additionally requires positive `maxStorage`. Pay4Resource requires a resolvable load, positive transfer rate, exactly two hour values in 0–24, and non-negative cost; optional fill percentage must be a valid fraction. Unknown fields on these compatibility payloads are captured/preserved rather than necessarily rejected.

RF runtime contains native `Model.Ops.LoadExporter`, `Model.Ops.LoadImporter`, and `Model.Ops.SimplePassengerStop`, but the supplied RF guide does not provide complete authoring contracts for them. They are **RF runtime-only** for this matrix, not recommended automatic FUSE conversions.

### Component validity before translation

| FUSE component family | Minimum native FUSE requirement | RF consequence |
| --- | --- | --- |
| New, non-partial, non-removal component | `type` and `name` | New RF component needs a loaded, concrete `Model.Ops.IndustryComponent` type; an existing RF component may omit type while patching. |
| `loader`, `unloader`, `repairTrack`, `teamTrack`, `interchange`, `interchangedLoader`, `interchangedUnloader`, `progression` | At least one `trackSpanIds` entry | Preserve the FUSE source requirement and translate spans; RF enforcement differs by concrete type as noted below. |
| `passengerStop` | `passengerStopId` and `timetableCode`; a legacy virtual stop may be spanless | Translate to RF passenger proxy plus station-agent/reference design. |
| `formulaic` | At least one of `inputTermsPerDay` or `outputTermsPerDay` | Formulaic does not require spans in RF. |
| `teamTrack` | At least one `teamProfiles` entry | Translate every profile. |
| `teleportLoading` | At least one of `inputSpanIds` or `outputSpanIds` | Translate to `inputSpans`/`outputSpans`; RF's loader base also requires usable `trackSpans`, so derive/author the intended base bindings explicitly. |

For both runtimes, `carTypeFilter` is a comma-separated token list used verbatim: write `FB,XM`, not `FB, XM`. A token ending in `*` is a prefix match.

RF's exact live-span guard covers `IndustryLoaderBase` (including Loader and TeleportLoading), `IndustryUnloader`, `InterchangedIndustryLoader`, `Interchange`, `TeamTrack`, and `RepairTrack`; Formulaic is explicitly exempt. It does not apply that same guard to ProgressionIndustryComponent or an external InterchangedIndustryUnloader. A guarded non-empty list resolving to zero live spans rejects, rolls back, or disables the component; `[]` explicitly detaches and omission preserves existing bindings.

### Complete common component field ledger

| FUSE component field | RailForge field/form | Status |
| --- | --- | --- |
| `remove: true` | `components.<id>: null` | Reshape; supported at the keyed RF component path. |
| `partial: true` | Ordinary RF merge with omitted fields | Reshape; remove the flag. |
| `type` | `type` | Translate alias to an RF concrete/proxy type. Existing RF component IDs cannot change live type in place. |
| `name` | `name` | Direct; runtime also accepts `displayName`/`label`. |
| `carTypeFilter` | `carTypeFilter` | Direct. |
| `loadId` | `loadId` | Direct; runtime alias `load` also exists. |
| `convertedLoadId` | `convertedLoadId` on embedded CaptiveConversion components | Direct for those types; no general RF common field. |
| `sharedStorage` | `sharedStorage` | Direct. |
| `storageChangeRate` | `storageChangeRate` | Direct canonical form. RF runtime has many production/consumption aliases; do not emit aliases unnecessarily. |
| `maxStorage` | `maxStorage` | Direct; runtime alias `storageMax`. |
| `carTransferRate` | `carTransferRate` | Direct canonical form; type-specific load/unload aliases exist. |
| `costPerUnit` | `costPerUnit` on embedded Pay4Resource | Direct for that type; no general RF common field. |
| `notBeforeHour` | Pay4Resource `hours[0]` | Reshape for that type. |
| `notAfterHour` | Pay4Resource `hours[1]` | Reshape for that type. |
| `fillPercentage` | `fillPercentage` on embedded Pay4Resource | Direct for that type. |
| `title` | `title` on embedded CaptiveConversion/Pay4Resource | Direct for those types; otherwise do not substitute `name` without checking semantics. |
| `orderAroundEmpties` | `orderAroundEmpties` | Direct. |
| `orderAroundLoaded` | `orderAroundLoaded` | Direct. |
| `idealCars` | `idealCars` | Direct numeric/float value for TeamTrack. |
| `canOverhaul` | `canOverhaul` | Direct for RepairTrack. |
| `passengerStopId` | Passenger component identity / station-agent `passengerStop` reference | Context-dependent reshape; not an ordinary RF passenger field. |
| `timetableCode` | `timetableCode` | Direct for RF passenger station. |
| `basePopulation` | `basePopulation` | Direct for RF passenger station. |
| `branch` | `branch` | Direct; colon-separated RF branch syntax has additional junction semantics. |
| `carLoadPeriod` | `carLoadPeriod` | Direct for TeleportLoading. |
| `carLengthFeet` | `carLengthFeet` | Direct for TeleportLoading. |
| `trackSpanIds[]` | `trackSpans[]` | Rename. Omitted preserves RF bindings; `[]` explicitly detaches. |
| `bookReasons[]` | `bookReasons[]` on embedded Pay4Resource | Direct for that type; no general RF common field. |
| `inputSpanIds[]` | `inputSpans[]` | Rename for TeleportLoading. |
| `outputSpanIds[]` | `outputSpans[]` | Rename for TeleportLoading. |
| `neighborIds[]` | `neighborIds[]` | Direct for passenger station; RF runtime alias `neighbors`. |
| `branchDefinitions[]` | RF passenger `branches[]` | Rename/reshape each nested branch. |
| `trackSpanPatch` | RF array operator program on `trackSpans`, or calculate final array | Reshape. See operation mapping below. |
| `inputTermsPerDay` | `inputTermsPerDay` | Direct dictionary `loadId → number`; RF runtime also accepts term arrays and aliases. |
| `outputTermsPerDay` | `outputTermsPerDay` | Direct dictionary. |
| `teamProfiles` dictionary | RF accepts `teamProfiles` as a dictionary or array | The FUSE dictionary can remain a dictionary; its key becomes the tag. If converted to an array, add `tag`/`id` to each entry. |
| `fields` arbitrary object | `data`/extra-data only when the target component/provider supports it | Manual; RF nested payload aliases are `data`, `componentData`, `settings`, and `config`. Never assume every key is recognized. |

### `trackSpanPatch` operations

| FUSE operation | RailForge equivalent |
| --- | --- |
| `add[]` | Array program entries using `$add`, or calculate a final `trackSpans` array. |
| `append[]` | `$append`/`$add`. |
| `prepend[]` | No one-token prepend; use `$find`/`$moveTo` where safe or emit the complete final array. |
| `insert[]` | Current FUSE runtime treats this as append-phase distinct additions, not positional insertion. Use RF `$add`/`$append` or emit the calculated final array. |
| `replace[]` | Ordinary complete array or `$replace`. |
| `remove[]` | `$remove` values or `$find` + `$remove`. |

### Nested team and passenger records

| FUSE nested field | RailForge field | Status |
| --- | --- | --- |
| `teamProfiles.<tag>.loadId` | dictionary entry `loadId`, or array entry `loadId` plus `tag` | Direct in dictionary form; reshape only if choosing RF array form. |
| `teamProfiles.<tag>.isExport` | `isExport` | Direct. |
| `teamProfiles.<tag>.loadingTimeDays` | `loadingTimeDays` | Direct. |
| `teamProfiles.<tag>.carTypeFilter` | `carTypeFilter` | Direct. |
| branch `branch` | `branches[].branch` | Direct. |
| branch `traverseTimeToNext` | `branches[].traverseTimeToNext` | Direct. |
| branch `mapFeature` | `branches[].mapFeature` | Direct reference after ID validation. |
| branch `intermediates` dictionary | RF passenger intermediates shape | Runtime/provider-specific reshape; preserve each `code` and `traverseTimeToNext`. |
| intermediate `code` | `code` | Direct where supported. |
| intermediate `traverseTimeToNext` | `traverseTimeToNext` | Direct where supported. |
| — | RF passenger `timetableOrder` / `order` | RF-only runtime extension. |
| — | RF interchange `disabled` / `interchangeDisabled` | RF-only component field. |
| — | RF interchange/interchanged loader `progressionDisabled` | RF-only component field. |
| — | RF interchanged loader `ledgerCategory` / `ledger` | RF-only component field. |

RF passenger timetable insertion requires an explicit authored `branch` and final area `order`. Native support is one main occurrence plus at most one junction duplicate; `branches[].intermediates` is deferred unless explicit timetable-topology support exists. Duplicate timetable codes preserve the earlier stable owner and fail closed rather than silently replacing it.

On a FUSE `partial: true` component, non-empty `trackSpanIds[]` appends unique IDs, while `trackSpanIds: []` does nothing. To clear, use `trackSpanPatch.replace: []`. Existing IDs named by `prepend[]` are not moved; only new distinct IDs are prepended.

### RF component compatibility aliases

Prefer the canonical field names in the main ledger. RF `1.98` additionally accepts this complete compatibility surface:

| Concept | Accepted RF inputs |
| --- | --- |
| Component nested payload | `data`, `componentData`, `settings`, `config` |
| Output/production rate | `storageChangeRate`, `productionRate`, `storageProductionRate`, `outputRate`, `loadRate`, `unitsPerDay`, `perDay`, `loadsPerDay`, `dailyRate`, `ratePerDay`, `amountPerDay`, `quantityPerDay`, `productionPerDay`, `producePerDay`, `supplyPerDay`, `outputUnitsPerDay`, `loadUnitsPerDay` |
| Input/consumption rate | `storageChangeRate`, `storageConsumptionRate`, `consumptionRate`, `inputRate`, `unloadRate`, `unitsPerDay`, `perDay`, `loadsPerDay`, `dailyRate`, `ratePerDay`, `amountPerDay`, `quantityPerDay`, `demandPerDay`, `consumePerDay`, `consumptionPerDay`, `inputUnitsPerDay`, `unloadUnitsPerDay` |
| Rate tier wrappers | Objects: `tiers`, `tierRates`, `tieredRates`, `ratesByTier`, `progressionRates`, `levels`, `stages`; arrays: `storageChangeRates`, `productionRates`, `outputRates`, `supplyRates`, `consumptionRates`, `inputRates`, `demandRates` |
| Empty-car ordering | `orderAroundEmpties`, `orderEmpties`; unloader also accepts `orderAwayEmpties`, `orderLoads` |
| Loaded-car ordering | `orderAroundLoaded`, `orderAwayLoaded`, `orderLoaded` |
| Loader transfer rate | `carTransferRate`, `carLoadRate`, `loadCarRate` |
| Unloader transfer rate | `carTransferRate`, `carUnloadRate`, `unloadCarRate` |
| Formulaic input root | `inputTermsPerDay`, `inputTerms`, `inputs`, `input` |
| Formulaic output root | `outputTermsPerDay`, `outputTerms`, `outputs`, `output` |
| Formulaic term load key | `loadId`, `load`, `id`, `identifier` |
| Formulaic term quantity key | `unitsPerDay`, `perDay`, `rate`, `amountPerDay`, `storageChangeRate`, `quantity` |
| Team profile root | `teamProfiles`, `profileEntries`, `entries` |
| Team entry fields | `tag`/`id`, `loadId`/`load`, `carTypeFilter`, `isExport`/`export`, `loadingTimeDays`/`loadingTime` |
| Passenger stop identity | `id`, `identifier`, `passengerStop` |
| Passenger name/code | `name`/`displayName`, `timetableCode`/`code` |
| Passenger load | `loadId`/`load`; provider forms `Load`, `PassengerLoad`, `passengerLoad`, `LoadId` |
| Passenger car filter | `carTypeFilter`/`carTypes`/`carTypeFilterForLoad`; wrapper/value forms `CarTypeFilter`, `carTypeFilter`, `filter`, `pattern`, `value` |
| Passenger population/spans | `basePopulation`/`population`, `trackSpans`/`spans` |
| Passenger branch entry | `branch`/`name`/`id`, `traverseTimeToNext`/`traverseTime`, `mapFeature`/`mapFeatureId`, `intermediates` |
| Passenger timetable order | `timetableOrder`, `order` |

These aliases are **runtime compatibility**, not a recommendation to mix naming styles in new RF graphs.

FUSE's legacy converter likewise normalizes these field aliases before native validation: `type`/`Type`; `trackSpanIds`/`trackSpans`/`spans`; `loadId`/`LoadId`/`load`; `convertedLoadId`/`convertedLoad`; `inputSpanIds`/`inputSpans`; `outputSpanIds`/`outputSpans`; `passengerStopId`/`passengerStop`; `neighborIds`/`neighbors`; and `fields`/`extraData`/`ExtraData`. New FUSE files should use the canonical first name in each group.

Legacy FUSE list directives accept `$add`/`add`, `$append`/`append`, `$insert`/`insert`, `$prepend`/`prepend`, `$replace`/`replace`, `$remove`/`remove`, and `$delete`/`delete`; delete normalizes to remove. These are legacy conversion inputs, not the RF operator contract merely because some spellings resemble it.

## 7. Loaders, stations, and turntables

These are world facilities/visual builders. They are distinct from operational industry components.

### Loader visual

| FUSE `operations.loaders.<id>` | RF `loaders.<id>` / promoted spliney | Status |
| --- | --- | --- |
| `position` | `position` or `localPosition` | Direct coordinates; prefer `position`. |
| `rotation` | `rotation` or `localRotation` | Direct coordinates; prefer `rotation`. |
| `prefab` | `prefab` | Direct if URI resolves in RF. |
| `industryId` | `industry` | Rename. |
| implicit native handling | Handler `AlinasMapMod.Loaders.LoaderBuilder` | RF promoter supplies canonical handler for convenience root. |

RF also accepts historical `AlinasMapMod.LoaderBuilder`. A visible loader does not create freight logic; the industry still needs a Loader/Unloader/etc. component bound to live spans.

During RF convenience-root promotion, a null loader becomes a spliney tombstone, an explicit `splineys.<same-id>` record wins, and malformed non-object/non-null entries are skipped.

### Station agent

| FUSE `operations.stations.<id>` | RF station-agent spliney | Status |
| --- | --- | --- |
| `position` | `position` or `localPosition` | Direct coordinates; prefer `position`. |
| `rotation` | `rotation` or `localRotation` | Direct coordinates; prefer `rotation`. |
| `prefab` | `prefab` | Direct if RF-compatible. |
| `passengerStopId` | `passengerStop` | Rename. |
| implicit native handling | `handler: "AlinasMapMod.Stations.StationAgentBuilder"` | Add handler. Historical `AlinasMapMod.StationAgentBuilder` also accepted. |

Documented RF station prefab URIs include `vanilla://flagStopStation`, `vanilla://brysonDepot`, `vanilla://dillsboroDepot`, `vanilla://dillsboroStation`, and `vanilla://southernCombinationDepot`, in addition to appropriate `path://`/`scenery://` sources. A station prefab must contain exactly one `Model.StationAgent`. The FUSE public schema makes `passengerStopId` optional, but its runtime validator requires it; follow the runtime and supply it.

### Turntable

| FUSE field | RF turntable-handler field | Status |
| --- | --- | --- |
| `position` | `position` | Direct. |
| `rotation` | `rotation` | Direct. |
| `radius` | `radius` | FUSE requires >0; RF requires 5–50. Values outside RF range need redesign. |
| `subdivisions` | `subdivisions` | FUSE allows integer 4–32 and defaults to 16; RF requires an even integer 2–50. An odd FUSE value is not directly portable. |
| `legacyIdentifier` | `id`/`identifier` is not the same semantic contract | No direct mapping; manual. |
| `roundhouse.stalls` | `roundhouseStalls` | Flatten. FUSE requires at least 1; RF permits 0 through `subdivisions`. |
| `roundhouse.trackLength` | `roundhouseTrackLength` | Flatten. FUSE requires >0 and defaults to 46; RF requires 1–500. |
| `roundhouse.startAngle` | No verified RF handler field | Manual. |
| `roundhouse.stallAngle` | No verified RF handler field | Manual. |
| `roundhouse.startPrefab` | `startPrefab` | Flatten; direct when source resolves. RF default `vanilla://roundhouseStart`. |
| `roundhouse.endPrefab` | `endPrefab` | Flatten; direct when source resolves. RF default `vanilla://roundhouseEnd`. |
| `roundhouse.stallPrefab` | `stallPrefab` | Flatten; direct when source resolves. RF default `vanilla://roundhouseStall`. |
| `visuals.pitAssetIdentifier` | No supplied RF handler field | Manual/provider-specific. |
| `visuals.pitPosition`, `pitRotation`, `pitScale` | No supplied RF handler fields | Manual/provider-specific. |
| `visuals.bridgeAssetIdentifier` | No supplied RF handler field | Manual/provider-specific. |
| `visuals.bridgePosition`, `bridgeRotation`, `bridgeScale` | No supplied RF handler fields | Manual/provider-specific. |
| `visuals.bridgeTrackEnabled` | No supplied RF handler field | Manual/provider-specific. |
| `visuals.bridgeTrackGauge`, `bridgeTrackLength`, `bridgeTrackYOffset` | No supplied RF handler fields | Manual/provider-specific. `bridgeTrackGauge` is metres and defaults to 1.435; it is not the segment gauge enum. |
| `controllerType`, `interactionRadius` | No supplied RF handler fields | Manual/provider-specific. |
| — | `defaultStopIndex` | RF-only builder field. |
| — | `bridgeGroupId` | RF-only builder field. |
| — | `deckStyle` | RF-only builder field; default `Stock (original)`. |

Use `handler: "AlinasMapMod.Turntable.TurntableBuilder"` for the verified RF compatibility family, but treat it as **runtime-only** until the missing AMM contract is available.

RF validates all authored turntable transforms as finite. Its native builder accepts lower- and PascalCase forms of `radius`, `subdivisions`, `defaultStopIndex`, `bridgeGroupId`, `roundhouseStalls`, `roundhouseTrackLength`, `position`, `rotation`, `deckStyle`, `startPrefab`, `endPrefab`, and `stallPrefab`.

RF preflight requires `radius`, `subdivisions`, `roundhouseStalls`, `roundhouseTrackLength`, `position`, and `rotation`. Even when the FUSE turntable has no roundhouse, emit `roundhouseStalls: 0` and a valid RF `roundhouseTrackLength` from 1–500.

### RF infrastructure override handler

RF's native infrastructure editor persists the reserved record `splineys.RF_MapInfrastructureEdits` with `handler: "RailForge.MapEditor.InfrastructureOverrides"`, current writer `schemaVersion: 2`, and keyed/object `crossings` plus `turntables`. This is distinct from the ordinary turntable builder and from the legacy crossings sidecar adapter.

| RF crossing override field | Contract |
| --- | --- |
| `enabled` | Enable/disable crossing. |
| `segmentId`, `distance`, `end` | Exact track location; distance ≥0 and end A/B. |
| `width` | 0.5–100. |
| `whistleDistance` | 0 or 25–2,000. |
| `operation`, entry `schemaVersion` | A missing crossing requires `operation: "create"` at schema version ≥2 and is cloned from a complete current-map crossing template; creation is not universally portable. |
| `rotationOffset` | Model rotation adjustment. |
| `modelIdentifier`, `modelScale`, `hideBaseModel` | Model override; identifier ≤256 characters, scale 0.01–100, and `hideBaseModel` requires a replacement model. |

| RF turntable override field | Contract |
| --- | --- |
| `position`, `rotation` | Finite transforms. |
| `radius` | 5–50, corresponding to diameter 10–100. |
| `subdivisions` | Even integer 2–50. |
| `defaultStopIndex` | 0 through `subdivisions - 1`. |
| `roundhouseStalls` | 0 through subdivisions. |
| `roundhouseTrackLength` | 1–500. |
| `firstStallWorldAngle` | Any finite runtime value; editor input is 0–360. RF normalizes after subtracting one stall step (`360 / subdivisions`). |
| `bridgeGroupId` | At most 128 characters. |
| `modelIdentifier`, `modelScale`, `hideBaseModel` | Same model rules as crossing override. |
| `requiresFullReload`, `transformSchemaVersion` | Current transform writer emits `true` and `1` respectively. |

There is no one-to-one FUSE infrastructure-override root. Translate only those fields that correspond to a FUSE crossing/turntable design and keep the RF record runtime-specific.

## 8. World content

### Scenery

| FUSE `world.scenery.<id>` | RF `scenery.<id>` | Status |
| --- | --- | --- |
| `assetIdentifier` | `modelIdentifier` | Rename; verify exact RF asset ID. |
| deprecated `model` | `modelIdentifier` | Normalize first. |
| `position` | `position` | Direct. |
| `rotation` | `rotation` | Direct. |
| `scale` | `scale` | Direct. |
| `anchorSpanIds[]` | No RF core scenery field | Manual; bake the final transform or use a provider. |
| — | RF custom/extra data where recognized | RF/provider-specific; FUSE scenery has no general arbitrary-fields bag. |

Do not strip `asset://` mechanically unless the RF asset viewer confirms that the raw identifier is correct.

RF `1.98` runtime aliases are `modelIdentifier`/`identifier` for asset identity and `active`/`enabled`/`visible` for visibility. Prefer `modelIdentifier`; visibility has no native FUSE scenery field and may require `world.sceneClones`/suppression design. Conversely, FUSE runtime recognizes `definitionIdentifier` as an `assetIdentifier` compatibility alias, but the public FUSE schema rejects it, so do not use it in new native FUSE files.

### Spawn points

FUSE `world.spawnPoints` is an array. Each entry contains `name`, `position`, optional `rotation`, `radius` (default 3), and integer `priority`.

RF `1.98` provides the closest data-only format at package root `spawn-points.json`. Discovery requires the package file to be anchored by an editable-file mixinto identifier for `game-graph`; an arbitrary unindexed standalone file is not enough.

| FUSE | RF spawn-point file | Status |
| --- | --- | --- |
| root `world.spawnPoints[]` | closed root object `{ title?, spawnPoints: [...] }` | Move/reshape. |
| entry `name` | `name` | Direct; RF maximum 80 characters. |
| entry `position {x,y,z}` | `position {x,y,z}` | Direct; each coordinate magnitude must be ≤1,000,000. |
| entry `rotation` | No RF field | FUSE-only. |
| entry `radius` | No RF field; RF registers radius 3 | Difference/allowlist. |
| entry `priority` | No RF field; RF registers `Int32.MinValue` | Difference/allowlist. |

RF limits are 1 MiB and 256 entries per file, 128 scanned files, and 1,024 total points. Earlier indexed package wins a duplicate name; a collision with a non-RF point of the same name at a different position is skipped. This is a global travel/location format, not an RF Company-start `playerSpawn` replacement.

The root is case-sensitive and closed to `title` plus required `spawnPoints` (which may be empty); each entry is closed to required `name` and `position`, and position is closed to x/y/z. Title trims to at most 120 characters with no controls. Names trim non-empty to at most 80, reserve `ds` case-insensitively and prefix `---`, and forbid quote, slash, backslash, or control characters. Within-file duplicate names compare case-insensitively. JSON depth is limited to 16 and trailing content is rejected.

### Generic FUSE spliney fields

| FUSE field | RF translation |
| --- | --- |
| `type` | Select a compatible RF `handler`; there is no universal value rename. |
| `profile` | Handler-specific `profile`/`splineProfile` when supported. |
| `style` | Handler-specific `style`. |
| `offsetY` | Handler-specific `offsetY`/`yOffset`. |
| `headStyle`, `tailStyle` | Verified for AutoTrestle-family runtime handler only. |
| `assetIdentifier` or `prefab` | Handler-specific asset source. |
| `spacing` | Object-line/modular-scenery handler-specific. |
| `instanceScale` | Handler-specific. |
| `rotationOffset` | Handler-specific. |
| `lateralOffset` | Handler-specific. |
| `verticalOffset` | Handler-specific. |
| `snapToTerrain` | Handler-specific. |
| `alignToSlope` | Handler-specific. |
| `placeAtEnd` | Handler-specific. |
| `maximumInstances` | Handler-specific. |
| `points[]` | Many RF handlers accept points, but their contract varies. A handlerless RF record with non-empty `points` attempts `RailForge.ExactSplineyPointReplacement`, which only supports one unambiguous existing AutoTrestle or RiverPath; it is not a general FUSE spliney builder. |
| point `position`, `rotation`, `width` | Direct only when the selected handler defines the same point fields. |

FUSE object-line families (`objectLine`, `object-line`, `fence`, `retainingWall`) require at least two points and exactly one of `assetIdentifier` or `prefab`; `spacing` must be positive and `maximumInstances` must be 1–4096. A legacy `StrangeCustoms.FlowyThingBuilder` converted to a river defaults a missing FUSE `offsetY` to -0.1.

Complete FUSE spliney `type` values are:

| FUSE type | Closest RF family | Translation |
| --- | --- | --- |
| `river` | `RiverSplineBuilder` or Flowy handler with river profile | Handler/manual. |
| `road` | Flowy/road handler | Handler/manual. |
| `terrainRoad` | Terrain/road handler | Handler/manual. |
| `trestle` | `AutoTrestle`/`AutoTrestleBuilder` | Handler/manual. |
| `waterfall` | Flowy/custom handler | Handler/manual. |
| `objectLine` | ModularScenery/object-line handler | Handler/manual. |
| `object-line` | Same as `objectLine` | Normalize alias, then handler/manual. |
| `fence` | Object-line/modular-scenery handler | Handler/manual. |
| `retainingWall` | Object-line/modular-scenery handler | Handler/manual. |

RF `1.98` also recognizes DKW/KRE, CLB shop, crossing, telegraph, turntable, road, modular-scenery, and dynamically loaded `BuildSpliney` handler families. Broad name matching is implementation compatibility, not a stable public schema. Require the owning mod/provider and test exact handler data.

### RF handler and namespace ledger

The following names are recognized by RF `1.98`. “Normalizes to” describes compatibility migration, not a requirement to author the historical form.

| Accepted/historical RF handler | Preferred or effective RF handler/family | Notes |
| --- | --- | --- |
| `AlinasMapMod.LoaderBuilder` | `AlinasMapMod.Loaders.LoaderBuilder` | Embedded loader source port. |
| `AlinasMapMod.MapLabelBuilder` | `AlinasMapMod.Map.MapLabelBuilder` | Embedded label source port. Additional accepted names: `AlinasMapMod.MapLabels.MapLabelBuilder`, `AlinasMapMod.Labels.MapLabelBuilder`. |
| `AlinasMapMod.StationAgentBuilder`, any name containing `.Stations.StationAgentBuilder`, or ending `.StationAgentBuilder` | `AlinasMapMod.Stations.StationAgentBuilder` family | Embedded station source port/compatibility matcher. |
| `AlinasMapMod.TelegraphPoleBuilder` | `AlinasMapMod.TelegraphPoles.TelegraphPoleBuilder` | Telegraph-family compatibility. |
| `AlinasMapMod.TelegraphPoleMover` | `AlinasMapMod.TelegraphPoles.TelegraphPoleMover` | Telegraph-family compatibility. |
| `AlinasMapMod.MapMaskBuilder` | `AlinasMapMod.Map.MapMaskBuilder` | Embedded map-mask source port. |
| `AlinasMapMod.Turntable.TurntableBuilder` | same | Embedded/compatibility turntable family. |
| `RailForge.ExactSplineyPointReplacement` | same | Exact replacement only for one unambiguous existing AutoTrestle or RiverPath; handlerless non-empty `points[]` attempts the same restricted path. |
| `RailForge.MapEditor.TelegraphPoleEdits` | same | RF-owned parallel-array pole edit handler. |
| `RailForge.MapEditor.TelegraphPoles.TelegraphPoleBuilder` | same | RF-owned individual telegraph-pole builder. |
| `RailForge.MapEditor.InfrastructureOverrides` | same | RF-owned crossing/turntable override handler; dedicated contract above. |
| `RailForge.NativeTerrainBrushes` | same | RF-owned persistent terrain-brush record; RF-only. |
| `RailForge.FlowyThingBuilder` | same | RF-owned editor output for Road/River splineys. |
| Name containing `RiverSplineBuilder` | river family | Runtime-recognized family. |
| Name containing `AutoTrestleBuilder`, exact `AutoTrestle`, or suffix `.AutoTrestle` | AutoTrestle family | Runtime-recognized family. |
| Name containing `FlowyThingBuilder` | Flowy/road/river family | Runtime-recognized family. |
| `CLB.Shop01`, `CLB.REX_Shop02`, or name containing `.Shop01`, `.Shop02`, `.REX_Shop02` | CLB shop family | Runtime-recognized family. |
| Name containing `ModularScenery` | modular-scenery family | Provider/runtime-specific. |
| Name containing `RRCrossing`, `CrossingGate`, `GradeCrossing`, `Crossbuck`, or `Wigwag`; also compound `Crossing` + (`Rail` or `Road`) + (`Gate` or `Signal`) | crossing family | Provider/runtime-specific. |
| Name containing `TelegraphPole` | telegraph family | Provider/runtime-specific. |
| Name containing `Turntable` and `Builder` | turntable family | Provider/runtime-specific. |
| Name containing `Road` | road handler | Provider/runtime-specific. |
| Name containing `DKW`, `DKW.KRESpliney`, or suffix `.KRESpliney` | DKW/KRE family | Runtime-recognized generated topology. |
| Loaded type exposing `BuildSpliney(id,parent,data)` | dynamic handler | Its assembly/provider is a real dependency. |

Known RF handler record shapes:

| Family | Runtime-confirmed fields/constraints |
| --- | --- |
| `RailForge.FlowyThingBuilder` / Flowy/River new builder | `points[]`, each with `position`, `rotation`, `width`; direct authored fields `profile`, `style`, `offsetY` (and PascalCase forms). Style is Road or River; the exact SplineProfile must load. Runtime defaults point width 1 and offsetY -0.1. Editor completion requires at least two points with ≥1 m horizontal separation, clamps width 0.25–200 and offsetY -20–20, and defaults Road to width 10/offset -0.1 and River to width 20/offset -1. `splineProfile`/`yOffset` occur only in classification or existing-target compatibility, not direct new-builder input. |
| AutoTrestle | At least two point objects with finite `position` and `rotation`; consecutive positions must differ; `headStyle`, `tailStyle`, plus runtime/internal `controlPoints` and profile behavior. |
| Exact RiverPath replacement | Existing target must carry RiverPath; `points[]` entries use `position`, `rotation`, `width`; optional `style`, `offsetY`/`yOffset`, `profile`/`splineProfile` and PascalCase aliases. |
| Exact AutoTrestle replacement | Existing target must carry AutoTrestle; at least two finite points with `position` and optional `rotation`, non-zero consecutive segments and valid endpoint terrain; existing profile is preserved, with optional `headStyle`/`tailStyle`. |
| DKW/KRE | `position`, `rotation`, signed `crossingAngle`, handler/ID, and generated topology. RF writer also emits `flipSwitchStandA: false`, `flipSwitchStandB: false`, and `style: "Standard"`. Angle must be -15…-4 or 4…15 degrees, and authoring preflight still requires a compatible DKW provider. |
| Loader, station, map label, map mask, turntable | Fields are enumerated in their dedicated tables above/below. |
| CLB shop, ModularScenery, crossing, other dynamic handler | Provider-specific; a recognized name does not prove a complete public data contract. |

### FUSE legacy-handler import ledger

FUSE's in-memory legacy converter recognizes these source namespaces and translates them into native FUSE collections:

| Legacy handler inputs | Native FUSE destination |
| --- | --- |
| `StrangeCustoms.LoaderBuilder`, `StrangeCustoms.UnloaderBuilder`, `AlinasMapMod.LoaderBuilder`, `AlinasMapMod.Loaders.LoaderBuilder`, `AlinasMapMod.UnloaderBuilder` | `operations.loaders` |
| `StrangeCustoms.StationBuilder`, `AlinasMapMod.StationBuilder`, `AlinasMapMod.Stations.StationAgentBuilder` | `operations.stations` |
| `StrangeCustoms.TurntableBuilder`, `StrangeCustoms.Turntable.TurntableBuilder`, `AlinasMapMod.TurntableBuilder`, `AlinasMapMod.Turntable.TurntableBuilder` | `operations.turntables` |
| `StrangeCustoms.TelegraphPoleMover`, `AlinasMapMod.TelegraphPoleMover`, `AlinasMapMod.TelegraphPoles.TelegraphPoleMover` | `world.telegraphPoleMovements` |
| `StrangeCustoms.MapLabelBuilder`, `AlinasMapMod.MapLabelBuilder` | `world.mapLabels` |
| `StrangeCustoms.RailroadCrossingBuilder`, `StrangeCustoms.RRCrossingBuilder`, `AlinasMapMod.RailroadCrossingBuilder`, `AlinasMapMod.RRCrossingBuilder` | `world.scenery` crossing representation |
| `StrangeCustoms.FlowyThingBuilder`, `StrangeCustoms.Tracks.FlowyThingBuilder`, `AlinasMapMod.FlowyThingBuilder` | `world.splineys` type `road`, or `river` when Flowy style/profile identifies river intent |
| `StrangeCustoms.RiverBuilder`, `AlinasMapMod.RiverBuilder` | `world.splineys` type `river` |
| `StrangeCustoms.RoadBuilder`, `AlinasMapMod.RoadBuilder` | `world.splineys` type `road` |
| `StrangeCustoms.AutoTrestle`, `StrangeCustoms.AutoTrestleBuilder`, `StrangeCustoms.TrestleBuilder`, `AlinasMapMod.TrestleBuilder` | `world.splineys` type `trestle` |

An unknown legacy handler with no points is routed to the plugin host; one with at least two points can become a native FUSE spliney; otherwise it is preserved under `extensions.legacySplineyObjects`.

The separate offline FUSE converter has a deliberately different/narrower compatibility set. In addition to some handlers above, it recognizes `StrangeCustoms.WaterfallBuilder`, `StrangeCustoms.TerrainRoadBuilder`, `DKW.DKWSpliney`, and crossings `cutil.rrcrossing`/`cutil.railroadcrossing`. Do not assume the offline and in-memory converters accept identical aliases.

### Water surfaces

FUSE `world.waterSurfaces.<id>` fields are `points[]` (at least three), `sourceLakePath`, `materialName`, `lockHeight`, `snapToTerrain`, `enableCollider`, `uvScale`, `triangleDensity`, `maximumTriangleArea`, and `yOffset`. The FUSE documentation explicitly treats this as native-only; no supplied RF authoring equivalent was found.

### RF native terrain brushes

RF `1.98` has an RF-only persistent `splineys` handler unrelated to FUSE water surfaces:

| RF path/field | Contract |
| --- | --- |
| `splineys.<id>.handler` | `RailForge.NativeTerrainBrushes` |
| `formatVersion` | Integer `1` |
| `strokes[]` | Up to 8,192 terrain operations |
| stroke `operation` | `raise`, `lower`, `flatten`, `round`, `smooth`, `paint`, `erasePaint`, `addTrees`, `removeTrees`, `paintGrass`, or `eraseGrass` |
| stroke location/shape | `terrain`, `centerX`, `centerZ`, `radius`, `square` |
| stroke strength | `amount`, `hardness`, `targetHeight`, `paintStrength` |
| paint selection | `paintLayerIndex`, `paintLayerName` |
| vegetation selection | `vegetationPrototypeIndex`, `vegetationPrototypeName`, `vegetationDensity`, `vegetationScale`, `randomSeed` |
| terrain identity | `originX`, `originY`, `originZ`, `sizeX`, `sizeY`, `sizeZ`, `heightmapResolution`, `alphamapWidth`, `alphamapHeight`, `scenePath`, `terrainName`, `terrainDataName` |

Runtime bounds are radius 0.5–256, amount 0–60, hardness/paintStrength 0–1, absolute target height ≤10,000, paint layer 0–255, vegetation prototype 0–4,095, density 0–128, and scale 0–10. FUSE has no native terrain-brush root, so preserve this only as RF-side content.

### Telegraph pole lines

FUSE `world.telegraphPoles.<id>` contains `profile`, `polePrefab`, `wirePrefab`, `spacing`, and `points[]` (at least two). Translating the line to RF remains a manual expansion, but RF `1.98` has a native individual-pole handler, `RailForge.MapEditor.TelegraphPoles.TelegraphPoleBuilder`.

Its record fields are `handler`, `position`, `rotation`, `scale`, `variant`, `height`, `crossarmWidth`, `wireCount`, and `poleRadius` (PascalCase aliases are also accepted). Built-in variants are `Standard Wood Pole`, `Tall Wood Pole`, `Short Yard Pole`, `Double Crossarm Pole`, `Junction Angle Pole`, `No Wire Pole`, and `Old Leaning Pole`. FUSE's line profile, spacing, point path, prefab sources, and wire topology must be resolved into the required RF pole records/connections; they are not direct field renames.

RF editor clamps are height 2.5–10, crossarm width 0.4–4, integer wire count 0–12, and pole radius 0.03–0.25. Standard defaults are 5.5/1.8/4/0.09. Variant defaults are Tall 7.25/2.1/6, Short 4.2/1.35/2, Double 5.8/1.9/4, Junction 6.1/2/6, No Wire 5.2/1.6/0, and Old Leaning 4.9/1.5/2; radius remains unchanged when switching variant. These are editor clamps/defaults, not strict guarantees of the more permissive runtime parser.

### Telegraph pole movement and edit arrays

FUSE movement entry:

- `poleIndices[]`: unique non-negative integers;
- `offset`: one Vector3 applied to every listed pole.

RF representation uses one handler record and parallel arrays:

| FUSE | RailForge `RailForge.MapEditor.TelegraphPoleEdits` |
| --- | --- |
| one movement entry | one `splineys.<stableId>` handler record |
| `poleIndices[]` | `polesToMove[]` |
| one `offset` | repeat the vector once per ID in `poleMovement[]` |

Complete RF `1.98` telegraph-edit fields found in the runtime:

| RF field(s) | Meaning | FUSE equivalent |
| --- | --- | --- |
| `schemaVersion` | Handler schema version | None at movement-entry level. |
| `createOnly` | Limit operation mode | None. |
| `polesToCreate[]`, `createdPolePositions[]` | Create poles at positions | No native FUSE movement equivalent. |
| `createdPoleRotations[]` | Rotations for created poles | No equivalent. |
| `createdPoleScales[]` | Scales for created poles | No equivalent. |
| `createdPoleTags[]` | Tags for created poles | No equivalent. |
| `polesToMove[]`, `poleMovement[]` | Offset existing poles | Direct intent after repeating FUSE offset. |
| `polesToRaise[]` | Raise each listed pole by fixed `{x:0,y:2,z:0}`; no authored amount | No native FUSE movement equivalent. |
| target IDs `polesToSet[]`; paired vectors `polesToPosition[]`/`polePositions[]`/`setPolePositions[]` | Set absolute positions | No native FUSE movement equivalent. |
| `polesToRotate[]`, `poleRotations[]` | Rotate existing poles | No native FUSE movement equivalent. |
| `polesToScale[]`, `poleScales[]` | Scale existing poles | No native FUSE movement equivalent. |
| `polesToDelete[]` | Delete indexed poles | No native FUSE movement equivalent. |
| `poleConnections[]`/`createdPoleConnections[]` | Connect pole ID pairs | No native FUSE movement equivalent. |

Parallel RF arrays must have matching order and exact cardinality; connections must be exact two-ID pairs. RF handler version 2 requires a tag for every created pole, version 3 also requires a scale for every created pole, and deletion requires RF-owned schema version 4. `createOnly` forbids move, raise, set, rotate, scale, and delete operations. Runtime aliases for create positions are `createdPolePositions`, `createPolePositions`, and `poleCreatePositions`.

Created pole IDs are positive unique integers and must not already be RF-owned; created tags are exact integers. Position and rotation vectors must be finite in schema version 2+, and every created/set scale component must be finite within 0.01–100. Primary fields also accept PascalCase aliases.

### Map labels

| FUSE `world.mapLabels.<id>` | RF map-label handler | Status |
| --- | --- | --- |
| `text` | `text` | Direct. |
| `position` | `position` or `localPosition` | Direct coordinates; prefer `position`. |
| `rotation` | No fully documented RF map-label rotation field | Manual. |
| `style: label` | Generic RF label behavior | Manual/default. |
| `style: speedLimit` / `speed-limit` | No confirmed direct handler field | Manual. |
| `speedLimitMph` | No confirmed direct handler field | Manual. |
| `size` | No confirmed direct handler field | Manual. |
| `color` | No confirmed direct handler field | Manual. |
| — | RF `alignment`, `alignYUp` | RF-only handler fields. |

RF convenience-root `mapLabels` is promoted to `splineys` with `AlinasMapMod.Map.MapLabelBuilder`. Historical namespaces `AlinasMapMod.MapLabelBuilder`, `AlinasMapMod.MapLabels.MapLabelBuilder`, and `AlinasMapMod.Labels.MapLabelBuilder` are runtime-compatible.

As with loaders, a null map-label record promotes to a spliney tombstone, an explicit `splineys.<same-id>` wins, and malformed entries are skipped.

The RF handler accepts `text`/`Text`, `position`/`Position`/`localPosition`/`LocalPosition`, `alignment`/`Alignment`, and `alignYUp`/`AlignYUp`. FUSE constrains `speedLimitMph` to 1–80, `size` to >0, and `color` to `#RRGGBB` or `#RRGGBBAA`; legacy text such as `15 MPH` is auto-detected by FUSE.

RF requires nonblank label text, finite position, a recognized native alignment value, and boolean `alignYUp`.

### Map masks

FUSE mask fields `falloff`, `enableSetHeight`, `enableCutTrees`, `enableMaskModifier`, `maskName`, and `order` have similarly named RF handler fields. FUSE runtime constrains `maskName` to `Object`, `Terrain`, or `Road` and defaults to `Object`, although its public schema only says string. Shape geometry still requires conversion:

| FUSE shape | FUSE geometry | RF AMM handler geometry | Status |
| --- | --- | --- | --- |
| `circle` | `center`, `radius`; serialized `rotation` is unused | `position`, `radius` | Rename `center` → `position`; do not invent rotation semantics. |
| `rectangle` | `center`, `size`, `rotation` | `position`, `size`, `rotation` | Rename `center` → `position`. |
| `curve` | `points[]`, optional `width`; serialized `center` and `rotation` are unused fillers | `positionA`, `rotationA`, `positionB`, `rotationB`, `sizeA`, `sizeB`, plus optional `radiusNoise`, `noiseScale` | **Reshape/manual**; a multi-point curve is not a blind endpoint rename. |

Use the verified RF `1.98` source-port family `AlinasMapMod.Map.MapMaskBuilder`. Its implementation is embedded in RailForge, although the standalone AMM contract linked by the guide is missing from the supplied documentation.

RF validates `type` as circle/rectangle/curve and requires finite positions. Circle radius must be positive and falloff non-negative; rectangle size must be positive and rotation finite; curve endpoints, rotations, sizes, and noise values must be finite.

### Map tiles

FUSE `world.mapTiles.<id>` fields are `directory`, `sourceFolder`, and `priority`. RF has no corresponding graph root, but RF `1.98` can index exact layout `<mod>/Maps/<active-map-directory>/tile_<signed-int32-x>_<signed-int32-y>.data` using direct tile children only. Each payload is 33 bytes–8 MiB and an exact 513×513, non-interlaced, 8-bit RGBA PNG; a map may expose at most 65,536 coordinates. RF reverses resolved roots so the later/higher-priority source wins duplicate coordinates. This is a **runtime-only/manual package translation candidate**, not a field-for-field graph conversion; translate FUSE `directory`, `sourceFolder`, and `priority` into package layout/order deliberately.

### Scene clones and RF mandelas

| FUSE `world.sceneClones.<id>` | RF `mandelas.<id-or-path>` | Status |
| --- | --- | --- |
| `targetPath` | dictionary key or runtime aliases `path`/`destination`/`target` | Reshape; RF Full Guide has no public mandela field contract. |
| `source` | runtime-preferred `instantiateFrom`; aliases `source`, `copyFrom` | Rename; runtime-only field contract. |
| `enabled` | `enabled` | Direct; runtime alias `active`. |
| `localPosition` | `localPosition` | Direct; RF also accepts `position`. |
| `localRotation` | `localRotation` | Direct; RF also accepts `rotation`. |
| `localScale` | `localScale` | Direct; RF also accepts `scale`. |

In FUSE, null/missing local transforms preserve existing values. Confirm the same intended result in RF. `mandelas.<path>: null` is hide-only in RF; it does not destroy the scene object.

## 9. Progression and map features

RF accepts canonical top-level `mapFeatures` and `progressions`, plus compatibility forms `features` and a nested `progression` wrapper. Prefer canonical lowercase top-level roots.

RF can hydrate the definitions even when **Enable progression materializer** is off, but RF-managed visibility, forced locks, synthetic milestone gates, and `trackGroupsDisableOnUnlock` require that setting. Initial visibility additionally requires **Enforce progression initial visibility**; its safe default is off. The presence of records in Diagnostics therefore does not prove that RF gating is active.

### Map-feature fields and every array

| FUSE `progression.mapFeatures.<id>` | RF `mapFeatures.<id>` | Status |
| --- | --- | --- |
| dictionary key / identity | dictionary key; optional `identifier`/`id` | Preserve stable ID. |
| `displayName` | `displayName` | Direct. |
| `description` | `description` | Direct. |
| `initiallyEnabled` | `defaultEnableInSandbox` | Rename; review Company behavior separately. |
| `groupIds[]` | Populate `trackGroupsEnableOnUnlock[]` and/or `trackGroupsAvailableOnUnlock[]` only when the corresponding explicit FUSE patch is absent | FUSE compatibility fallback, not an unconditional combination. |
| `prerequisiteFeatureIds[]` | `prerequisites[]` | Rename. |
| `trackGroupsEnableOnUnlock[]` | same | Direct. |
| `trackGroupsAvailableOnUnlock[]` | same | Direct. |
| `areasEnableOnUnlock[]` | same | Direct. |
| `gameObjectsEnableOnUnlock[]` | same | Direct. |
| `unlockIncludeIndustries[]` | same | Direct after reference validation. |
| `unlockExcludeIndustries[]` | same | Direct after reference validation. |
| `unlockIncludeIndustryComponents[]` | same, canonical objects `{areaId,industryId,componentId}` | Reshape compact FUSE IDs into RF object references. |
| — | `name` | RF optional identity/display field. |

FUSE `initiallyEnabled` affects Sandbox. It is not a Company-start grant. RF `defaultEnableInSandbox` has the same Sandbox boundary.

### Progression fields

| FUSE `progression.progressions.<id>` | RF `progressions.<id>` | Status |
| --- | --- | --- |
| dictionary key | dictionary key and optional `identifier`/`id` | Preserve stable ID. |
| — | `name` | RF optional progression name. |
| `baseProgression` | No supplied RF field | Manual; flatten/inherit explicitly. |
| `sections` | `sections` dictionary | Direct if already keyed; flat root sections must be nested. |
| `enableFeaturesAtStart` array/patch | `enableFeaturesAtStart[]` | Calculate final array or translate patch to RF operators. |

FUSE root `progression.progressionId` selects/associates a progression and defaults to the package ID for flat root sections. Declaring an existing `progressions.<id>` patches it in place. In RF, progression is keyed by ID; a Company start separately names `progressionId`.

FUSE `enableFeaturesAtStart` is Company-only: it is reapplied on load, persisted, and self-heals existing saves. A start-granted feature must not also appear in a section's enable, disable, or available arrays. Translate this lifecycle deliberately into the RF progression plus Company-start design.

### Section fields and every array

Every entry in FUSE's flat root `progression.sections[]` requires an `id` before it can be nested under the selected progression.

| FUSE section field | RF section field/form | Status |
| --- | --- | --- |
| `id` | dictionary key and/or `identifier`/`id` | Reshape identity. |
| `progressionId` | parent `progressions.<id>` | Reshape/nest. |
| — | RF section `name` | RF optional field. |
| `displayName` | `displayName` | Direct. |
| `description` | `description` | Direct. |
| `prerequisiteSections[]` or `prerequisiteSectionIds[]` | `prerequisiteSections[]` | Normalize/rename. |
| `enableFeaturesOnUnlock[]` | same | Direct. |
| `disableFeaturesOnUnlock[]` | same | Direct. |
| `enableFeaturesOnAvailable[]` | same | Direct. |
| `deliveryPhases[]` | same | Reshape component references and deliveries. |
| `unlockIncludeIndustries[]` | Usually synthesize/extend an RF map feature and enable it from the section | Do not blindly copy; not a documented ordinary RF section field. |
| `unlockExcludeIndustries[]` | Synthesize/extend an RF map feature | Manual semantic conversion. |
| `unlockIncludeIndustryComponents[]` | Synthesize/extend an RF map feature with object references | Reshape/manual. |
| `areasEnableOnUnlock[]` | Synthesize/extend an RF map feature | Manual. |
| `gameObjectsEnableOnUnlock[]` | Synthesize/extend an RF map feature | Manual. |
| `trackGroupsEnableOnUnlock[]` | Synthesize/extend an RF map feature | Manual. |
| `trackGroupsAvailableOnUnlock[]` | Synthesize/extend an RF map feature | Manual. |
| `interchangeTransfers` dictionary | No supplied RF section field; preserve source-component → destination-component intent through an RF-compatible interchange/provider design | Manual. FUSE permits string or null destination values. |
| — | `trackGroupsDisableOnUnlock[]` | RF-only reversible gate; not deletion. |

### Delivery phase and delivery arrays

| FUSE | RailForge | Status |
| --- | --- | --- |
| phase `cost` | `cost` | Direct integer. |
| phase `industryComponentId` string, or runtime alias `industryComponent` | `industryComponent: {areaId,industryId,componentId}` | Reshape reference. Canonical RF object is safer. |
| phase `deliveries[]` | `deliveries[]` | Translate every entry. |
| delivery `carTypeFilter` | `carTypeFilter` | Direct. |
| delivery `count` | `count` | Direct integer. |
| delivery `loadId`, or runtime alias `load` | `loadId` | Normalize to canonical field. |
| delivery `direction: loadToIndustry` or `import` | `direction: LoadToIndustry` | Normalize enum and casing. FUSE runtime also recognizes `0`, `toIndustry`, and `to`. |
| delivery `direction: loadFromIndustry` or `export` | `direction: LoadFromIndustry` | Normalize enum and casing. FUSE runtime also recognizes `1`, `fromIndustry`, and `from`. |
| delivery `destinationIndustryId` | Encoded through the phase's RF component reference/design | No direct RF delivery field. |

Runtime note: the inspected RF version rejects an invalid authored phase atomically enough to retain the previous complete phase array. This is stricter than wording in the supplied guide suggesting individual bad entries are merely skipped.

FUSE may infer a missing phase `industryComponentId` only when all deliveries target one industry and that industry has exactly one ProgressionIndustryComponent. Do not depend on inference during translation; write the full RF component reference.

### FUSE string/id patch forms

All FUSE map-feature and section fan-out ID fields are backed by `FuseStringPatch` at runtime: omitted means no change, a plain array replaces the collection, `[]` clears it, and a boolean dictionary merges case-insensitively by ID (`true` adds and `false` removes; resulting merge order is unspecified). This applies to every prerequisite, feature, group, area, game-object, industry, and industry-component collection listed in the two tables above, plus `progressions.<id>.enableFeaturesAtStart`.

Only `enableFeaturesAtStart` publicly exposes the array-or-boolean-dictionary union in the supplied FUSE schema. Boolean dictionaries on the other fan-out fields are runtime-compatible but schema-invalid; prefer schema-valid arrays for new FUSE authoring unless deliberately patching through this known discrepancy.

RF ordinary arrays replace. Before translation, resolve a FUSE boolean dictionary to the intended final ordered array, or express the exact change with RF operators. Do not copy a FUSE boolean map into an RF array field.

RF `1.98` accepts these progression aliases: map feature `identifier`/`id` and `prerequisites`/`prerequisiteFeatures`; progression `identifier`/`id`; section `identifier`/`id`, `prerequisiteSections`/`prerequisites`, and `deliveryPhases`/`phases`. Component-reference keys accept canonical `areaId`, `industryId`, `componentId` and compatibility aliases `area`, `industry`, `subIdentifier`, `component`, `id`, and `identifier`. Prefer the canonical fields shown in the main tables.

When importing legacy data, FUSE maps `defaultEnableInSandbox` to `initiallyEnabled` and `prerequisites` to `prerequisiteFeatureIds`. These are conversion aliases, not additional preferred native FUSE fields.

For newly materialized RF features, progressions, and sections, omitted collection fields normalize to non-null empty arrays. This differs from patching an already existing ordinary RF array, where omission preserves its value.

## 10. Settings and feature rules

### Complete FUSE setting definition

`settings.<id>` supports:

| Field | Type/values | RF translation |
| --- | --- | --- |
| `type` | Canonical `bool`, `enum`, `number`, `path`, `color`, `text` | No supplied native RF data-only equivalent. |
| accepted type aliases | `boolean`; `choice`/`select`; `float`/`double`; `int`/`integer`; `file`/`folder`; `colour`; `string` | Normalize in FUSE; do not project automatically. |
| `label`, `description` | strings | No supplied RF equivalent. |
| `scope` | Canonical `user`, `profile`, `server` | No supplied RF equivalent. |
| accepted scope aliases | `local`/`client`; `modset`/`mod-set`; `shared`/`multiplayer` | Normalize in FUSE only. |
| `default` | arbitrary setting value | No supplied RF equivalent. |
| `values[]` | enum choices | No supplied RF equivalent. |
| `min`, `max`, `step` | numeric constraints | No supplied RF equivalent. |
| `advanced`, `reloadRequired` | booleans | No supplied RF equivalent. |

Enum `values[]` are exact strings and are not normalized by FUSE.

### Complete feature-rule definition

Each `featureRules.<id>` contains required `setting`, `value`, and `targets`. Optional `operator` defaults to `equals`; accepted values are `equals`, `notEquals`, `greaterThan`, `greaterThanOrEqual`, `lessThan`, and `lessThanOrEqual`.

The setting and every target must be authored in the same FUSE definition, at least one target is required, and numeric ordering operators require a `number` setting. The controlling setting should be marked `reloadRequired`; a false rule filters the runtime copy when it is reapplied rather than acting as a permanent graph deletion. The older supplied `AUTHORING_RECIPES.md` statement that declarative conditional settings are unreleased is stale relative to the current schema/runtime.

All FUSE target arrays and their RF interpretation:

| FUSE `targets` array | RF target namespace | Translation |
| --- | --- | --- |
| `trackNodes[]` | `tracks.nodes` | No RF settings-rule engine; create separate RF variants/patches manually. |
| `trackSegments[]` | `tracks.segments` | Same. |
| `trackSpans[]` | `tracks.spans` | Same. |
| `trackAreas[]` | `areas` | Same. |
| `loads[]` | `loads` | Same; RF loads cannot be tombstoned. |
| `industries[]` | nested `areas.*.industries` | Same plus area reshaping. |
| `industryComponents[]` | nested `components` | IDs are `industryId/componentId` in FUSE; RF also needs area context. |
| `loaders[]` | promoted `splineys` | Handler/manual. |
| `turntables[]` | handler-backed `splineys` | Handler/manual. |
| `stations[]` | handler-backed `splineys` | Handler/manual. |
| `scenery[]` | `scenery` | Manual variant/patch. |
| `splineys[]` | `splineys` | Handler/manual. |
| `waterSurfaces[]` | No RF root | No translation. |
| `telegraphPoles[]` | handler-backed `splineys` | Handler/manual. |
| `mapLabels[]` | promoted `splineys` | Handler/manual. |
| `mapMasks[]` | handler-backed `splineys` | Handler/manual. |
| `mapTiles[]` | No RF graph root; package `Maps` tile root exists | No RF settings-rule engine; create/select a package-level tile variant manually. |
| `sceneClones[]` | `mandelas` | Manual variant/patch. |
| `progressions[]` | `progressions` | Manual variant/patch. |
| `mapFeatures[]` | `mapFeatures` | Could be redesigned as RF feature gating; not a settings-rule translation. |
| `whistles[]` | RF runtime catalog mixinto `whistles` | No RF settings-rule engine; create/select a runtime-specific catalog variant manually. |
| `horns[]` | RF runtime catalog mixinto `horns` | Same. |
| `bells[]` | RF runtime catalog mixinto `bells` | Same. |

There are no FUSE feature-rule target arrays for `spawnPoints` or `telegraphPoleMovements`.

## 11. Audio

RF `1.98` has built-in audio catalog mixinto targets even though the supplied Full Guide does not document them. Declare source files under the RF manifest's `whistles`, `horns`, or `bells` mixinto targets. RF consumes a root array for each catalog, whereas FUSE stores each catalog as a dictionary keyed by ID. RF catalog identity is the profile `name`, not the discarded FUSE dictionary key: bell names compare case-sensitively, horn and whistle names case-insensitively; deterministic first owner/occurrence wins. Preserve or deliberately replace the FUSE ID by a stable unique `name` during the reshape.

| FUSE path/field | RailForge source entry | Status |
| --- | --- | --- |
| `audio.whistles.<id>` | one entry in the `whistles` root array | Runtime-only reshape. |
| whistle `name` | `name` | Direct. |
| whistle `clip` | `clip` | Direct. |
| whistle `model.assetPackIdentifier` | `model.assetPackIdentifier` | Runtime diagnostics metadata. A missing/ambiguous model keeps the locomotive's baseline physical model while audio remains usable. |
| whistle `model.assetIdentifier` | `model.assetIdentifier` | Same diagnostics-only caveat. |
| whistle `rampUpPitch` | No RF catalog field found | FUSE-only. |
| whistle `lerpSpeed` | No RF catalog field found | FUSE-only. |
| whistle `airLerpSpeed` | No RF catalog field found | FUSE-only. |
| `audio.horns.<id>` | one entry in the `horns` root array | Runtime-only reshape. |
| horn `name` | `name` | Direct. |
| horn `layers[]` | `layers[]` | Direct array; RF requires one or two layers. |
| layer `file` | `file` | Direct; RF horn files must be `.wav` or `.ogg`. |
| layer `keyframes[]` | `keyframes[]` | Direct array after key conversion; RF requires non-empty, ordered, finite keyframes within 0–1. |
| keyframe `t` | `time` | Rename. |
| keyframe `value` | `value` | Direct. |
| `audio.bells.<id>` | one entry in the `bells` root array | Runtime-only reshape. |
| bell `name` | `name` | Direct. |
| bell `file` | `file` | Direct; RF bell files may be `.wav`, `.ogg`, or `.mp3`. Whistle clips use the same three extensions. |
| bell `indexTimes[]` | `indexTimes[]` | Direct; RF requires finite, positive, strictly ordered times. |

FUSE exposes no native audio removal arrays. RF catalog replacement/removal semantics must be verified against the specific mixinto/catalog contract before translating a deletion.

Bell and horn names are limited to 128 characters; whistle names to 256. Exact bell name `Default` is reserved, while horn `Default` is reserved case-insensitively. Whistle `clip` paths are limited to 1,024 characters, model asset-pack IDs to 256, and model asset IDs to 512.

RF runtime safety caps are 64 keyframes per horn layer, 64 bell `indexTimes`, and 2,048 profiles per bell/horn/whistle catalog, with 4,096 whistle profiles total. Whistle-specific limits include 8 MiB catalog JSON, 64 MiB audio, 33,554,432 decoded samples, JSON depth 16, and a minimum loop of 8,192 frames. These are runtime limits rather than fields in the supplied public Full Guide.

## 12. Complete removal and suppression ledger

| FUSE removal/suppression | RailForge representation | Caveat |
| --- | --- | --- |
| `tracks.removals.nodes[]` | `tracks.nodes.<id>: null` | Remove dependent spans/segments first. |
| `tracks.removals.segments[]` | `tracks.segments.<id>: null` | Ensure no retained span references it. |
| `tracks.removals.spans[]` | `tracks.spans.<id>: null` | Supported. |
| No FUSE area removal array | `areas.<id>: null` | RF-only tombstone; true deletion, not visibility gating. |
| `operations.removals.industries[]` | `areas.<area>.industries.<id>: null` | Area is required. |
| component `remove: true` | `...components.<id>: null` | Supported. |
| No FUSE load removal | RF load tombstone unsupported | Neither side offers a general safe load deletion here. |
| `world.removals.scenery[]` by authored ID | `scenery.<id>: null` | Supported keyed tombstone. |
| scenery/full scene path removal | `mandelas.<path>.enabled: false` | Hide existing scene object; not destruction. |
| `world.removals.splineys[]` | `splineys.<id>: null` | Supported. |
| `world.removals.waterSurfaces[]` | No generic RF root | No translation. |
| `world.removals.telegraphPoles[]` | Handler-specific delete/edit | No generic RF tombstone. |
| `world.removals.mapLabels[]` | Tombstone promoted/handler spliney ID | Only after proving the effective RF spliney ID. |
| `world.removals.mapMasks[]` | Tombstone handler spliney ID | Same caveat. |
| `world.removals.sceneClones[]` | `mandelas.<id/path>: null` or `enabled:false` | RF is hide-only. |
| `suppressBaseScenePaths[]` | One disabled RF mandela per exact path | Direct intent after reshape. |
| `suppressBaseTrackGroups[]` | RF progression/feature visibility design | Not deletion. |
| `suppressBaseAreas[]` | RF progression/feature visibility design | Not deletion. |

FUSE has no removal arrays for spawn points, telegraph-pole movement operations, or map tiles. RF does not support tombstones for loads, map features, progressions, texts, whole roots, or nested scalar/array fields.

Every FUSE `world.removals.*[]` entry may be a package-owned ID or a full scene path. World removals run before new world objects are created, and defining and removing the same ID in one document is invalid. A full path is not automatically an RF keyed ID; translate it to an exact mandela hide or handler-specific operation after verifying identity.

RF tombstones remain subject to package ownership, base/mod delete guards, live references, and safety deferral. Industry components tied to passengers/progression or occupied infrastructure may soft-remove or defer. A `null` record is an instruction, not proof that the live object disappeared.

## 13. RF-only package formats and surfaces

### Native discovery folders

| RailForge path | Purpose | FUSE counterpart |
| --- | --- | --- |
| `RailForge/game-graph/**/*.json` | Graph patches | FUSE files named in `FuseDataFiles[]`. |
| `RailForge/game-migrations/**/*.json` | ID migrations | FUSE can execute only a limited scalar subset under `extensions.gameMigrations`; see below. |
| `RailForge/containers/<container-id>/**/*.json` | Container/mixinto patch group | No direct FUSE graph counterpart. Runtime also accepts singular `container`, but canonical authoring should use documented `containers`. |
| `RailForgeAssetPacks/**/Catalog.json` | Asset catalog | FUSE asset-pack packaging; requires manual conversion. |
| `RailForgeAssetPacks/**/Definitions.json` | Asset definitions | FUSE definitions/overrides; requires manual conversion. |
| Package-root `spawn-points.json` | Global RF map locations | Partial counterpart to FUSE `world.spawnPoints[]`; runtime-only contract described above. |

Inside an RF container patch only, a root object may use `$objectsByIdentifier: { "<identifier>": <object-or-null> }`; RF rewrites those entries against the container's `objects[]`. This is not a general game-graph root or general-purpose patch operator.

### RF compatibility sidecars/adapters

RF `1.98` contains two migration adapters that are not ordinary canonical graph roots:

| Input surface | RF output | Accepted shape and limits |
| --- | --- | --- |
| `Info.json.LatomsCrossingsEditable.Crossings` | `splineys` handler `RailForge.MapEditor.InfrastructureOverrides` | RF internally labels the adapter `latoms-editable-crossings-v1` and synthesizes the graph mixinto. Input entries require `id` and `location {segmentId,distance}`, with optional end default A, `enabled` default true, `width` default 8, and `whistleDistance` default 0. Output supplies `operation: "create"` and handler schema version 2. |
| `tile-editor.fuse.json` / mixinto `railforge-tile-editor-object-lines-v1` | RF scenery generated from object-line splineys | Strict FUSE subset described below; it is not general FUSE-graph support. |
| Loose legacy load-catalog JSON | Promoted as a game-graph load patch | Runtime migration heuristic requires non-empty `loads`/`Loads` and a path/name/ID containing `SCOLD`, `StandardCatalog`, `CatalogofLoad`, `LoadID`, `LoadIDs`, `Load_ID`, or `loads`. Prefer canonical RF game-graph placement. |

Treat all three as runtime-only import paths for legacy data, not preferred native RF authoring.

The crossings adapter allows an empty array, at most 4,096 entries, crossing IDs up to 128 characters, segment IDs up to 256, finite non-negative distance, width 0.5–100, and whistle distance 0 or 25–2,000. IDs use Unicode letters/digits plus `.`, `-`, `_`, and `:`. Property names are case-insensitive, ambiguous case aliases and case-insensitive duplicate IDs are rejected, finite numeric strings are accepted, and extra entry properties are ignored.

The tile-editor adapter requires top-level `schemaVersion`, `id`, `coordinateSpace`, and `world`; it also recognizes `$schema`, `name`, `author`, `modVersion`, and optional `tracks`. `world.splineys` must contain 1–512 records. If tracks are supplied, `tracks.nodes`/`segments`/`spans` and track removal arrays must be empty. Each spliney is closed and may contain only required `type`, `assetIdentifier`, `spacing`, `instanceScale`, `rotationOffset`, `lateralOffset`, `verticalOffset`, `snapToTerrain`, `alignToSlope`, `placeAtEnd`, `maximumInstances`, and `points[]`; each point contains closed `position` and `rotation` objects with x/y/z.

Property names are generally case-insensitive with ambiguous casing rejected, but values `schemaVersion: "1.0"` and `type: "objectLine"` are case-sensitive; `coordinateSpace: world` is case-insensitive. Safe unknown top-level metadata names up to 64 characters are allowed only with null, boolean, finite number, or string up to 1,024; nested objects are closed. `$schema` is limited to 1,024 characters, name/author to 512, and `modVersion` to 64. Asset IDs reject blank/control text, `..`, rooted paths, `scene://`, and `hierarchy://`.

For this adapter, rotation/lateral/vertical offsets must be zero, `snapToTerrain` and `alignToSlope` must be false, and `placeAtEnd` must be true. Points must be between 1 and `maximumInstances`; the default/max is 4,096. Other caps are 512 splineys, 4,096 points per spliney, 8,192 total points, ID length 160, asset ID length 512, spacing 0.01–10,000, scale components 0.001–1,000, world magnitude 10,000,000, and rotation magnitude 1,000,000. Output is one RF `scenery` entry per point with `modelIdentifier`, position, rotation, scale, and active state.

### Game migrations

RF migration roots are `waybillDestinations`, `properties`, and `carTypes`, each a dictionary from old ID to either a scalar/string or an object using `value`, `to`, `target`, `new`, or `id`.

FUSE has no public native graph schema for migrations. Its compatibility applier can execute only scalar `waybillDestinations` and `properties` dictionaries stored under `extensions.gameMigrations` (including their known PascalCase aliases). It does not execute `carTypes` or RF object-valued records. Preserve those only as data for a future converter/provider, or redesign them manually; do not claim functional FUSE migration parity.

### Company starts v1

RF Company starts are conventionally stored in the documented `starts/` folder and declared through the manifest mixinto `railforgeCompanyStarts`; placing the file there without the mixinto declaration is insufficient. The detailed linked schema is missing locally, so this inventory is **runtime-confirmed for RF 1.98**.

Version 1 is a closed schema: unknown properties are rejected, duplicate `startId` values have no load-order winner, and each file is a complete start plan rather than a partial patch. Starts apply only while creating a new Company and must never reapply to an existing save.

All root fields below are required except optional `$schema` and optional/null `locomotiveOverrideSlot`. The runtime permits `$schema` without type-checking it. Property names are case-insensitive, but duplicate casing aliases are rejected; exactly one start object is allowed per file.

| RF field | Type | Closest FUSE concept |
| --- | --- | --- |
| `$schema` | optional, runtime-untyped; conventionally string | Format metadata. |
| `schemaVersion` | integer `1` | Format metadata. |
| `startId` | string | No native FUSE Company-start graph root. |
| `displayName` | string | No direct counterpart. |
| `progressionId` | string | Related to FUSE progression selection, but Company-start semantics differ. |
| `showTutorial` | boolean | No counterpart. |
| `openingCash` | integer | No counterpart. |
| `playerSpawn` | object | Do not equate automatically with global `world.spawnPoints[]`. |
| `enabledFeatures[]` | string array | Could derive from intended FUSE initial feature state, but is start-specific. |
| `trainPlacements[]` | object array | No public FUSE graph counterpart. |
| `locomotiveOverrideSlot` | object | No counterpart. |

`playerSpawn` fields are `position`, `rotationEuler`, `radius`, and `priority`; its two vectors are exact three-number arrays in this format.

Each `trainPlacements[]` entry contains:

- `cars[]` of string car identifiers;
- `track` with `segmentId`, `distance`, and exact `end: A` or `B`;
- required boolean `wreck`;
- optional `oil` finite fraction 0–1, default 1;
- optional `load: null` or `{id, percent}`, where `percent` is required and 0–1;
- optional `initialFuelWaterPercent` fraction 0–1 with the conditional default described below.

`locomotiveOverrideSlot` contains `placementIndex` and `carIndex`.

At least one complete train placement is required, and every placement requires a non-empty `cars[]`; duplicate car IDs are allowed. `enabledFeatures[]` is required, may be empty, and rejects duplicates. A locomotive override must select a car in a non-wreck placement, and `initialFuelWaterPercent` is valid only when `load` is null or absent.

If `initialFuelWaterPercent` is omitted, its default is `load.percent` when the placement is wrecked or has a non-null load; otherwise it is 1. Consequently, a wreck with no load defaults to 0.

`playerSpawn.position` and `rotationEuler` are required; radius is optional 0–1,000,000 with default 3, and priority is an optional integer defaulting to 1. `openingCash` is 0–1,000,000,000. Runtime caps include 8 MiB per file, depth 64, 256 start files, ID length 256, display-name length 160, 512 enabled features, 128 train placements, 256 cars per placement, and 2,048 cars total. Override indices are placement 0–127 and car 0–255.

RF `1.98` also imports legacy target `railforgeLegacyCompanyStarts` through internal adapter `joo-custom-spawn-points-v1` (the latter is not an author-facing target). Compatibility fields are `identifier`, `name`, `progressionId`, `showTutorial`, `initialMoney`, `spawnPoint {location,rotation,range,priority}`, `enabledFeatures`, and `carPlacements[]`. Despite its singular name, `carIdentifier` is a required array of 1–256 car IDs; other placement fields are `location`, `wreck`, optional `oiled` fraction 0–1 defaulting to 1, `loadPercent`, and `loadId`. Legacy Start/End markers normalize to A/B. Treat this as runtime-only migration input, not recommended authoring.

### Other RF-only or under-documented roots

| RF surface | FUSE treatment |
| --- | --- |
| `texts` | RF's own `texts` schema remains undocumented, but the supplied FUSE legacy converter performs a best-effort conversion: object/scalar entries become `world.mapLabels`, null becomes `world.removals.mapLabels`. It recognizes text, position/localPosition, rotation/localRotation, size/fontSize, color, and `NN MPH` speed labels. Verify every converted label. |
| `mandelas` | Use `world.sceneClones` for clone/transform intent and suppression/removal lists for hide intent. |
| `trackGroupsDisableOnUnlock[]` | No direct FUSE section field; redesign using map features or runtime-specific progression. |
| `LoadExporter`, `LoadImporter`, `SimplePassengerStop` | Use a fully qualified custom/native type in FUSE only if FUSE can resolve and validate it; field parity is not documented. |
| `ConfusingSupplements.Bodygroups`, `DestinationSign`, `LabelPrinter`, `CS.LiverySwap`, `Refiller` | RF `1.98` embeds concrete Definition-component contracts, and FUSE has legacy compatibility for these families. They are asset Definition data, not graph roots; see the table below. |
| `IndustryRefiller` / `ConfusingSupplements.IndustryComponents.RefillerComponent` | Despite its namespace, this is another rolling-stock Definition-component discriminator using the Refiller contract, not a game-graph industry component. |

### Supplemental Definition-component namespaces

| Normalized type and aliases | Runtime-confirmed data shape | Cross-runtime note |
| --- | --- | --- |
| `ConfusingSupplements.Bodygroups`; aliases `Bodygroups`, `bodygroups` | Base `name`, `enabled`; `groups` dictionary; each group has `name`, `options` dictionary; each option has `name`, `path[]` | FUSE legacy compatibility exists; translate within asset Definitions, not the game graph. |
| `ConfusingSupplements.DestinationSign`; aliases `DestinationSign`, `destinationSign`, `destination-sign` | `name`, `transform`, `parent`, `enabled`, `model` AssetReference, `size` default `[1,1,1]`, `destinations[]` | FUSE legacy compatibility exists. |
| `ConfusingSupplements.LabelPrinter`; aliases `LabelPrinter`, `labelPrinter`, `label-printer` | `name`, `enabled`, `transform`, `parent`, `size`, `content`, `forceColor`, `group`; saved-text ID uses group, otherwise name | FUSE legacy compatibility exists. |
| `CS.LiverySwap`; aliases `Livery`, `livery`, `Liveries` | Base component fields only | FUSE legacy compatibility exists. |
| `ConfusingSupplements.Refiller`; aliases `Refiller`, `refiller` | `name`, `enabled`, `transferRate` integer, default 36,000 units/game-hour, ≥0; values 1–3,599 round to 0 units/second | FUSE legacy compatibility exists. |
| `IndustryRefiller`; alias `ConfusingSupplements.IndustryComponents.RefillerComponent` | Same Definition-component `name`, `enabled`, `transferRate` contract as Refiller; no extra fields | The CLR namespace is misleading: this derives from `Model.Definition.Component`, not `Model.Ops.IndustryComponent`. |

`CS.LiverySwap` gets its selectable liveries from an exact directory-mixinto target named `livery:<rolling-stock-definition-id>`. `Standard` is the implicit baseline; each confined mixinto directory basename becomes the option ID and label, with duplicate keys removed case-insensitively. FUSE has corresponding legacy livery-mixinto compatibility. RF stores the selected option under save-state key `cs.livery` and applies these safety caps: 256 directories, 2,048 image files, 16 MiB per file, 8,192 pixels per dimension, 16,777,216 pixels per texture, 64 loaded textures, and 67,108,864 pixels in total.

## 14. URI and prefab translation

| URI family | FUSE | RailForge |
| --- | --- | --- |
| `vanilla://` | Documented runtime source | Documented for loader/station sources. |
| `path://` | Documented runtime source | Documented for scene paths. |
| `scenery://` | Documented runtime source | Documented reference to RF scenery. |
| `empty://` | Documented runtime source | Documented empty source. |
| `asset://` | Accepted by FUSE schema/tooling | RF translation requires exact asset ID/catalog lookup. |
| `fuse://` | Reserved/FUSE-oriented | No automatic RF translation. |
| `rail://` | Reserved/tooling | No automatic RF translation. |
| `file://` | Accepted/reserved by FUSE schema/tooling | No automatic RF translation; do not ship machine-local paths. |

Documented RF loader prefab names are `coalConveyor`, `coalTower`, `dieselFuelingStand`, `waterTower`, and `waterColumn` under `vanilla://`.

## 15. Editor state, extensions, and unsupported systems

FUSE `editor` contains `projectName`, `lastEditedUtc`, `viewport.position`, `viewport.rotation`, and `selectedObject` with `id` and `type`. It is editor state, not runtime content, and has no RF translation.

FUSE `extensions` accepts arbitrary namespaced objects. Translate an extension only with the owning provider's RF contract. RF component `data`/extra-data and handler records are possible destinations, but they are not a universal passthrough.

Neither supplied public schema defines a general signal/CTC authoring root. FUSE documentation treats that area as deferred, and the supplied RF guide does not establish a signal root. Mark it unsupported until a documented provider contract exists.

## 16. Do not translate these by resemblance alone

- FUSE `map.suppressBaseWorld` is not one RF switch.
- FUSE global `spawnPoints[]` is not RF Company `playerSpawn`.
- FUSE `groupIds[]` shorthand is not one RF scalar `groupId`.
- FUSE feature rules are not RF progression rules.
- FUSE scene paths are not automatically RF scenery IDs.
- FUSE `asset://` values are not automatically raw RF model identifiers.
- A handlerless RF `points[]` spliney is exact-point replacement, not a generic FUSE spliney.
- An RF loader visual is not an operational industry loader.
- An omitted RF property preserves the previous value; it does not clear it.
- An empty RF object does not delete anything.
- A null RF mandela hides; it does not destroy the scene object.
- A matching display name does not prove matching node, span, industry, or component identity.
- A runtime-supported alias is not automatically a stable authoring contract.

## 17. Recommended conversion order

1. Freeze and validate the complete FUSE graph.
2. Create the RF package manifest and native folder structure.
3. Move/translate roots using the master namespace table.
4. Translate nodes, segments, spans, then areas in dependency order.
5. Translate loads, then nest industries under verified RF areas.
6. Translate every component type, field, span reference, and array deliberately.
7. Convert scenery and scene suppressions.
8. Convert handler-backed loaders, stations, turntables, splineys, masks, labels, and pole edits only with the correct provider contract.
9. Normalize map features, progressions, sections, phases, and object references.
10. Record every FUSE-only and RF-only behavior in the project's runtime-difference allowlist.
11. Run static reference checks, then full restart, visual, functional, and save/reload tests in each runtime independently.

## Source coverage

FUSE coverage comes from the supplied `fuse-mod.schema.json`, `umm-info.schema.json`, `FUSE_JSON_SCHEMA.md`, examples, and the included FUSE source/runtime normalization code. RailForge coverage comes from the supplied `RailForge FULL GUIDE.md`, `Info.json`, and inspection of the included RF `1.98.BRAVO99` DLL. The missing linked RF schemas/contracts remain an explicit documentation gap; entries derived only from the DLL are labeled runtime-only or handler/manual.

When newer documentation or runtime versions become available, update this matrix and re-run both runtime test suites before treating any new mapping as release-safe.
