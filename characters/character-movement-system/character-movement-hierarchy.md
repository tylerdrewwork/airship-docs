# Character Movement Hierarchy

The default AirshipCharacter prefab has a working setup for the movement system

The "AirshipCharacter" transform has the `CharacterMovement` and `CharacterMovementData` components on it for controlling your movement options. This root transform tracks the position of your character, but never rotates (to preserve an axis aligned collider). Access it in TS by grabbing the `RootTransform` variable.

The "NetworkedGraphicsHolder" is controlled by the movement code and always points towards the direction the character is trying to move towards. You can access it in TS by grabbing the `AirshipTransform` variable

The "AirshipRigHolder"  is the visible direction the character faces which is an interpolated value based on the characters desired direction and the camera angle.&#x20;

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

The "AirshipRig" transform holds the Animator for animating the character, as well as a Rig component that gives you reference to all of the bones of our avatars. You can access it in TS by grabbing the `rig` variable on the character.&#x20;

The children of the "AirshipRig" hold the visual elements of the character as well as the rig bone transforms

<figure><img src="../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>
