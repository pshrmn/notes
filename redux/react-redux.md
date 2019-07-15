# React Redux

 Source code on [GitHub](https://github.com/reactjs/react-redux)

* [Overview](#overview)
* [Installation](#installation)
* [React](#react)
  * [Smart Components](#smart-components)
  * [Dumb Components](#dumb-components)
* [React Redux](#react-redux)
  * [`Provider`](#provider)
  * [`connect`](#connect)
  * [Dispatching Actions](#dispatching-actions)
* [Example](#example)

## Overview

React Redux provides a way to use the Redux store with a React app. Select components are wrapped using React Redux to create smart components, which are provided props that correspond to the state of the store as well as a dispatch prop to send actions to the store.

## Installation

```bash
npm install --save react-redux
```

## React

When using Redux with React, your app should have two types of components, smart and dumb. Smart components are aware of Redux and dumb ones are not.

### Smart Components

Smart components are connected to the Redux store. They have access to the state of the store and are able to dispatch actions to the store.

```jsx
<SmartComponent onClick={() => this.props.dispatch({type: "LOG_CLICK"})}>
  {/* child elements */}
</SmartComponent>
```

### Dumb Components

Dumb components are regular React components. Any values from the store and any action dispatching functions have to be passed down to them as props.


```jsx
<DumbComponent onClick={this.props.handleClick}>
  {this.props.text}
</DumbComponent>
```

## React Redux

There are two main components that react-redux uses to connect your components to the redux store. They are a Provider component and a connect function which returns a component.


### `Provider`

```jsx
<Provider store={store}>
  {/* ... */}
</Provider>
```

The Provider is the root of your components and is reponsible for giving your other components access to the redux store. With that in mind, the Provider requires a store prop be passed to it. The Provider uses the context to make the store available to other components.

### `connect`

```js
class Component extends React.Component {...}

export default connect(
  mapStateToProps,
  mapDispatchToProps,
  mergeProps,
  options
)(Component);
```

`connect` connects your components to the redux store. It has four optional arguments and returns another function. The returned function takes a component that you want to give access to (specific parts of) the redux store.

#### `mapStateToProps`

`mapStateToProps` takes the current state of your store returns an object. The object maps store values to values that will be available as props in your component.

```js
connect(
  store => {
    return {
      name: store.name,
      url: store.url
    };
  }
)(Component);
```

#### `mapDispatchToProps`

`mapDispatchToProps` gives you a shortcut to dispatching actions from your component to the store. This can either be an object or a function that returns an object. When this is provided to the connect function, the wrapped component won't be provided with a dispatch prop.

```js
// your actions
import * as actions from "../actions";

// as an object
connect(
  null,
  {
    foo: actions.foo,
    bar: actions.bar
  }
)(Component);

// as a function
import { bindActionCreators } from "redux";

connect(
  null,
  dispatch => bindActionCreators(actions, dispatch)
  }
)(Component);
```

#### `mergeProps`

`mergeProps` takes the object returned by `mapStateToProps`, the object returned by `mapDispatchToProps`, and the `props` objected passed to your component, and lets you reconcile values to a final object which will be passed to your component as its props.

#### `options`

`options` gives you an opportunity to affect how connect works. The `pure` property (default `true`) adds a shallow comparison to `shouldComponentUpdate` to prevent unnecessary re-renders. The `withRef` property (default `false`) adds a `ref` to the component.

### Dispatching Actions

In order to update the store, you need to dispatch an action to it. Typically you will create functions that return an action object with a type and any other necessary properties. You then need to dispatch the result of those functions to the store.

Smart components are given the `dispatch` prop which allows them to send these action objects to the Redux store.

```js
// the action function
function addNumber(number) {
  return {
    type: "ADD_NUMBER",
    number: number
  }
}

class SmartComponent extends React.Component {
  handeEvent(event){
    const number = parseInt(event.target.value, 10);
    const action = addNumber(number);
    this.props.dispatch(action);
  }

  render() {
    // ...
  }
}
```

#### `bindActionCreators`

Redux provides a shortcut function called `bindActionCreators` to simplify dispatching actions. `bindActionCreators` maps action functions to an object using the names of the action functions. These functions automatically dispatch the action to the store when the function is called. This is especially useful for passing action functions to dumb components.

```jsx
// from "actions" module
const ActionFns = {
  addNumber: function(number) {
  return {
    type: "ADD_NUMBER",
    number: number
  }
};

class SmartComponent extends React.Component {
  constructor(props) {
    super(props);
    this.actions = bindActionCreators(ActionFns, props.dispatch);
  }
    
  render() {
    return (
      <SmartComponent>
        <DumbComponent add={this.actions.addNumber} />
      </SmartComponent>
    );
  }
}

class DumbComponent extend React.Component {
  handleEvent(event) {
    const number = parseInt(event.target.value, 10);
    // 
    this.props.add(number);
  }

  render() { /*... */}
}
```

## Example
  
### File Layout

```
  actions         // functions that return actions
  +-- index.js
  |
  components      // dumb components
  +-- parent.js
  +-- child.js
  |
  constants       // reference values, such as the types of actions 
  +-- ActionTypes.js
  |
  containers      // smart components
  +-- app.js
  |
  reducers        // reducers for the store
  +-- index.js
```

### Code

```js
// constants/actionTypes.js
export const ADD_ONE = "ADD_ONE";
export const SUBTRACT_ONE = "SUBTRACT_ONE";
export const DOUBLE_VALUE = "DOUBLE_VALUE";
```

```js
// actions/index.js
import * as types from "../constants/actionTypes";

export function addOne() {
  return {
    type: types.ADD_ONE
  };
}

export function subtractOne() {
  return {
    type: types.SUBTRACT_ONE
  };
}

export function doubleValue() {
  return {
    type: types.DOUBLE_VALUE
  };
}
```

```js
// reducers/index.js
import * as types from "../constants/actionTypes";

export default function(state, action) {
  switch (action.type) {
  case types.ADD_ONE:
    return state + 1;
  case types.SUBTRACT_ONE:
    return state - 1;
  case types.DOUBLE_VALUE:
    return state * 2;
  default:
    return state;
  }
}
```

```js
// containers/app.js
import React from "react";
import { bindActionCreators } from "redux";
import { connect } from "react-redux";
import * as MathActions from "../actions";
import Calculator from "../components/calculator"

class App extends React.Component {
  constructor(props) {
    super(props);
    // bind the action creators to automatically dispatch the actions
    this.actions = bindActionCreators(MathActions, props.dispatch);
  }
  
  render() {
    const { value } = this.props;
    return (
      <Calculator actions={this.actions} value={value} />
    );
  }
}

export default connect(
  state => ({ value: state}),
)(App);
```

```jsx
// components/calculator.js
import React from "react";

export default class Calculator extends React.Component {
  render() {
    const { actions } = this.props;
    return (
      <div className="calculator">
        <p>{this.props.value}</p>
        <button onClick={actions.addOne}>+1</button>
        <button onClick={actions.subtractOne}>-1</button>
        <button onClick={actions.doubleValue}>{String.fromCharCode(215)}2</button>
      </div>
    );
  }
}
```

```jsx
// index.js
import React from "react";
import { createStore } from "redux";
import { Provider } from "react-redux";
import App from "./containers/app";
import reducer from './reducers';

const store = createStore(reducer, 1);

React.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById("content")
);
```

The above files can be bundled using webpack

```js
// webpack.config.js
module.exports = {
  context: __dirname + "/",
  entry: "./index.js",
  resolve: {
  extensions: ["", ".js", ".jsx"]
  },
  externals: {
  "react": "React"
  },
  output: {
  path: __dirname + "/public/static/js/",
  filename: "bundle.js",
  },
  module: {
  loaders: [
   {
    test: /\.jsx?$/,
    exclude: /node_modules/,
    loader: "babel-loader"
    }
  ]
  }
}
```

```bash
# .babelrc
{
  "presets": ["es2015", "react"]
}
```
