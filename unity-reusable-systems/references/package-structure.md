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
│   ├── SampleSceneGenerator.cs
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

## Sample Scene Generator (Required)

Every package must include an Editor menu item that creates a fully-wired sample scene. This is non-negotiable — users must be able to see the system working with one click.

**Menu path:** `Tools/{Company}/{System}/Create Sample Scene`

**The generator must:**

1. Create a new scene (or add to current scene based on user prompt)
2. Instantiate all required SO assets (config, event channels, variables, runtime sets) into a `SampleScene_Data/` folder
3. Create GameObjects with all MonoBehaviour components attached
4. Wire every serialized field — SO references, event channels, variables, runtime sets
5. The scene must enter Play Mode and demonstrate the system working without manual setup

**Generator script location:** `Editor/SampleSceneGenerator.cs`

```csharp
using UnityEditor;
using UnityEditor.SceneManagement;
using UnityEngine;
using UnityEngine.SceneManagement;

namespace MyStudio.Inventory.Editor
{
    public static class SampleSceneGenerator
    {
        private const string MenuPath = "Tools/MyStudio/Inventory/Create Sample Scene";
        private const string DataFolder = "Assets/SampleScene_Data/Inventory";

        [MenuItem(MenuPath)]
        public static void CreateSampleScene()
        {
            // 1. Create fresh scene
            var scene = EditorSceneManager.NewScene(NewSceneSetup.DefaultGameObjects, NewSceneMode.Single);

            // 2. Ensure data folder exists
            if (!AssetDatabase.IsValidFolder("Assets/SampleScene_Data"))
                AssetDatabase.CreateFolder("Assets", "SampleScene_Data");
            if (!AssetDatabase.IsValidFolder(DataFolder))
                AssetDatabase.CreateFolder("Assets/SampleScene_Data", "Inventory");

            // 3. Create SO assets
            var itemDef = ScriptableObject.CreateInstance<ItemDefinition>();
            itemDef.name = "SampleSword";
            AssetDatabase.CreateAsset(itemDef, $"{DataFolder}/SampleSword.asset");

            var onItemAdded = ScriptableObject.CreateInstance<GameEvent>();
            onItemAdded.name = "OnItemAdded";
            AssetDatabase.CreateAsset(onItemAdded, $"{DataFolder}/OnItemAdded.asset");

            // ... create all SO assets the system needs

            // 4. Create GameObjects with components, wire references
            var inventoryGO = new GameObject("Inventory");
            var inventoryComp = inventoryGO.AddComponent<InventoryManager>();
            // inventoryComp.config = configAsset;
            // inventoryComp.onItemAdded = onItemAdded;

            var listenerGO = new GameObject("UI_Listener");
            var listener = listenerGO.AddComponent<GameEventListener>();
            // listener.gameEvent = onItemAdded;

            // 5. Save scene
            AssetDatabase.SaveAssets();
            EditorSceneManager.SaveScene(scene, $"{DataFolder}/SampleScene.unity");
            EditorSceneManager.MarkSceneDirty(scene);

            Debug.Log("Sample Inventory scene created. Press Play to test.");
        }
    }
}
```

Adapt the pattern above to the specific system — the key requirement is that **every SO field on every component is assigned, every event channel is connected to its listeners, and pressing Play demonstrates the system working**.

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
