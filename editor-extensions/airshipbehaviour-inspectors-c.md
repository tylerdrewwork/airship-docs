# AirshipBehaviour Inspectors (C#)

{% hint style="danger" %}
<i class="fa-flask-round-potion">:flask-round-potion:</i> This is currently a beta feature being tested in Airship and includes a rewrite of the inspector system. The API is subject to change.

***

To enable this beta feature, go to Project Settings -> Editor Beta Features -> and set **AirshipEditors v2** to `Enabled`.\
\
This feature will be set as the default when it's determined that it is stable.
{% endhint %}

While Airship will generate a default inspector for `AirshipBehaviour` components, you may want to create a custom inspector to:

* Create a more user-friendly interface for complex components
* Organize and group properties
* Conditionally show/hide UI based on user choices.

If you need to just _categorize or add constraints_ to properties, you can refer to [Using Component Decorators](../typescript/airshipbehaviour/adding-inspector-properties.md#organizing-your-properties). <br>

{% hint style="warning" %}
Currently editor scripts can only be written through the C# _AirshipEditor_ API. Support for TypeScript-based editor scripts may come in future.
{% endhint %}

You can declare an `AirshipBehaviour` as per usual -

```typescript
export default class ExampleComponent extends AirshipBehaviour {
    public name = "Bob";
    public age = 20;
    public favouriteColor = Color.blue;
}
```

<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption><p>Default inspector for the "Example Component" object</p></figcaption></figure>

### Creating a custom inspector script

{% hint style="warning" %}
Airship only supports using IMGUI for custom inspectors at this time.
{% endhint %}

Creating a custom inspector for any _Airship_ serialized object is pretty straightforward. You will need to create a class that derives from `AirshipEditor` and add the `AirshipEditor` attribute to it. This will let Airship know what class this custom inspector represents.

{% code title="Assets/Editor/ExampleComponentInspector.cs" %}
```csharp
[CustomAirshipEditor("ExampleComponent")]
public class ExampleComponentEditor : AirshipEditor {
    public override void OnInspectorGUI() {
        EditorGUILayout.LabelField("This is a custom inspector");
    }
}
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption><p>Custom Inspector with label</p></figcaption></figure>

{% hint style="info" %}
You should make sure that your editor scripts are in an _Editor_ folder.
{% endhint %}

`AirshipEditor` is similar to `UnityEngine.Editor` and contains Airship parallels to the unity custom editor API

* It contains a `serializedObject` and `target` property
* You can access airship properties via `serializedObject.FindAirshipProperty("propertyName")` - this will return an `AirshipSerializedValue` which you can use to check or modify the property with; similar to `SerializedProperty`.
* You can render properties using the Airship editor system using `AirshipEditorGUI.PropertyField(airshipProperty)` by default, however you are not limited to it.

### Example 1: Showing/Hiding properties based on user choices

There may be cases where you want to show/hide properties based on what the user has selected

{% code title="Assets/Code/ExampleComponent.ts" %}
```typescript
export default class ExampleComponent extends AirshipBehaviour {
	public name: string;
	public color = Color.blue;

	public hiddenProperties = false;
	public hiddenNumber: number;
	public hiddenBoolean: boolean;
	public hiddenString: string;
}

```
{% endcode %}

{% code title="Assets/Editor/ExampleComponentInspector.cs" %}
```csharp
[CustomAirshipEditor("ExampleComponent")]
public class ExampleComponentEditor : AirshipEditor {
    public override void OnInspectorGUI() {
        // PropertyField here can be used as shortcut to show property fields
        PropertyField("name");
        PropertyField("color"); 
        
        // If the property is a boolean one, it can be used to conditionally check
        if (PropertyField("hiddenProperties")) {
            // Properties can also be queried using 'FindAirshipProperty'
            var secretValue = serializedObject.FindAirshipProperty("hiddenNumber");
            PropertyField(secretValue);
            
            // values of properties can be queried as well!
            if (secretValue.numberValue > 100) {
                EditorGUILayout.LabelField("The number is greater than 100!");
            }
            // You can also show multiple properties in one line
            PropertyFields("hiddenBoolean", "hiddenString");
        }
    }
}

```
{% endcode %}

The resulting inspector behaviour:

<figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption><p>With 'hidden properties' unchecked<br></p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption><p>With 'hidden properties' checked</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption><p>With 'hidden number' > 100</p></figcaption></figure>
