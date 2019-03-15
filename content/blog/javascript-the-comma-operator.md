+++
title = "The Comma Operator in JavaScript"
description = "The comma operator evaluates each of its operands (from left to right) and returns the value of the last operand."
tags = [
    "javascript",
    "coding",
    "programming",
    "comma operator"
]
image = "javascript.png"
date = "2016-02-25"
highlight = "true"
aliases = [
    "/blog/javascript-the-comma-operator"
]
+++

>The comma operator evaluates each of its operands (from left to right) and returns the value of the last operand.
-- <cite>[MDN][1]</cite>

An operand is the data on which an operation will be performed. An operator describes the specific operation to be performed.

In the simple example below, we perform the comma operation on two operands:

```javascript
> a = (1, 2);
< 2
```

The two operands are evaluated as `1` and `2` respectfully. The value of the last operand, `2`, is returned and assigned to the variable `a`.

The parentheses (or brackets) are used to increase the priority of the comma operation within the overall statement. This is because the comma operator has the lowest precedence of all operators. 

If we remove the parentheses from the above code, you will notice that, although the returned value is correct, when we check the variable `a`, it is set to the value of `1` instead. 

```javascript
> a = 1, 2;
< 2
> a
< 1
```

The assignment operator `=` is three levels above the comma operator in the precedence hierarchy, so the assignment operation will be performed first. The value `2` will then be returned and discarded. For more information, check out [Order Precedence](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Operator_Precedence). 

The benefits of using the comma operator can be seen when we want to evaluate multiple expressions for their side effects, but return the value of the last expression where only one expression is expected. 

The most common usage is within a `for` loop, to allow multiple variables to be incremented/decremented for each iteration of the loop.

```javascript
for (var i = 1, j = 5; i <= 5; i++, j--) {  
    console.log("i: " + i, "j: " + j); 
}   

i: 1 j: 5
i: 2 j: 4
i: 3 j: 3
i: 4 j: 2
i: 5 j: 1
```
Note that `var i = 1, j = 5;` is a multiple variable assignment statement and not a use of the comma operator. However, `i++, j--` is using the comma operator to evaluate both operands in the third statement of the `for` loop. The last operand is returned, but in this case, it doesn't matter what gets returned because the return value is never actually used. 

We can add to this loop by making use of the return value:

```javascript
for (var i = 1, j = 5, k = 1; i <= 5; k = (i++, j--, (i * j))) {  
    console.log("i: " + i, "j: " + j, "k: " + k); 
}   

i: 1 j: 5 k: 1
i: 2 j: 4 k: 8
i: 3 j: 3 k: 9
i: 4 j: 2 k: 8
i: 5 j: 1 k: 5
```
In this case, we declare and assign the new variable `k`, increment and decrement the variables `i` and `j` respectfully, and then we perform the calculation `(i * j)`, which will be returned to the variable `k`.

The comma operator is not a heavily used feature because of the confusion it can cause with other uses of commas in the language. It is always possible to use standard assignment statements to get the same results, but the comma operator in a few cases can be more elegant. 

[1]:https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Comma_Operator