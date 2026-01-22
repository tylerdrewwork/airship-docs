# Airship on Linux

* Playing Airship on Steam - While we don't have official Linux support yet, Airship works perfectly fine through Steam's _Proton_ compatibility layer.
* Editing your Airship game  - This is mostly functional. Do note that Unity _officially_ supports Ubuntu, so with other distributions your mileage may vary. Your Unity Hub version should be `≥ 3.15.4` as earlier versions have issues with module installation with other distros.

### Client (via Proton)

#### Known issues

* **KDE**: Maximizing windows (this is a Unity Engine related bug with Proton 10 and KDE)&#x20;
  * This can be fixed by setting the compatibility tool to `Proton 9.0`.

### Editor

The best way to install node on Linux would be through [NVM](https://github.com/nvm-sh/nvm) as that is what we support for detecting your node installation right now.

The verified working versions of Linux for Unity Editor + Airship are:

* CachyOS (Arch Linux)

