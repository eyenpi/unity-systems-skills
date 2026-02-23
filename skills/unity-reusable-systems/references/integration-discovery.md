# Integration Discovery

## Integration Doc Location

Every package's integration surface is documented at:

```
docs/packages/<package-name>.md
```

Example: `docs/packages/com.mystudio.health.md`

This is a **project-level** file, not inside the package itself. All integration docs live together so Claude can read them all at once.

## Full Integration Doc Template

```markdown
# com.mystudio.health — Integration Surface

## Event Channels

| Event | Payload Type | Raised When | Suggested Listeners |
|-------|-------------|-------------|---------------------|
| `OnDeath` | `void` | Health reaches zero | Loot spawner, respawn system, score tracker |
| `OnDamaged` | `DamageInfo` | Any damage applied | VFX, sound, UI health bar, combat log |
| `OnHealed` | `float` | Health restored | UI health bar, achievement tracker |

## ScriptableVariables

| Variable | Type | Purpose |
|----------|------|---------|
| `CurrentHealth` | `FloatVariable` | Current health value, readable by any system |
| `MaxHealth` | `FloatVariable` | Maximum health, used for UI percentage display |

## RuntimeSets

| Set | Item Type | Purpose |
|-----|-----------|---------|
| `AliveEnemiesSet` | `HealthComponent` | All currently alive enemies, useful for AI targeting |

## Interfaces (Package Boundary)

| Interface | Purpose | When to Implement |
|-----------|---------|-------------------|
| `IDamageable` | Apply damage to any object | When your package has objects that can take damage but don't use HealthComponent |

## Assembly & Version Define

- **Assembly:** `MyStudio.Health`
- **Package ID:** `com.mystudio.health`
- **Version Define symbol:** `MYSTUDIO_HEALTH`

## Integration Examples

- **Loot system** → Listen to `OnDeath`, spawn drops at dead entity position
- **UI** → Read `CurrentHealth` / `MaxHealth` variables, listen to `OnDamaged` and `OnHealed` for bar animations
- **Combat** → Raise `OnDamaged` through event channel after hit detection, or call `IDamageable` on target
- **Save system** → Read `CurrentHealth` on save, write on load
- **Achievement system** → Listen to `OnDeath` to count kills, `OnDamaged` for "survive X damage" checks
```

## Discovery Workflow Example

When creating a new **Loot** package, Claude reads all existing docs:

**1. Read `docs/packages/com.mystudio.health.md`**

Found relevant:
- `OnDeath` event (void) — raised when health reaches zero
- `AliveEnemiesSet` RuntimeSet — tracks living enemies

**2. Read `docs/packages/com.mystudio.inventory.md`**

Found relevant:
- `ItemDefinition` SO Config — defines items
- `OnItemAdded` event — raised when item added to inventory

**3. Produce Integration Plan for Loot package:**

```markdown
### Integration Plan

#### Listen to (via Version Defines)
- `com.mystudio.health` → `OnDeath`: spawn loot table drops at entity position
  - Version Define: `MYSTUDIO_HEALTH` in Loot's asmdef

#### Publish (new events)
- `OnLootDropped` (LootDropInfo): raised when loot spawns in world
- `OnLootCollected` (ItemDefinition): raised when player picks up loot

#### Read/Write (existing ScriptableVariables)
- None directly — loot is event-driven

#### Expose (new ScriptableVariables)
- `LootDropRate` (FloatVariable): global loot drop rate multiplier

#### Implement (existing interfaces)
- None

#### Bridge needed?
- No — loose event coupling via Version Defines is sufficient

#### Suggested changes to existing packages
- [ ] `com.mystudio.health` — add `versionDefines` entry for `com.mystudio.loot` with symbol `MYSTUDIO_LOOT` (optional: health could raise OnDeath with position data if loot needs it)
- [ ] `com.mystudio.inventory` — add `versionDefines` entry for `com.mystudio.loot` with symbol `MYSTUDIO_LOOT` (optional: inventory could listen to `OnLootCollected` to auto-add items)
```

## Bridge Package Decision

Use a Bridge Package only when:

1. Integration logic needs **concrete types from both packages** (not just events)
2. A MonoBehaviour must **reference components from two packages** on the same GameObject
3. The integration involves **complex orchestration** that doesn't fit in a simple event listener

Example where a bridge IS needed:

```
com.mystudio.combat-inventory-bridge/
```

A `LootableEnemy` component needs `HealthComponent` (from Health) AND `InventoryHolder` (from Inventory) to transfer items on death. Neither package should depend on the other, so the bridge handles it.

Example where a bridge is NOT needed:

Loot system listens to `OnDeath` and spawns items. No concrete type from Health is needed — just the event. Use Version Defines + SO Event Channel.

## Section Rules

When generating a package integration doc:

- **Always include:** Event Channels, Assembly & Version Define, Integration Examples
- **Include if applicable:** ScriptableVariables, RuntimeSets, Interfaces
- **Omit if empty:** Don't add sections with "None" — just leave them out
- **Suggested Listeners column:** List 3-5 realistic systems that would subscribe, to help future package authors spot integration opportunities
- **Integration Examples:** Give concrete, actionable descriptions — not vague "could be used by other systems"
