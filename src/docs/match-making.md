---
title: Match Making
type: documentation
order: 3
---

Colyseus right now provides the simplest match-making system you'd expect.

![Match-making Diagram](http://www.gliffy.com/go/publish/image/10069321/L.png)

As you've seen in [registering rooms](registering-rooms) section, you can
register as many room handlers as you want, possibly having custom options when
initializing them.

The server will always try to instantiate a new room in case the client couldn't
join in any of the existing rooms.

Each room you register on the server may implement the `(bool) requestJoin(options)`
method, which will receive the join request options from the client. This method
may accept or not the new client is joining in, by returning a boolean value.

**Example**: Allowing only 10 clients to connect on ChatRoom

```javascript
class ChatRoom extends Room {
  requestJoin(options) {
    return this.clients.length < 10;
  }
}
```

