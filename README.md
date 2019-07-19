# koa-resource-react
## Experimental reactive koa middleware resembling JSX

> NOTE: This middleware is very experimental. It is meant to be an inspiration
> for myself & others. If something else comes out of it, whether by me or
> someone else, great! If not, the experience will (hopefully) have been a
> good one. That being said, the middleware is still meant to work, so if you
> find something wrong with it, please [open an issue][issues] and I will try
> to fix it or guide you on how to fix it.

Middleware for koa especially suited for IoT setups, or extremely complicated APIs, but flexible enough to be useful in any server or anywhere you use JavaScript.

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

Usually, you add event listeners, as props, to add the actions, and then call them with something like `refs`. You generally only need listener props on the server, as the actions get called when the server gets a request with a certain method, using the table below. You may need event listeners, refs, or both on the client.

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
| DELETE | create |
-------------------

## Full API

### JSX Attributes

#### url

Creates a simple proxy to `url`, sending the same body and/or query string as the incoming request. Each method can be overridden by prepending the method, e.g. `getUrl`, `postUrl`, etc. These can also be used instead of `url`, for example if `getUrl` and `deleteUrl` are given, but not `url`, only `GET and DELETE will be proxied. A value of `null` or `undefined` cancels the proxy. If given a function, the return value will be used.

#### action

Callback upon getting a request. As with `url`, can be overridden per method, e.g. `getAction`, and can be a memo-ized function, as in a function that returns a function. You can also use a method action like `getAction` with no standalone `action`. `null` or `undefined` cancels the action.

Both `url` and `action` can be used on the same element without conflicting with each other, which is useful if the proxied URL is on a logging server, or if it is another `Resource` server.`

[issues]: https://github.com/trisys3/koa-resource-react/issues
