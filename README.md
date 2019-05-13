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
const server = new Koa();
```

[issues]: https://github.com/trisys3/koa-resource-react/issues
