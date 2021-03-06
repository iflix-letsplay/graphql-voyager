# GraphQL Voyager
[![David](https://img.shields.io/david/APIs-guru/graphql-voyager.svg)](https://david-dm.org/APIs-guru/graphql-voyager)
[![David](https://img.shields.io/david/dev/APIs-guru/graphql-voyager.svg)](https://david-dm.org/APIs-guru/graphql-voyager?type=dev)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

![graphql-voyager logo](./docs/cover.png)

Represent any GraphQL API as an interactive graph. It's time to finally see **the graph** behind GraphQL.
You can also explore number of public GraphQL APIs from [our list](https://github.com/APIs-guru/graphql-apis).

> With graphql-voyager you can visually explore your GraphQL API as an interactive graph. This is a great tool when designing or discussing your data model. It includes multiple example GraphQL schemas and also allows you to connect it to your own GraphQL endpoint. What are you waiting for, explore your API!

_[GraphQL Weekly #42](https://graphqlweekly.com/issues/42)_

## [Live Demo](https://apis.guru/graphql-voyager/)
[![voyager demo](./docs/demo-gif.gif)](https://apis.guru/graphql-voyager/)

## Features
  + Quick navigation on graph
  + Left panel which provides more detailed information about every type
  + "Skip Relay" option that simplifies graph by removing Relay wrapper classes
  + Ability to choose any type to be a root of the graph

## Roadmap
  - [x] Major refactoring
  - [ ] Publish as a library ([issue 1](https://github.com/APIs-guru/graphql-voyager/issues/1))
  - [ ] Tests + CI + CD
  - [ ] Try to optimize graph auto-layout
  - [ ] [ < place for your ideas > ](https://github.com/APIs-guru/graphql-voyager/issues/new)

## Usage
GraphQL Voyager exports `Voyager` React component and helper `init` function. If used without
module system it is exported as `GraphQLVoyager` global variable.

### Properties
`Voyager` component accepts the following properties:

+ `introspection` [`object` or function: `(query: string) => Promise`] - the server introspection response. If `function` is provided GraphQL Voyager will pass introspection query as a first function parameter. Function should return `Promise` which resolves to introspection response object.
+ `displayOptions`
  + `displayOptions.skipRelay` [`boolean`, default `true`] - skip relay-related entities
  + `displayOptions.rootType` [`string`] - name of the type to be used as a root
  + `displayOptions.sortByAlphabet` [`boolean`, default `false`] - sort fields on graph by alphabet

### `init` function
The signature of the `init` function:

```js
(hostElement: HTMLElement, options: object) => void
```

+ `hostElement` - parent element
+ `options` - is the JS object with [properties](#Properties) of `Voyager` component

### Using pre-bundled version
You can get GraphQL Voyager bundle from the following places:
+ our GitHub Pages-based CDN
  + some exact version - https://apis.guru/graphql-voyager/releases/v1.0.0-rc.3/voyager.min.js
  + latest 1.x version - https://apis.guru/graphql-voyager/releases/v1.x/voyager.min.js
+ download zip from [releases](https://github.com/APIs-guru/graphql-voyager/releases) page and use files from `dist` folder
+ from `dist` folder of the npm package `graphql-voyager`

**Important:** for the latest two options make sure to copy `voyager.worker.js` to the same
folder as `voyager.min.js`.

The HTML with minimal setup (see the full [example](./example))

```html
<!DOCTYPE html>
<html>
  <head>
    <script src="//cdn.jsdelivr.net/react/15.4.2/react.min.js"></script>
    <script src="//cdn.jsdelivr.net/react/15.4.2/react-dom.min.js"></script>

    <link rel="stylesheet" href="./node_modules/graphql-voyager/dist/voyager.css" />
    <script src="./node_modules/graphql-voyager/dist/voyager.min.js"></script>
  </head>
  <body>
    <div id="voyager">Loading...</div>
    <script>
      function introspectionProvider(introspectionQuery) {
        // ... do a call to server using introspectionQuery provided
        // or just return pre-fetched introspection
      }

      // Render <Voyager />
      GraphQLVoyager.init(document.getElementById('voyager'), {
        introspection: introspectionProvider
      })
    </script>
  </body>
</html>
```

### Using as a dependency
You can install lib using `npm` or `yarn`:

    npm i --save graphql-voyager
    yarn add graphql-voyager

And then use it:

```js
import * as React from 'react';
import * as ReactDOM from 'react-dom';
import {Voyager} from 'graphql-voyager';
import fetch from 'isomorphic-fetch';

function introspectionProvider(query) {
  return fetch(window.location.origin + '/graphql', {
    method: 'post',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({query: query}),
  }).then(response => response.json());
}

ReactDOM.render(<Voyager introspection={introspectionProvider} />, document.getElementById('voyager'));
```

Build for the web with [webpack](https://webpack.js.org/) ([example](./example/webpack-example)) or
[browserify](http://browserify.org/)

**Important:** make sure to copy `voyager.worker.js` from `node_modules/graphql-voyager/dist` to the same folder as your main bundle.

## Middleware
Graphql Voyager has middleware for the next frameworks:

### Express

#### Properties
Express middleware supports the following properties:

+ `options`
  + `endpointUrl` [`string`] - the GraphQL endpoint url.
  + `displayOptions` [`object`] - same as [here](#Properties)

#### Usage
```js
import express from 'express';
import { express as middleware } from 'graphql-voyager/middleware';

const app = express();

app.use('/voyager', middleware({ endpointUrl: '/graphql' }));

app.listen(3000);
```

### Hapi

#### Properties
Hapi middleware supports the following properties:

+ `options`
  + `path` [`string`] - the Voyager middleware url
  + `voyagerOptions`
      + `endpointUrl` [`string`] - the GraphQL endpoint url.
      + `displayOptions` [`object`] - same as [here](#Properties)

#### Usage
```js
import hapi from 'hapi';
import { hapi as middleware } from 'graphql-voyager/middleware';

const server = new Hapi.Server();

server.connection({
  port: 3001
});

server.register({
  register: middleware,
  options: {
    path: '/voyager',
    endpointUrl: '/graphql'
  }
},() => server.start());
```

### Koa

#### Properties
Koa middleware supports the following properties:

+ `options`
  + `endpointUrl` [`string`] - the GraphQL endpoint url.
  + `displayOptions` [`object`] - same as [here](#Properties)

#### Usage
```js
import Koa from 'koa';
import KoaRouter from 'koa-router';
import { koa as middleware } from 'graphql-voyager/middleware';

const app = new Koa();
const router = new KoaRouter();

router.all('/voyager', voyagerMiddleware({
  endpointUrl: '/graphql'
}));

app.use(router.routes());
app.use(router.allowedMethods());
app.listen(3001);
```

## Credits
This tool is inspired by [graphql-visualizer](https://github.com/NathanRSmith/graphql-visualizer) project.
