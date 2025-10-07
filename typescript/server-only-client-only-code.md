---
description: >-
  How to strip code from the server or client, making code server or client
  only.
---

# Server-only/Client-only code

When developing a game in Airship, it may be prudent to not have server logic on the client. For example, if you have an anti-cheat system or other code that the client should not know about.&#x20;

{% hint style="danger" %}
Even with server only code, it's recommended _to **not**_ store API keys or any other sensitive data in your components or scripts.

* **Properties**: Properties are serialized regardless of server or client, so are accessible by the client.
* Through a server-only _getter_: They will still be accessible to anyone who has access to your source code.&#x20;
{% endhint %}

## Method 1: Server-only/Client-only methods

Inside `AirshipBehaviour` classes, you can use the `@Server()` and `@Client()` [method decorators](https://www.typescriptlang.org/docs/handbook/decorators.html#method-decorators) to tell the compiler you want the given methods to be server or client only.

```typescript
export default class ServerAndClientOnlyMethods extends AirshipBehaviour {
    @Server()
    public ServerOnlyMethod() {
        print("This will only be on, and callable by the server!")
    }
    
    @Client()
    public ClientOnlyMethod() {
        print("This will only be on, and callable by the client!");
    }
}
```

{% hint style="warning" %}
If you try to use a server-only method on the client, or a client-only method on the server - it will error.

However, there is an exception to this rule: Lifecycle methods such as `Awake`, `Start`, `OnCollisionEnter` etc; as this allows then to make those lifecycles server or client only without throwing an error.
{% endhint %}

### Server/Client only lifecycle methods

Normal methods by default will _error_ if called in the wrong place. However, will lifecycle methods this is not the case as they may be intended to be server or client only, but not error!

```typescript
export default class ServerComponent extends AirshipBehaviour {
    @Server()
    protected Start() {
         print("I am a server component, printing hello from the server!");   
    }
}
```

## Method 2: Conditional directives

There may be cases you want to mark blocks of code server/client only rather than entire functions. This will work even if it's not inside an `AirshipBehaviour`. To do this, we use _directives_. There are two directives:

* `$SERVER` - when in a condition, will check if it's the server
* `$CLIENT` - when in a condition, will check if it's the client

{% hint style="warning" %}
The compiler will also try to implicitly turn `Game.IsServer()` and `Game.IsClient()` calls if it can into these if `airship.stripImplicitContextCalls` is `true` (enabled by default currently) in `tsconfig.json` - this is more for backwards compatibility and it's recommended you use the actual directives above if your intention is to strip the code.
{% endhint %}

```typescript
export default class DirectivesExample extends AirshipBehaviour {
    protected Start() {
       // You can use directives within ternaries!
       const contextGreeting = $SERVER ? "Hi I'm the server" : "Hi I'm the client!";
    
       // this will print different depending on server or client! :D
       print(contextGreeting);
    
       // Server only code block
       if ($SERVER) {
          print("This will only show up in server code, and print on the server!");
       }
       
       // Client only code block
       if ($CLIENT) {
          print("This will only show up in client code, and print on the client!");
       } else {
          // Of course you can also use else for the opposite!
          print("This will show up in server code and print on the server!");
       }
       
       // Dedicated server block (no shared mode)
       if ($SERVER && !$CLIENT) {
          print("This will only be on a dedicated server (no shared mode)");
       }
       
       // Dedicated client block (no shared mode)
       if ($CLIENT && !$SERVER) {
          print("This will only be on a dedicated client (no shared mode)")
       }
    }
}
```

You can mix these with the decorators as well, to do cool things like create server/client lifecycle event methods:

```typescript
export default class MyComponent extends AirshipBehaviour {
    protected Start() {
        // It will call 'StartServer' on the server!
        if ($SERVER) this.StartServer();
        // It will call 'StartClient' on the client!
        if ($CLIENT) this.StartClient();
    }
    
    @Server() // server start method :D
    protected StartServer() {
        print("Hello, server!");
    }
    
    @Client() // client start method :D
    protected StartClient() {
        print("Hello, client!");
    }
}
```
