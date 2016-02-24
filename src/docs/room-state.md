---
title: The Room State
type: documentation
order: 4
---

Each room has its own state. A broadcast occurs every time the state changes, notifying all connected clients of all patches occurred in the interval specified.

![Room State Diagram](http://www.gliffy.com/go/publish/image/10069469/L.png)

There's two ways of dealing with the room state: using plain JavaScript objects, or having a state handler class.

## Plain Object State

The simplest way to deal with the room state is using a plain JavaScript object.

```javascript
class ChatRoom extends Room {
  constructor(options) {
    super(options, 1000) // sync state every 1 second

    // use a plain object state, holding the room "messages".
    this.setState({
      messages: []
    })
  }

  onMessage (client, data) {
    this.state.messages.push(data.message)
  }
}
```

## Appling patch state

At the client-side, on this example, a patch of the room state is sent every 1 second, if something changed since last patch.

The patch state broadcasted by the server follows JSON Patch structure ([RFC 6902](http://tools.ietf.org/html/rfc6902)), which looks like this:

```javascript
[
  {
    "op":"add",
    "path":"/messages/1",
    "value":"VyX6zAHse: New message comming!"
  }
]
```

## State Handler Class

The state handler class allows you to have private state variables while
explicitly exposing public ones. Only what is exposed through `toJSON` method
will be sent to all connected clients.

Here's the a simple example of a state handler class:

```javascript
// StateHandler.js

class StateHandler {

  constructor () {
    this.players = {}
    this.mapBuilder = new MapBuilder()
    this.map = this.mapBuilder.build()
  }

  restart () {
    this.map = this.mapBuilder.build()
  }

  addPlayer (client) {
    this.players[ client.id ] = {}
  }

  toJSON () {
    return {
      map: this.map
      players: this.players
    };
  }

}
module.exports = StateHandler
```

On this example the `StateHandler` create a `MapBuilder` instance, which would
be responsible for creating the map data. It expose some methods to be called
from the parent `Room` instance, such as `addPlayer` and `restart`. Note that
the `MapBuilder` instance is not exposed through `toJSON` method, which means
it's only for internal usage and shouldn't be public.

To use this `StateHandler` class in a `Room` instance it would look like this:

```javascript
var Room = require('colyseus').Room
var StateHandler = require('./StateHandler')

class MapRoom extends Room {

  constructor (options) {
    super(options, 1000 / 20) // sync every 50ms

    // initialize StateHandler
    this.setState( new StateHandler() )

    // reset the state every 3 minutes
    setInterval(() => this.state.restart(), 1000 * 60 * 3)
  }

  onJoin (client) {
    // add player instance to room state
    this.state.addPlayer(client)
  }
}

module.exports = MapRoom
```

What makes this approach powerful is the ability to have nested instances that
responds to `toJSON` method. This allows you to create an internal structure
that mutates itself, calling methods from each other back and forth. Let's use
`Player` as an example:

```JavaScript
class Player {
  constructor () {
    this.life = 50
    this.maxLife = 50
    this.damage = 9

    // recovers a life a little every 2 seconds
    setInterval(this.recoverLife.bind(this), 2000)
  }

  recoverLife () {
    if (this.life < this.maxLife) {
      this.life++
    }
  }

  takeDamage (otherPlayer) {
    this.life -= otherPlayer.damage
  }

  toJSON () {
    return {
      life: this.life
    }
  }
}
module.exports = Player
```

Having defined the `Player` class, you'd instantiate it rather than using plain
objects in the `addPlayer` method:

```javascript
// StateHandler.js
// ...
  addPlayer (client) {
    this.players[ client.id ] = new Player()
  }
// ...
```
