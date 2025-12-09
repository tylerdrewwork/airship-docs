# Airship Scriptable Objects

{% hint style="danger" %}
<i class="fa-flask">:flask:</i>`AirshipScriptableObject` is in beta pending full release, and may not work on some platforms yet

* <i class="fa-xmark">:xmark:</i> Editor
* <i class="fa-xmark">:xmark:</i> Windows and MacOS
* <i class="fa-xmark">:xmark:</i> Android and iOS

Using this feature on currently unsupported platforms may not work as expected.
{% endhint %}

`AirshipScriptableObject` is a class you can derive from to create objects that live independently of GameObjects.  This allows you to save data as an asset to use at runtime.

Like [ScriptableObject](https://docs.unity3d.com/6000.2/Documentation/ScriptReference/ScriptableObject.html) in Unity, they are accessible from scenes and assets within a project.

<pre class="language-typescript"><code class="lang-typescript">// This will show in Create -> ScriptableObjects -> Example Data Object by default
@CreateAssetMenu()
export default class ExampleDataObject extends AirshipScriptableObject {
<strong>    public message = "Hello, world!";
</strong>    
    protected Awake() {
         // This will be fired when the scriptable object is first referenced/created.
         // The properties of this object will be set up on Awake
         print(this.message, "from the example scriptable object awake!");   
    }
    
    protected OnDestroy() {
         // This will be called if:
         //  - A CreateInstance() ScriptableObject is destroyed
         print("This object was destroyed!");
    }
}
</code></pre>

This allows you to save data as an asset to use at runtime. This can be useful for global data shared between objects, avoiding the necessity for duplicate data.

### Referencing a ScriptableObject in your game

As an example, we can use a ScriptableObject to create "templates" for NPCs we want to spawn

```typescript
@CreateAssetMenu()
export default class NPCTemplate extends AirshipScriptableObject {
	@Header("Metadata")
	public name = "Example NPC";
	@Min(1)
	public maxHealth = 100;

	@Header("Appearance")
	public customPrefab?: GameObject;
}
```

Then we can create it using the menu item:

<figure><img src="../.gitbook/assets/image (126).png" alt="" width="536"><figcaption><p>Since NPCTemplate has <code>CreateAssetMenu</code> on it, it will show up here by default.</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (125).png" alt=""><figcaption><p>The resulting ScriptableObject - we can now set a name, the health and even a custom prefab if need be.</p></figcaption></figure>

Then to reference the ScriptableObject

```typescript
import { Airship } from "@Easy/Core/Shared/Airship";
import NPCTemplate from "Code/ScriptableObjects/NPCTemplate";

export default class NPCCharacterSpawner extends AirshipBehaviour {
	// Just like referencing other AirshipBehaviours
	// We can also reference AirshipScriptableObject classes
	public template: NPCTemplate;

	@Server()
	protected Start(): void {
		if (this.template === undefined) return;

		const character = Airship.Characters.SpawnNonPlayerCharacter(this.transform.position, {
			customCharacterTemplate: this.template.customPrefab,
		});

		character.SetHealth(this.template.maxHealth);
		character.SetDisplayName(this.template.name);
	}
}
```

<figure><img src="../.gitbook/assets/image (127).png" alt=""><figcaption><p>You will see we now have a reference field for an NPC template</p></figcaption></figure>

Then click on the <kbd>None (NPC Template)</kbd> <i class="fa-circle-o">:circle-o:</i> button - and it will give you a prompt to select a scriptable object of that type, which we can then select -

<figure><img src="../.gitbook/assets/image (129).png" alt=""><figcaption><p>The template is now provided</p></figcaption></figure>

Then when you run the game - it spawns the NPC with the name and health!

<figure><img src="../.gitbook/assets/image (130).png" alt="" width="310"><figcaption><p>The spawner in action</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (131).png" alt="" width="375"><figcaption><p>Example of using the same scriptable object with multiple spawners</p></figcaption></figure>
