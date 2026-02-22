# Patterns

## Component Design Rules

- **Single responsibility**: one component = one job. `HealthComponent` tracks HP. `DamageFlash` handles hit visuals. Never combine them.
- **`[RequireComponent]`**: if a component always needs another, declare it. `[RequireComponent(typeof(Rigidbody))]` on a `Mover`.
- **Events over direct calls**: components expose C# events. Siblings subscribe in `OnEnable`/`OnDisable`. No `GetComponent` chains in `Update`.

## Composition Example

One entity, multiple focused components:

```csharp
// Each does ONE thing. Composed on a single GameObject.
[RequireComponent(typeof(HealthComponent))]
public class Mover : MonoBehaviour
{
    [SerializeField] private FloatVariable moveSpeed;
    public void Move(Vector3 direction) =>
        transform.Translate(direction * moveSpeed.Value * Time.deltaTime);
}

public class WeaponController : MonoBehaviour
{
    [SerializeField] private WeaponData weaponData;  // SO Config
    [SerializeField] private GameEvent onAttack;      // SO Event Channel
    public void Attack() => onAttack.Raise();
}
```

Components communicate through SO Event Channels and ScriptableVariables — not direct references to each other.

## Cross-Package Integration

| Situation | Pattern | Example |
|-----------|---------|---------|
| Package B is always optional | Version Defines in asmdef | Inventory defines `MYSTUDIO_CRAFTING` when crafting package detected |
| Code that needs both packages | Define Constraints assembly | `Inventory.CraftingIntegration.asmdef` with `defineConstraints: ["MYSTUDIO_CRAFTING"]` |
| Loose event-based communication | SO Event Channel | Combat raises `OnHitDealt`, health system listens — no shared types |
| Complex integration logic | Bridge package | `com.mystudio.combat-health-bridge` depends on both, neither depends on it |
| Shared runtime state | ScriptableVariable asset | Both packages reference the same `FloatVariable` asset for player HP |

**Default to SO Event Channels.** Only use a bridge package when integration logic requires concrete types from both packages. Only use interfaces when you need a type contract that SO Events can't express (rare).

## Bridge Package Pattern

A bridge package depends on two+ packages. Neither source package knows the bridge exists.

```json
{
  "name": "MyStudio.Inventory.CraftingBridge",
  "references": [
    "MyStudio.Inventory",
    "MyStudio.Crafting"
  ],
  "defineConstraints": [
    "MYSTUDIO_CRAFTING"
  ]
}
```

The bridge auto-compiles only when both packages are installed. Contains MonoBehaviours that reference concrete types from both.

## SerializeReference + Polymorphism

For extensible behavior without inheritance hierarchies:

```csharp
[Serializable]
public abstract class Effect
{
    public abstract void Apply(GameObject target);
}

[Serializable]
public class DamageEffect : Effect
{
    [SerializeField] private float amount;
    public override void Apply(GameObject target) =>
        target.GetComponent<HealthComponent>()?.TakeDamage(amount);
}

// In a MonoBehaviour or SO:
[SerializeReference] private List<Effect> effects = new();
```

Unity serializes the concrete types. Designers add/remove effects in the Inspector. New effects = new classes, zero existing code changes.

## Async: Awaitable

Unity 6+ replaces coroutines with `Awaitable`. Always pass `destroyCancellationToken` to cancel on object destruction.

```csharp
private async Awaitable RespawnAfterDelay(float seconds)
{
    await Awaitable.WaitForSecondsAsync(seconds, destroyCancellationToken);
    Respawn();
}
```

## System Decomposition

| System | Data (SOs) | Behavior (MonoBehaviours) | Communication |
|--------|-----------|--------------------------|---------------|
| Inventory | ItemDefinition, SlotConfig | InventoryHolder, SlotUI | SO Events: OnItemAdded, OnItemRemoved |
| Combat | WeaponData, AttackConfig | HitDetector, WeaponController | SO Events: OnHitDealt, OnHitReceived |
| Health | HealthConfig | HealthComponent, DamageFlash | SO Events: OnDamaged, OnDied; ScriptableVariable: CurrentHP |
| Dialogue | DialogueTree, NodeData | DialogueRunner, ChoiceUI | SO Events: OnDialogueStarted, OnChoiceMade |
| Save/Load | SaveConfig | SaveableEntity, SaveManager | SO Events: OnSaveRequested, OnLoadComplete; RuntimeSet: SaveableEntities |
