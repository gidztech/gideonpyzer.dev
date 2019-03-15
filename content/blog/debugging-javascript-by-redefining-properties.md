+++
title = "Debugging JavaScript with Monkey Patching Properties"
description = "In Debugging JavaScript with Monkey Patching Functions, we discussed how to inject our own debugging function calls into existing function definitions. Now we will do the same for properties, which involves a slightly different approach."
tags = [
    "javascript",
    "debugging",
    "properties",
    "prototype",
]
image = "javascript.png"
date = "2016-04-05"
highlight = "true"
aliases = [
    "/blog/debugging-javascript-by-redefining-properties"
]
+++

In <a href="/blog/debugging-javascript-by-redefining-functions/" ga-link-track="debugging-javascript-by-redefining-functions">Debugging JavaScript with Monkey Patching Functions</a>, we discussed how to inject our own debugging function calls into existing function definitions. Now we will do the same for properties, which involves a slightly different approach.

A `property` in JavaScript can either be a data descriptor, which has a simple `value` that is retrieved or assigned, or it can be an accessor descriptor, which has `get` and `set` functions. `cookie` is a property of the `document` object. It does a lot more than store a value; it has to interface with the browser, so it an example of an accessor property descriptor.

In order to inject our debugging function before the existing logic, we need to store a copy of the original property for later. We do this by using the `Object.getOwnPropertyDescriptor` method. This takes the object on which the property is defined, and the property name itself.


```javascript
var originalCookie =  
  Object.getOwnPropertyDescriptor(Document.prototype, 'cookie') ||
  Object.getOwnPropertyDescriptor(HTMLDocument.prototype, 'cookie');
```

You will notice that we are actually using the prototype of the `Document` class. This is because the property is defined on the object's prototype, so that it can be inherited. You will also notice that we are checking both `Document` and `HTMLDocument` for the property. This is due to browser implementation differences. In Chrome, Safari and IE, the `cookie` property is defined on `Document`, whereas in FireFox it is defined on `HTMLDocument`, which is a more specific implementation.

In order to create a property, we us the `Object.defineProperty` method. This takes the object on which the property will be added to, and the property name. We then define the `get` and `set` functions. We use the `call` method to invoke the original functions. This is different from `apply`, which was used in the previous article. They are very similar methods, but instead of passing the whole `arguments` array, you explicitly pass the individual parameters. For the `cookie` property, we know exactly what parameters are expected, so there is no need to use `apply`. 

```javascript
var myDebugFunction = function() {
    debugger;
}

var originalCookie =
    Object.getOwnPropertyDescriptor(Document.prototype, 'cookie') ||
    Object.getOwnPropertyDescriptor(HTMLDocument.prototype, 'cookie');
if (originalCookie && originalCookie.configurable) {
    Object.defineProperty(document, 'cookie', {
        get: function() {
            myDebugFunction();
            return originalCookie.get.call(document);
        },
        set: function(val) {
            myDebugFunction();
            originalCookie.set.call(document, val);
        }
    });
}
```

We can then run this as a code snippet in the browser, and then whenever our app calls attempts to access or modify the `cookie` property, the debugger will pause on execution.

```javascript
// execution would pause during this invocation.
document.cookie = "a=1";
```

You can find a more generic code snippet written by [Matt Zeunert](https://twitter.com/mattzeunert) here: https://gist.github.com/dmethvin/1676346#gistcomment-1732477