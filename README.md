# notes on data flow primitives

## fundamental elements

* message: any value (string, obj, etc)
* source: an emitter of messages
* sink: a destination for messages
* channel: uni-directional path between a source and a sink

[![nomnoml 1][nomnoml-1-img]][nomnoml-1-link]

[nomnoml-1-img]: https://cloud.githubusercontent.com/assets/1474978/20232340/79b0a95c-a81c-11e6-8e7f-9b33d0a3d6e5.png
[nomnoml-1-link]: http://www.nomnoml.com/#view/%5B%3Creceiver%3EA%20(in)%5D%0A%5B%3Csender%3EA%20(out)%5D%0A%5B%3Creceiver%3EA%20(in)%5D-%3E%5B%3Csender%3EA%20(out)%5D%0A

## composition

* transform: a sink and a source. mutates the message.

[![nomnoml 2][nomnoml-2-img]][nomnoml-2-link]

[nomnoml-2-img]: https://cloud.githubusercontent.com/assets/1474978/20232637/7d301e76-a81e-11e6-89ac-d9d7ca1fc19e.png
[nomnoml-2-link]: http://www.nomnoml.com/#view/%5B%3Creceiver%3E(in)%5D%0A%5B%3Cframe%3E%20transform%5D%0A%5B%3Csender%3E(out)%5D%0A%5B%3Creceiver%3E(in)%5D-%3E%5B%3Cframe%3E%20transform%5D%0A%5B%3Cframe%3E%20transform%5D-%3E%5B%3Csender%3E(out)%5D%0A

* multichannel: you can wrap multiple channels in a single channel by wrapping the message with metadata. this is useful if you want to send multiple channels over a single network connection.

```js
function multiplexTransform(originChannel, message) {
  return {
    channel: originChannel,
    data: message,
  }
}
```
[![nomnoml 3][nomnoml-3-img]][nomnoml-3-link]

[nomnoml-3-img]: https://cloud.githubusercontent.com/assets/1474978/20232867/2d48f048-a820-11e6-850e-0e83fdfe40df.png
[nomnoml-3-link]: http://www.nomnoml.com/#view/%5B%3Creceiver%3EA%20(in)%5D%0A%5B%3Csender%3EA%20(out)%5D%0A%5B%3Creceiver%3EB%20(in)%5D%0A%5B%3Csender%3EB%20(out)%5D%0A%5B%3Creceiver%3EMulti%20(in)%5D%0A%5B%3Csender%3EMulti%20(out)%5D%0A%5B%3Cframe%3E%20multiplex%5D%0A%5B%3Cchoice%3E%20de-multiplex%5D%0A%0A%5B%3Creceiver%3EA%20(in)%5D-%3E%5B%3Cframe%3E%20multiplex%5D%0A%5B%3Creceiver%3EB%20(in)%5D-%3E%5B%3Cframe%3E%20multiplex%5D%0A%0A%5B%3Cframe%3E%20multiplex%5D-%3E%5B%3Creceiver%3EMulti%20(in)%5D%0A%5B%3Creceiver%3EMulti%20(in)%5D-%3E%5B%3Csender%3EMulti%20(out)%5D%0A%5B%3Csender%3EMulti%20(out)%5D-%3E%5B%3Cframe%3E%20de-multiplex%5D%0A%0A%5Bde-multiplex%5D-%3E%5B%3Creceiver%3EA%20(out)%5D%0A%5Bde-multiplex%5D-%3E%5B%3Creceiver%3EB%20(out)%5D%0A

* duplex channel: you can have bi-directional communication by pairing 2 uni-directional channels in opposite directions
[![nomnoml 4][nomnoml-4-img]][nomnoml-4-link]

[nomnoml-4-img]: https://cloud.githubusercontent.com/assets/1474978/20233258/e4dad10c-a822-11e6-9c58-33b9bbbdfb21.png
[nomnoml-4-link]:
http://www.nomnoml.com/#view/%5BDuplexChannel%7C%0A%5B%3Creceiver%3Elocal%20(in)%5D%0A%5B%3Csender%3Elocal%20(out)%5D%0A%5B%3Creceiver%3Eremote%20(in)%5D%0A%5B%3Csender%3Eremote%20(out)%5D%0A%0A%5B%3Creceiver%3Elocal%20(in)%5D-%3E%5B%3Csender%3Eremote%20(out)%5D%0A%5B%3Csender%3Elocal%20(out)%5D%3C-%5B%3Creceiver%3Eremote%20(in)%5D%0A%5D

## data flow

### data flow direction

- pull:
  - send request, get response
  - calling a fn
  - lazy-computed values
  - pull-streams
  - sending eth rpc requests

- push:
  - send data
  - handler being called
  - event emitters
  - observables (redux store)
  - current block (via poller)
  - standard streams
  - receiving dapp eth rpc requests

### flow direction adapters

 - push->pull (cache/disk)
 - pull->push (poll/network)

## flow transforms

both
  - transform

push
  - filter
    - debounce
    - throttle
    - mux/demux
    - waitForValuesPopulated

pull
  - cache

## layers

- obervable obj (observ)
- state transition process (simple set, reducer)
- semantic api (wrapper / dnode)

## ideal API design notes

same behavior remote+local ( async set's )
same API for observing ( despite reducer or obervable )

## properties

recompute strategies:
* lazy (pull only except for 'wait for first handlers' optimization)
* eager

flow API:
* streams
* handlers

channels:
* single (simpler API)
* multiple (more control over granularity/type of update)

locality:
* local
* remote (can we stream in a remote store with the same API?)


## prior art

### [Node.js|EventEmitter](https://nodejs.org/api/events.html#events_class_eventemitter)

pubsub. multiple channels.

```js
ee.on('a', fn)
ee.on('b', fn)
ee.emit('a', data)
```

### [React|Redux](https://github.com/reactjs/redux)

observable store. processes state transitions via reducers. emits new result immediately.

```js
function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1
  case 'DECREMENT':
    return state - 1
  default:
    return state
  }
}

// Create a Redux store holding the state of your app.
// Its API is { subscribe, dispatch, getState }.
let store = createStore(counter)

// You can use subscribe() to update the UI in response to state changes.
// Normally you’d use a view binding library (e.g. React Redux) rather than subscribe() directly.
// However it can also be handy to persist the current state in the localStorage.

store.subscribe(() =>
  console.log(store.getState())
)

// The only way to mutate the internal state is to dispatch an action.
// The actions can be serialized, logged or stored and later replayed.
store.dispatch({ type: 'INCREMENT' })
```

### [Raynos|observ](https://github.com/raynos/observ)

observable store. single channel. *calls new listener immediately.*
allows simple usage of `set`.

```js
// create + set initial
var v = Observable("initial value")
// listen
var stopListening = v(function onchange(newValue) {
  assert.equal(newValue, "new value")
})
// set
v.set("new value")
```

### [Raynos|observ/computed](https://github.com/raynos/observ#example-computed)

compose observables

```js
var one = Observable(1)
var two = Observable(2)
var together = computed([one, two], function (a, b) {
  return a + b
})
```

### [Raynos|observ/observ-struct](https://github.com/Raynos/observ-struct)

children are also obserables

```js
var state = ObservStruct({
    fruits: ObservStruct({
        apples: Observ(3),
        oranges: Observ(5)
    }),
    customers: Observ(5)
})

state(function (current) {
  console.log("apples", current.fruits.apples)
  console.log("customers", current.customers)
})

state.fruits(function (current) {
  console.log("apples", current.apples)
})

var initialState = state()
assert.equal(initialState.fruits.oranges, 5)
assert.equal(initialState.customers, 5)

state.fruits.oranges.set(6)
state.customers.set(5)
state.fruits.apples.set(4)
```


### [Ember|EmberObject](https://github.com/emberjs/ember.js/blob/master/packages/ember-runtime/lib/mixins/observable.js)

magical overkill. name re-compute dependencies with a string.
some fancy stuff for pattern matching properties against children of Arrays.

```javascript
Ember.Object.extend({
  valueObserver: Ember.observer('value', function(sender, key, value, rev) {
    // Executes whenever the "value" property changes
    // See the addObserver method for more information about the callback arguments
  })
});
```

### [Dominictarr|pull-stream](https://github.com/pull-stream/pull-stream)

streams that pull data instead of push, allowing lazy evaluation.

```js
pull(
  pull.values(['file1', 'file2', 'file3']),
  pull.asyncMap(fs.stat),
  pull.collect(function (err, array) {
    console.log(array)
  })
)
```

### [RxJS](https://github.com/Reactive-Extensions/RxJS)

stream-like flow control lib

```js
/* Get stock data somehow */
const source = getAsyncStockData();

const subscription = source
  .filter(quote => quote.price > 30)
  .map(quote => quote.price)
  .subscribe(
    price => console.log(`Prices higher than $30: ${price}`),
    err => console.log(`Something went wrong: ${err.message}`);
  );

/* When we're done */
subscription.dispose();
```
