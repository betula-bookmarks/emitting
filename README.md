# Emitting

[![Build Status](https://travis-ci.com/sergeysova/emitting.svg?branch=master)](https://travis-ci.com/sergeysova/emitting)
[![Codecov](https://img.shields.io/codecov/c/github/sergeysova/emitting)](https://codecov.io/gh/sergeysova/emitting)
[![Codacy](https://api.codacy.com/project/badge/Grade/3c6e3befee084c1d8e6ed87ef7b17327)](https://app.codacy.com/manual/sergeysova/emitting)
[![dependencies Status](https://david-dm.org/sergeysova/emitting/status.svg)](https://david-dm.org/sergeysova/emitting)
[![npm bundle size](https://img.shields.io/bundlephobia/min/emitting)](https://bundlephobia.com/result?p=emitting)

[[Github](https://github.com/sergeysova/emitting) | [NPM](https://npmjs.com/package/emitting) | [Typedoc](https://emitting.sova.dev/)]

Emitting is a simple event emitter designed for **TypeScript** and **Promises**. There are some differences from other emitters:

- **Exactly typing** for event payloads `new EventEmitter<Events>()`
- **Waiting for event** with `.take("event"): Promise<Payload>` and same
- **Do not `throw`** an error when you emit an `error` event and nobody is listening
- **Functional** — methods don't rely on `this`
- **Small size**. 500 bytes without typings. No dependencies. [Size Limit](https://github.com/ai/size-limit) controls the size.

## Table of contents

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Installation](#installation)
- [Polyfills](#polyfills)
- [Usage](#usage)
  - [JavaScript](#javascript)
  - [TypeScript](#typescript)
- [API](#api)
  - [Constructor](#constructor)
  - [`.on(eventName, handler)` — add event listener](#oneventname-handler--add-event-listener)
  - [`.once(eventName, handler)` — listen event once](#onceeventname-handler--listen-event-once)
  - [`.emit(eventName, payload)` — send event to listeners](#emiteventname-payload--send-event-to-listeners)
  - [`.emitCallback(eventName)` — create emitter function](#emitcallbackeventname--create-emitter-function)
  - [`.take(event): Promise` — wait for event](#takeevent-promise--wait-for-event)
  - [`.takeTimeout(event, ms): Promise` — wait for event or reject](#taketimeoutevent-ms-promise--wait-for-event-or-reject)
  - [`.takeEither(successEvent, failureEvent): Promise` — resolve or reject promise with events](#takeeithersuccessevent-failureevent-promise--resolve-or-reject-promise-with-events)
  - [Unsubscribe from events](#unsubscribe-from-events)
  - [`.off(eventName)` — remove all listeners](#offeventname--remove-all-listeners)
- [Roadmap](#roadmap)
- [License](#license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Installation

```sh
# NPM
npm instal --save emitting

# Yarn
yarn add emitting
```

## Polyfills

Module built to ECMAScript 5.

Be sure to add polyfills if neccessary for:

- `Promise`
- `Set`
- `Map`

## Usage

### JavaScript

After installation the only thing you need to do is require the module:

```js
const { EventEmitter } = require("emitting")

const emitter = new EventEmitter()
```

### TypeScript

After installation you need to import module and define events:

```ts
import { EventEmitter } from "emitting"

type Events = {
  hello: { name: string }
  bye: void
}

const emitter = new EventEmitter<Events>()
// Now you have typed event emitter 🚀 yay!
```

## API

Documentation for API generated by TypeDOC — [emitting.sova.dev](https://emitting.sova.dev/)

### Constructor

Receives type parameter `Events` that should be object type or interface, where key is an event name and value is an payload as a single parameter type.

```ts
type Events = {
  eventName: PayloadType
}
```

Pass `Events` to constructor generic parameter:

```ts
import { EventEmitter } from "emitting"

type Events = {
  hello: { name: string }
  bye: void
}

const emitter = new EventEmitter<Events>()
```

Now you can emit events and subscribe to.

### `.on(eventName, handler)` — add event listener

`.on()` receives an event name and an event handler.<br/>
Event handler should be a function that receives a payload.<br/>
`.on()` returns `unsubscribe` function to remove created subscribtion.

```ts
function helloHandler(payload: { name: string }) {
  console.log(payload.name)
}

function byeHandler() {
  console.log("Goodbye!")
}

emitter.on("hello", helloHandler)
emitter.on("bye", byeHandler)
```

### `.once(eventName, handler)` — listen event once

Subscribes to event, and when it received immediately unsubscribe.<br/>
Subscribtion can be canceled at any time.

```ts
const cancel = emitter.on("hello", helloHandler)

cancel() // subscription cancelled
```

### `.emit(eventName, payload)` — send event to listeners

Executes all listeners with passed payload.<br/>
Accepts only one payload parameter. Use object type or tuple type to pass multiple payloads.<br/>
If no listeners nothing happens.

```ts
emitter.emit("hello", { name: "world" })
```

### `.emitCallback(eventName)` — create emitter function

Create function that emit event when called.<br/>
Payload should be passed to returned callback.

```ts
const hello = emitter.emitCallback("hello")

hello({ name: "world" }) // emitted "hello" event
```

### `.take(event): Promise` — wait for event

Creates promise that resolves when specified event is received. <br/>
Returns `Promise` resolved with payload of the passed event. <br/>
Listeners removed after event is received.

```ts
type Events = {
  example: number
}
const emitter = new EventEmitter<Events>()

emitter.take("example").then((payload) => {
  console.log("Received", payload)
})

emitter.emit("example", 10) // Received 10
```

With `async/await`:

```ts
const payload: number = await emitter.take("example")
```

### `.takeTimeout(event, ms): Promise` — wait for event or reject

Creates a promise that resolves when specified event is received.<br/>
Promise resolves with payload of the received event. <br/>
Promise is rejected when timeout is reached. <br/>
Listeners removed after timeout is reached, and event is received.

```ts
try {
  const { name } = await emitter.takeTimeout("hello", 300)
} catch (_) {
  console.info("Event `hello` is not emitted in 300 ms after subscribing to")
}
```

### `.takeEither(successEvent, failureEvent): Promise` — resolve or reject promise with events

Returns promise that resolves when `successEvent` is emitted with the payload of event. <br/>
Promise rejected when `failureEvent` is emitted, as error passed payload of the `failureEvent`. <br/>
Listeners removed when `successEvent` or `failureEvent` is received and promise resolves just once. <br/>

```ts
emitter
  .takeEither("success", "failure")
  .then((payload) => console.log("Yeeah", payload))
  .catch((error) => console.log("Noooo", error))

emitter.emit("success", 500) // Yeeah 500
// or
emitter.emit("failure", "Vader is my father!") // Noooo Vader is my father
```

### Unsubscribe from events

The `.on()`, `.once()` and same returns `unsubscribe` function, that can be called multiple times at any time.

```ts
const unsubscribeHello = emitter.on("hello", helloHandler)
const unsubscribeBye = emitter.on("bye", helloHandler)

unsubscribeHello() // now helloHandler will not be called when "hello" is emitted
```

### `.off(eventName)` — remove all listeners

> Note: destructive operation

The method removes all listeners from emitter. Use it with caution, if called `.take` methods, **promises will never be fullfilled**.

```ts
emitter.off("hello") // all listeners removed
```

## Roadmap

- [ ] Add `emitter.off(eventName, handler)`
- [ ] Add `*` event to listen all events
- [ ] Add `emitter.events(): string[]`
- [ ] Add `emitting::listenerAdded`, `emitting::listenerRemoved`
- [ ] Add support for reactive (`.subscribe`)
- [ ] Add async iterators (`.iterate(eventName)`)
- [ ] Add benchmarks

## License

[MIT](https://github.com/sergeysova/emitting/blob/HEAD/LICENSE)
