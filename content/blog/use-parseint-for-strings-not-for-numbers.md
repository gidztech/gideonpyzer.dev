+++
title = "JS Tip: Use parseInt for strings, NOT for numbers"
description = "So you have a decimal number that you want converted into an integer. You've come across `parseInt` and think to yourself, 'Boy, that sounds like the right function for me!'. So you try the following."
tags = [
    "javascript",
    "math",
    "parseint",
    "rounding",
    "bitwise",
    "floats",
]
image = "parseint/binary.jpg"
date = "2017-06-06"
highlight = "true"
+++
![](/img/blog/parseint/binary.jpg)

So you have a decimal number that you want converted into an integer. You've come across `parseInt` and think to yourself, "Boy, that sounds like the right function for me!". So you try the following.

```javascript
parseInt(1.655)
-> 1
```

Great. It worked!

But later on...

```javascript
parseInt(0.0000000060)
-> 6
```

Whoops. Clearly something went wrong here.

So what happened? If you cast `0.000000006` as a `Number` in your DevTools console, you'll see the value `6e-9`. This is a shorthand mathematical form used when there's lots of consecutive 0s.

Let's quickly go to the MDN spec to see what `parseInt` actually does. 

> The parseInt() function parses a **string** argument and returns an integer of the specified radix (the base in mathematical numeral systems).

> If the string argument is not a string, then it is converted to a string (using the ToString abstract operation).
-- <cite>[MDN][1]</cite>

Back to our problem, what has happened is that the number `0.000000006` was first converted to a string, which is `"6e-9"`, as per the spec. 

It then tried to parse the string, but came across an `e` character, which is not a number. Well, technically it is! It's approximately 2.718, but let's not get into logarithms! It bails out at `e`, and returns what it has to that point. In this case, `6`.

If we now perform the same action using a string representation of the original number, we get the correct answer:

```javascript
parseInt('0.000000006')
> 0
```
It's clear that `parseInt` is not super safe. It tries to be clever by coercing, but it fails in some circumstances. One solution is to put quotes around your numbers and use the function as it was primarily intended. 

However, it's probably not very memory efficient or speedy to use strings to get around this problem, particularly if you are dealing with lots of values.

# Solution 1
You can use `Math` functions to round or truncate numbers to form integers, such as [`Math.round`][2], [`Math.ceil`][3], [`Math.floor`][4] and [`Math.trunc`][5].

My preferred solution is `Math.trunc` because it simply truncates everything from the decimal point, and doesn't apply any rounding logic.


## Example 1
```javascript
Math.trunc(1.655)
> 1
```
## Example 2
```javascript
Math.trunc(0.000000006)
> 0
```

You can implement the truncation logic very simply using `Math.ceil` and `Math.floor` in cases where you cannot run ES2015 functions.

```javascript
function ToInteger(v) { 
    if (v > 0) {
        return Math.floor(v);
    }
    return Math.ceil(v);
}
```
```javascript
ToInteger(0.000000006)
> 0
```

# Solution 2
A potentially faster method of converting a number to an integer is to use [bitwise operators](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators). Bitwise in JavaScript works by converting the operands of an operation to 32-bit signed integer representation, and then flips or shifts bits where applicable.

We can use the OR operator as below.

```javascript
1.655 | 0
-> 1
```

`1.655` gets converted to `0b00000001` when used in a bitwise operation. This is 1 in binary. That's it. It's done the truncation and returned the integer for us. 

In order to coerce the JavaScript engine to do this, we had to provide two operands and a bitwise operator, but we didn't want to make any changes to the left operand.

The OR operator yields a 1 for each bit column if either operand has a 1 in the corresponding column. By using 0 as the right operand, the return value in every column is the same.

```javascript
00000001
   |
00000000
-> 00000001
```
The same behaviour can be achieved with the [shift operator][6], `>>`

The drawback to this solution is that we can only use numbers in 32-bit representation. Numbers in JavaScript are 64-bit floating points, so we can represent much higher values if we wanted to.

If we attempt to convert `4000000000000000000000.1` to an integer with this method, we get an overflow.

```javascript
4000000000000000000000.1 | 0
-> 2055208960
```

Method 1 didn't suffer from this. If you can guarantee not to have large values (+/- 2^32), use bitwise, otherwise I recommend using `Math.trunc`, or perhaps `Math.round` if you prefer to round up/down.

[1]:https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/parseInt
[2]:https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/round
[3]:https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/ceil
[4]:https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/floor
[5]:https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/trunc
[6]:https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators#Bitwise_shift_operators