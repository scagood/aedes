# Aedes
[![Build Status](https://travis-ci.org/mcollina/aedes.svg?branch=master)](https://travis-ci.org/mcollina/aedes)
[![Dependencies Status](https://david-dm.org/mcollina/aedes/status.svg)](https://david-dm.org/mcollina/aedes)
[![devDependencies Status](https://david-dm.org/mcollina/aedes/dev-status.svg)](https://david-dm.org/mcollina/aedes?type=dev)
<br/>
[![Known Vulnerabilities](https://snyk.io/test/github/mcollina/aedes/badge.svg)](https://snyk.io/test/github/mcollina/aedes)
[![Coverage Status](https://coveralls.io/repos/mcollina/aedes/badge.svg?branch=master&service=github)](https://coveralls.io/github/mcollina/aedes?branch=master)
[![NPM version](https://img.shields.io/npm/v/aedes.svg?style=flat)](https://www.npmjs.com/package/aedes)
[![NPM downloads](https://img.shields.io/npm/dm/aedes.svg?style=flat)](https://www.npmjs.com/package/aedes)

[![js-standard-style](https://cdn.rawgit.com/feross/standard/master/badge.svg)](https://github.com/feross/standard)

Barebone MQTT server that can run on any stream server.

| QoS 0 | QoS 1 | QoS 2 | auth | [bridge][bridge_protocol] | $SYS topics | SSL | [dynamic topics][dynamic_topics] | cluster | websockets | plugin system | MQTT 5 |
| ----- | ----- | ----- | ---- | ------------------------- | ----------- | --- | -------------------------------- | ------- | ---------- | ------------- | ------ |
| ✔     | ✔     | ✔     | ✔    | ?                         | ✔           | ✔   | ✔                                | ✔       | ✔          | ✔             | ✘      |

* [Install](#install)
* [Example](#example)
* [API](#api)
* [TODO](#todo)
* [Plugins](#plugins)
* [Collaborators](#collaborators)
* [Acknowledgements](#acknowledgements)
* [License](#license)

<a name="install"></a>
## Install
To install aedes, simply use npm:

```
npm install aedes --save
```

<a name="example"></a>
## Example

```js
var aedes = require('aedes')()
var server = require('net').createServer(aedes.handle)
var port = 1883

server.listen(port, function () {
  console.log('server listening on port', port)
})
```

### TLS

```js
var fs = require('fs')
var aedes = require('aedes')()

var options = {
  key: fs.readFileSync('YOUR_TLS_KEY_FILE.pem'),
  cert: fs.readFileSync('YOUR_TLS_CERT_FILE.pem')
}

var server = require('tls').createServer(options, aedes.handle)

server.listen(8883, function () {
  console.log('server started and listening on port 8883')
})
```

### WEBSOCKETS

```js
var aedes = require('./aedes')()
var server = require('net').createServer(aedes.handle)
var httpServer = require('http').createServer()
var ws = require('websocket-stream')
var port = 1883
var wsPort = 8888

server.listen(port, function () {
  console.log('server listening on port', port)
})

ws.createServer({
  server: httpServer
}, aedes.handle)

httpServer.listen(wsPort, function () {
  console.log('websocket server listening on port', wsPort)
})
```

<a name="api"></a>
## API

  * <a href="#constructor"><code><b>aedes()</b></code></a>
  * <a href="#Server"><code><b>aedes.Server()</b></code></a>
  * <a href="#handle"><code>instance.<b>handle()</b></code></a>
  * <a href="#subscribe"><code>instance.<b>subscribe()</b></code></a>
  * <a href="#publish"><code>instance.<b>publish()</b></code></a>
  * <a href="#unsubscribe"><code>instance.<b>unsubscribe()</b></code></a>
  * <a href="#preConnect"><code>instance.<b>preConnect()</b></code></a>
  * <a href="#authenticate"><code>instance.<b>authenticate()</b></code></a>
  * <a href="#authorizePublish"><code>instance.<b>authorizePublish()</b></code></a>
  * <a href="#authorizeSubscribe"><code>instance.<b>authorizeSubscribe()</b></code></a>
  * <a href="#authorizeForward"><code>instance.<b>authorizeForward()</b></code></a>
  * <a href="#published"><code>instance.<b>published()</b></code></a>
  * <a href="#close"><code>instance.<b>close()</b></code></a>
  * <a href="#client"><code><b>Client</b></code></a>
  * <a href="#clientid"><code>client.<b>id</b></code></a>
  * <a href="#clientclean"><code>client.<b>clean</b></code></a>
  * <a href="#clientconn"><code>client.<b>conn</b></code></a>
  * <a href="#clientreq"><code>client.<b>req</b></code></a>
  * <a href="#clientpublish"><code>client.<b>publish()</b></code></a>
  * <a href="#clientsubscribe"><code>client.<b>subscribe()</b></code></a>
  * <a href="#clientunsubscribe"><code>client.<b>unsubscribe()</b></code></a>
  * <a href="#clientclose"><code>client.<b>close()</b></code></a>

-------------------------------------------------------
<a name="constructor"></a>
### aedes([opts])

Creates a new instance of Aedes.

Options:

* `mq`: an instance of [MQEmitter](http://npm.im/mqemitter),
  such as [MQEmitterRedis](http://npm.im/mqemitter-redis)
  or [MQEmitterMongoDB](http://npm.im/mqemitter-mongodb). Used to share messages between multiple brokers instances (ex: clusters)
* `persistence`: an instance of [AedesPersistence](http://npm.im/aedes-persistence),
  such as [aedes-persistence-redis](http://npm.im/aedes-persistence-redis),
  [aedes-persistence-nedb](http://npm.im/aedes-persistence-nedb)
  or [aedes-persistence-mongodb](http://npm.im/aedes-persistence-mongodb). It's used to store QoS > 1 packets, retained packets, will messages and subscriptions in memory or on disk (if not specified default persistence is in memory)
* `concurrency`: the max number of messages delivered concurrently,
  defaults to `100`
* `heartbeatInterval`: the interval at which the broker heartbeat is
  emitted, it used by other broker in the cluster, defaults to
  `60000` milliseconds
* `connectTimeout`: the max number of milliseconds to wait for the CONNECT
  packet to arrive, defaults to `30000` milliseconds
* `id`: id used to identify this broker instance in `$SYS` messages,
  defaults to `uuidv4()`
* `decodeProtocol`: function called when a valid buffer is received, see
  [instance.decodeProtocol()](#decodeProtocol)
* `preConnect`: function called when a valid CONNECT is received, see
  [instance.preConnect()](#preConnect)
* `authenticate`: function used to authenticate clients, see
  [instance.authenticate()](#authenticate)
* `authorizePublish`: function used to authorize PUBLISH packets, see
  [instance.authorizePublish()](#authorizePublish)
* `authorizeSubscribe`: function used to authorize SUBSCRIBE packets, see
  [instance.authorizeSubscribe()](#authorizeSubscribe)
* `authorizeForward`: function used to authorize forwarded packets, see
  [instance.authorizeForward()](#authorizeForward)
* `published`: function called when a new packet is published, see
  [instance.published()](#published)

Events:

* `client`: when a new [Client](#client) successfully connects and register itself to server, [connackSent event will be come after], arguments:
  1. `client`
* `clientReady`: when a new [Client](#client) received all its offline messages, it is ready, arguments:
  1. `client`
* `clientDisconnect`: when a [Client](#client) disconnects, arguments:
  1. `client`
* `clientError`: when a [Client](#client) errors, arguments:
  1. `client`
  2. `err`
* `connectionError` When a [Client](#client) connection errors and there is no clientId attached , arguments:
  1. `client`
  2. `err`
* `keepaliveTimeout`: when a [Client](#client) keepalive times out, arguments:
  1. `client`
* `publish`: when a new packet is published, arguments:
  1. `packet`
  2. `client`, it will be null if the message is published using
     [`publish`](#publish). It is by design that the broker heartbeat will be on publish event, in this case `client` is null
* `ack`: when a packet published to a client is delivered successfully with QoS 1 or QoS 2, arguments:
  1. `packet`, this will be the original PUBLISH packet in QoS 1, and PUBREL in QoS 2
  2. `client`
* `ping`: when a [Client](#client) sends a ping, arguments:
  1. `packet`
  2. `client`
* `subscribe`: when a client sends a SUBSCRIBE, arguments:
  1. `subscriptions`, as defined in the `subscriptions` property of the
     [SUBSCRIBE](https://github.com/mqttjs/mqtt-packet#subscribe)
packet.
  2. `client`
* `unsubscribe`: when a client sends a UNSUBSCRIBE, arguments:
  1. `unsubscriptions`, as defined in the `subscriptions` property of the
     [UNSUBSCRIBE](https://github.com/mqttjs/mqtt-packet#unsubscribe)
packet.
  2. `client`
* `connackSent`: when a CONNACK packet is sent to a client, arguments:
  1. `packet`
  2. `client`
* `closed`: when the broker is closed

-------------------------------------------------------
<a name="Server"></a>
### new aedes.Server([opts])

Same as [`aedes(opts)`](#constructor).
Creates a new instance of Aedes.
This variant is useful with TypeScript or ES modules.

-------------------------------------------------------
<a name="handle"></a>
### instance.handle(duplex)

Handle the given duplex as a MQTT connection.

```js
var aedes = require('./aedes')()
var server = require('net').createServer(aedes.handle)
```

-------------------------------------------------------
<a name="subscribe"></a>
### instance.subscribe(topic, func(packet, cb), done)

After `done` is called, every time [publish](#publish) is invoked on the
instance (and on any other connected instances) with a matching `topic` the `func` function will be called.

`func` needs to call `cb` after receiving the message.

It supports backpressure.

-------------------------------------------------------
<a name="publish"></a>
### instance.publish(packet, done)

Publish the given packet to subscribed clients and functions. It supports backpressure.

A packet contains the following properties:

```js
{
  cmd: 'publish',
  qos: 2,
  topic: 'test',
  payload: new Buffer('test'),
  retain: false
}
```

Only the `topic` property is mandatory.
Both `topic` and `payload` can be `Buffer` objects instead of strings.

-------------------------------------------------------
<a name="unsubscribe"></a>
### instance.unsubscribe(topic, func(packet, cb), done)

The reverse of [subscribe](#subscribe).
------------------------------------------------------
<a name="decodeProtocol"></a>
### instance.decodeProtocol(client, buffer)

It will be called when aedes instance trustProxy is true and that it receives a first valid buffer from client. client object state is in default and its connected state is false. A default function parse https headers (x-real-ip | x-forwarded-for) and proxy protocol v1 and v2 to retrieve information in client.connDetails. Override to supply custom protocolDecoder logic, if it returns an object with data property, this property will be parsed as an mqtt-packet.


```js
instance.decodeProtocol = function(client, buffer) {
  var protocol = yourDecoder(client, buffer)
  return protocol
}
```
-------------------------------------------------------
<a name="preConnect"></a>
### instance.preConnect(client, done(err, successful))

It will be called when aedes instance receives a first valid CONNECT packet from client. client object state is in default and its connected state is false. Any values in CONNECT packet (like clientId, clean flag, keepalive) will pass to client object after this call. Override to supply custom preConnect logic.
Some use cases:
1. Rate Limit / Throttle by `client.conn.remoteAddress`
2. Check `instance.connectedClient` to limit maximum connections
3. IP blacklisting

```js
instance.preConnect = function(client, callback) {
  callback(null, client.conn.remoteAddress === '::1') {
}
```
```js
instance.preConnect = function(client, callback) {
  callback(new Error('connection error'), client.conn.remoteAddress !== '::1') {
}
```
-------------------------------------------------------
<a name="authenticate"></a>
### instance.authenticate(client, username, password, done(err, successful))

It will be called when a new client connects. Override to supply custom
authentication logic.

```js
instance.authenticate = function (client, username, password, callback) {
  callback(null, username === 'matteo')
}
```
Other return codes can passed as follows :-

```js
instance.authenticate = function (client, username, password, callback) {
  var error = new Error('Auth error')
  error.returnCode = 1
  callback(error, null)
}
```
The return code values and their responses which can be passed are given below:

*  `1` - Unacceptable protocol version
*  `2` - Identifier rejected
*  `3` - Server unavailable
*  `4` - Bad user name or password

-------------------------------------------------------
<a name="authorizePublish"></a>
### instance.authorizePublish(client, packet, done(err))

It will be called when a client publishes a message. Override to supply custom
authorization logic.

```js
instance.authorizePublish = function (client, packet, callback) {
  if (packet.topic === 'aaaa') {
    return callback(new Error('wrong topic'))
  }

  if (packet.topic === 'bbb') {
    packet.payload = new Buffer('overwrite packet payload')
  }

  callback(null)
}
```

-------------------------------------------------------
<a name="authorizeSubscribe"></a>
### instance.authorizeSubscribe(client, pattern, done(err, pattern))

It will be called when a client subscribes to a topic. Override to supply custom
authorization logic.

```js
instance.authorizeSubscribe = function (client, sub, callback) {
  if (sub.topic === 'aaaa') {
    return callback(new Error('wrong topic'))
  }

  if (sub.topic === 'bbb') {
    // overwrites subscription
    sub.qos = sub.qos + 2
  }

  callback(null, sub)
}
```

To negate a subscription, set the subscription to `null`:

```js
instance.authorizeSubscribe = function (client, sub, callback) {
  if (sub.topic === 'aaaa') {
    sub = null
  }

  callback(null, sub)
}
```

-------------------------------------------------------
<a name="authorizeForward"></a>
### instance.authorizeForward(clientId, packet)

It will be called when a client is set to recieve a message. Override to supply custom
authorization logic.

```js
instance.authorizeForward = function (client, packet) {
  if (packet.topic === 'aaaa' && client.id === "I should not see this") {
    return null
    // also works with return undefined
  }

  if (packet.topic === 'bbb') {
    packet.payload = new Buffer('overwrite packet payload')
  }

  return packet
}
```

-------------------------------------------------------
<a name="published"></a>
### instance.published(packet, client, done())

It will be called after a message is published.
`client` will be null for internal messages.
Override to supply custom authorization logic.

-------------------------------------------------------
<a name="close"></a>
### instance.close([cb])

Disconnects all clients.

Events:

* `closed`, in case the broker is closed

-------------------------------------------------------
<a name="Client"></a>
### Client

Classes for all connected clients.

Events:

* `error`, in case something bad happended

-------------------------------------------------------
<a name="clientid"></a>
### client#id

The id of the client, as specified by the CONNECT packet, defaults to 'aedes_' + shortid()

-------------------------------------------------------
<a name="clientclean"></a>
### client#clean

`true` if the client connected (CONNECT) with `clean: true`, `false`
otherwise. Check the MQTT spec for what this means.

-------------------------------------------------------
<a name="clientconn"></a>
### client#conn

The client's connection stream object.

In the case of `net.createServer` brokers, it's the connection passed to the `connectionlistener` function by node's [net.createServer](https://nodejs.org/api/net.html#net_net_createserver_options_connectionlistener) API.

In the case of [websocket-stream](https://www.npmjs.com/package/websocket-stream) brokers, it's the `stream` argument passed to the `handle` function described in [that library's documentation](https://github.com/maxogden/websocket-stream/blob/e2a51644bb35132d7aa477ae1a27ff083fedbf08/readme.md#on-the-server).

-------------------------------------------------------
<a name="clientreq"></a>
### client#req

The HTTP Websocket upgrade request object passed to websocket broker's `handle` function by the [`websocket-stream` library](https://github.com/maxogden/websocket-stream/blob/e2a51644bb35132d7aa477ae1a27ff083fedbf08/readme.md#on-the-server).

If your clients are connecting to aedes via websocket and you need access to headers or cookies, you can get them here. **NOTE:** this property is only present for websocket connections.

-------------------------------------------------------
<a name="clientpublish"></a>
### client#publish(message, [callback])

Publish the given `message` to this client. QoS 1 and 2 are fully
respected, while the retained flag is not.

`message` is a [PUBLISH](https://github.com/mqttjs/mqtt-packet#publish) packet.

`callback`  will be called when the message has been sent, but not acked.

-------------------------------------------------------
<a name="clientsubscribe"></a>
### client#subscribe(subscriptions, [callback])

Subscribe the client to the list of topics.

`subscription` can be:

1. a single object in the format `{ topic: topic, qos: qos }`
2. an array of the above
3. a full [subscribe
   packet](https://github.com/mqttjs/mqtt-packet#subscribe),
specifying a `messageId` will send suback to the client.

`callback`  will be called when the subscription is completed.

-------------------------------------------------------
<a name="clientunsubscribe"></a>
### client#unsubscribe(topicObjects, [callback])

Unsubscribe the client to the list of topics.

The topic objects can be as follows :-

1. a single object in the format `{ topic: topic, qos: qos }`
2. an array of the above

`callback`  will be called when the unsubscriptions are completed.

-------------------------------------------------------
<a name="clientclose"></a>
### client#close([cb])

Disconnects the client

-------------------------------------------------------
<a name="clientpresence"></a>
### client presence

You can subscribe on the following `$SYS` topics to get client presence:

 - `$SYS/+/new/clients` - will inform about new clients connections
 - `$SYS/+/disconnect/clients` - will inform about client disconnections.
The payload will contain the `clientId` of the connected/disconnected client

## Plugins

- [aedes-persistence](https://github.com/moscajs/aedes-persistence): In-memory implementation of an Aedes persistence
  - [aedes-persistence-mongodb](https://github.com/moscajs/aedes-persistence-mongodb): MongoDB persistence for Aedes
  - [aedes-persistence-redis](https://github.com/moscajs/aedes-persistence-redis): Redis persistence for Aedes
  - [aedes-persistence-level](https://github.com/moscajs/aedes-persistence-level): LevelDB persistence for Aedes
  - [aedes-persistence-nedb](https://github.com/ovhemert/aedes-persistence-nedb#readme): NeDB persistence for Aedes
- [mqemitter](https://github.com/mcollina/mqemitter): An Opinionated Message Queue with an emitter-style API
  - [mqemitter-redis](https://github.com/mcollina/mqemitter-redis): Redis-powered mqemitter
  - [mqemitter-mongodb](https://github.com/mcollina/mqemitter-mongodb): Mongodb based mqemitter
  - [mqemitter-child-process](https://github.com/mcollina/mqemitter-child-process): Share the same mqemitter between a hierarchy of child processes
  - [mqemitter-cs](https://github.com/mcollina/mqemitter-cs): Expose a MQEmitter via a simple client/server protocol
  - [mqemitter-p2p](https://github.com/mcollina/mqemitter-p2p): A P2P implementation of MQEmitter, based on HyperEmitter and a Merkle DAG
  - [mqemitter-aerospike](https://github.com/GavinDmello/mqemitter-aerospike): Aerospike mqemitter based on @mcollina 's mqemitter
- [aedes-logging](https://github.com/moscajs/aedes-logging): Logging module for Aedes, based on Pino
- [aedes-stats](https://github.com/moscajs/aedes-stats): Stats for Aedes

## Collaborators

* [__Gavin D'mello__](https://github.com/GavinDmello)
* [__Behrad Zari__](https://github.com/behrad)
* [__Gnought__](https://github.com/gnought)

## Acknowledgements

This library is born after a lot of discussion with all
[Mosca](http://npm.im/mosca) users and how that was deployed in
production. This addresses your concerns about performance and
stability.

## License

MIT

[dynamic_topics]: https://github.com/mqtt/mqtt.github.io/wiki/are_topics_dynamic
[bridge_protocol]: https://github.com/mqtt/mqtt.github.io/wiki/bridge_protocol