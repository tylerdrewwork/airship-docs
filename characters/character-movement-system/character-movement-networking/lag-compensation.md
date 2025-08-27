---
description: Performing lag compensated checks using Airship's Character Networking.
---

# Lag Compensation

{% hint style="warning" %}
This is an advanced topic. Most games do not need lag compensation, basic checks can be enough. Lag Compensation is best used in places where your game has strong competitive integrity requirements.
{% endhint %}

## Background

If you're not familiar with the concept of lag compensation, Valve's [Source Multiplayer Networking](https://developer.valvesoftware.com/wiki/Source_Multiplayer_Networking#Lag_compensation) wiki is a good place to start. Valve's wiki talks about lag compensation in terms of server authoritative networking, but Airship's Lag Compensation works with both server authoritative and client authoritative models.

## Example

{% code fullWidth="true" %}
```typescript
// You submit lag compensation checks for processing by calling the LagCompensationCheck function on a Player object. This can only
// be called on the server. It's important that you do not yield in the passed functions.
this.character.player?.LagCompensationCheck(
	// The first parameter is the check function. The check function will run after the position of each player has been
	// updated to reflect the position the client saw.
	// This check function should NOT modify the world, it should only read the positions of player and perform raycast checks.
	// If you make changes to the position or velocity of players, it will be overwritten.
	() => {
		// Do a raycast hit check.
		const [hit, position, normal, collider] = Physics.Raycast(
			this.character.movement
				.GetPosition()
				.add(new Vector3(0, 1.5, 0)),
			this.character.movement.GetLookVector(),
			WEAPON_RAYCAST_DISTANCE,
			this.hitLayers,
		);
		// Pass the results to the processing function
		return { hit, position, normal, collider };
	},
	// The second parameter is the processing function. You can process the result of the check here. This function runs
	// with the latest state of the world, so it is safe to modify positions, apply forces, etc.
	({ hit, position, normal, collider }) => {
		// If there was no hit, return
		if (!hit) return;
		// Get the character we hit.
		const hitChar = collider.gameObject.GetAirshipComponentInParent<Character>();
		// If there was no character, return.
		if (!hitChar) return;
		// Inflict damage on the character we hit.
		hitChar.InflictDamage(10, this.character.gameObject);
		hitChar.movement.AddImpulse(new Vector3(0, 20, 0));
	},
);
```
{% endcode %}

## Airship Networked Object

The AirshipNetworkedObject component can be placed on any game object with a Rigidbody to ensure that it is also moved to the correct position during Lag Compensation requests. Any objects that don't have this component will be untouched.

