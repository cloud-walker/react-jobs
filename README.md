## ___WIP___

This library is in a complete state of flux at the moment.  I haven't done any releases, not even an alpha.  I am evolving the API through a test process.  This message will disappear after it settles enough for an alpha release.  Until then I wouldn't pay too much attention to this library. :)

---

# react-jobs 🕴

Attach "jobs" to your components, with SSR support.

[![npm](https://img.shields.io/npm/v/react-jobs.svg?style=flat-square)](http://npm.im/react-jobs)
[![MIT License](https://img.shields.io/npm/l/react-jobs.svg?style=flat-square)](http://opensource.org/licenses/MIT)
[![Travis](https://img.shields.io/travis/ctrlplusb/react-jobs.svg?style=flat-square)](https://travis-ci.org/ctrlplusb/react-jobs)
[![Codecov](https://img.shields.io/codecov/c/github/ctrlplusb/react-jobs.svg?style=flat-square)](https://codecov.io/github/ctrlplusb/react-jobs)

## TOCs

  - [Introduction](#introduction)
  - [Installation](#installation)
  - [Usage](#usage)
  - [Server Side Rendering Usage](#server-side-rendering-usage)
  - [API](#api)
    - [job](#job)
    - [ClientProvider](#clientprovider)
    - [ServerProvider](#serverprovider)
    - [createRenderContext](#createRenderContext)
    - [runJobs](#runJobs)
  - [FAQs](#faqs)
  - [Credits](#credits)

## Introduction

This library provides you with a generic mechanism of attaching "jobs" to your React Components, and includes support for server rendering (i.e. within SSR/universal/isomorphic apps).

You can use these "jobs" to do any of the following:
 - Resolve additional data for your components in a sync/async manner.
 - ?

It provides you with a simple `function` and `Promise` based API which allows you to easily compose additional features such as data caching or 3rd party integrations (e.g. Redux).

## Installation

__npm__

```bash
npm i react-jobs -S
```

__yarn__

```bash
yarn add react-jobs
```

## Usage

Below we will detail how to use this library for the typical _browser_-only use.

In the naive example below we will use the `fetch` API (you'll need to [polyfill it](https://github.com/github/fetch) for older browsers) to retrieve data from an API endpoint.

```js
import React from 'react';
import { job } from 'react-jobs';
import Product from './components/Product';

function Products({ job, categoryID }) {
  // We provide a "job" prop containing the status/results of
  // the work you attached to your component.

  if (job.inProgress) {
    // If the work is still being processed (i.e. the Promise
    // is not yet resolved) then the `job.inProgress` boolean flag
    // value will be set to `true`.
    return <div>Retrieving...</div>;
  }

  if (job.error) {
    // If an error occurred whilst trying to execute your work
    // then the `job.error` will contain the error.
    return <div>Oh noes!</div>;
  }

  // Yay! We have the results!
  return (
    <div>
      {
        job.results.map(product =>
          <Product key={product.id} item={product} />
        )}
    </div>
  );
}

export default job(
  // Provide a function defining the work that needs to be done.
  // This function will be provided the props passed to your
  // component and must return a Promise for any asynchronous work.
  function work(props) {
    // Fetch the products for the given `categoryID` prop,
    return fetch(`/products/category/${props.categoryID}`)
      // then convert the response to JSON.
      .then(response => response.json());
  }
)(Products);
```

This component can then be used like so:

```jsx
<Products categoryID={1} />
```

## Server Side Rendering Usage

Below we will detail how to use this library for a _server side rendering_ application.

We have created an "ssr" import point so that the server side rendering related modules of this library will only be bundled into your source when used.

__Shared Code__

Let's say you had the following React application component you want to render on the server/client:

```jsx
import React from 'react';
// Note! We import from '/ssr'. This provides us with an extended version
// of the job helper with SSR specific behaviours.
import { job } from 'react-jobs/ssr';

function MyApp({ job }) {
  if (job.result) {
    return <div>{job.result}</div>;
  }
  return null;
}

export default job(() => fetch('/stuff').then(r => r.json()))(MyApp);
```

__Server__

```js
import React from 'react';
import { renderToString } from 'react-dom/server';
import { runJobs } from 'react-jobs/ssr';
import serialize from 'serialize-javascript';
import MyApp from './shared/components/MyApp';

export default function expressMiddleware(req, res, next) {
  runJobs(<MyApp />).then(({ app, state, STATE_IDENTIFIER }) => {
    const appString = renderToString(app);
    const html = `
      <html>
        <head>
          <title>Example</title>
        </head>
        <body>
          <div id="app">${appString}</div>
          <script type="text/javascript">
            window.${STATE_IDENTIFIER} = ${serialize(state)}
          </script>
        </body>
      </html>`;
    res.send(html);
  });
}
```

__Client__

```js
import React from 'react';
import { render } from 'react-dom';
import { rehydrateJobs } from 'react-jobs/ssr';
import MyApp from './shared/components/MyApp';

rehydrateJobs(<MyApp />).then(({ app }) => {
  render(app, document.getElementById('app'));
});
```

## API

### `jobs`

TODO

### `runJobs`

TODO

### `rehydrateJobs`

TODO

## FAQs

> Let me know your questions...

## Credits

Inspiration has been taken from the following amazing projects:

  - [`react-apollo`](https://github.com/apollostack/react-apollo)
