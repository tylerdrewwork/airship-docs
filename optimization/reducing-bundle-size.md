# Reducing Bundle Size

Any assets used in your game (outside of core) will increase the download size of your game. Below are tips for ensuring your players enter the game quickly.

### Artwork, Audio and Video

The biggest culprits for bundle size are media assets. Textures, Audio Clips, and especially video will increase your game size quickly.&#x20;

Only include assets you really need. Be sure to compress your assets as much as possible

### Compression

Most assets in Unity has optional compression settings. Be sure to intentionally set this on your assets.&#x20;

{% hint style="success" %}
If you want to change the default compression settings on all imported assets you can use [Unity's Preset Manager](https://docs.unity3d.com/6000.1/Documentation/Manual/class-PresetManager.html)
{% endhint %}

* Keep an eye on the file size indicator at the bottom of the inspector
* Compression can be set per platform if you want smaller sizes for mobile

### Textures

<div align="left"><figure><img src="../.gitbook/assets/image (103).png" alt="" width="481"><figcaption></figcaption></figure></div>

* Make sure to set textures size as small as possible.
* Use crunch compression for smaller file sizes.
* Set compressor quality to 100 for the smallest file size.&#x20;
* For UI icons, combine them into sprite sheets when possible.&#x20;

{% hint style="info" %}
Find more info on [Textures Here](https://docs.unity3d.com/Manual/class-TextureImporter.html)
{% endhint %}



### Audio Files

<div align="left"><figure><img src="../.gitbook/assets/image (105).png" alt="" width="563"><figcaption></figcaption></figure></div>

* Lower the quality until you start to hear a noticeable loss
* Use Vorbis compression for the smallest file size but use ADPCM for short reused clips like footsteps

{% hint style="info" %}
Find more info on [Audio Files Here](https://docs.unity3d.com/6000.1/Documentation/Manual/class-AudioClip.html)
{% endhint %}



### 3D Models

<div align="left"><figure><img src="../.gitbook/assets/image (106).png" alt="" width="563"><figcaption></figcaption></figure></div>

* Use FBX 3D files
* If you aren't using animations or materials from this file make sure to disable them in the inspector. By default Unity will create materials for each mesh and import any animation clips it detects.&#x20;
* Enable compression on the model tab if your file is a large size and you don't need exact precision on the vertices.&#x20;

{% hint style="info" %}
Find more info on [Mode Imports Here](https://docs.unity3d.com/6000.1/Documentation/Manual/FBXImporter-Model.html)
{% endhint %}



### Resources Folder

Avoid placing assets in the Resources Folder. Assets in the Resources folder will always be included even if they are not referenced by anything in your game.&#x20;
