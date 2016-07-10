---
title: Registering rooms
type: documentation
order: 2
---

You may define as many rooms as you want in the server.

When creating a room class, you'll need to implement its basic API in order to change its state to deal with your game, such as:

- `requestJoin (options)`
- `onJoin (client, options)`
- `onLeave (client)`
- `onMessage (client, data)`

```javascript
var Room = require('colyseus').Room

class ChatRoom extends Room {
  constructor(options) {
    super(options)
  }

  onJoin (client, options) {
    console.log(client.id, "joined!")
  }

  onLeave (client) {
    console.log(client.id, "left!")
  }

  onMessage (client, data) {
    console.log(client.id, "sent", data)
  }
}

module.exports = ChatRoom
```

Considering you have a `colyseus.Server` instance in your `index.js` file ([as in the starter project example](https://github.com/endel/colyseus-starter)), you'll need to specify a unique identifier for each room handler you register.

```javascript
gameServer.register('chat_room', ChatRoom)
```

Note that you're able to register the same room class multiple tiles, using different unique identifiers. The third parameter of `register` is used to for custom constructor options, allowing you to make the room have a different behaviour, based on these options.

```javascript
gameServer.register('chat_room_with_more_players', ChatRoom, {
  maxClients: 20
})
```

In this example, we're going to limit the maximum clients allowed on rooms called `'chat_room_with_more_players'` to `20`. In order to make this `maxClients` option work as expected, we'll need to implement the `requestJoin` method:

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
