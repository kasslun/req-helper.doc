# Req Helper

The Req Helper  is an ajax, fetch and other interface request help library. It provides some functions to reduce the number of HTTP requests. Proxy functions are used to control the sending frequency of requests and cache responses.

It is applicable to client-side JavaScript (such as Browser, React Native, Electron) and server-side Node environment.

## Install and usage

Install via npm or yarn.
```shell
npm install req-helper
```
or
```shell
yarn add req-helper
```

It supports es module, commonJs module and AMD module loading and running.
```js
// es module
import { polling } from 'req-helper'
```

```js
// commonJs module
const { polling } = require('req-helper')
```
```js
// AMD module
require('req-helper', ({ polling }) => { 
  // Do something
})
```

## Function description
- [cache](#cache): The response to a request is cached in memory for a period of time.
- [deResend](#deResend): Prevent duplicate requests.
- [latest](#latest): Control frequent queries in a short time through cache and concurrency restrictions.
- [packing](#packing): Merge frequently requested data and use the requested data for batch interface.
- [polling](#polling): Polling for the same request.


## Some precautions

#### About parameter fn
Most functions of Req Helper need to take a function as a parameter. This function needs to send an ajax request and return a promise object to express the status of the request. 

We can handle ajax through promise at any time, such as [Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) and [Axios](https://axios-http.com/). You can also use `XMLHttpRequest` with [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).


#### About promise status of fn returned
The Req Helper needs to get this state accurately to ensure that the program can run normally.
- `pending`: The client has not received a response.
- `fulfilled`: The client received the correct response.
- `rejected`: Request error or the client received an incorrect response.

The following code takes the `cache()` of Req Helper as an example
```js
// Baaaaaaad!
// The Req Helper Unable to know the request error
cache(() => {
  return fetch(url).then(() => {
    // Do something
  }).catch(() => {
    // Do other thing
  })
})

// Gooooood!
cache(() => fetch(url)).then(() => {
  // Do something
}).catch(() => {
  // Do other thing
})

```

#### About optional parameters
Optional parameters can be passed to `undefined` or `null`.
```js
cache(fn)
// Equals to
cache(fn, null)
```

#### About time 
Most functions of Req Helper involve time parameters, such as interval time and cache time. All times are in milliseconds. Some parameters allow 0, and the score parameter must be greater than 0

Some browsers (such as chrome) have a phenomenon: The value of `setTimeout` parameter 2 is `0`, which is actually 1 ms. Moreover, `setTimeout` is nested in multiple, and the time is more unpredictable.
```js
// In chrome
let now = Date.now()
setTimeout(() => console.log('parameter set 1:' + (Date.now() - now) + 'ms'), 1)
setTimeout(() => console.log('parameter set 0:' + (Date.now() - now) + 'ms'), 0)
// Print
// parameter set 1:1ms
// parameter set 0:1ms
```
The processing of timer time by Req Helper is as follows:
```js
const setDelay = (cb, time) => {
  if (time === 0) {
    Promise.resolve().then(fn)
  } else {
    setTimeout(fn, time)
  }
}
```

#### About proxy function
Most functions of req helper will be passed into a function `fn` to produce a proxy function `proxyFn`. The functions of the proxy function are as follows:
- Call parameter function with control `fn`.
- In general, pass the parameter to `fn`.
- In general, return the result of `fn`.
- Bind this for `fn`.

```js
const proxyFn = (function (fn) {
  return function (...args) {
    // Some control 
    return fn.call(this, ...args)
  }
})(fn);
```
 