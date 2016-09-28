<!--
  This file was generated by 'make-readme.js', edit README.tmpl.md and run 'make build' instead
-->
Universal Variable API
----------------------

[![js-standard-style](https://cdn.rawgit.com/feross/standard/master/badge.svg)](https://github.com/feross/standard)

[ ![Codeship Status for qubitdigital/uv-api](https://codeship.com/projects/f8884a40-8ad8-0132-dedc-76c1126cf0b3/status?branch=master)](https://codeship.com/projects/60163)



_The Universal Variable API moves business level data into another dimension by introducing a completely event based approach to representing page level data flow._

This project contains the source code and tests for the Universal Variable API. A script tag with a minified version of the API should be inserted inline for all html documents, at the top of head, immediately after the `<meta charset... />` tag.

Methodology
===========

Universal Variable is used for exposing internal data to third party businesses integrating technology with websites and apps. Until recently it has sufficed to provide a global object with an ecommerce-based schema however as web and mobile apps become more prominant it has become difficult to represent data changes as the visitor navigates the app with few or no page reloads.


Moving forward all data is exposed as events using a global API. There is a range of event schemas to represent data for many different use cases, namespacing is used to group events that are related. Event schemas contain nested objects that also have schema. Note that the schemas have been built so that events carry a lot of relevant embedded information to avoid the need to join between events. Though there is a lot of replicated data in the schema, the embeddeding severly reduces lookup time and increases ease of use.

Compatibility
=============

Tested in IE8+, Firefox, Opera, Chrome and Safari.

Minified snippet
===

For embedding onto client websites

```js
function createUv(){function e(e,n){n=s(n||{}),n.meta=n.meta||{},n.meta.type=e,u.push(n),1===u.length&&t(u[0]),a.listeners=c(a.listeners,function(e){return!e.disposed})}function t(e){a.events.push(e),o(a.listeners,function(t){if(!t.disposed&&i(t.type,e.meta.type))try{t.callback.call(t.context,e)}catch(e){console&&console.error&&console.error("Error emitting UV event",e.stack)}}),u.shift(),u.length>0&&t(u[0])}function n(e,t,n){function r(){return o(a.events,function(r){c.disposed||i(e,r.meta.type)&&t.call(n,r)}),u}function s(){return c.disposed=!0,u}var c={type:e,callback:t,disposed:!1,context:n||window};a.listeners.push(c);var u={replay:r,dispose:s};return u}function r(e,t,n){var r=a.on(e,function(){t.apply(n||window,arguments),r.dispose()});return r}function o(e,t){for(var n=e.length,r=0;r<n;r++)t(e[r],r)}function s(e){var t={};for(var n in e)e.hasOwnProperty(n)&&(t[n]=e[n]);return t}function i(e,t){return"string"==typeof e?e===t:e.test(t)}function c(e,t){for(var n=e.length,r=[],o=0;o<n;o++)t(e[o])&&r.push(e[o]);return r}var u=[],a={emit:e,on:n,once:r,events:[],listeners:[]};return a}"object"==typeof module&&module.exports?module.exports=createUv:window&&(window.uv=createUv());
```

API
===

### Emit

`uv.emit(type, [data])`

Emits an event with the __type__ and __data__ specified. The __data__ should conform to the schema for the event __type__ emitted. All events that are emitted are given a `meta` property with the event `type`.

```js
uv.emit('ec.ProductView', {
  product: {
    id: '112-334-a',
    price: 6.99,
    name: '18th Birthday Baloon',
    category: ['Party Accessories', 'Birthday Parties']
  },
  color: 'red',
  stock: 6
})
// => emits an ec.ProductView event
```

The emitted event will have meta attached.

```json
{
  "meta": {
    "type": "ec.ProductView"
  },
  "product": {
    "id": "112-334-a",
    "price": 6.99,
    "name": "18th Birthday Baloon",
    "category": ["Party Accessories", "Birthday Parties"]
  },
  "color": "red",
  "stock": 6
}
```


### On

`uv.on(type, handler, [context])`

Attaches an event __handler__ to be called when a certain event __type__ is emitted. The __handler__ will be passed the event data and will be bound to the __context__ object, if one is passed. If a regex is passed, the handler will execute on events that match the regex.

```js
uv.on('ec.Product.View', function (data) {
  console.log(data)
})
// => logs data when an `ec.Product.View` event is emitted

var subscription = uv.on(/.*/, function (data) {
  console.log(data)
})
// => logs data for all events
```

The on method returns a subscription object which can detatch the handler using the dispose method and can also be used to replay events currently in the event array. Note that subscription that have been disposed will not call the handler when replay is called.

```js
subscription.dispose()
// => detatches the event handler

subscription.replay()
// => calls the handler for all events currently in uv.events
```


### Once

`uv.once(type, handler, [context])`

Attaches an event __handler__ that will be called once, only on the next event emitted that matches the __type__ specified. The __handler__ will be passed the event data and will be bound to the context object, if one is passed. Returns a subscription object which can detatch the __handler__ using the dispose method. If a regex is passed, the handler will execute on the next event to match the regex.


```js
uv.once('ec.Product.View', function (data) {
  console.log(data)
})
emit('ec.Product.View')
// => logs data

emit('ec.Product.View')
// => does not log
```

### Events

The events array is a cache of events emitted since the last page load. By iterating over the array it is possible to interpret the user journey or the current state of the page.

### Deliver module

The uv-api is available on the deliver

```bash
deliver install @qubit/uv-api
```

The module exports a createUv function if required using commonjs.

```js
var createUv = require('@qubit/uv-api')
var uv = createUv()
uv.emit('ec.View')
```

### Log Level

The API has a `logLevel` method that can be used to set logging to `OFF`, `ERROR`, `INFO` or `ALL`. The default log level is `ERROR`.
```js
var createUv = require('@qubit/uv-api')
var uv = createUv()

// Turns on info logging (verbose)
uv.logLevel(uv.logLevel.INFO)
```
