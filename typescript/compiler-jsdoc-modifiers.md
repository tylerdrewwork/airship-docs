# Compiler JSDoc Modifiers

Since _TypeScript_ does not support decorators on anything but classes, class fields and class methods, you can use the following JSDoc _modifiers_ to apply some customization:

### InspectorName

To change the name of an enum member, akin to unity's [InspectorNameAttribute](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/InspectorNameAttribute.html):

```typescript
export enum ModelImporterIndexFormat {
    Auto,
    /** @InspectorName 16 bits **/
    UInt16,
    /** @InspectorName 32 bits **/
    UInt32,
}
```

{% hint style="warning" %}
Do note _currently_ with the editor API, this will also affect the name of the enum values.
{% endhint %}

### Flags

To mark an enum as a flag enum, you can use `@flags`

```typescript
/** @flags */
export enum ExampleFlags {
    A = 1,
    B = 2,
    C = 4,
    D = 8,
    E = 16,
    // ...
}
```

Do note that if it doesn't follow powers of 2, it will fall back to an integer enum.
