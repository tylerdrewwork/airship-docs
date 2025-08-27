---
description: >-
  How to utilize the predicted command manager to easily create server
  authoritative commands with client prediction.
---

# Predicted Command Manager

## Basic Concepts

The predicted command system allow you to create a set of logic and data that will be predicted on the client when using Server Authoritative character networking. For example, if you want to create an ability that launches the player in the air when activated, you would need to ensure that the server and client agree on when the ability was used, as well as ensure that both the client and server use the same logic to apply the force.

### Commands

Commands are a set of logic that will run as part of the character movement processing. To make a new command, you extend the `PredictedCommand` class. Commands should implement the `GetCommand()`, `OnTick()`, `OnCaptureSnapshot()`, `ResetToSnapshot()`, and `CompareSnapshots()` functions. Do not yield in any of the predicted command functions.

#### Predicted Command Manager

The `PredictedCommandManager` is a singleton running on both the client and server. The manager handles starting new commands and ensuring the proper execution of running commands. The `PredictedCommandManager` can get information about which commands are running for a character, validate which commands should run, and provides command related signals such as when a command stops running.

## Example

The following example implementation is an ability which impulses your character upwards after charging for a short amount of time. While the command is charging, the character will move slowly.

### Command

{% code title="TestPredictedCommand.ts" fullWidth="true" %}
```typescript
import { Keyboard } from "../../UserInput";
import PredictedCustomCommand from "./PredictedCustomCommand";

// The input data for this command. This is added to every character input command.
// We use arrays for these since it's less data to send over the network
type InputCommand = [charging: boolean, completed: boolean];

// The state data associated with this command. We network the progress so that if the client and
// server disagree on how much progress has been made, the client will reconcile to use the server
// progress.
type StateData = [progress: number];

// We define our predicted command by extending PredictedCustomCommand and passing in our data types.
// We will register this predicted command with the PredictedCommandManager later. Keeping mind, this is
// NOT an AirshipBehavior. You don't need to add it to a game object.
export class TestPredictedCommand extends PredictedCustomCommand<InputCommand, StateData> {
	// state
	private progress = 0; // How far along we are in the charge process.
	private completed = false;

	// constants
	private CHARGE_TIME_SEC = 1; // How long we want to charge for. 

        // This function is called every time a new instance of this command is created. On replays, this command
        // might be destroyed and recreated many time. Keep that in mind since any state you include (like progress
        // above) will be reset. ResetToSnapshot is always called after Create(), so you should load any state you
        // need from the snapshot there.
	Create(): void {
		// Slow the character movement speed down while this command is running.
		this.character.movement.movementSettings.speed *= 0.1;
	}

	Destroy(): void {
		// Reset the character movement speed back to normal when it ends.
		this.character.movement.movementSettings.speed /= 0.1;
	}

	// Called when the client is generating a new command. This is where you collect any data you
	// need to process in OnTick(). Remember that this is being generated on the client, so you should
	// pass things like input here.
	GetCommand(): false | InputCommand {
		// If the player is holding down the ability button, we continue to charge the ability.
		if (Airship.Input.IsDown("Ability")) {
			return [true]; // Send true since the ability is being held down.
		}
		return false; // If the ability is not being held down, we just exit the command commpletely by returning false.
	}

	// Runs on both the client and the server either before (default) or after character movement is processed depending
	// on the configuration provided when registering the command with the PredictedCommandManager. We can access
	// the data we generated in GetCommand() and perform our logic based on that data.
	override OnTick(input: Readonly<InputCommand> | undefined, replay: boolean, fullInput: CharacterInputData) {
		if (!input) return false;

		this.progress++;

		if (this.progress >= this.CHARGE_TIME_SEC / Time.fixedDeltaTime) {
			this.character.movement.AddImpulse(Vector3.up.normalized.mul(20));
			this.completed = true;
			return false;
		}
	}

	// This runs after OnTick() and after the physics simulation completes. Here we return the progress data since that's
	// what we want to make sure stays in sync between the client and server.
	OnCaptureSnapshot(): StateData {
		return [this.progress, this.completed];
	}

	// This function gets called whenever there's a mismatch between the client and server. It may also get called
	// if the client replays or if the server is processing lag compensation. The purpose of this function is to
	// reset the state of this command to match whatever snapshot is provided to it.
	ResetToSnapshot(state: Readonly<StateData>): void {
		this.progress = state[0];
		this.completed = state[1];
	}

	// Called whenever the client needs to compare it's local predicted result to the actual result provided by the server.
	// We want to ensure that the state data we have locally actually matches the state data returned by the server.
	CompareSnapshots(a: Readonly<StateData>, b: Readonly<StateData>): boolean {
		if (a[0] !== b[0]) return false; // If it doesn't match, then we return false
		if (a[1] !== b[1]) return false;
		return true; // If it matches we are good!
	}
}

```
{% endcode %}

### Registering the Command

To register a command, you need to call `PredictedCommandManager.Get().RegisterCommands()`.  You can call `RegisterCommands()` multiple times if needed. The command needs to be registered on both the client and the server for it to work. Here's a manager singleton that registers the command, and sets up the other configuration needed to actually use our test command.

{% code title="TestCommandManager.ts" fullWidth="true" %}
```typescript
import { Airship } from "@Easy/Core/Shared/Airship";
import { Game } from "@Easy/Core/Shared/Game";
import { Binding } from "@Easy/Core/Shared/Input/Binding";
import PredictedCommandManager, {
	CommandInstanceIdentifier,
} from "@Easy/Core/Shared/Network/PredictedCommands/PredictedCommandManager";
import { TestPredictedCommand } from "@Easy/Core/Shared/Network/PredictedCommands/Test";

export default class TestCommandManager extends AirshipSingleton {
	// Constants
	private CMD_IDENTIFIER = "cmd"; // Identifier for the command. Can be anything, just needs to be unique
	private CMD_COOLDOWN_SEC = 3; // Cooldown for the ability
	private CMD_ACTION_NAME = "Ability"; // The ability action name (This is for user input)

	// Cooldown tracking
	private canUseAt = new Map<number, number>(); // Map of when a character can use their ability again. We use a map because the server
						      // has to track all of the players cooldowns.

	// Client only local command tracking
	private localCommand: CommandInstanceIdentifier | undefined; // Used only for tracking the running command on the local client.

	override Start(): void {
		// Registers the command with the predicted command manager. This allows us to run the command using the provided
		// identifier.
		PredictedCommandManager.Get().RegisterCommands({
			[this.CMD_IDENTIFIER]: {
				handler: TestPredictedCommand,
			},
		});

		// Validate that the command should be able to run. This runs on both the client and server.
		PredictedCommandManager.Get().onValidateCommand.Connect((event) => {
			// This even fires for all commands that start, so make sure we only run these checks
			// on our specific command identifier.
			if (event.commandId !== this.CMD_IDENTIFIER) return;
			// Get the next time the player should be able to run this command.
			const nextUseTime = this.canUseAt.get(event.character.id) ?? 0;
			// If the players ability is on cooldown, then we set the event to cancelled. This stops the command
			// from running.
			if (nextUseTime > Time.time) {
				event.SetCancelled(true);
				return;
			}
		});

		// This event fires when the command ends authoritatively, meaning when this event fires, the server has confirmed
		// the that command ended, and we shouldn't see it be ticked again (although it could still run during resimulations!).
		PredictedCommandManager.Get().onCommandEnded.Connect((command, result) => {
			// The finalState is the last snapshot that was captured on the tick the command ended. We use this to decide
			// if the ability was actually used. Remember that our ability has a charge up time, and we don't want to
			// reset the cooldown unless they actually got launched in the air.
			const finalState = result as [progress: number, completed: boolean];
			if (finalState && finalState[1] === true) { // Check if the "completed" state was true
				// Set the next time the player can use this ability in our cooldown tracking map.
				this.canUseAt.set(command.characterId, Time.time + this.CMD_COOLDOWN_SEC);
			}
		});

		// Observe all the characters in the game.
		Airship.Characters.ObserveCharacters((character) => {
			// The returned function is called when the character despawns.
			return () => {
				// Remove any cooldowns for characters which are despawned. This avoids memory leaks!
				this.canUseAt.delete(character.id);
			};
		});
		
		if (Game.IsClient()) {
			// Create an action for our input.
			Airship.Input.CreateAction(this.CMD_ACTION_NAME, Binding.Key(Key.Q));
			// Listen to when the action button is pressed.
			Airship.Input.OnDown(this.CMD_ACTION_NAME).Connect(() => {
				// If we are running the ability, then we don't want to start another one.
				if (
					this.localCommand &&
					// The last two parameters are checking if the command is queued, meaning it will run next tick,
					// and checking if the command used to be running, but we don't know if it's officially ended yet.
					// It's important to include those parameters in this case because we can't run the ability twice,
					// and we wouldn't want to start running the ability again before we know if it's ended.
					PredictedCommandManager.Get().IsCommandInstanceActive(this.localCommand, true, true)
				) {
					// Ignore the input if the command is already running.
					return;
				}
				// If the command isn't running, we run the command!
				this.localCommand = PredictedCommandManager.Get().RunCommand(this.CMD_IDENTIFIER);
			});
		}
	}
}

```
{% endcode %}
