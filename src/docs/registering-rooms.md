---
title: Registering rooms
type: documentation
order: 2
---

You may define as many rooms as you want in the server.

When creating a a room class, you'll need to implement its basic API in order to
change its state to deal with your game, such as:

- `requestJoin (options)`
- `onJoin (client, options)`
- `onLeave (client)`
- `onMessage (client, data)`

```javascript
var Room = require('colyseus').Room

class ChatRoom extends Room {
  constructor(options) {
    super(options, 1000 / 20)
  }

  onJoin (client, options) {
    console.log(client.id, "joined!")
  }

  onLeave (client) {
    console.log(client.id, "leaved!")
  }

  onMessage (client, data) {
    console.log(client.id, "sent", data)
  }
}
```

Considering you have a `colyseus.Server` instance in your `index.js` file ([as
in the default project structure](https://github.com/endel/colyseus-starter)),
you'll need to specify an unique identifier for each room you register.

```javascript
gameServer.register('chat_room', ChatRoom)
```

Note that you're able to register the same room class using different
identifiers. You may use the third parameter to send custom options to the room
constructor, making the room have a different behaviour.

```javascript
gameServer.register('chat_room_with_more_players', ChatRoom, {
  maxClients: 20
})
```

In order to make this `maxClients` option works, we'll need to implement it
inside `requestJoin` method:

```javascript
class ChatRoom extends Room {
  // ...
  requestJoin(options) {
    return this.clients.length < this.options.maxClients;
  }
  // ...
}
```

If not implemented, `requestJoin(options)` will always return `true`.
