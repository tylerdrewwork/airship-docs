---
description: Quick toggling of ragdoll physics for characters
---

# Character Ragdoll

If you wish to be able to trigger ragdoll effects on your characters, be sure to make a variant of the prefab AirshipCharacterRagdoll, which has RigidBodies setup for ragdoll effects.

If your character prefab has a ragdoll on it, you can access it in TS by getting a reference to the CharacterRagdoll Airship component on the character rig. This component also has easy functions for adding forces to all of the ragdoll joints at once.

```typescript
const ragdoll = this.gameObject.GetAirshipComponent<CharacterRagdoll>();
if (ragdoll) {
	ragdoll.SetRagdoll(true);
	ragdoll.AddExplosiveForce(
		80, // Force value
		ragdoll.transform.position, // Explosion center point
		3, // Explosion radius
		0.25, // Upwards force modifier
		ForceMode.Impulse,
	);
}
```

### OnDeath Example

A common pattern is to enable a character's ragdoll when the character dies. This is a trivial task using the `Airship.Damage` singleton. Let's create a `RagdollManager` singleton. Remember to attach this component to a GameObject in the scene so that it starts up by itself.

```typescript
import { Airship } from "@Easy/Core/Shared/Airship";
import { DamageInfo } from "@Easy/Core/Shared/Damage/DamageInfo";
import CharacterRagdoll from "@Easy/Core/Shared/Character/Animation/CharacterRagdoll";

export default class RagdollManager extends AirshipSingleton {
	@Client()
	protected Start() {
		// Fetch ragdoll component on death and hand it off to the EnableRagdoll function
		Airship.Damage.onDeath.Connect((damageInfo) => {
			const character = damageInfo.gameObject.GetAirshipComponent<Character>();
			const ragdoll = character?.rig.gameObject.GetAirshipComponent<CharacterRagdoll>();
			if (ragdoll) {
				this.EnableRagdoll(damageInfo, ragdoll);
			}
		});
	}

	private EnableRagdoll(damageInfo: DamageInfo, ragdoll: CharacterRagdoll) {
		ragdoll.SetRagdoll(true);
		
		// Add other effects here as desired, e.g. ragdoll.AddExplosiveForce()
	}
}
```

{% hint style="info" %}
The ragdoll effect needs to run on the client, so we will use the `@Client()` decorator on our Start lifecycle method to ensure this code only runs on the client.
{% endhint %}
