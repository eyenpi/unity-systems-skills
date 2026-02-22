# Package Structure

## UPM Directory Layout

```
com.{company}.{system}/
├── package.json
├── CHANGELOG.md
├── LICENSE.md
├── README.md
├── Runtime/
│   ├── {Company}.{System}.asmdef
│   ├── Components/          # MonoBehaviours
│   ├── Data/                # ScriptableObjects (config, definitions)
│   ├── Events/              # SO Event Channels
│   └── Variables/           # ScriptableVariables, RuntimeSets
├── Editor/
│   ├── {Company}.{System}.Editor.asmdef
│   └── Inspectors/
├── Tests/
│   └── Editor/
│       ├── {Company}.{System}.Tests.asmdef
│       └── *Tests.cs
└── Samples~/
    └── BasicUsage/
        ├── Scenes/
        ├── Scripts/
        └── Data/            # Example SO assets
```

## Assembly Definition — Runtime

```json
{
  "name": "MyStudio.Inventory",
  "rootNamespace": "MyStudio.Inventory",
  "references": [],
  "includePlatforms": [],
  "excludePlatforms": [],
  "versionDefines": [
    {
      "name": "com.mystudio.crafting",
      "expression": "1.0.0",
      "define": "MYSTUDIO_CRAFTING"
    }
  ]
}
```

`versionDefines` auto-set `#define` symbols when another package is installed. Use for optional cross-package features without hard dependencies.

## Define Constraints — Conditional Integration Assembly

For code that should only compile when two packages coexist, create a separate assembly:

```json
{
  "name": "MyStudio.Inventory.CraftingIntegration",
  "rootNamespace": "MyStudio.Inventory.CraftingIntegration",
  "references": [
    "MyStudio.Inventory",
    "MyStudio.Crafting"
  ],
  "defineConstraints": [
    "MYSTUDIO_CRAFTING"
  ]
}
```

This assembly is invisible to the compiler until `MYSTUDIO_CRAFTING` is defined (by the versionDefines above). No `#if` blocks needed in code.

## SemVer Rules

| Bump | When |
|------|------|
| **Major** | Public API removed or changed (SO fields renamed, events removed, method signatures changed) |
| **Minor** | New features added (new SO types, new events, new components) — existing API untouched |
| **Patch** | Bug fixes only — no API changes |

## Distribution

**Git + tags (recommended):**

```bash
git tag v1.2.0 && git push origin v1.2.0
```

Users install via Package Manager: `https://github.com/org/com.mystudio.inventory.git#v1.2.0`

**OpenUPM:** Register at openupm.com for discoverability and automatic version tracking.

**Embedded development:** Clone into `Packages/com.mystudio.inventory/` for active development alongside a game project. Move to Git distribution when stable.
