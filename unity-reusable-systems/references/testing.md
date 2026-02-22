# Testing

## Test Assembly Definition

```json
{
  "name": "MyStudio.Inventory.Tests",
  "references": [
    "MyStudio.Inventory",
    "UnityEngine.TestRunner",
    "UnityEditor.TestRunner"
  ],
  "optionalUnityReferences": [],
  "includePlatforms": ["Editor"],
  "overrideReferences": true,
  "precompiledReferences": ["nunit.framework.dll"],
  "testAssemblies": true
}
```

Place in `Tests/Editor/`. `overrideReferences: true` + `precompiledReferences` enables NUnit. `includePlatforms: ["Editor"]` = Edit Mode tests.

## Testing ScriptableObjects

Create instances in code, test, destroy. Never depend on asset files.

```csharp
[TestFixture]
public class FloatVariableTests
{
    private FloatVariable variable;

    [SetUp]
    public void SetUp()
    {
        variable = ScriptableObject.CreateInstance<FloatVariable>();
    }

    [TearDown]
    public void TearDown()
    {
        Object.DestroyImmediate(variable);
    }

    [Test]
    public void Value_AfterSet_ReturnsNewValue()
    {
        variable.Value = 42f;
        Assert.AreEqual(42f, variable.Value);
    }
}
```

**Always `DestroyImmediate` in TearDown.** `CreateInstance` allocates native memory. Leaks accumulate across test runs and pollute the Editor.

## Testing SO Event Channels

```csharp
[TestFixture]
public class GameEventTests
{
    private GameEvent gameEvent;

    [SetUp]
    public void SetUp()
    {
        gameEvent = ScriptableObject.CreateInstance<GameEvent>();
    }

    [TearDown]
    public void TearDown()
    {
        Object.DestroyImmediate(gameEvent);
    }

    [Test]
    public void Raise_WithSubscriber_InvokesCallback()
    {
        bool wasCalled = false;
        gameEvent.Subscribe(() => wasCalled = true);

        gameEvent.Raise();

        Assert.IsTrue(wasCalled);
    }
}
```

## Edit Mode vs Play Mode

**Edit Mode** unless you need MonoBehaviour lifecycle (`Awake`, `Start`, `Update`) or physics (`OnCollisionEnter`, raycasts). Edit Mode tests are faster and more reliable.

Play Mode tests require `[UnityTest]` returning `IEnumerator`, a separate test asmdef without `includePlatforms`, and `yield return null` to advance frames.

## Mocking

Use NSubstitute for interface mocking at package boundaries. Add `nsubstitute.dll` to `precompiledReferences` in the test asmdef. Only mock interfaces you own — never mock Unity types.
