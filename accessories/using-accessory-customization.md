---
description: >-
  Accessories can have special customization data that users can modify per
  outfit
hidden: true
---

# Using Accessory Customization

* **Platform Gear** Specifies the type of custom data on the set of accessories
* **Outfit Data** holds the players chosen customization's on the server
* **Custom Acc Setter** components use the data and apply it to the graphics

<div data-full-width="false"><figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure></div>

###

### Customization Options

Currently there are two types of custom data. Colors and Variants

**Colors** - Colors can be used by the setters to modify any visuals needed. The key is the value displayed to the user and the value used to save into the outfit. The color is what color the setter will use. The scheme determines what color swatches to display in the avatar editor.&#x20;

<div align="left"><figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure></div>

**Variants** - Versions of accessories that swap out based on the users chosen index.

<div align="left"><figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure></div>

###

### Custom Options in Avatar Editor

When selecting an accessory to equip the available customization options will display if available.&#x20;

Colors display as a set of swatches and a color picker.

<div align="left"><figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure></div>

Variants display as buttons with 1 selected at a time.

<div align="left"><figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure></div>

###

### Accessory Prefab Setters

Accessory Components have optional references to color and variant setters. These setters are made in C# and will update the graphics whenever new customization data is set on the accessory. More setters can be made as artists determine new accessory needs.&#x20;

#### Colors

Add the component `CustomAccSetter_Color` to your AccessoryComponent prefab

Colors can be set per renderer or by shader using our custom color mask shader.

* Using **Material Color URP** per renderer
  * For each color key, specify a list of MaterialColorURP references ![](<../.gitbook/assets/image (7).png>)
  * The Material Color index specifies what material index the MaterialColorURP will use (**Set to -1 to apply to every material)**
* Using **RGB Color Masks** shader
  * Make sure your materials are using the LitOpaqueRBGColorMask ![](<../.gitbook/assets/image (8).png>)
  * Fill the Color Mesk Renderer list with renderers that use the mask ![](<../.gitbook/assets/image (10).png>)

#### Variants

Add the `CustomAccSetter_Variant` to your AccessoryComponent prefab

The variant setter can swap meshes or materials on renderers

* Add renderers that will change to the Affected Renderers list ![](<../.gitbook/assets/image (11).png>)
*   Add an element per possible variant

    * Specify the new mesh (optional)
    * Specify the new material (optional)&#x20;

    ![](<../.gitbook/assets/image (13).png>)

