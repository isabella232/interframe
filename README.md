# Interframe<br/>[![Sponsored by][sponsor-img]][sponsor] [![Version][npm-version-img]][npm] [![Downloads][npm-downloads-img]][npm] [![Build Status Unix][travis-img]][travis] [![Dependencies][deps-img]][deps]

Communication made easy between browser frames.

[sponsor-img]: https://img.shields.io/badge/Sponsored%20by-Sebastian%20Software-692446.svg
[sponsor]: https://www.sebastian-software.de
[deps]: https://david-dm.org/sebastian-software/interframe
[deps-img]: https://david-dm.org/sebastian-software/interframe.svg
[npm]: https://www.npmjs.com/package/interframe
[npm-downloads-img]: https://img.shields.io/npm/dm/interframe.svg
[npm-version-img]: https://img.shields.io/npm/v/interframe.svg
[travis-img]: https://img.shields.io/travis/sebastian-software/interframe/master.svg?branch=master&label=unix%20build
[travis]: https://travis-ci.org/sebastian-software/interframe

# Using Interframe

_Interframe_ provides a factory function that takes a `window` and an `origin`
to open a communication channel. Please provide the `window` object of the
counterpart frame to open a communication channel with that frame.

```
import interframe from "interframe"

/* get reference to iframe */
const iframe = document.getElementById("myIframe")

const channel = interframe(iframe.contentWindow, "*")
```

Using `*` as origin allows communication with every other message provider.

Inside of the iframe you can open the channel via

```
const channel = interframe(window.top)
```

All communication data are stored in memory as long as the handshake is not done. As soon as the handshake is done the data
are sent through the channel.

## Listening for messages

_Interframe_ allowes to add message event listeners to receive messages from
the opposite side. As long as no message listener for the specific namespace
is assigned messages are cached.

```
channel.addListener("namespace", (message) =>
{
  console.log(message.id)
  console.log(message.namespace)
  console.log(message.data)
  console.log(message.channel)
})
```

## Sending messages

A message consist of a namespace and, optional, a serializable object.

```
channel.send("namespace", { foo: "bar" })
```

## Responding to messages

As each message has a unique _id_ interframe is able to response to messages.
For this the `send()` method returns a _promise_ that is resolved with a message.
If response channel is not opened inside message callback the promise is rejected.

```
const channel1 = interframe(window, "*")
const channel2 = interframe(window, "*")

channel1.addListener("my namespace", (message) =>
{
  const responseChannel = message.open()

  setTimeout(() =>
  {
    responseChannel.response({
      hello: `Hi ${message.data.username}`
    })
  }, 1000)
})

channel2
  .send("my namespace", { username: "Sebastian" })
  .then((message) =>
  {
    console.log(message.id)
    console.log(message.namespace)
    console.log(message.data)
    console.log(message.channel)
  })
```

`response()` is a shortcut of send with preset namespace of source message.

# API

```
function interframe(targetWindow, [origin = "*"])

returns

{
  addListener,
  removeListener,
  send,
  hasHandshake
}
```

This factory function returns a channel.

```
function addListener(namespace, callback)

returns

callback
```

Add callback for new messages. callback is a function with the signature
`(message) => {}`

```
function removeListener(callback)
```

Disconnect specific callback from message events.

```
function send(namespace, [data])

returns

Promise<message>
```

Send message to opposite side. `namespace` is a string that defines the type
of the message. `data` is an optional argument that must be serializable by
`JSON.stringify`.

The returned message consists of

```
{
  id,
  data,
  namespace,
  response,
}
```

* id is the id of the message
* data is the optional data object (given in send())
* namespace is the namespace (given in send())

The promise only resolves if the `response()`function of the message inside addListener callback is used.

```
function hasHandshake([callback])

returns

boolean
```

Returns if handshake is successfull. An optional callback is called as
soon as there is a handshake.

## Copyright

<img src="https://cdn.rawgit.com/sebastian-software/sebastian-software-brand/0d4ec9d6/sebastiansoftware-en.svg" alt="Sebastian Software GmbH Logo" width="460" height="160"/>

Copyright 2016-2019<br/>[Sebastian Software GmbH](http://www.sebastian-software.de)
