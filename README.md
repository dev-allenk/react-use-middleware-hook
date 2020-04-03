# react-use-middleware-hook

[![](https://img.shields.io/npm/v/react-use-middleware-hook)](https://www.npmjs.com/package/react-use-middleware-hook)

An enhanced version of react useReducer hook for handling asynchronous logics with your own middlewares.  
Middleware is a function that takes an action from dispatch, process some sync/async logics, and passes the action to reducer.  
Be aware that it is not compatible with redux-middlewares.

## Demo

- [code sandbox](https://codesandbox.io/s/usemiddleware-gjv7v)

## Getting started

### Installation

```
$ npm install react-use-middleware-hook
$ yarn add react-use-middleware-hook
```

### Usage

#### Syntax

```jsx
const [state, dispatch] = useMiddleware(reducer, initialState, middleware);
```

- reducer
  - Type: `Function`
  - A pure function of type `(state, action) => newState`
- initialState(optional)
  - Type: `any`
- middleware(optional)
  - Type: `Function` or `Object` or `Array`
  - A function or a collection of functions of type `payload => newPayload`
  - Name of functions must be same as action types. Check below examples for detail.

> When dispatching an action, it MUST contains _type_ property.
> Also, action may include _payload_ property, if any value needs to be passed to middlewares or a reducer.

#### Example

React useMiddleware hook provides two ways to use it.

1. with middleware
2. without middleware

Although both ways work fine, using middleware is recommended in the point of SOC(separation of concerns).

### With middleware

With middleware, useMiddleware hook accepts a middleware or collection of middlewares as 3rd parameter.  
Middlewares may contain asynchronous logics.

```jsx
//App.jsx
import useMiddleware from "react-use-middleware-hook";
import { loading, getPost, end } from "./actions";
import reducer from "./reducer";
import middleware from "./middleware";

export default function App() {
  const [state, dispatch] = useMiddleware(reducer, { data: {} }, middleware);

  const getPost = async postId => {
    dispatch(loading());
    await dispatch(getPost(postId));
    dispatch(end());
  };
  return; /* some elements */
}
```

```jsx
//actions.js
export const loading = () => ({ type: "loading" });
export const end = () => ({ type: "end" });
export const getPost = postId => ({ type: "getPost", payload: postId });
export const fetchError = err => ({ type: "fetchError", payload: err });
```

```jsx
//middleware.js
import * as actions from "./actions";

//middleware function accepts a payload which dispatched from an action,
//and returns a new payload or triggers a different action.
const getPost = async postId => {
  try {
    const res = await fetch(
      `https://jsonplaceholder.typicode.com/posts/${postId}`
    );
    const data = await res.json();
    return data;
  } catch (err) {
    return actions.fetchError(err);
  }
};
export default { getPost }; //exports an object which contains middleware functions in it.
```

```jsx
//reducer.js
const reducer = (state, { type, payload }) => {
  switch (type) {
    case "loading":
      return { ...state, status: "loading" };
    case "end":
      return { ...state, status: "end" };
    case "getPost":
      return { ...state, data: payload };
    case "fetchError":
      return { ...state, error: payload };
    default:
      return state;
  }
};
export default reducer;
```

### Without middleware

Without middleware, leave the 3rd parameter empty.
In this case, actions may contain asynchronous logics.

```jsx
//App.jsx
import useMiddleware from "react-use-middleware-hook";
import { loading, getPost, end } from "./action";
import reducer from "./reducer";

export default function App() {
  const [state, dispatch] = useMiddleware(reducer, { data: {} });

  const getPost = async postId => {
    dispatch(loading());
    await dispatch(getPost(postId));
    dispatch(end());
  };
  return; /* some elements */
}
```

```jsx
//actions.js
export const loading = () => ({ type: "loading" });
export const end = () => ({ type: "end" });

export const getPost = async postId => {
  try {
    const res = await fetch(
      `https://jsonplaceholder.typicode.com/posts/${postId}`
    );
    const data = await res.json();
    return { type: "getPost", payload: data };
  } catch (err) {
    return fetchError(err);
  }
};

export const fetchError = err => ({ type: "fetchError", payload: err });
```

```jsx
//reducer.js
const reducer = (state, { type, payload }) => {
  switch (type) {
    case "loading":
      return { ...state, status: "loading" };
    case "end":
      return { ...state, status: "end" };
    case "getPost":
      return { ...state, data: payload };
    case "fetchError":
      return { ...state, error: payload };
    default:
      return state;
  }
};
export default reducer;
```

[⬆ back to top](#react-use-middleware-hook)
