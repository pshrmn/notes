# Redux

Source code on [GitHub](https://github.com/reactjs/redux), documentation at [http://redux.js.org/](http://redux.js.org/).

* [Overview](#overview)
* [Installation](#installation)
* [Actions](#actions)
* [Reducers](#reducers)
  * [Splitting Reducers](#splitting-reducers)
  * [Combining Reducers](#combining-reducers)
* [Store](#store)
  * [`createStore`](#createstorereducer-initialstate-enhancer)
  * [`subscribe`](#storesubscribecallback)
  * [`dispatch`](#storedispatchaction)
* [Enhancers](#enhancers)
  * [Middleware](#middleware)
* [Example](#example)

## Overview

Redux manages the state of an app using a single store. To modify the state, an action is dispatched to the store and a reducer updates the store based on the type and other values of the action. In order to use the new state, the store can be subscribed to. Subscribing requires a callback function which is called whenever the store updates.

## Installation

```bash
npm install --save redux
```

## Actions

An action is an object that describes how the store should be updated. Actions require a type, which is used by reducers to only update properties for specific actions. Other properties can be included in the action to pass any relevant data to the store.


```js
{
  type: "SET_NAME",
  name: "Wilbur"
}
```

## Reducers

A reducer updates the state of the store. A reducer takes two arguments, the current state and the action that has been dispatched. The current state is manipulated and returns a new state based on the action.

```js
function reducer(state, action) {
  switch (action.type) {
  case "SET_NAME":
    return Object.assign({}, state, {
      name: action.name
    });
  default:
    return state;
  }
}
```


#### State

The first argument to a reducer function is the current state. The state can be of any type, but when the state is an object and you return a new state, the new state should be a new object, not a modified version of the old state object. This should be done so that a comparison between the objects will identify them as being different.

```js
// good
return Object.assign({}, oldState, {name: "foo"});

//bad
oldState.name = "foo";
return oldState;
```

#### Action

The second argument to a reducer function is the action that was dispatched to the store. The current state should be updated based on the type of the action and any additional properties of the action. The easiest way to handle the various action types is by using a switch statement for the known types, and a default case that returns the current state for unknown types.


### Splitting Reducers

For simple cases a single reducer function will suffice, but when the code starts to get too complicated, it is useful to split the reducer into multiple reducer functions. Those functions can then be combined to create a root reducer using redux's combineReducers function. Each reducer only cares about actions that affect the part of the state that they maintain, returning their current state for all other actions.

When splitting the reducer, there should be a function for each base property of the state. For example if your state has a "name" property and an "attrs" property, there should be a name reducer and an attrs reducer. While the first argument to the root reducer function is the current state of the store, the first argument passed to the sub-reducer functions is the current value of that property in the store.

```js
/*
 * store = {
 *     name: "",
 *     attrs: ""   
 * }
 */

const initialState = {
  name: "foo",
  attrs: "bar"
};

function rootReducer(state=initialState, action) {
  return {
    name: nameReducer(state.name, action),
    attrs: attrReducer(state.attrs, action)
  };
}

function nameReducer(state="", action) {
  switch (action.type) {
  case "SET_NAME":
    return action.name;
  default:
    return state;
  }
}

function attrReducer(state={}, action) {
  switch (action.type) {
  case "ADD_ATTR":
    return Object.assign({}, state, {
      [attr.key]: action.value
    });
  case "REMOVE_ATTR":
    const stateCopy = Object.assign({}, state);
    delete stateCopy[action.key];
    return stateCopy;
  case "GET_ATTRS":
    return state;
  default:
    return state;
  }
}
```

### Combining Reducers

Instead of writing your own root reducer, you can combine your property specific reducers using redux's `combineReducers` function.

```js
import { combineReducers } from "redux";
import { nameReducer, attrsReducer } from "./reducers";

const rootReducer = combineReducers({
  name: nameReducer,
  attrs: attrsReducer
});
```

## Store

A store is created by passing it a reducer function. It can also take an optional initialState value and an enhancer function.

### `createStore(reducer, initialState, enhancer)`

`createStore` creates your store. When the store's dispatch method is called, the action is passed to the reducer to update the state. The `initialState` is an optional way to give start values for the state. The enhancer is a function that is used in the creation of the store, for example when applying middleware.


```js
import { createStore } from "redux";

const store = createStore(reducer);
```

### `store.subscribe(callback)`

`subscribe` takes a callback function and appends it to a list of listeners. Whenever the store is updated (as a result of a dispatch), all of the subscribed callback functions are called. To get the current state, call `store.getState()` in the `store.subscribe` callback.


```js
store.subscribe(() => {
  var state = store.getState();
  // ...
});

`store.subscribe` returns a function that can be called to unsubscribe from listening to the store.
  
```js
const unsub = store.subscribe(() => { ... });
// listening for updates
unsub();
// no longer listening for updates
```

### `store.dispatch(action)`

`dispatch` takes an action object and updates the store. The dispatch function verifies that the action is legitimate (is an object with a "type") and calls the store's reducer function (the one used in `createStore`). The arguments passed to the reducer are the current state of the store and the dispatched action. The result returned by the reducer is set as the new state of the store and any subscribed callbacks are then called.

```js
store.dispatch({
  type: "SET_NAME",
  name: "Wilma"
});
```

## Enhancers

Enhancers "enhance" the Redux store by adding extra functionality to the store when you call createStore.


Enhancers are created by composing functions together and passing composed function the createStore method. Read a brief explantion of function composition [here](https://github.com/pshrmn/notes/tree/master/functional-programming/compose.md).


### Middleware

Middleware enhances the store by enabling you to interact with dispatched actions before they reach the store. To do this, the store's dispatch function is modified to run the middleware functions before the dispatch function is called.

#### Creating a store with middleware

```js
import { applyMiddleware } from "redux";

/*
 * The functional way of enhancing a store with middleware
 */
const store = applyMiddleware(
  middlewareOne,
  middlewareTwo
)(createStore)(reducers, initialState);

/*
 * However, this is fairly awkward JavaScript code, so
 * you can instead pass the enhancer function returned
 * by applyMiddleware directy to createStore
 */

const store = createStore(
  reducers,
  initialState,
  applyMiddleware(
    middlewareOne,
    middlewareTwo
  )
);
```

## Example

```js
// Action Types: "ADD", "SUBTRACT", and "MULTIPLY"

function reducer(state, action) {
  switch(action.type) {
  case "ADD":
    return state + action.value;
  case "SUBTRACT":
    return state - action.value;
  case "MULTIPLY":
    return state * action.value;
  default:
    return state;
  }
}

var store = Redux.createStore(reducer, 1);
// store = 1
store.dispatch({
  type: "ADD",
  value: 4
});
// store = 5
store.dispatch({
  type: "MULTIPLY",
  value: 3
});
// store = 15
store.dispatch({
  type: "SUBTRACT",
  value: 5
});
// store = 10

// for unknown types, the current state is returned
store.dispatch({
  type: "UNKNOWN"
});
// store = 10
```
