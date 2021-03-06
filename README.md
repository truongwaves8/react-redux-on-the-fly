# React Redux on The Fly

> Reducers lazy loading for React

[![NPM](https://img.shields.io/npm/v/react-redux-on-the-fly.svg)](https://www.npmjs.com/package/react-redux-on-the-fly)
[![JavaScript Style Guide](https://img.shields.io/badge/code_style-standard-brightgreen.svg)](https://standardjs.com)
[![Build Status](https://travis-ci.com/MatteoGioioso/react-redux-on-the-fly.svg?branch=master)](https://travis-ci.com/MatteoGioioso/react-redux-on-the-fly)
[![Coverage Status](https://coveralls.io/repos/github/MatteoGioioso/react-redux-on-the-fly/badge.svg?branch=master)](https://coveralls.io/github/MatteoGioioso/react-redux-on-the-fly?branch=master)

- Lazy load dynamic reducers 🚀
- Create reducers and actions with almost zero boiler plate 👷🏻
- Enforce consistency with actions and types naming schema 👮🏻

#### This library and this guide is still under construction, so be gentle 😏

## Why?

You can read more about the motivation of this library on my blog article: https://blog.hirvitek.com/post/redux-advanced-pattern:-dynamic-reducers_bZCV33Fkdvegf4LWwvYe1

## Install

```bash
npm install --save react-redux-on-the-fly
```

## Usage

There are fundamental 2 type of usage: high-level and low-level

### High Level usage

To create your `Root` element containing the `redux` store and `react-redux` provider

```javascript
import { Provider } from "react-redux";
import { createRoot } from "react-redux-on-the-fly";

const Root = createRoot(Provider, staticReducers, [yourMiddleware]);
```

After that simply wrap your `App` component or your routes with the created `Root`
Then into your component you can inject your dynamic reducer:

```jsx
import React, { Component } from "react";
import { customerReducer, customerActions } from "./reducers";
import { connect } from "react-redux";

import {
  withAsyncReducer,
  createNamedWrapperReducer
} from "react-redux-on-the-fly";

class Example extends Component {
  render() {
    return <MyComponent />;
  }
}

const mapStateToProps = (state, props) => omerReducer = createNamedWrapperReducer(customerReducer);

export default withAsyncReducer(
  "customer",
  "customerId",
  wrappedCustomerReducer
)(
  connect(
    mapStateToProps,
    customerActions
  )(Example)
);
```

If you inspect your redux `store` you are going to find a reducer named `customer/<customerId>`.
That is your dynamic "dynamically" injected reducer

### Low Level

If you wish to have more control over your `Root` and `store` you can simply do it your way by just adding the `ReactReduxOnTheFlyProvider`

```javascript
import React from "react";
import { Provider } from "react-redux";
import { createStore, applyMiddleware } from "redux";
import logger from "redux-logger";
import {
  ReactReduxOnTheFly,
  ReactReduxOnTheFlyProvider,
  baseArrayReducer
} from "react-redux-on-the-fly";

const staticReducers = {
  myStaticReducer: baseArrayReducer("STATIC_REDUCER")
  //other static reducers here...
};
const reactReduxOnTheFly = new ReactReduxOnTheFly(staticReducers);
const reducers = reactReduxOnTheFly.asyncCombineReducers();

export default props => {
  const store = createStore(reducers, applyMiddleware(logger));
  store.asyncReducers = {};
  return (
    <Provider store={store}>
      <ReactReduxOnTheFlyProvider
        store={store}
        reactReduxOnTheFly={reactReduxOnTheFly}
      >
        {props.children}
      </ReactReduxOnTheFlyProvider>
    </Provider>
  );
};
```

## Reducer and actions creation

`react-redux-on-the-fly` give you the possibility to create pre-made reducers and sets of actions with two simple methods

- Reducer data structure:
  At the moment there are two types of data structures:
  - Array
  - Single entity (could be any type, but it is not intended to be used with collections)

Array structures contains the classic CRUD methods, while the Single entity contains only the update method as it suppose to contain a single value or object.

### Create a reducer

To create a reducer, import your reducer data structure and give it a `section` name.
The `section` name is going to be root of the actions naming.
Ex:

```javascript
const section = "CUSTOMER";
//action name FIND_CUSTOMERS, FIND_CUSTOMER_BY_ID, RECEIVE_CUSTOMERS, SUBSTITUTE_CUSTOMER, ...
```

```javascript
import { baseArrayReducer } from "react-redux-on-the-fly";

const myReducer = baseArrayReducer(section);
```

### Create actions

Same as the reducer

```javascript
import { createActions } from "react-redux-on-the-fly";

const customerActions = createActions(section);
```

This method will return an object containing actions types and functions, this can and should be used with middleware like `redux-saga` or `redux-observable` which dispatch an action to trigger an API request and then dispatch an action to modify the state, you can check below the list of the methods and their signatures

```javascript
const { actions, types } = customerActions;
```

### Naming convention

- types:

```javascript
FIND: `FIND_${section}S`,
FIND_BY_ID: `FIND_${section}_BY_ID`,
CREATE: `CREATE_${section}`,
UPDATE_BY_ID: `UPDATE_${section}_BY_ID`,
DELETE_BY_ID: `DELETE_${section}_BY_ID`,
RECEIVE_MANY: `RECEIVE_${section}S`,
RECEIVE_MANY_ON_TOP: `RECEIVE_${section}S_ON_TOP`,
RECEIVE_ONE: `RECEIVE_${section}`,
RECEIVE_ONE_ON_TOP: `RECEIVE_${section}_ON_TOP`,
SUBSTITUTE_ONE: `SUBSTITUTE_${section}`,
DESTROY_ONE: `DESTROY_${section}`,
DESTROY_MANY: `DESTROY_${section}S`,
RESET: `RESET_${section}S`,
```

- Actions
  actions follow the same naming conventions as types but in camelCase
  `urlObject`: you can put everything related to the request, id, url, ...depending on your implementation.
  `callback`: typically when a saga terminate you can decide to send a callback, you can also omit it
  `data`: used for post and put request
  If you do not like the signature [you can easily override it](#add-more-reducer-methods). 

```
find<SectionName>s: (urlObject, callback) => ...
find<SectionName>ById: (urlObject, callback) => ...
create<SectionName>: (urlObject, data, callback) => ...
update<SectionName>ById: (urlObject, data, callback) => ...
delete<SectionName>ById: (urlObject, data, callback) => ...
receive<SectionName>s: (data, name) => ...
receive<SectionName>sOnTop: (data, name) => ...
receive<SectionName>: (data, name) => ...
receive<SectionName>OnTop: (data, name) => ...
substitute<SectionName>: (data, name) => ...
destroy<SectionName>: (data, name) => ...
destroy<SectionName>s: (data, name) => ...
reset<SectionName>s: name => ...
```

- ### Reducer methods

You can override each one of the standard reducer methods by importing the `BaseArrayReducerMethods` base class
and create a custom class that inherits from the base class

```javascript
class myCustomMethods extends BaseArrayReducerMethods {
  receiveMany = (state, payload) => {...};

  receiveManyOnTop = (state, payload) => {...};

  substituteOne = (state, payload) => {...};

  destroyOne = (state, payload) => {...};

  destroyMany = (state, payload) => {...};

  resetAll = (state, payload) => {...};
}
```

Then simply pass it as arguments to your reducer creator

```javascript
const reducer = baseArrayReducer("CUSTOMER", null, myCustomMethods);
```

- #### Add more reducer methods
  If you need to add more reducer methods create your custom piece of reducer and pass it as arguments to your reducer creator

```javascript
const overrideReducer = (state, action) => {
   switch (action.type) {
    case "MY_CUSTOM_ACTION":
      return myCustomMethod(state, action.payload);
      //...
};
const reducer = baseArrayReducer("CUSTOMER", null, undefined, overrideReducer);
```

- #### Add custom actions
  If you have the need to add more actions and types, you can easily to that

```javascript
const options = {
  types: {
    CUSTOM_TYPE: "CUSTOM_TYPE"
  },
  actions: {
    myCustomAction: data => createActions("CUSTOMER", options);
```

## Upcoming features

- Map or Dictionary data structure
- Typescript support
- Hooks

## Some other stuff

Code reviews, feature requests and questions are highly welcome. Just open an issue!

---

## License

MIT © [Matteo Gioioso](https://github.com/MatteoGioioso)
