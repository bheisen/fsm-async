# fsm-async
[![Build Status](https://travis-ci.org/bheisen/fsm-async.svg?branch=master)](https://travis-ci.org/bheisen/fsm-async)
[![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/bheisen/fsm-async/master/LICENSE)
[![Semver](https://img.shields.io/SemVer/2.0.0.png)](https://semver.org/spec/v2.0.0.html)

A state machine implementation featuring:

1.  `on<state>` life-cycle events, allowing the triggering of further (internal)
     events during the callback.

2.  `async` event functions that can be awaited. Depending
    on the implemented logic, multiple state changes can be awaited.

3.  Generic and awaitable `waitUntilStateEnters(<state>)` and
    `waitUntilStateLeaves(<state>)` functions providing full flexibility to
    state machine clients business logic.

## Example

Define the transition table as a json object,

```javascript
const transitionTable = {
  initial: 'disconnected',
  transitions: [
    { ev: 'connect', from: 'disconnected', to: 'connecting' },
    { ev: '_connectDone', from: 'connecting', to: 'connected' },
    { ev: 'disconnect', from: 'connected', to: 'disconnecting' },
    { ev: '_disconnectDone', from: 'disconnecting', to: 'disconnected' }
  ]
}
```

then apply this logic to your object:

```javascript
const StateMachine = require('fsm-async')

class MyClient extends StateMachine {

  constructor () {
    const transitionTable = {
      initial: 'disconnected',
      transitions: [
        { ev: 'connect', from: 'disconnected', to: 'connecting' },
        { ev: '_connectDone', from: 'connecting', to: 'connected' },
        { ev: 'disconnect', from: 'connected', to: 'disconnecting' },
        { ev: '_disconnectDone', from: 'disconnecting', to: 'disconnected' }
      ]
    }
    super(transitionTable)
  }
}
```

This injects the events as proper callable functions to your instance,
hence you can write:

```javascript
myClient = new MyClient()
myClient.connect()
```

In the body of your class you can define life-cycle functions `on<event>` and
`on<state>`, which are automatically called and can be used to trigger
further events:

```javascript
const StateMachine = require('fsm-async')

class MyClient extends StateMachine {

  constructor () {
    const transitionTable = {
      initial: 'disconnected',
      transitions: [
        { ev: 'connect', from: 'disconnected', to: 'connecting' },
        { ev: '_connectDone', from: 'connecting', to: 'connected' },
        { ev: 'disconnect', from: 'connected', to: 'disconnecting' },
        { ev: '_disconnectDone', from: 'disconnecting', to: 'disconnected' }
      ]
    }
    super(transitionTable)
  }

  // Use async here to be able to await internally
  async onConnecting () {
    // Simulate connection establishment
    await new Promise(resolve => setTimeout(resolve, 1000))
    // Internally trigger an event bringing the machine to connected state
    this._connectDone()
  }

  async onDisconnecting () {
    // Simulate disconnection
    await new Promise(resolve => setTimeout(resolve, 1000))
    // Internally trigger an event bringing the machine to disconnected state
    this._disconnectDone()
  }
}
```

Now, outer code can `await` the `connect()` of your client and/or use other
utility functions injected by the `StateMachine`. The utility functions are:
1. `getState()` returns current state
2. `waitUntilStateEnters(<state>)` waits until a given state is entered
3. `waitUntilStateLeaves(<state>)` waits until a given state is left
4. `onStateChange(<callback(state)>)` notifies about state changes
5. `onInvalidTransition(<callback(event, state)>)` notifies about invalid transitions

The `StateMachine` class at the same time is an event emitter. Hence,

```
stateMachine.on('state', <callback(state)>)
stateMachine.on('invalidTransition', <callback(event, state)>)
```
is also possible.

Please see the provided example code (`examples` folder) for more details and
usage patterns.

You can run the example code via:

```
npm run example
```
