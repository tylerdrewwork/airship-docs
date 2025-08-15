# Game Loading Screens

You can customize what users see while your game is downloading and loading. These also show up when users transport to different servers within your game.&#x20;

The image marked default will be the image shown unless you specify a different image via TS Code.

<figure><img src="../../.gitbook/assets/image (109).png" alt=""><figcaption></figcaption></figure>

Use the Image ID to set an image during server transports:

```typescript
const config: AirshipGameTransferConfig = {
    loadingScreenImageId: "123123123123",
}
Platform.Server.Transfer.TransferToServer(player, serverId, config);
```

For more information on server transfers see the [Platform Server Transfers](../../platform-services/server-transfers.md) page
