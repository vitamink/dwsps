# ps *(publish/subscribe)*

**ps** is a distributed nodejs pub/sub system. It uses websockets to transmit messages between client and server, and between peered servers.

- [Topics](#topics)
- [Messages](#messages)
- [Acknowledgements](#acknowledgements)
- [Distribution](#distribution)
- [Usage](#usage)
  - [Basic server example](#basic-server-example)
  - [Basic client example](#basic-client-example)
- [License](#license)

## Topics

**ps** uses a heirarchical topic system for client subscriptions, each level divided by a full-stop `.`. For example, a client could subscribe to the topic `news.uk`. They would then recieve messages published to `news.uk`, `news.uk.london`, `news.uk.birmingham` etc. but not from `news.fr` or `news.de`.

Note: if a client is subscribed to a parent topic and a sub-topic of the parent, unsubscribing from the parent topic will *not* also unsubscribe the client from the sub-topic. Each subscription must be unsubscribed from explicitly.

## Messages

Messages are JSON format and look like the following:
```
{
  "type": "publish",
  "timestamp": "2020-01-21T17:03:13.625Z",
  "topic": "news.uk",
  "context": "news",
  "message": "Hello news.uk channel!"
}
```
* `topic` lets a client know where the message was published to
* `context` lets a client know *why* they are receiving a certain message - in this instance the client is subscribed to `news`, where they received it, but not `news.uk`, where it was sent.
* `message` can be a string or an object

## Acknowledgements

When a client performs an action, the server will send an acknowledgment in reply if it received the message and executed the action correctly. These can be listened for with the client `ack` event e.g.:

```js
client.on('ack', message => {
  console.log(message)
}
```

Ack messages look like this:
```
{
  "type": "ack",
  "action": "subscribe",
  "timestamp": "2020-01-22T15:44:04.674Z",
  "topic": "news"
}
```

## Distribution

Multiple **ps** servers can be peered with one another to create a distributed network. Once servers are peered, then a message published to one server will also be published to all servers, and delivered to their own subscribed clients respectively. Subscribed clients are not replicated between servers, instead held in memory on a per server basis.

Messages that were forwarded from another server will also contain a `fromPeerServer: true` flag.

See [example.js](./example.js) for an example of peering servers.

## Usage

### Basic server example

```js
const PSServer = require('ps/server')

const server = new PSServer({ port: 8000 })

server.on('subscribe', (topic, client) => {
  console.log(`${client} subscribed to ${topic}`)
})
```

### Basic client example

```js
const PSClient = require('ps/client')

const client = new PSClient('ws://localhost:8000')

// Must wait for client to establish connection before performing actions
client.on('open', () => {
  client.subscribe('news')
  client.publish('news', 'Hello news channel!')
  client.publish('news.uk', 'Hello news.uk channel!')
})

// Log messages received by the client
client.on('message', () => {
  console.log(message)
})

// => {"type":"publish","timestamp":"2020-01-21T17:03:13.625Z","topic":"news","message":"Hello news channel!","context":"news"}
// => {"type":"publish","timestamp":"2020-01-21T17:03:13.625Z","topic":"news.uk","message":"Hello news.uk channel!","context":"news"}
```

## License

MIT. See [LICENSE](./LICENSE).
