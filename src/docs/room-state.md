---
title: The Room State
type: documentation
order: 4
---

Each room has its own state. The state is broadcasted to all connected clients
if the state changed since last broadcast.

![Room State Diagram](http://www.gliffy.com/go/publish/image/10069469/L.png)

There's two ways of dealing with the room state: using plain JavaScript objects,
or having a state handler class.

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

At the client-side, on this example, a patch of the room state is sent every 1
second, if something changed since last patch.

The patch state broadcasted by the server follows JSON Patch structure ([RFC
6902](http://tools.ietf.org/html/rfc6902)), which looks like this:

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

> TODO
