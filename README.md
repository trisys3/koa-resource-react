# koa-resource-react
## Experimental reactive koa middleware resembling JSX

> NOTE: This middleware is very experimental. It is meant to be an inspiration
> for myself & others. If something else comes out of it, whether by me or
> someone else, great! If not, the experience will (hopefully) have been a
> good one. That being said, the middleware is still meant to work, so if you
> find something wrong with it, please [open an issue][issues] and I will try
> to fix it or guide you on how to fix it.

### Install:

```bash
npm install koa-resource-react
```

## Create the resource

```javascript
import {Resource} from 'koa-resource-react';

export function MyResource() {
  return <Resource url='/my-resource' />;
}
```

## Create your server

```javascript
import Koa from 'koa';
import render from 'koa-resource-react';

const server = new Koa('My Resource Server');

const middleware = render(MyResource);

server.use(middleware);
```

## Full API

### Server JSX Attributes

#### url

Creates a simple proxy to `url`, sending the same body and/or query string as the incoming request. Each method can be overridden by prepending the method, e.g. `getUrl`, `postUrl`, etc. These can also be used instead of `url`, for example if `getUrl` and `deleteUrl` are given, only `GET and DELETE will be proxied. A value of `null` or `undefined` cancels the proxy. If given a function, the return value will be used. Any other variable type is invalid.

#### action

Callback upon getting a request. As with `url`, can be overridden per method, e.g. `getAction`, and can be a memo-ized function, as in a function that returns a function. You can also use a method action like `getAction` with no standalone `action`. `null` or `undefined` cancels the action.

Both `url` and `action` can be used, which is useful if the proxied URL is on a logging server, or if it is another `Resource` server.`

[issues]: https://github.com/trisys3/koa-resource-react/issues
