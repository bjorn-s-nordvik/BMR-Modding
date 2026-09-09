# One Package, Two Runtimes

## How to translate a legacy data mod and ship it for both FUSE and RailForge

> **Status:** Community-tested compatibility pattern, not an official promise from either loader. The proof of concept was verified on 4 September 2026 with FUSE development build `0.0.0+1eeef152ec57066bb7a90e1ef6edb27f6018e690` and RailForge `1.98.BRAVO99` (`0.12.99.0`). Re-test after updating Railroader, FUSE, RailForge, or a required asset pack.

> **New to the setup?** Start with the [simple folder-and-manifest quickstart](./FUSE-RailForge-Dual-Runtime-Quickstart.md). This document is the detailed conversion, troubleshooting, and validation reference. Use the [complete Legacy → FUSE ↔ RailForge translation matrix](./FUSE-RailForge-Complete-Translation-Matrix.md) when you need legacy source roots, every supplied native namespace, arrays, fields, component types, compatibility paths, and unsupported cases aligned side by side.

This guide shows how to translate an existing legacy data mod, when applicable, and distribute one mod folder that works when the player chooses **either FUSE or RailForge**. The two loaders do not consume the same native graph dialect. The trick is to keep one FUSE graph and one RailForge graph in loader-specific locations inside the same package, while treating RailLoader/Strange Customs-family content as source or an explicitly selected compatibility path rather than a third duplicate graph.

The recommended authoring workflow is:

1. If the source is legacy, inventory its manifests, data roots, mixintos, handlers, assets, and binaries.
2. Convert or rebuild the supported data in FUSE and finish that native version.
3. Freeze the FUSE graph as the semantic source of truth.
4. Add a RailForge manifest and a translated RF sidecar graph, or deliberately select RF's supported legacy reader for a specific payload.
5. Keep runtime-specific differences in a small, documented allowlist.
6. Test the exact same package in a clean FUSE environment.
7. Test it again in a clean RailForge environment.

The result is **one installation package with two native graph branches**, not one universal JSON file and not a bridge between the loaders.

## Scope and limits

This method is intended for data-only graph/content mods: track, spans, areas, industries, scenery, removals, progression, and compatible handler-backed data.

It does **not** automatically make compiled DLL mods dual-runtime. Code mods may depend on loader APIs, assemblies, patch points, and licences. Those need a separate engineering and distribution plan.

Important boundaries:

- FUSE and RailForge must be treated as mutually exclusive runtime choices.
- RailForge's legacy compatibility is substantial but contract-specific; it does not make every legacy plugin or managed-object mixinto natively portable.
- FUSE's converter translates JSON/data, not arbitrary compiled behavior.
- Players need the dependency editions appropriate to the runtime they selected.
- Stable object IDs help, but do not by themselves guarantee that one save can move safely between runtimes.
- The folder-layout pattern was proven with a normal/manual UMM-style folder installation. Test any installer that performs its own cross-manifest dependency checks before advertising installer compatibility.

## Why the layout works

Use this package structure:

```text
Railroader/
└── Mods/
    └── YourName.ExampleYard/
        ├── Info.json
        ├── Definition.json
        ├── example-yard.fuse.json
        └── RailForge/
            └── game-graph/
                └── example-yard.json
```

FUSE is pointed explicitly at the root graph by `FuseDataFiles`. RailForge recursively discovers JSON under `RailForge/game-graph`. This directory boundary prevents either loader from interpreting the other loader's graph.

Do not:

- put the RF graph in the mod root;
- add the RF graph to `FuseDataFiles`;
- put a backup ending in `.fuse.json` in the root;
- add a `game-graph` mixinto just to make the native RF folder work;
- install an original copy and a test copy with the same package ID together.

For a new project, use one neutral, stable identity everywhere:

```text
Folder name                  YourName.ExampleYard
Info.json -> Id              YourName.ExampleYard
Definition.json -> id        YourName.ExampleYard
```

Using the same exact ID in both manifests is the safest tested pattern. If an already-published FUSE mod has a `.FUSE` suffix, preserving its existing ID is normally safer than breaking saves, dependencies, and load-order references merely to make the name neutral.

## The two manifests

### `Info.json` for FUSE discovery

```json
{
  "$schema": "https://hunterr.dev/fuse/schemas/umm-info.schema.json",
  "Id": "YourName.ExampleYard",
  "DisplayName": "Example Yard",
  "Author": "Your name",
  "Version": "1.0.0",
  "ManagerVersion": "0.27.10",
  "Requirements": [],
  "LoadAfter": [
    "FUSE"
  ],
  "FuseLoadPriority": 100,
  "FuseRequires": [],
  "FuseLoadAfter": [],
  "FuseLoadBefore": [],
  "FuseConflictsWith": [],
  "FuseDataFiles": [
    "example-yard.fuse.json"
  ]
}
```

List every intended FUSE definition explicitly in `FuseDataFiles`, in load order.

The unusual but important dual-runtime detail is this:

```json
"Requirements": [],
"LoadAfter": ["FUSE"]
```

A normal FUSE-only package may use a hard generic `Requirements: ["FUSE"]`. In the tested dual package, RailForge also audited that generic requirement and reported missing FUSE in the RF-only environment. Explicit `FuseDataFiles` allowed FUSE to discover and load the package. Keeping FUSE only as an optional `LoadAfter` target preserved UMM ordering when present without creating a hard cross-runtime dependency.

Use `FuseRequires`, `FuseLoadAfter`, and `FuseLoadBefore` for real relationships between FUSE data packages. Do not use ordering as a substitute for a required dependency. Any generic UMM dependency placed in `Requirements` must also make sense in the RF-only installation, or it can defeat the dual-runtime design.

### `Definition.json` for RailForge discovery

```json
{
  "manifestVersion": 5,
  "id": "YourName.ExampleYard",
  "name": "Example Yard",
  "author": "Your name",
  "version": "1.0.0",
  "requires": [
    {
      "id": "Example.NativeDependency",
      "notBefore": "1.2.0"
    }
  ],
  "loadAfter": [
    "Example.NativeDependency"
  ]
}
```

Remove the example dependency when none is required. In RailForge:

- `requires` declares hard native/RF dependencies and their version bounds;
- `loadAfter` and `loadBefore` create deterministic ordering edges;
- an ordering edge should exist only when your patch must see another mod's result;
- ordering cannot reconcile two logically incompatible edits to the same record.

Do not add FUSE as a RailForge dependency. Do not add RailForge as a FUSE dependency. The content package supports both; the runtimes remain alternatives.

Keep the ID, display metadata, and version synchronized in both manifests for every release.

## Legacy source and RailForge legacy logic

RailForge `1.98` does use legacy logic. The inspected runtime includes RailLoader-facing mod and mixinto interfaces, a legacy modding context, exact scans for `StrangeCustoms`/`SC` and legacy asset-pack roots, conditional file/directory mixinto parsing, plus adapters for selected AMM, ConfusingSupplements, audio, Company-start, and migration contracts.

That means a supported data-only legacy payload can be applicable directly in RF. It does **not** mean that every legacy format is native RF, or that the legacy payload should remain active after the same content has been translated into `RailForge/game-graph`.

| Legacy content | FUSE behavior | RailForge behavior | Decision |
| --- | --- | --- | --- |
| Track/route JSON | Converter/runtime adapter reshapes supported roots to native FUSE | Legacy reader can ingest supported file/directory mixintos | Prefer the two audited native branches for maintained releases. |
| `StrangeCustoms/` or `SC/` | Recognized legacy data can be converted | Exact legacy roots are scanned by RF | Use as input or a deliberate RF fallback, never alongside an equivalent RF-native graph. |
| Legacy asset-pack roots | Supported packs can be mounted/discovered | Exact legacy roots and aliases are indexed | Reuse only when both runtimes resolve the same catalog and Definition IDs. |
| AMM/ConfusingSupplements serialized contracts | Selected handlers/components convert or use compatibility implementations | Selected handlers/components have embedded adapters | Verify the exact discriminator and required provider. |
| Horn/whistle/bell data | Converts to FUSE audio dictionaries | RF runtime catalogs accept the corresponding legacy mixinto families | Validate paths, profiles, and keyframes separately. |
| Legacy DLL or managed object | No code conversion; compatibility hosting is partial | External assembly/plugin support is compatibility-dependent | Reimplement or explicitly document the required provider. |

### Conditional mixintos in the dual package

Conditions must retain **fragment scope**. A missing requirement or matching conflict should skip only the optional fragment; package-level requirements block the entire package.

FUSE lists the optional fragment in `Info.json`:

```json
"FuseDataFiles": [
  "example-yard.fuse.json",
  "optional-yard.fuse.json"
]
```

The optional FUSE fragment carries its own metadata:

```json
{
  "schemaVersion": "1.0",
  "id": "YourName.ExampleYard.Optional",
  "name": "Example Yard Optional Layer",
  "author": "Your name",
  "mixinto": {
    "target": "game-graph",
    "sourceFile": "optional-yard.fuse.json",
    "requires": [
      { "id": "Example.BaseRoute", "notBefore": "1.2.0" }
    ],
    "conflictsWith": [
      { "id": "Example.IncompatibleRoute" }
    ]
  }
}
```

RF points the same logical condition at a separately translated RF payload:

```json
{
  "mixintos": {
    "game-graph": [
      {
        "mixinto": "file(RailForge/conditional/optional-yard.json)",
        "requires": [
          { "id": "Example.BaseRoute", "notBefore": "1.2.0" }
        ],
        "conflictsWith": [
          { "id": "Example.IncompatibleRoute" }
        ]
      }
    ]
  }
}
```

Keep `RailForge/conditional/optional-yard.json` outside `RailForge/game-graph`. Files below `game-graph` are discovered natively and therefore bypass the condition. RF accepts legacy aliases and plural path forms, but new authoring should use canonical `mixinto`, `requires`, and `conflictsWith` spellings.

`sourceFile` is provenance metadata on the FUSE fragment; `FuseDataFiles` performs FUSE discovery. `loadAfter`/`loadBefore` are ordering, not eligibility. If FUSE and RF editions of a dependency use different IDs, write the runtime-correct ID in each branch.

## Authoring workflow

### 0. Classify a legacy source

Before conversion, keep an untouched source copy outside the release folder and inventory:

- `Info.json`, `Definition.json`, redirects, requirements, conflicts, and ordering;
- every file/directory/conditional mixinto target;
- top-level graph roots and patch operators;
- handler and component discriminators;
- asset-pack, map-tile, audio, migration, and Company-start payloads;
- every DLL/PDB and behavior that cannot be expressed as data.

Mark each item `converted`, `RF legacy-direct`, `preserved-only`, `provider/code`, or `unsupported`. This prevents a successful track conversion from hiding a missing script behavior.

### 1. Finish the FUSE version first

Build the complete mod in native FUSE format. Test its tracks, operations, scenery, progression, and save behavior before translating anything.

When it is accepted:

- make a protected working copy;
- record the SHA-256 hash of the FUSE graph;
- treat the FUSE file as the semantic source of truth;
- never patch the FUSE branch merely to satisfy an RF-specific problem.

For example:

```powershell
Get-FileHash ".\example-yard.fuse.json" -Algorithm SHA256
```

### 2. Add the RF sidecar

Add `Definition.json` and create `RailForge/game-graph/example-yard.json`. Translate from the frozen FUSE graph into RF's native schema.

Do not maintain two unrelated designs. When the FUSE graph changes:

1. update and retest FUSE;
2. translate the same semantic change into RF;
3. compare IDs and references;
4. retest both runtimes;
5. update the intentional-difference allowlist.

### 3. Preserve identity wherever semantics match

Node, segment, span, area, industry, scenery, map-feature, and component IDs are contracts. Preserve them across branches whenever they describe the same logical object.

Use a different runtime-specific ID only when there is a proven semantic or lifecycle reason. Document the old ID, new ID, reason, affected references, and save-compatibility impact.

## Legacy/FUSE-to-RailForge conversion reference

This is a conversion guide, not a blind search-and-replace recipe. RailForge is a merge/patch format, while FUSE describes authored intent and also supports conveniences and aliases.

The tables below cover the most common conversion work. The separate [complete translation matrix](./FUSE-RailForge-Complete-Translation-Matrix.md) is the exhaustive Legacy → FUSE ↔ RailForge ledger and should be the authority when a source root, compatibility handler, namespace, or array is not shown here.

### Top-level structure

| FUSE path | RailForge path | Action |
| --- | --- | --- |
| FUSE file metadata such as `$schema`, `schemaVersion`, `id`, `author`, `coordinateSpace` | `Definition.json` or no RF graph equivalent | Keep package metadata in the manifests; do not copy FUSE-only graph metadata blindly. |
| `tracks.nodes` | `tracks.nodes` | Preserve IDs and supported geometry fields. |
| `tracks.segments` | `tracks.segments` | Preserve IDs; rename endpoint fields and convert enums. |
| `tracks.spans` | `tracks.spans` | Preserve IDs; translate endpoint names. |
| `tracks.areas` | `areas` | Move to the RF top level. |
| `operations.loads` | `loads` | Move to the RF top level. |
| `operations.industries` | `areas.<areaId>.industries` | Nest each industry under its real RF area. |
| `world.scenery` | `scenery` | Move to the RF top level and translate asset identity. |
| `world.splineys` | `splineys` | Move to the RF top level, then convert to the required RF handler format. |
| `progression.mapFeatures` | `mapFeatures` | Move to the RF top level and review visibility semantics. |
| `progression.progressions` | `progressions` | Move to the RF top level and normalize references. |

### Nodes

For ordinary PoC nodes, the ID plus `position`, `rotation`, and `flipSwitchStand` transferred directly. Keeping vectors as `{ "x": ..., "y": ..., "z": ... }` reduces conversion mistakes even where RF also accepts arrays.

FUSE-only or less common fields such as `isDiamond`, `gauge`, `tags`, partial/preserve controls, and some group/area conveniences do not have a universal one-to-one rule. Resolve them against the current RF schema/editor and test the resulting runtime object.

### Segments

| FUSE field/value | RailForge field/value |
| --- | --- |
| `startNodeId` | `startId` |
| `endNodeId` | `endId` |
| `style: "standard"` | `style: "Standard"` |
| `style: "bridge"` | `style: "Bridge"` |
| `style: "tunnel"` | `style: "Tunnel"` |
| `style: "yard"` | `style: "Yard"` |
| `trackClass: "main"` | `trackClass: "Mainline"` |
| `trackClass: "branch"` | `trackClass: "Branch"` |
| `trackClass: "industrial"` | `trackClass: "Industrial"` |
| `priority`, `speedLimit` | Copy when semantically unchanged. |
| `groupId` | Preserve only when RF has the intended controlling progression/feature; otherwise decide explicitly. |

FUSE example:

```json
"S_example_01": {
  "style": "yard",
  "trackClass": "industrial",
  "startNodeId": "N_example_A",
  "endNodeId": "N_example_B",
  "priority": 0,
  "speedLimit": 0
}
```

RailForge equivalent:

```json
"S_example_01": {
  "style": "Yard",
  "trackClass": "Industrial",
  "startId": "N_example_A",
  "endId": "N_example_B",
  "priority": 0,
  "speedLimit": 0
}
```

### Spans

| FUSE | RailForge |
| --- | --- |
| `segmentId` | `segmentId` |
| `distance` | `distance` |
| `end: "Start"` | `end: "A"` |
| `end: "End"` | `end: "B"` |
| `normalize` omitted | Write `"normalize": true` when matching FUSE's default normalized behavior. |

FUSE example:

```json
"example_service": {
  "upper": {
    "segmentId": "S_example_01",
    "distance": 0.0,
    "end": "Start"
  },
  "lower": {
    "segmentId": "S_example_01",
    "distance": 0.0,
    "end": "End"
  }
}
```

RailForge equivalent:

```json
"example_service": {
  "upper": {
    "segmentId": "S_example_01",
    "distance": 0.0,
    "end": "A"
  },
  "lower": {
    "segmentId": "S_example_01",
    "distance": 0.0,
    "end": "B"
  },
  "normalize": true
}
```

If a FUSE endpoint uses a normalized value instead of canonical `distance`, calculate the RF distance against the final segment length. FUSE endpoint offsets have no confirmed general RF equivalent and require a manual solution.

### Areas and industries

FUSE industries are stored centrally and identify their area. RF industries are nested under that area.

| FUSE | RailForge |
| --- | --- |
| `operations.industries.<industryId>` with `areaId` | `areas.<areaId>.industries.<industryId>` |
| industry `position` | industry `localPosition` |
| `name`, `usesContract` | Copy. |
| `replaceComponents: true` | `"components": { "$replace": { ... } }` |
| `mergeComponents: true` | Normal RF `components` merge. |
| component `trackSpanIds` | component `trackSpans` |
| FUSE component alias | Fully qualified concrete RF component type. |

When replacing an existing vanilla industry that has no `areaId` in the FUSE patch, find its actual area in the RF baseline/viewer. RF does not infer area membership from display name or physical proximity.

Minimal FUSE component example inside `operations.industries`:

```json
"example-engine": {
  "name": "Example Engine Service",
  "position": { "x": 10.0, "y": 0.0, "z": -20.0 },
  "usesContract": false,
  "replaceComponents": true,
  "components": {
    "coal": {
      "type": "unloader",
      "name": "Coal Track",
      "trackSpanIds": ["example_service"],
      "loadId": "coal",
      "carTypeFilter": "HM,HT"
    }
  }
}
```

RailForge equivalent inside `areas.example.industries`:

```json
"example-engine": {
  "name": "Example Engine Service",
  "localPosition": { "x": 10.0, "y": 0.0, "z": -20.0 },
  "usesContract": false,
  "components": {
    "$replace": {
      "coal": {
        "type": "Model.Ops.IndustryUnloader",
        "name": "Coal Track",
        "trackSpans": ["example_service"],
        "loadId": "coal",
        "carTypeFilter": "HM,HT"
      }
    }
  }
}
```

Copy applicable standard fields such as storage limits/rates, car filters, ordering flags, and `canOverhaul`. Then validate the live component and its span; a valid merged JSON object does not prove that the runtime component is active.

### Standard industry component types

| FUSE alias | RailForge concrete type |
| --- | --- |
| `loader` | `Model.Ops.IndustryLoader` |
| `unloader` | `Model.Ops.IndustryUnloader` |
| `formulaic` | `Model.Ops.FormulaicIndustryComponent` |
| `repairTrack` | `Model.Ops.RepairTrack` |
| `teamTrack` | `Model.Ops.TeamTrack` |
| `interchange` | `Model.Ops.Interchange` |
| `interchangedLoader` | `Model.Ops.InterchangedIndustryLoader` |
| `teleportLoading` | `Model.Ops.TeleportLoadingIndustry` |
| `progression` | `Model.Ops.ProgressionIndustryComponent` |

Passenger stops and custom components need special care. For RF `1.98`, consult the current RF guide/editor for the supported passenger-station representation. A custom component must resolve to an available, concrete `IndustryComponent` type, so its DLL/provider must exist in that runtime.

RailForge cannot change the runtime type of an existing component in place under the same ID. If the type must change, use an explicitly planned remove/add sequence or a new component ID, and test save implications.

### RF merge semantics you must understand

RailForge graph files are patches:

| RF value | Meaning |
| --- | --- |
| Property omitted | Preserve the existing value. |
| Scalar/object property supplied | Merge/update that property. |
| Array supplied | Replace the array. |
| `[]` | Clear the array. |
| `{}` | Empty merge; it does not delete the existing object. |
| `null` at a supported keyed record | Tombstone/remove the record. |
| `{ "$replace": { ... } }` | Replace the supported subtree with the supplied complete value. |

Examples of common removals:

```json
{
  "tracks": {
    "spans": {
      "old-span-id": null
    },
    "segments": {
      "old-segment-id": null
    },
    "nodes": {
      "old-node-id": null
    }
  }
}
```

An omitted `groupId` does not mean “remove the group”; it means “preserve what is already there.” This distinction was central to the East Whittier PoC.

Advanced FUSE partial array operations may need an explicit RF `$find`/array patch or a precomputed final array. Do not assume RF appends to an array simply because the property appears in a patch.

### Scenery and vanilla-object removal

| FUSE | RailForge |
| --- | --- |
| `world.scenery.<id>` | `scenery.<id>` |
| `assetIdentifier` | `modelIdentifier` |
| `asset://SomeId` | Often `SomeId`, but confirm the exact RF identifier in the asset viewer. |
| `position`, `rotation`, `scale` | Copy when supported. |
| remove authored keyed scenery | `scenery.<id>: null` |
| hide an existing scene object | `mandelas.<full scene path>.enabled: false` |

Example of a native scene-object hide:

```json
{
  "mandelas": {
    "World/Large Scenery/Area/Exact Object Name": {
      "enabled": false
    }
  }
}
```

Use the exact full scene path reported by RF tools. A FUSE removal name is not automatically a valid RF Mandela path.

### Splineys and telegraph-pole edits

Handler-backed data is not a mechanical field rename. Convert it to the handler schema supported by the RF version you target.

For movement of existing telegraph poles, RF `1.98` used this shape:

```json
{
  "splineys": {
    "your-mod-telegraph-moves": {
      "handler": "RailForge.MapEditor.TelegraphPoleEdits",
      "schemaVersion": 3,
      "polesToMove": [581, 583, 585],
      "poleMovement": [
        [1.725, 0.0, 2.454],
        [1.725, 0.0, 2.454],
        [1.725, 0.0, 2.454]
      ]
    }
  }
}
```

There must be one movement vector per pole, in matching order. Creating new pole sets and converting FUSE-native road, river, terrain-road, trestle, or object-line splineys was not proven by this PoC; author or export those with the current RF handler/editor instead.

### Progression and map features

Progression needs semantic review, not just path movement. Common mappings include:

| FUSE | RailForge |
| --- | --- |
| `progression.mapFeatures.<id>` | `mapFeatures.<id>` |
| `progression.progressions.<id>` | `progressions.<id>` |
| `initiallyEnabled` | `defaultEnableInSandbox` |
| `prerequisiteFeatureIds` | `prerequisites` |
| section `prerequisiteSectionIds` | `prerequisiteSections` |
| phase `industryComponentId` | RF object reference containing `areaId`, `industryId`, and `componentId` |

FUSE `groupIds` shorthand may need expansion into both `trackGroupsEnableOnUnlock` and `trackGroupsAvailableOnUnlock`. Flat FUSE sections may need normalization under the correct RF progression and section IDs. Component include/exclude references likewise need RF area/industry/component objects rather than compact strings.

RF can materialize progression definitions without enforcing their visibility. If the mod relies on RF-managed hide/show behavior, verify the current RF progression-materializer and initial-visibility settings. Test Sandbox and Company separately; `defaultEnableInSandbox` is not a Company-start mechanism.

## Runtime-specific differences: use an allowlist

Most IDs and semantics should match. Put necessary differences in a small table stored with the project, for example:

| Object | FUSE value | RF value | Why | Retest |
| --- | --- | --- | --- | --- |
| segment group | `MY_GROUP` | `""` | RF branch intentionally has no group controller | Track counts and progression |
| component ID | `old-component` | `new-component` | Proven RF lifecycle/type replacement issue | UI, operations, save reload |

Never hide a mismatch by simply excluding it from comparison. An exception should be narrow, justified by observed behavior, and covered by its own test.

## Lessons from the East Whittier proof of concept

The following fixes made the PoC pass, but they are **not universal conversion rules**.

### 1. Ten RF segments disappeared because of a FUSE-owned group

Ten segments were present in the merged RF graph but absent from the rendered track network. They retained `groupId: "EWH"`, while the RF environment had no active controller for that FUSE-owned group. RF's native rebuild filtered all ten.

The intended RF behavior was “these tracks always exist,” so only the RF branch explicitly used:

```json
"groupId": ""
```

The FUSE branch kept `groupId: "EWH"`. Normally, preserve the group and translate its progression. Clear it only when always-enabled track is the intended RF behavior.

### 2. An industry definition existed, but its UI row did not

The vanilla `rip` and `rip-parts` components shared a runtime GameObject. During RF `$replace`, removal of the stale `rip` component deactivated that shared host. Reusing `rip-parts` left the definition present but its live child inactive.

The PoC used a fresh RF-only ID, `repair-parts`, which forced creation of an active component. FUSE retained `rip-parts`. This is a targeted RF `1.98` workaround, not a rule to rename every component.

### 3. Visual geometry was not enough to prove span identity

The custom repair spans had the same physical geometry as two vanilla spans. The in-game highlight therefore looked correct even when identity remained ambiguous. Runtime binding logs were required to prove that the industry used the authored span IDs.

### 4. Scene paths and pole indexes are map-specific

The engine-shed Mandela path and telegraph-pole indexes used by East Whittier belong only to that map state. Every author must obtain exact paths and IDs for their own target and version.

## Test the two runtimes independently

A successful menu load is not proof. Each runtime must independently prove that it discovered the right file, ignored the other branch, built the complete graph, resolved operations, applied progression, and survived a save/reload cycle.

### Prepare two clean environments

| Environment | Active runtime | Dependencies |
| --- | --- | --- |
| FUSE test | FUSE only | FUSE editions of required assets/content |
| RailForge test | RailForge only | Native or RF-compatible editions |
| RF legacy-compatibility test, only if that path will ship | RailForge only, with the equivalent native RF payload removed from the disposable test copy | Exact legacy providers/assets required by that payload |

Never swap files or loaders while Railroader is running. Fully exit and confirm the process has stopped. Keep exactly one copy of the dual package active.

Use two stages for each runtime:

1. **Minimal:** loader, the dual package, and direct dependencies only.
2. **Representative:** the normal route and companion-mod stack.

The minimal run isolates your package. The representative run finds ordering conflicts and competing edits.

### Static checks before launch

At minimum, automate or manually verify:

- both manifests and all graphs parse as JSON;
- manifest IDs and versions match;
- `FuseDataFiles` contains only the intended root FUSE files;
- unconditional RF graphs exist below `RailForge/game-graph`, while conditional RF payloads live outside auto-discovered roots and are referenced once from `Definition.json.mixintos`;
- no logical object is supplied through both an active native branch and a legacy compatibility path;
- every legacy DLL or managed-object behavior is either proven through its compatibility provider or explicitly classified unsupported;
- intended node, segment, span, and scenery ID sets match;
- segment endpoint references survive the field rename;
- all span segment IDs exist in the final target graph;
- every industry points at the intended area and span IDs;
- component aliases became valid RF concrete types;
- `$replace`, arrays, omissions, and tombstones express the intended final state;
- every runtime-specific difference is in the allowlist;
- the frozen FUSE hash is unchanged.

If a probe calls private loader/scanner code, treat it as version-sensitive. It supplements live testing; it never replaces it.

### FUSE run

Inspect `FUSE.log` and confirm:

- the package is `ready` and comes from the expected folder;
- only the declared root `.fuse.json` files are loaded;
- every conditional FUSE fragment is either applied or skipped for the documented requirement/conflict reason;
- the nested RF graph is never listed as a FUSE definition;
- the package's apply report has the expected counts;
- the package has no unexplained warnings, errors, skips, or fatal result;
- replaced industries have the intended final component counts;
- live components bind to the intended spans;
- progression-locked objects have the expected initial state.

Useful FUSE diagnostics include:

```text
/fuse.report
/fuse.loaded
/fuse.conflicts
/fuse.graph
/fuse.operations
/fuse.progressions
/fuse.assets
/fuse.dumpgraph
/fuse.dumpruntimegraph
```

An intentional registry conflict can be acceptable when your package deliberately replaces an upstream object, but prove the expected load order and winner.

### RailForge run

Start with `RailForge-support-report.txt`, then inspect `Player.log`. Confirm:

- dependency audit succeeds;
- the expected package and nested graph file are indexed;
- every conditional RF/legacy mixinto is applied or skipped for the documented reason and is not also found through native discovery;
- every expected patch merges and commits;
- there are no package-related materializer failures;
- node/segment/span/area/industry/scenery/spliney counts match your baseline;
- every industry has the expected definition and active live component count;
- deferred scenery is built and activated;
- Mandela targets and handler-owned IDs resolve;
- track rebuild counts are correct before and after saved progression is applied.

RF may emit `RF-FUSE-DATA-ONLY-001` for a dual data package. That informational boundary diagnostic explains that FUSE-only material was restricted while RF-owned content remained eligible. It does not excuse dependency or graph errors.

Judge logs per package. A large mod stack can contain unrelated warnings; filter by package ID, graph filename, industry ID, and affected object IDs. Every warning from your package must still be understood.

### Visual and functional checks

In each runtime, test:

- all new and modified rail from several angles;
- switches, crossings, frogs, bumpers, ballast, and continuity;
- track before and after progression state is restored;
- scenery plus every vanilla object intended to be hidden;
- Company → Locations and every expected industry row;
- the highlighted span for each component;
- actual loading, unloading, repair, interchange, purchase, and passenger behavior used by the mod;
- locked and unlocked progression states;
- every published Company start;
- map unload and reload.

Test visuals and operations separately. A loader model can exist without a working industry component, and a working industry can exist without its expected model.

### Mandatory save tests

Use disposable copies of saves.

For each runtime:

1. Start a fresh game.
2. Inspect and operate the changed area.
3. Save to a new slot.
4. Exit Railroader completely.
5. Restart with the same runtime.
6. Reload and repeat the checks.
7. Unload/reload the map and inspect logs again.

If existing saves are supported, test a copied pre-mod save. If cross-runtime save portability is advertised, test both directions on disposable copies:

```text
FUSE save -> RailForge -> save -> RailForge reload
RailForge save -> FUSE -> save -> FUSE reload
```

The same package ID does not guarantee save portability. Saves can retain rolling-stock positions, segment IDs, progression state, component identity, inventory, waybill destinations, and Company-start state.

## Troubleshooting

| Symptom | Likely cause | First checks |
| --- | --- | --- |
| RF reports missing FUSE with `RF-DEP-001` | `Info.json` declares FUSE as a hard generic requirement | Use the tested data-only dual pattern: no hard generic FUSE requirement, explicit `FuseDataFiles`, and optional `LoadAfter: ["FUSE"]`. |
| RF graph is ignored | Wrong folder, bad manifest, duplicate ID, or dependency block | Check `RailForge/game-graph/**/*.json`, the support report, package ID, and indexed file count. |
| FUSE loads the wrong graph | Fallback discovery or an unwanted root backup | Pin `FuseDataFiles`; remove release backups ending in `.fuse.json`; verify the logged path. |
| Conditional RF content always loads | Its JSON is beneath `RailForge/game-graph`, so native discovery bypasses the condition | Move it to a non-auto-discovered path such as `RailForge/conditional` and reference it once from the conditional mixinto. |
| Nodes, rails, industries, or scenery appear twice | Equivalent legacy and native payloads are both active | Choose one active path per runtime; remove the duplicate legacy mixinto/root or its native replacement from that test/release design. |
| Legacy JSON works but a feature is missing | The original behavior came from a DLL, managed-object mixinto, or unsupported handler | Read the conversion report, identify the owning provider, and reimplement or document the missing behavior. |
| Graph contains tracks but rail is missing | Unknown/disabled `groupId` | Compare logical and rebuilt counts; translate the controller or explicitly clear only the intended RF groups. |
| Span is invalid | Its segment was filtered/removed, or endpoint/distance is wrong | Prove the live segment first, then verify `A`/`B`, distance, and exact IDs. |
| Industry retains vanilla spans/components | RF merged where FUSE intended replacement | Check the component `$replace`, stale-component removal, and final live bindings. |
| Correct definition count, missing UI row | Reused component host is inactive or its type cannot change in place | Enable focused verbose logging; use a fresh RF-only ID only when the lifecycle/type issue is proven. |
| Loader model exists, but cars cannot load | Visual object without operational component | Check concrete component type, `loadId`, and live `trackSpans`. |
| Operations work, but model is absent | Scenery/loader asset failed | Check asset ID, dependency edition, deferred build, and activation. |
| Progression exists, visibility is wrong | Materializer settings, mode, or saved state differ | Test Sandbox, fresh Company, and existing Company independently. |
| Live reload works, clean restart fails | Cached state masked ordering/lifecycle errors | Treat a complete process restart as authoritative. |
| Fresh save works, old save fails | A stored segment/component/progression identity changed | Preserve IDs, test a copied save, or provide/document a migration. |
| Results vary between launches | Both loaders, duplicate packages, stale files, or stale logs | Fully exit, rebuild the active mod set, keep one package copy, and capture a clean log. |

## Release checklist

- [ ] The FUSE graph is complete, frozen, and hashed.
- [ ] The RF branch was regenerated or reviewed after the last FUSE edit.
- [ ] Both manifests have the exact intended ID and version.
- [ ] Every JSON file parses.
- [ ] `FuseDataFiles` names only intended root FUSE graphs.
- [ ] All unconditional RF graphs are beneath `RailForge/game-graph`; conditional payloads are outside native discovery and referenced exactly once.
- [ ] No equivalent legacy graph and native graph are both active in the same runtime.
- [ ] Every retained legacy compatibility path is named, version-bounded where needed, and tested in isolation.
- [ ] Every legacy binary/managed-object dependency is proven or explicitly unsupported.
- [ ] No backup graph, log, test save, private path, editor cache, or unredistributable binary is packaged.
- [ ] Every intentional cross-runtime difference is documented and tested.
- [ ] Minimal and representative FUSE tests pass after full restarts.
- [ ] Minimal and representative RF tests pass after full restarts.
- [ ] Package-scoped logs contain no unexplained warnings or errors.
- [ ] Tracks, spans, scenery, removals, industries, and progression work visually and functionally.
- [ ] Fresh-game save/reload and map reload pass in both runtimes.
- [ ] Existing-save behavior is tested or explicitly marked unsupported.
- [ ] Cross-runtime save movement is tested both ways if advertised.
- [ ] Temporary verbose diagnostics are disabled.
- [ ] The final archive expands to exactly one direct child folder under `Railroader/Mods`.
- [ ] The supported Railroader, FUSE, RailForge, and dependency versions are recorded.

## What the proof of concept established

The East Whittier test demonstrated all of the following with one unchanged installed content package:

- FUSE loaded the explicitly declared root graph and ignored the nested RF graph.
- RailForge loaded the nested graph and restricted the FUSE-only branch.
- Both manifests coexisted with the same package identity.
- FUSE discovered and loaded the package through explicit `FuseDataFiles`; `LoadAfter: ["FUSE"]` supplied only the optional UMM ordering hint.
- Track, spans, scenery, removal, telegraph-pole movement, progression, and a replaced multi-component industry could be represented natively in both branches.
- RF-only fixes could live in the sidecar without altering the FUSE source graph.
- A final FUSE regression still passed after the RF work was complete.

It did not establish automatic compatibility for every FUSE feature, legacy package, compiled mod, managed-object mixinto, installer, future loader version, or save. Treat unsupported handlers and unfamiliar component types as new conversion work, then extend the mapping only after both runtime tests pass.

## The maintenance rule

**Import legacy once; author in FUSE; translate deliberately to RailForge; verify every active path.**

That keeps one creative source of truth, one distributable folder, and two honest native implementations. The folder trick provides isolation. The quality comes from preserving semantics, documenting exceptions, and testing the real runtime objects instead of trusting JSON alone.
