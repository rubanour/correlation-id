[![Build Status](https://travis-ci.org/toboid/correlation-id.svg?branch=master)](https://travis-ci.org/toboid/correlation-id)
[![Coverage Status](https://coveralls.io/repos/github/toboid/correlation-id/badge.svg?branch=master)](https://coveralls.io/github/toboid/correlation-id?branch=master)
[![Dependencies](https://david-dm.org/toboid/correlation-id.svg)](https://github.com/toboid/correlation-id/blob/master/package.json)
[![npm version](https://badge.fury.io/js/correlation-id.svg)](https://badge.fury.io/js/correlation-id)
[![Greenkeeper badge](https://badges.greenkeeper.io/toboid/correlation-id.svg)](https://greenkeeper.io/)

# Correlation id
Correlation id maintains a consistent id across asynchronous calls in node.js applications.
This is extremely useful for logging purposes. For example within a web application, each incoming request can be assigned an id that will be available in all function calls made processing that request, so we can see which requests caused errors.

## Installation
```shell
npm i correlation-id --save
```
## Simple example
As demonstrated by this example, all calls to `getId()` within the same `withId()` block will return the same id. The id can be supplied, otherwise a v4 uuid will be generated.
``` javascript
const correlator = require('correlation-id');

function printCurrentId (name) {
  console.log('%s id: %s', name, correlator.getId());
}

correlator.withId(() => {
  setTimeout(() => {
    printCurrentId('withId block 1, call 1');
  });
  setTimeout(() => {
    printCurrentId('withId block 1, call 2');
  }, 1000);
});

correlator.withId('my-custom-id', () => {
  setTimeout(() => {
    printCurrentId('withId block 2, call 1');
  }, 500);
});

// Output:
// withId block 1, call 1 id: 5816e2d3-6b90-43be-8738-f6e1b2654f39
// withId block 2, call 1 id: my-custom-id
// withId block 1, call 2 id: 5816e2d3-6b90-43be-8738-f6e1b2654f39
```

## API
### `withId([id,] work)`
Executes function `work` within a correlation scope. Within work and any other function executions (sync or async) calls to `getId()` will return the same id. The id for the context may be set explicitly with the optional `id` parameter, otherwise it will be a v4 uuid. Calls to `withId()` may be nested.

```javascript
correlator.withId(() => {
  console.log(correlator.getId()); // Writes a uuid to stdout
});
correlator.withId('my-custom-id', () => {
  console.log(correlator.getId()); // Writes 'my-custom-id' to stdout
});
```

### `bindId([id,] work)`
Returns function `work` bound with a correlation scope. When `work` is executed all calls to `getId()` will return the same id. The id for the context may be set explicitly with the optional `id` parameter, otherwise it will be a v4 uuid. Arguments passed to the bound function will be applied to `work`.

```javascript
const boundFunction = correlator.bindId((p1) => {
  console.log('p1 is', p1);
  console.log(correlator.getId());
});
boundFunction('foo'); // Writes 'p1 is foo' and then a uuid to stdout

const boundFunction2 = correlator.bindId('my-custom-id', (p1) => {
  console.log('p1 is', p1);
  console.log(correlator.getId());
});
boundFunction('foo'); // Writes 'p1 is foo' and then 'my-custom-id' to stdout
```

### `getId()`
Returns the id for the current correlation scope (created via `withId` or `bindId`). If called outside of a correlation scope returns `undefined`.

```javascript
correlator.getId(); // Returns the current id or undefined
```

## Limitations
Async/await is not yet supported.

## How does it work?
Currently this module a slim wrapper over [continuation-local-storage](https://github.com/othiym23/node-continuation-local-storage). I intend to move to async-hook when it's generally available.

## License
MIT