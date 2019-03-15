+++
title = "Debugging JavaScript with Monkey Patching Functions"
description = "There are hundreds of amazing debugging tools available for JavaScript in modern web browsers today. Breakpoints are particularly useful for debugging problems or can simply be used to understand a particular code base."
tags = [
    "javascript",
    "debugging",
    "functions",
]
image = "javascript.png"
date = "2016-03-26"
highlight = "true"
aliases = [
    "/blog/debugging-javascript-by-redefining-functions"
]
+++

There are hundreds of amazing debugging tools available for JavaScript in modern web browsers today. Breakpoints are particularly useful for debugging problems or can simply be used to understand a particular code base.

This is great when you are debugging your own code, but for APIs, such as Web Storage, if you wanted to debug API function calls, you'd have the tedious job of placing breakpoints in many places in your code, particularly in a larger application.

If we wanted to use the debugger to pause execution every time an API function was invoked, we could do this by redefining the API functions, such that debugging code was injected before the original logic.

---
To begin with, let us create a generic function that allows us to debug. In this case, we will use a simple debugger statement, but this could be a console log or anything else. 

```javascript
var myDebugFunction = function () {
    debugger;
}
```

We can now create the logic to redefine functions from the Web Storage API. The important thing to do first is to store a copy of the original function declarations, as we will need them later.

```javascript
var originalSetItem = window.localStorage.setItem;
var originalGetItem = window.localStorage.getItem;
```

We can now redefine the `setItem` function, to inject our own debugger function before the original logic. We use the `apply` method to invoke the original function with the arguments array.

```javascript
window.localStorage.setItem = function () {
  myDebugFunction();
  return originalSetItem.apply(this, arguments);
}
```
---
## How does `apply` work?

> The apply() method calls a function with a given this value and arguments provided as an array (or an array-like object).
-- <cite>[MDN][1]</cite>

### Arguments
In JavaScript, function declarations do not need to specify the accepted parameters. Every parameter can be accessed via the `arguments` array.

The following example demonstrates this:

```javascript
function getMaximumInteger() {
    var max = 0;
    for (var i in arguments) {
        if (arguments[i] > max) {
            max = arguments[i];
        }
    }
    return max;
}
```

```javascript
console.log(getMaximumInteger(4, 5, 2, 9, 4));
> 9
```

It is important to note that from an API perspective, it is very bad practice to do this, as it causes ambiguity. We typically declare all parameters, so that other programmers know exactly what the function expects, without having to understand implementation details.

Back to the Web Storage scenario, we make use of the `arguments` from the redefined function, and pass them through to the original function. This means we do not have to know about the implementation of the original function, specifically, how many parameters are required or expected.

### Context
If you knew a function didn't expect any arguments or you knew, in advance, how many arguments it expected you could write invoke it like this:

```javascript
window.localStorage.setItem = function (a, b) {
  myDebugFunction();
  return originalSetItem(a, b);
}
```

In many cases, where the function you are calling is self contained, the above code would work. However, a problem occurs when a method requires information from the object it was invoked on. This is known as the *context*. 

The following example demonstrates context:

```javascript
var SomeClass = function () {
  this._someField = 4,
  this.execute =  function() {
    console.log(this._someField);
  }
} 

var myObj = new SomeClass();
myObj.execute();
```

```javascript
> 4
```

When you invoke the `execute` function on `myObj`, it will log the value of `_someField`, which is 4.

Look what happens when we create a copy of the `execute` function definition and invoke it:

```javascript
var copyOfExecute = myObj.execute;
copyOfExecute();
```

```javascript
> undefined
```
The `execute` function makes a reference to the `_someField` field within `myObj`. Since we are no longer invoking the function on the instance, `myObj`, the context `this` is `undefined`.

Back to the Web Storage scenario, the `setItem` function implementation makes reference to information in the `localStorage` object, hence why we can't call the function without context. 

### Using `apply`
The `apply` method in JavaScript is used to invoke another function, specifying the context and arguments (as an array). In the case of `window.localStorage.setItem`, the context `this` refers to `localStorage`, so we can simply pass that as the context. For the arguments parameter, we pass `arguments` from the original calling function.

```javascript
var originalSetItem = window.localStorage.setItem;
var originalGetItem = window.localStorage.getItem;

window.localStorage.setItem = function() {
    myDebugFunction();
    return originalSetItem.apply(this, arguments);
}

window.localStorage.getItem = function() {
    myDebugFunction();
    return originalGetItem.apply(this, arguments);
}
```

We can then run this as a code snippet in the browser, and then whenever our app calls one of the Web Storage functions, the debugger will pause on execution. 

```javascript
// execution would pause during this invocation.
window.localStorage.setItem("name", 123);

```

[1]:https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply