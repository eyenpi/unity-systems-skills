---
name: unity-reusable-systems
description: >-
  Use when building reusable Unity game systems, creating UPM packages, or
  designing ScriptableObject-based modular architecture for gameplay systems
  like inventory, combat, dialogue, quests, or save/load. Also use when
  connecting multiple independent Unity packages that need to communicate
  without direct dependencies.
---

# Unity Reusable Systems

Build every gameplay system as a self-contained UPM package. Data lives in
ScriptableObjects, behavior lives in small single-responsibility MonoBehaviours,
and systems talk through SO Event Channels — never direct references.

Target: Unity 6+ / C# 11.

## The Rules

| ALWAYS | NEVER |
|--------|-------|
| One system = one UPM package | Create a "shared contracts" package with interfaces |
| Data in ScriptableObjects, behavior in MonoBehaviours | Put data and behavior in the same class |
| SO Event Channels for cross-system communication | Use singletons, service locators, or static managers |
| One assembly definition per folder (Runtime, Editor, Tests) | Ship without assembly definitions |
| Version Defines for optional cross-package awareness | Use `#if` defines without versionDefines in asmdef |
| Define Constraints for conditional integration assemblies | Create hard dependencies between gameplay packages |
| Small MonoBehaviours with `[RequireComponent]` | God classes that handle input + logic + rendering |
| ScriptableVariable for shared runtime state | Public static fields or global state holders |
| RuntimeSet for tracking active instances | FindObjectsOfType or singleton registries |
| `Awaitable` with `destroyCancellationToken` for async | Coroutines for new async work |
| `[SerializeReference]` for polymorphic serialized data | Deep inheritance hierarchies |
| Composition: many small components on one GameObject | One MonoBehaviour doing everything |
| Interfaces only at package boundaries when SO Events can't solve it | Interfaces inside a single package |
| Tests using ScriptableObject.CreateInstance in Edit Mode | Skipping tests because "it's just a SO" |

## Decision Flowchart: Where Does This Code Belong?

```dot
digraph where {
  rankdir=TB;
  node [shape=diamond];

  start [label="New code" shape=box];
  q1 [label="Is it data\nor config?"];
  q2 [label="Does it need\nMonoBehaviour\nlifecycle?"];
  q3 [label="Is it Editor-\nonly tooling?"];
  q4 [label="Is it pure\nlogic with no\nUnity deps?"];

  so [label="ScriptableObject" shape=box style=filled fillcolor="#d4edda"];
  mono [label="MonoBehaviour" shape=box style=filled fillcolor="#cce5ff"];
  editor [label="Editor/ folder\nEditorWindow or\nPropertyDrawer" shape=box style=filled fillcolor="#fff3cd"];
  util [label="Plain C# class\nin Runtime/" shape=box style=filled fillcolor="#f8d7da"];

  start -> q1;
  q1 -> so [label="yes"];
  q1 -> q2 [label="no"];
  q2 -> mono [label="yes"];
  q2 -> q3 [label="no"];
  q3 -> editor [label="yes"];
  q3 -> q4 [label="no"];
  q4 -> util [label="yes"];
  q4 -> mono [label="no"];
}
```

## Decision Flowchart: How Should These Systems Talk?

```dot
digraph talk {
  rankdir=TB;
  node [shape=diamond];

  start [label="System A needs\nto notify System B" shape=box];
  q1 [label="Same package?"];
  q2 [label="Same GameObject\nor parent-child?"];
  q3 [label="Need to share\nruntime state?"];
  q4 [label="Both packages\nalways installed\ntogether?"];

  csharp [label="C# event or\ndelegate on the\ncomponent" shape=box style=filled fillcolor="#cce5ff"];
  sovar [label="ScriptableVariable\n(shared SO asset)" shape=box style=filled fillcolor="#d4edda"];
  sochan [label="SO Event Channel\n(fire and forget)" shape=box style=filled fillcolor="#d4edda"];
  vdef [label="Version Defines +\nSO Event Channel\nor bridge package" shape=box style=filled fillcolor="#fff3cd"];

  start -> q1;
  q1 -> q2 [label="yes"];
  q1 -> q3 [label="no"];
  q2 -> csharp [label="yes"];
  q2 -> sochan [label="no"];
  q3 -> sovar [label="yes\n(read state)"];
  q3 -> q4 [label="no\n(notify)"];
  q4 -> sochan [label="yes"];
  q4 -> vdef [label="no"];
}
```

## New Package Checklist

Before shipping any package, verify:

- [ ] `package.json` with correct `name`, `version`, `unity` (6000.0+), `displayName`
- [ ] `Runtime/` with `{Company}.{Package}.asmdef` — zero external references
- [ ] `Editor/` with Editor-only asmdef (if any editor code exists)
- [ ] `Tests/Editor/` with test asmdef using `overrideReferences` and `testAssemblies`
- [ ] `Samples~/` with at least one importable sample
- [ ] `CHANGELOG.md` following SemVer
- [ ] All SOs have `[CreateAssetMenu]` with organized menu paths
- [ ] All MonoBehaviours use `[RequireComponent]` where applicable
- [ ] SO Event Channels for every output event (no direct subscriber lists)
- [ ] ScriptableVariables for any shared runtime state
- [ ] RuntimeSets for any "all active X" queries
- [ ] Version Defines in asmdef for any optional package awareness
- [ ] No `FindObjectsOfType`, no singletons, no static mutable state

## System Pattern Quick Reference

| System | SO Config | SO Event Channels | ScriptableVariable | RuntimeSet | Bridge Pattern |
|--------|-----------|-------------------|--------------------|------------|----------------|
| Inventory | ItemDefinition, SlotConfig | OnItemAdded, OnItemRemoved | CurrentWeight, SlotCount | ActiveInventories | Version Defines to shop/crafting |
| Combat | WeaponData, AttackConfig | OnHitDealt, OnHitReceived | — | ActiveCombatants | SO Event to health system |
| Health | HealthConfig, ResistanceProfile | OnDamaged, OnHealed, OnDied | CurrentHP | ActiveHealthComponents | Listens to combat's SO Events |
| Dialogue | DialogueTree, NodeData | OnDialogueStarted, OnChoiceMade | CurrentDialogue | — | Version Defines to quest system |
| Save/Load | SaveConfig | OnSaveRequested, OnLoadComplete | — | SaveableEntities | Each system implements ISaveable |
| Quest | QuestDefinition, ObjectiveData | OnQuestStarted, OnObjectiveComplete | ActiveQuest | ActiveQuests | Listens to dialogue/combat events |
| AI | BehaviorTreeData, StateConfig | OnStateChanged | CurrentTarget | ActiveAIAgents | Version Defines to combat/nav |

## Reference Files

Load these as needed — they contain code snippets and detailed patterns:

- **[references/package-structure.md](references/package-structure.md)** — UPM directory layout, asmdef setup, Version Defines, Define Constraints, SemVer rules, distribution
- **[references/so-architecture.md](references/so-architecture.md)** — ScriptableVariable, SO Event Channel, GameEventListener, RuntimeSet code patterns
- **[references/patterns.md](references/patterns.md)** — Component composition, cross-package integration, bridge packages, SerializeReference, Awaitable async, system decomposition
- **[references/testing.md](references/testing.md)** — Test asmdef setup, SO test patterns, event test patterns, Edit vs Play Mode guidance
