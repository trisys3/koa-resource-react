# koa-resource-react
## Experimental object-oriented middleware for koa resembling JSX

> NOTE: This middleware is very experimental. It is meant to be an inspiration
> for myself & others. If something else comes out of it, whether by me or
> someone else, great! If not, the experience will (hopefully) have been a
> good one. That being said, the middleware is still meant to work, so if you
> find something wrong with it, please [open an issue][issues] and I will try
> to fix it or guide you on how to fix it.

Middleware for koa especially suited for IoT setups, or extremely complicated APIs, but flexible enough to be useful in any server or anywhere you use JavaScript.

As much as possible, the client and server functionality are exactly the same. "Client" here usually means a browser or `electron` application, and "server" means a node server. The heuristics aren't foolproof, and we may look into adding support for forcing one or the other, though sone functionality may still not be possible for either or both cases depending on the environment.

### Install:

```bash
npm install koa-resource-react
```

## Create the resource

```javascript
// resource.js
import {useRef} from 'react';
import React from 'koa-resource-react';

import {Resource} from 'koa-resource-react';

const MyResource = forwardRef(({onChange = () => {}}, ref) => {
    useImperativeHandle(ref, () => ({
      change: () => {
        console.log('Changing the resource...');
        resource.set({some: 'prop'});
      },
    }));

  return <Resource ref={resource} />;
});

MyResource.propTypes = {onChange: PropTypes.func};

export default MyResource;
```

## Create the server

```javascript
// server.js
import Koa from 'koa';
import React, {render} from 'koa-resource-react';
import MyResource from './resource';

const server = new Koa('My Resource Server');

const myResource = <MyResource onChange={onChange} />;

const middleware = render(myResource);

server.use(middleware);

function onChange() {
  console.log('Everything just changed.');
}
```

## Create the client

```javascript
// client.js
const {body} = document;
const elem = body.querySelector('main');

// you can also import from 'koa-resource-react' like on the server
import React, {useRef} from 'react';
import {render} from 'react-dom';
import MyResource from './resource';

render(<App />, elem);

function App() {
const resource = useRef();

  useEffect(() => {
    console.log('Changing everything...');
    resource.change();
  });

  return <MyResource ref={resource} />;
}
```

## Actions

Actions are the guts of `koa-resource-react`. They control how each resource works, and what happens when certain things change. They use the pattern of CRUD, Create-Read-Update-Delete.

With syncing, these functions are automatically called whenever the props change. You can also add `onAction` or `onGet`, `onSet`, etc. as event listener props to add the actions, and then call them with something like `refs`. You generally only need listener props on the server, as the actions get called when the server gets a request with a certain method, using the table below. You may need event listeners, refs, or both on the client.

### Action <=> Method

This is a table of which action is called on the server when it gets a request with the corresponding method:

-------------------
| Method | Action |
|--------|--------|
|  GET   |  get   |
|--------|--------|
|  POST  |  set   |
|--------|--------|
|  PUT   |  add   |
|--------|--------|
| DELETE | delete |
-------------------

## Full API

### JSX Props

#### url

Adds a commection to `url`. On the server, this sends the same body and/or query string as the incoming request. On the client, all current properties are added to the request. Both respect the whitelist specified by `props`, if any. Each action can be overridden by prepending the method, e.g. `getUrl`, `setUrl`, etc. These can also be used instead of `url`, for example if `getUrl` and `deleteUrl` are given, but not `url`, only `get` and `delete` will be available for syncing. A value of `null` or `undefined` cancels the proxy. If given a function, the return value will be used as the URL.

#### onAction

Callback upon getting a request. Can be overridden per action, e.g. `onGet` or `onSet`, and can be a memo-ized function, as in a function that returns a function. You can also use a method action like `onGet` with no standalone `action`. `null` or `undefined` cancels the action.

If `url` and `sync` are both passed, a separate action listener is defined, sending all the props as a request to the said `url`, respecting the `props` whitelist.

#### props

Array of properties to retrieve or set, for example when syncing when incoming props change or when using an `on<action>` ref. No other props will be retrieved or set. If not passed, all props will be available for getting & setting, minus props defined by the component like `action` and `url`.

#### sync

Keep the props on this component in sync with the ones on the server given by `url`. As with almost everything else, available on both the client and server, though typically only useful on the client.

Can also be a number, for the number of milliseconds to wait before syncing. If props change again before the time is up, only the latest change is respected.

### Ref functions

### <action>

Call the action. This is typically only needed on the client, and only if `sync` is not used. If `url` and `sync` are both passed, this function is called internally whenever the props change, though it can still be called manually. The function can be defined on the server, too.

Can be passed a prop and a value, or an object with multiple props & values. Each property & its value is put into the body for `set`s, as these are `POST` requests, and the query string for other actions. If nothing is passed, uses all the properties and values currently passed as props, respecting the `props` whitelist. In this way, you can simulate the `sync` prop with debouncing or other more advanced cases like only syncing at certain times of day. Note that debouncing is also possible by passing a number to the `sync` prop

[issues]: https://github.com/trisys3/koa-resource-react/issues
