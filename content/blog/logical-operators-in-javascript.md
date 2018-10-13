+++
title = "Logical Operators in JavaScript"
description = "Logical operators are typically used with Boolean (logical) values. When they are, they return a Boolean value. However, the && and || operators actually return the value of one of the specified operands, so if these operators are used with non-Boolean values, they may return a non-Boolean value."
tags = [
    "javascript",
    "logical operators",
    "truthy",
    "falsy",
    "debugging"
]
image = "javascript.png"
date = "2016-05-11"
highlight = "true"
+++

> Logical operators are typically used with Boolean (logical) values. When they are, they return a Boolean value. However, the && and || operators actually return the value of one of the specified operands, so if these operators are used with non-Boolean values, they may return a non-Boolean value.
-- <cite>[MDN][1]</cite>

The important thing to note is that, although the expressions can be non-Boolean, they can all be converted into Boolean values.

The following are considered "falsy" values:

- `null`
- `NaN`
- `0`
- `""`
- `undefined`

All others are therefore "truthy" values.

When using the `&&` (AND) and `||` (OR) logical operators, the expressions are evaluated left-to-right. 

The `&&` operator will return the right operand if the first operand evaluates to `true`, otherwise it's "short-circuited" when the first operand evaluate to `false`.  

The `||` operator will return the right operand if the left operand evaluates to `false`, otherwise it's "short-circuited" when the first operand evaluates to `true`.

# Example 1
In the following snippet, we have a function `setName`, which returns a default value of "(No name)" when the `name` parameter passed in evaluates to `false` (any of the "falsy" values mentioned above).

```javascript
function setName(name) {
​    if (!name) {
        return "(No name)";
    }
    return name;
}
```

Since the logical operators return expressions as well as evaluating them as Boolean values, we can avoid an explicit `if` statement, and shorten the above code as follows:

```javascript
function setName(name) {
​    return name || "(No name)";
}
```

If `name` evaluates to `false`, "(No name)" is returned, otherwise `name`.

<span style="color:#FF0000; font-weight:bold">Take note:</span>
You should be careful using the `||` operator for defaulting values. If you pass in a valid value that happens to be "falsy", you could have unintended side effects. For example, for a function dealing with numerical values, if the value `0` is passed in as a parameter, and this is considered a valid number, it will be automatically replaced by the default value. In this case, an explicit check for `undefined` would be required. In ES6, check out [Default Parameters][2] to simplify this situation.

#Example 2
We can use the logical operators in other ways. In ES6, `Arrow functions` were introduced. The following function will take each value of the array and multiply it by two.

```javascript
var double = [1, 2, 3, 4, 5].map(v => v * 2); 
```
In this example, the arrow function is used as a shortcut instead of defining an anonymous function that has it's own scope. You can read up on many other great benefits [here](https://strongloop.com/strongblog/an-introduction-to-javascript-es6-arrow-functions/), but the logical operator comes to use when you need to debug your code.

```javascript
function debugFunc(value) {
    debugger;
    // console.log(value);  
    // ...
}

var double = [1, 2, 3, 4, 5].map(v => debugFunc(v) || (v * 2)); 
```

`debugFunc` is a "falsy" value, as the return value is `undefined`. Therefore the second operator `(v * 2)` is returned instead, which is equivalent to the original code we have. The difference is that we still executed the `debugFunc` function, and so the `debugger` statement would be hit when used in your debugging environment.

You could alternatively avoid the logical operators and put individual statements in curly braces (`{}`). You have to remember to add a `return` to the last statement, as it is no longer implicit. The problem here is that you miss out on the shorter, more concise syntax. Having said that, for shared code, the logical operators can be difficult to grasp, and could potentially add confusion. Use where deemed appropriate. 

```javascript
var double = [1, 2, 3, 4, 5].map(v => { 
    debugFunc(v); 
    return (v * 2); 
}); 
```

[1]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_Operators

[2]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Default_parameters