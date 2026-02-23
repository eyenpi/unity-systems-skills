# ScriptableObject Architecture

## ScriptableVariable\<T\>

Shared runtime state as an asset. Any component can read/write it. Resets on play mode exit.

```csharp
public abstract class ScriptableVariable<T> : ScriptableObject
{
    [SerializeField] private T initialValue;
    [System.NonSerialized] private T runtimeValue;

    public T Value
    {
        get => runtimeValue;
        set => runtimeValue = value;
    }

    private void OnEnable()
    {
        runtimeValue = initialValue;
    }
}
```

Create concrete types for the serializer:

```csharp
[CreateAssetMenu(menuName = "Variables/Float")]
public class FloatVariable : ScriptableVariable<float> { }

[CreateAssetMenu(menuName = "Variables/Int")]
public class IntVariable : ScriptableVariable<int> { }
```

**Reset strategy:** `OnEnable` runs when entering play mode and on domain reload. `[System.NonSerialized]` ensures `runtimeValue` is never saved to the asset — only `initialValue` persists. This prevents play mode edits from leaking into the asset on disk.

## SO Event Channel

Fire-and-forget broadcast. Publishers and subscribers share an asset reference — they never reference each other.

```csharp
[CreateAssetMenu(menuName = "Events/Game Event")]
public class GameEvent : ScriptableObject
{
    private readonly List<Action> listeners = new();

    public void Raise()
    {
        for (int i = listeners.Count - 1; i >= 0; i--)
            listeners[i]?.Invoke();
    }

    public void Subscribe(Action listener) => listeners.Add(listener);
    public void Unsubscribe(Action listener) => listeners.Remove(listener);
}
```

Generic variant for typed payloads:

```csharp
[CreateAssetMenu(menuName = "Events/Float Event")]
public class FloatGameEvent : ScriptableObject
{
    private readonly List<Action<float>> listeners = new();

    public void Raise(float value)
    {
        for (int i = listeners.Count - 1; i >= 0; i--)
            listeners[i]?.Invoke(value);
    }

    public void Subscribe(Action<float> listener) => listeners.Add(listener);
    public void Unsubscribe(Action<float> listener) => listeners.Remove(listener);
}
```

Create one concrete SO per payload type you need (void, float, int, Vector3, DamageInfo, etc.). Generic SO base classes are not directly serializable by Unity — always create concrete leaf types.

## GameEventListener

Bridges SO Event Channels to the scene via UnityEvent. Designers wire responses in the Inspector without code.

```csharp
public class GameEventListener : MonoBehaviour
{
    [SerializeField] private GameEvent gameEvent;
    [SerializeField] private UnityEvent response;

    private void OnEnable() => gameEvent.Subscribe(OnEventRaised);
    private void OnDisable() => gameEvent.Unsubscribe(OnEventRaised);

    private void OnEventRaised() => response.Invoke();
}
```

## RuntimeSet\<T\>

Self-registering collection of active instances. Replaces `FindObjectsOfType`.

```csharp
public abstract class RuntimeSet<T> : ScriptableObject
{
    private readonly List<T> items = new();
    public IReadOnlyList<T> Items => items;

    public void Add(T item) { if (!items.Contains(item)) items.Add(item); }
    public void Remove(T item) => items.Remove(item);
}
```

Components register themselves:

```csharp
public class RuntimeSetMember<T> : MonoBehaviour where T : Component
{
    [SerializeField] private RuntimeSet<T> runtimeSet;

    private void OnEnable() => runtimeSet.Add(this as T);
    private void OnDisable() => runtimeSet.Remove(this as T);
}
```

## SO Config

A ScriptableObject that holds only serialized fields — no runtime state, no methods beyond validation. Used for designer-tunable settings: health values, spawn rates, drop tables, UI layout params. **Every primitive or value-type field you'd put on a MonoBehaviour belongs here instead.**

```csharp
[CreateAssetMenu(menuName = "Combat/Weapon Config")]
public class WeaponConfig : ScriptableObject
{
    [Header("Damage")]
    public float baseDamage = 10f;
    public float critMultiplier = 2f;
    public AnimationCurve falloffCurve;

    [Header("Detection")]
    public LayerMask hitLayers;
    public float attackRange = 1.5f;
}
```

The MonoBehaviour references the SO — never duplicates its fields:

```csharp
[RequireComponent(typeof(Collider2D))]
public class HitDetector : MonoBehaviour
{
    [SerializeField] private WeaponConfig config;  // single SO ref
    [SerializeField] private GameEvent onHitDealt;  // SO Event Channel

    // Uses config.hitLayers, config.attackRange, etc.
}
```

This lets designers create multiple config variants (e.g. "Sword", "Spear", "Fists") as assets and swap them without touching code.
