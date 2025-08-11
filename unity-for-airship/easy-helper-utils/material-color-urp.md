---
description: Set the color of a material
---

# Material Color URP

Place this component on a renderer's game object to dynamically assign the color value of its materials. It works for multiple materials on one mesh and can also be changed at runtime.&#x20;

{% hint style="info" %}
This script sets the materials \_MainColor variable via a MaterialPropertyBlock. Use this name in your custom shader to have it support MaterialColorURP.&#x20;
{% endhint %}

<figure><img src="../../.gitbook/assets/MaterialColorURP.gif" alt=""><figcaption></figcaption></figure>
