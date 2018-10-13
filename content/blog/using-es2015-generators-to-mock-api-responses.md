+++
title = "Using ES2015 generators to mock API responses"
description = "Generators were first introduced in the ES2015 specification. They are special functions in JavaScript that allow you to 'return' some data, but later re-enter the function to continue further execution. The context (internal data) is maintained throughout."
tags = [
    "javascript",
    "es6",
    "es2015",
    "generators",
    "mocking",
    "apis",
    "sinon.js",
    "huddle"
]
image = "generators/generator.png"
date = "2017-07-18"
highlight = "true"
+++

![Generator](/img/blog/generators/generator.png)*<span style="font-size: small; text-align: center; display: block">Source: https://www.hondashop.com.au/</span>*

Generators were first introduced in the ES2015 specification. They are special functions in JavaScript that allow you to "return" some data, but later re-enter the function to continue further execution. The context (internal data) is maintained throughout.

When a generator function is first invoked, an `iterator` object is returned. When the `next` function is called on that object, the first `yield` statement is executed, and the value can be accessed. Each time we call `next`, we yield again. You can think of yields a bit like return statements, but the difference is that the function will not fully complete until all the `yield` statements have been executed.

# Basic example

```javascript
function* myGenerator() {
    yield 'Toby';
    yield 'Leonard';
    yield 'Archie';
}

var g = myGenerator();

console.log(g.next().value);
console.log(g.next().value);
console.log(g.next().value);
console.log(g.next().value);
```
```javascript
> Toby
> Leonard
> Archie
> undefined
```
You can see that the three names have been logged out. It is followed by an `undefined` value, because there was nothing left to yield the 4th time.

Generators are typically used with large or infinite data sets, because the values are computed on demand, so you save on memory and allocation. They can also be used in combination with Promises to give a synchronise looking API for asynchronous code. This grew into the new `async/await` feature in ES2017. Check out [Understanding JavaScript’s async await](https://ponyfoo.com/articles/understanding-javascript-async-await) for more details.

In the next section, I want to discuss how you can use utilise generators for mocking API responses for your development or test environment.

# Mock API responses

At [Huddle](https://www.huddle.com/), our components are developed in isolation as opposed to an integrated environment. One of the benefits of this is speed of development. When you have a Webpack dev server, with Hot Module Replacement enabled, you can quickly write code and see results instantly. 

Another subsequent benefit is the way it forces you to think about how you test your components. With a mocked development environment, you can write tests quickly, because all the hard work is already done. 

In many cases, you call a function and you expect the same result each time. This is essential for pure functional code. However, dealing with APIs is a different story. The response of the call may differ based on numerous possible conditions.

The example I will use is our `upload-widget` component. There are intermediate UI states between starting an upload and it completing. We need to test each of these. One way you could achieve this is by storing and modifying state outside of a function, and then using conditional return statements. However, generators allow us to yield different values, while encapsulating state within the function itself.

The following is a slightly modified version of a mock we wrote. [Sinon.JS](http://sinonjs.org/) can be used to mock XHR calls. We have a light wrapper around it to coordinate things, and allow us to save collections of requests/responses to later verify against in our tests.

The `get` function used below takes in two arguments. The first is either a URI string or a regular expression, to match on `HTTP GET` request URIs. The second is either a normal function or a generator for yielding responses. If a request URI matches, the function is executed. In the case of this generator, the first yield will occur, indicating an upload is in progress. The `while` loop then ensures that all subsequent calls to the same endpoint return a completed upload.

```javascript
    get('/api/files/upload/432423', function*() {
        yield { 
            status: 200, 
            response: fileProgress({ 
                totalBytes: 9518, 
                bytesWritten: 1533, 
                status: 'Uploading' 
            });
        };

        while (true) {
            yield { 
                status: 200, 
                response: fileProgress({
                    totalBytes: 9518, 
                    bytesWritten: 9518, 
                    status: 'Complete' 
                });
            };
        }
    });
```

<img src="/img/blog/generators/sequence.png" alt="Sequence of calls" class="full-img" style="border: none !important;">*<span style="font-size: small; text-align: center; display: block">Sequence diagram showing the request and responses</span>*

You could extend this, for example, to deal with loss of connection scenarios. The implementation details of the wrapper are not too important here, but the main point is that `next()` is invoked on the iterator object, and the value returned, each time we respond from the fake server. 

This kind of mocking API is really pleasant to use, and keeps all state encapsulated. In our tests, we can assert that the correct number of requests have been made, and then verify the UI updates accordingly.

We're hiring at [Huddle](https://www.huddle.com/). If you're interested, check out our [vacancies](https://talentcommunity.huddle.com/).