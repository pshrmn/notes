# Redux Middleware

Middleware provides a way to interact with actions that have been dispatched to the store before they reach the store's reducer. Examples of different uses for middleware include logging actions, reporting errors, making asynchronous requests, and dispatching new actions.

* [Using Middleware](#using-middleware)
* [How `applyMiddleware` Works](#how-applymiddleware-works)
* [What does a middleware function look like?](#what-does-a-middleware-function-look-like)
* [Writing your own middleware](#writing-your-own-middleware)
  * [Responding to Actions](#responding-to-actions)
  * [Stopping the Chain](#stopping-the-chain)

## Using Middleware

Middleware is used by composing the functions together and passing that function to your createStore call. The composed middleware function is used to replace the store's default dispatch method with one that dispatches the action to each middleware function in a chain, with the last middleware function dispatching the action to the store.

Read a brief explantion of function composition [here](https://github.com/pshrmn/notes/blob/master/functional-programming/compose.md).


```js
import { applyMiddleware } from "redux";

const store = createStore(
  reducers,
  initialState,
  applyMiddleware(
    middlewareOne,
    middlewareTwo
  )
);
```

## How `applyMiddleware` Works

To understand how middleware functions work, it is useful to look at what applyMiddleware actually does.

The code below is a slightly modified version of the applyMiddleware function in redux's source code with comments explaining what it is doing. Modifications are made to make sure that everything can be commented on clearly.

```js
// applyMiddleware receives a variadic (unspecified/any) number of
// middleware functions, which it stores in an array called middlewares
function applyMiddleware(...middlewares) {

  // applyMiddleware returns a function that takes a createStore
  // function as its argument
  return createStore => {

    // that function in turn returns another function, which takes a reducer,
    // initial state, and an enhancer. This is the same function signature
    // as redux's createStore function.
    return (reducer, initialState, enhancer) => {

      // when that function is called, the first thing it does
      // is create the store by calling the createStore function
      var store = createStore(reducer, initialState, enhancer)

      // a copy of the original store.dispatch function is made
      var dispatch = store.dispatch

      // a pseudo store is made.
      var middlewareAPI = {
        getState: store.getState,
        // calling the pseudo store's dispatch function does a full
        // dispatch using the function created by composing the
        // middleware and passing it the original dispatch
        dispatch: (action) => dispatch(action)
      }
      // each middleware function is given a copy of the pseudo store
      // so that it can dispatch using the middleware-enhanced dispatch
      var chain = middlewares.map(middleware => middleware(middlewareAPI))

      // The middleware functions are composed together to create a
      // composing function. The composing function is called with the original
      // dispatch function as its argument and returns the modified dispatch
      // function. Unlike the modified dispatch function, the store.dispatch
      // function passed to the composing function will dispatch directly to
      // the store, not calling any of the middleware functions.
      dispatch = compose(...chain)(store.dispatch)
      // at this point, calling dispatch(action) is effectively the same as
      // store.dispatch(middlewareN(middlewareN-1(...(2(1(action))))))

      // the store object is returned, with its dispatch method replaced
      // by the middleware enhanced dispatch function.
      return {
        ...store,
        dispatch
      }
    }
  }
}
```

## What does a middleware function look like?

At its most basic, a middleware function is a function that receives an action and interacts with it in some way.

```js
function middleware(action) { /*...*/}
```

Because the middleware works as a chain of functions where each middleware function dispatches the action to the next function in the chain, each middleware function needs a dispatch function. The middleware functions receive these dispatch functions when the composing function is called, with the nth middleware function receiving store.dispatch and each subsequent function receiving the previous middleware function as its dispatch.

```js
function middlewareWrapper(nextDispatch) {
  return function(action) {
    nextDispatch(action);
  }
}
```

And if you remember from above, in `applyMiddleware`, each middleware receives a copy of the pseudo store. This is done with yet another wrapper function.


```js
function storeWrapper(store) {
  function middlewareWrapper(nextDispatch) {
    return function(action) {
      nextDispatch(action);
    }
  }
}
```

Why is it necessary to pass the pseudo store to the middleware?


#### `pseudoStore.getState`

For one, it is useful to be able to access the current state of the store in the middleware. In order to get the current state of the store, your store needs to be in the scope of the middleware function. If it is not included in the creation of the middleware, then the middleware would need access to it through the global scope. To avoid that, redux makes sure that you have access to it through the pseudo store.


#### `pseudoStore.dispatch`

You might want to dispatch new actions from within your middleware and to do this you could call the "nextDispatch" function that the middleware has from your middleware composition. Dispatching an action this way would mean that it would reach the reducer, but it would never be seen by middleware functions in the chain that have already been called.


```js
/* 
 * Imagine that middleware has no pseudoStore, so its only way to dispatch
 * a method is through the dispatch function passed in through its scope. In
 * this example, we will use a middleware that logs all of our actions and
 * another one that dispatches new actions.
 */
import { applyMiddleware, createStore } from "redux";

function dispatcher(nextDispatch) {
  return function(action) {
    nextDispatch({ type: "NEW_ACTION" });
    // and the original
    return nextDispatch(action);
  }
}

function logger(nextDispatch) {
  return function(action) {
    console.log(action);
    return nextDispatch(action);
  }
}

/*
 * We can apply the middleware either with dispatcher first or with 
 * logger first. Without pseudoStore.dispatch, this order is very important
 */

const store_one = createStore(
  reducers,
  initialState,
  applyMiddleware(
    dispatcher,
    logger
  )
);
/*
 * Now we dispatch a new action through our middleware
 */
store_one.dispatch({
  type: "MAKE_DISPATCH"
});
/*
 * The first middleware in the chain to receive this action is the dispatcher.
 * The dispatcher recognizes the action's type and acts by dispatching
 * a new action. The new action is dispatched using the nextDispatch
 * function in its scope, which is the logger middleware.
 * logger receives the action, logs it, and calls its own nextDispatch method,
 * store.dispatch. The dispatcher also dispatches the original action, which
 * the logger also sees and logs. Everything works as expected.
 */

const store_two = createStore(
  reducers,
  initialState,
  applyMiddleware(
    logger,
    dispatcher
  )
);
store_two.dispatch({
  type: "MAKE_DISPATCH"
});
/*
 * Now imagine that we dispatch the same action to store_two.
 */
 /*
 * The logger is the first middleware to receive this action. The logger
 * middleware sees this original action and logs it, then calls its nextDispatch
 * method, which is the dispatcher middleware. The dispatcher middleware sees
 * that it should dispatch a new action and dispatches it using its nextDispatch
 * function, store.dispatch.
 * The store receives the newly dispatched action, but the logger never does.
 */
 ```

So in one case, only being able to dispatch further down the chain is fine, but in the other our logger never sees the newly dispatched action and the action is never logged. We could make sure that this doesn't happen by making sure that the middlewares are in the correct order, but that can become very complex when dealing with a larger number of middlewares and sometimes no ordering would provide a satisfactory result. Instead, by giving the middleware access to the full dispatch function that starts at the beginning of the middleware chain, we can guarantee that when we dispatch an action from inside of our middleware that middleware functions higher up in the chain will see it.

```js
import { applyMiddleware, createStore } from "redux";

/* 
 * Now, with the dispatcher having access to the full store.dispatch,
 * and actions that it dispatches will start at the beginning of the
 * middleware chain.
 */
function dispatchWithStore(store) {
  return function dispatcher(dispatch) {
    return function(action) {
      // dispatch the new action at the beginning of
      // the middleware chain, but only for other action
      // types (we don't want an infinite dispatch loop)
      if (action.type !== "NEW_ACTION") {
        store.dispatch({ type: "NEW_ACTION" });
      }
      // and the original can continue along the chain
      return dispatch(action);
    }
  }
}

function logWithStore(store) {
  return function logger(dispatch) {
    return function(action) {
      console.log(action);
      return dispatch(action);
    }
  }
}
```

## Writing your own middleware

Now we know have the basic layout of a middleware function:

```js
function middleware(store) {
  return function(nextDispatch) {
    return function(action) {
      nextDispatch(action);
    }
  }
}
```

Or using arrow functions:

```js
const middleware = store => nextDispatch => action {
  nextDispatch(action);
}
```

### Responding to Actions

Often, middleware doesn't care about the majority of actions that it sees. In those cases, it should check the type of the action and if it doesn't match one that it cares about, just continue passing the action down the chain.

```js
// this middleware only cares about actions
// where type === "SPECIAL"
function specialMiddleware(store) {
  return function(nextDispatch) {
    return function(action) {
      if ( action.type !== "SPECIAL" ) {
        return nextDispatch(action);
      }
      // react to the action
    }
  }
}
```

### Stopping the Chain

Sometimes an action doesn't need to reach the store's reducer because it doesn't update the store based on that action's type. In those cases, the middleware function does not need to call the `nextDispatch` for that action. This is most often the case when the middleware dispatches a new, but separate, action based on the dispatched action. That new action will instead eventually reach the reducer to update your store.

A good example of this is in the react-router-redux module. Its middleware (`syncHistory`) reacts to `TRANSITION` actions, but the reducer (`routeReducer`) reacts to `UPDATE_LOCATION` actions. Since `routeReducer` has no need for the `TRANSITION` action, and no other reducers have reason to care about the `TRANSITION` action, there is no reason to continue dispatching the `TRANSITION` action down the chain to the reducer.

```js
function middleware(store) {
  unsubscribeHistory = history.listen(location => {
    // a listener is attached to the history. When the
    // history changes locations, an UPDATE_LOCATION
    // action is dispatched to the store
    store.dispatch(updateLocation(location))
  })

  return next => action => {
    // the middleware only listens for TRANSITION actions
    if (action.type !== TRANSITION) {
      return next(action)
    }
    // when there is a TRANSITION action the middleware
    // calls a history function to update the location
    history[action.payload.method](action.payload.arg)
    // the middleware doesn't bother passing the TRANSITION
    // action down the middleware chain, since no reducer
    // will update the state of the store based on it
  }
}
```
