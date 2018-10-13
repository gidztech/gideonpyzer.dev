+++
title = "JavaScript: Function Composition and Pipelines"
description = "Function Composition is a mathematical concept, by which the result of one function becomes the input of the next, and so on. The return value of the final function call is the overall result. It is a key concept in functional programming. Composing functions from several small and simple functions allows for easy code reuse."
tags = [
    "javascript",
    "functions",
    "composition",
    "pipelines",
    "reduce",
    "accumulator",
    "es6"
]
image = "composition/pipes.png"
date = "2017-01-02"
highlight = "true"
+++

![Pipelines](/img/blog/composition/pipes.png)*<span style="font-size: small; text-align: center; display:block;">Pipeline functions are analogous to physical pipelines. See <a href="#pipelines">Pipelines</a> section.</span>*

[Function Composition](https://en.wikipedia.org/wiki/Function_composition_(computer_science)) is a mathematical concept, by which the result of one function becomes the input of the next, and so on. The return value of the final function call is the overall result. It is a key concept in functional programming. Composing functions from several small and simple functions allows for easy code reuse.

In mathematics, a composition of the functions `f` and `g` can defined as `(f ∘ g)(x)` or `f(g(x))`. We evaluate everything from right-to-left, which means the inner-most function returns first, then outer, and so on.

## Composite Functions
The following function takes two functions, `f` and `g`, and returns a new function that will execute the two functions on an argument `x` passed in.

```javascript
function compose(f, g) {
    return function(x) {
        return f(g(x));
    }
}
```
Let's take an example where `f` and `g` are defined as `multiplyBy5` and `add10` respectfully.

```javascript
let multiplyBy5Add10 = compose(multiplyBy5, add10);
```

We have now composed a new function that will multiply an argument by 5 and add 10. Let's try this out and log it to the console.

```javascript
let result = multiplyBy5Add10(4);
console.log(result);
> 70
```

Were you expecting the value `70`? If not, you may have expected `30` instead. Take a look at the `compose` function again. As explained earlier, we evaluate right-to-left. This means that we evaluate `x` first, then `g`, and then finally `f`. The intermediate results are:

* `x` = 4
* `g(x)` = 14
* `f(g(x)` = 70

Right now our `compose` function allows us to compose with just two functions. To make it more generic, we can create a function that takes in any number of functions, and generates a composed function from these.

We can achieve this by using the `reduceRight` function. 

> The reduceRight() method applies a function against an accumulator and each value of the array (from right-to-left) has to reduce it to a single value. 
-- <cite>[MDN][1]</cite>

```javascript
const compose = (...fns) => {
    return (initialVal) => {
        return fns.reduceRight((val, fn) => { 
            return fn(val);
        }, initialVal);
    }
}
```
The accumulator is the previous value returned from the callback function passed into `reduceRight`. For each item in the `fns` array, the previous value, `val`, and the next function, `fn`, are passed in. The function is called with that previous value, and that returned value becomes the `val` argument in the next iteration.

The `initialVal` is the initial argument value passed to the generated function - in our case, it's `4`. This is used as the previous `val` (accumulator) on the first iteration only.

Earlier, you may have automatically thought the result of the function would be `30` instead of `70`. I certainly did originally. For me, it seems more intuitive to evaluate from left-to-right, as if it were a TODO list. Do a, then b, then c. Let's now take a look at the concept of pipelines.

# <h1 id="pipelines">Pipelines</h1>
In Unix operating systems, a pipeline is multiple processes that are chained together. Each process takes an input from the previous process's output, and then passes it's output along to the next process. 

`process1 | process2 | process3`

The `|` symbolises the pipe.

This is very much similar to how our `compose` function works. The only difference is that it evaluates left-to-right instead. Below is an implementation of a `pipe` function.

```javascript
const pipe = (...fns) => {
    return (initialVal) => {
        return fns.reduce((val, fn) => {
            return fn(val);
        }, initialVal);
    }
}
```
Now let's use it, and you will see we get the value `30`.

```javascript
let multiplyBy5Add10 = pipe(multiplyBy5, add10); 
let result = multiplyBy5Add10(4);  
console.log(result);  
> 30
```

[1]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Array/reduceRight

