# Server Timing Explained

## What's All This About?
Server Timing allows developers to monitor the performance of server-handling of requests of every resource that makes up a page by overloading Navigation (for the base HTML) and Resource Timing (for scripts, images, etc). This new API enables developers to have insight into performance bottlenecks of the `Request` block below, that is, after `requestStart` and before `responseStart`.
 
![navigation timing image](./nav-timing.png)

Adhering to a standard response header format gives developer tools the ability to visualize backend timers that the developer (or intermediate proxies) deem important. The Javascript API gives analytics vendors the ability to collect and beacon more detailed timing data about each request. 

### Goals
The goal of this API is the standardization of the following:
* a response-header format
* the relationship between server timing data and other RUM APIs (Navigation and Resource Timing)

### Non-goals
It is _not_ the goal of this API to communicate arbitrary payloads of structured data to the browser. 

It is _not_ the goal of the API to provide a mechanism which relies on clock synchronization (between client, server, or any intermediaries), like providing `startTime`. More [here](https://github.com/w3c/server-timing/issues/10#issuecomment-282442919).

It is _not_ the goal of the API to [distinguish](https://github.com/w3c/server-timing/issues/13) between cached responses and those served by the origin. 

## Getting started
If you are using expressjs, [this](https://www.npmjs.com/package/server-timing) is a nice lightweight wrapper for writing the response headers. 

### Measuring a specific code path
If you want to measure and report on how long it took to validate that a user was authorized to be served "avatar.png", in the response, we’d write headers of the following format:
```
server-timing: auth = 100; authorization check
```

The entry will be available to javascript running in the page on the resource timing entry, like this:
```javascript
const {name, duration, description} = 
  performance.getEntriesByName('<path-to-file>/avatar.png')[0].serverTiming[0];
console.info(name, duration, description); // 'auth', 100, 'authorization check'
```

### Logging *all* Server Timing entries
```javascript
for (const entryType of ['navigation', 'resource']) {
  for (const {name: url, serverTiming} of performance.getEntriesByType(entryType)) {
    for (const {name, duration, description} of serverTiming) {
      console.log('Server Timing entry =',
        JSON.stringify({url, entryType, name, duration, description}, null, 2))
    }
  }
}
```

## "_Controversial_" decisions 

### Explicitly named metadata parameters:
There was [some discussion](https://github.com/w3c/server-timing/issues/12) about defining the `description` parameter as an explicitly named parameter. But as this API is intended to be as lightweight as possible, we've [suggested](https://github.com/w3c/server-timing/issues/22#issuecomment-317123400) [alternatives](https://github.com/w3c/server-timing/issues/12#issuecomment-317876891). 

### `duration` with no `startTime`? [wtf](https://github.com/w3c/server-timing/issues/10)?
"ST is not a general purpose tracing framework." - @igrigorik

## References & acknowledgements
@yoavweiss, @igrigorik
