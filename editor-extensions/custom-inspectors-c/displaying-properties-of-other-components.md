# Displaying properties of other components

There may be cases where you want to "embed" another component's properties in your inspector.

{% code fullWidth="false" %}
```csharp
// componentReference here would be any reference to an AirshipBehaviour
var componentProperty = serializedObject.FindAirshipProperty("componentReference");
if (componentProperty.objectReferenceValue is AirshipComponent component) {
    // this is similar to serializedObject, but pointing at a different component
    var componentSerializedObject = new AirshipSerializedObject(component);
    
    // Then you can use componentSerializedObject as you would with serializedObject
    
    // ApplyModifiedProperties *must* be called otherwise the properties will not change.
    componentSerializedObject.ApplyModifiedProperties();
}
```
{% endcode %}

If you want to use the custom inspector (if you have one defined for the component) you can do

{% code fullWidth="false" %}
```csharp
var componentProperty = serializedObject.FindAirshipProperty("componentReference");
if (componentProperty.objectReferenceValue is AirshipComponent component) {
    // garb the custom editor and render it
    var componentEditor = AirshipCustomEditors.GetEditor(component);
    componentEditor.OnInspectorGUI();
    // Apply any modified properties from the above call
    componentEditor.serializedObject.ApplyModifiedProperties();
}

```
{% endcode %}

