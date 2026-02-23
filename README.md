# Unity Systems Skills

A skill that teaches your agent how to architect and build **reusable Unity game systems** as modular UPM packages with ScriptableObject-driven architecture.

## What This Does

When installed, your agent gains deep knowledge of how to:

- Structure gameplay systems as **self-contained UPM packages**
- Use **ScriptableObject-driven architecture** (SO Event Channels, ScriptableVariables, RuntimeSets)
- Wire cross-system communication **without direct dependencies**
- Apply modern Unity 6+ / C# 11 patterns (Awaitable, SerializeReference, etc.)
- Follow strict rules: composition over inheritance, no singletons, no god classes

Covers common systems like Inventory, Combat, Health, Dialogue, Save/Load, Quests, and AI with concrete decomposition patterns and integration strategies.

## Install

### Claude Code Marketplace

Add the marketplace and install the plugin:

```bash
/plugin marketplace add eyenpi/unity-systems-skills
/plugin install unity-reusable-systems@unity-systems-skills
```

### Using `npx skills` (works with Claude Code, Cursor, Cline, and more)

```bash
npx skills add eyenpi/unity-systems-skills
```

## What's Inside

| File | Description |
|------|-------------|
| `SKILL.md` | Core rules, decision flowcharts, system pattern quick-reference, and a new-package checklist |
| `references/package-structure.md` | UPM layout, assembly definitions, versioning, and distribution |
| `references/so-architecture.md` | ScriptableVariable, SO Event Channel, RuntimeSet, and SO Config patterns with code |
| `references/patterns.md` | Component composition, cross-package integration table, bridge packages, async patterns |
| `references/testing.md` | Test assembly setup, SO testing patterns, Edit/Play mode guidance |

## Usage

Once installed, just ask Claude to build a system:

> "Create a reusable inventory system as a UPM package"

> "Add health and damage to my game using ScriptableObject events"

> "Set up cross-system communication between my combat and audio packages"

Or invoke the skill directly:

- **Claude Code marketplace**: `/unity-reusable-systems:unity-reusable-systems`
- **npx skills**: `/unity-reusable-systems`

## License

MIT
