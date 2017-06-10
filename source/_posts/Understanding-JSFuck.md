---
title: Understanding JSFuck
date: 2017-06-01 01:36:53
categories:
- JavaScript
tags:
- JSFuck
- ES5
thumbnail: /images/javascript.png
---

*JavaScript: The Good Parts* readers and **Douglas Crockford** fans know that JavaScript subsets matter. Due to the permissive and versatile nature of the language, it is reasonable to select only the most solid stones to build a wall. But hey, why would we need to bother with a full programming language when we just need 6 characters? Here comes [**JSFuck**](http://www.jsfuck.com/) by **Martin Kleppe**...

<!-- more -->

> JSFuck is an esoteric and educational programming style based on the atomic parts of JavaScript. It uses only six different characters to write and execute code.

I know what you think: "*6 characters? WTF!?*". Yes, JSFuck allows you to code using `(`, `)`, `[`, `]`, `+` and `!` only. How is this possible?

To understand what is going on behind JSFuck, we have to explain how JavaScript works with:

- **Type conversion/coercion**
- **Truthy/falsy values**
- **Operator precedence/associativity**
- **Bracket notation**

## Type conversion/coercion

### Explicit type conversion

For explicit type conversion in JavaScript, **unary operators** like `+` or `!` may be used:

- `+` can convert the operand to a number
- `!` can convert the operand to a boolean (and "negates" it)

```JavaScript
// MULTIPLE TYPES

var str = '',
    nb = 42,
    bool = true,
    arr = [],
    obj = {};

console.log(str, typeof str); // "" string
console.log(nb, typeof nb); // 42 number
console.log(bool, typeof bool); // true boolean
console.log(arr, Array.isArray(arr)); // [] true
console.log(obj, typeof obj); // {} object

// NUMBERS

var n_str = +str,
    n_nb = +nb,
    n_bool = +bool,
    n_arr = +arr,
    n_obj = +obj;

console.log(n_str, typeof n_str); // 0 number
console.log(n_nb, typeof n_nb); // 42 number
console.log(n_bool, typeof n_bool); // 1 number
console.log(n_arr, typeof n_arr); // 0 number
console.log(n_obj, typeof n_obj); // NaN number

// BOOLEANS

var b_str = !str,
    b_nb = !nb,
    b_bool = !bool,
    b_arr = !arr,
    b_obj = !obj;

console.log(b_str, typeof b_str); // true boolean
console.log(b_nb, typeof b_nb); // false boolean
console.log(b_bool, typeof b_bool); // false boolean
console.log(b_arr, typeof b_arr); // false boolean
console.log(b_obj, typeof b_obj); // false boolean
```

### Implicit type conversion (coercion)

There would be many things to say about type coercion in JavaScript... However, with JSFuck, we only need to understand why:

- `[]+[]` is an empty string
- `+[]` is `0`
- `true+false` = `1`

These three odd behaviors are linked to the same ambiguity: the `+` operator which can be used for concatenation (in a string context) and for addition/incrementation/conversion (in a numerical context).

In fact, `Array.prototype.toString()` is called internally during an array-to-primitive conversion, and we know that its output is different from the one given by `Object.prototype.toString()`: it will not return something like `[object Object]`, but a concatenation of all elements in the array with a comma as a separator.

```JavaScript
var arr = ['foo', 'bar', 'baz'];
console.log(arr.toString()); // foo,bar,baz
```

Of course, if the array is empty, `Array.prototype.toString()` returns an empty string. That is why `[]+[]` gives an empty string: this operation is actually no more than a concatenation of empty strings after coercion!

Moreover, when converted to a number, an empty string becomes `0`. `[object Object]` is not an empty string and does not contain numbers, so it becomes `NaN`. Here is what happens (roughly) under the hood:

```JavaScript
console.log([].toString() + [].toString()); // ""
console.log([].toString(), +[].toString()); // "" 0
console.log({}.toString(), +{}.toString()); // [object Object] NaN
```

An operation with `+` where operands are booleans is not evaluated in a string context, but in a numerical context. This means booleans are coerced to numbers. `true` becomes `1` and `false` becomes `0`.

```JavaScript
console.log(false + false); // 0
console.log(true + false); // 1
console.log(true + true); // 2
```

## Truthy/Falsy values

In boolean expressions, JavaScript accepts booleans (`true`/`false`), but it also evaluates **truthy** and **falsy** values:

- An object (object literal, array, function, etc.) is always **truthy**
- `undefined`, `null`, `NaN`, `0` and the empty string are **falsy**

```JavaScript
function truthyOrFalsy(arg) {
  arg ? console.log('truthy') : console.log('falsy');
}

truthyOrFalsy(''); // falsy
truthyOrFalsy('str'); // truthy
truthyOrFalsy(0); // falsy
truthyOrFalsy(1); // truthy
truthyOrFalsy([]); // truthy
truthyOrFalsy({}); // truthy

console.log(!'', !!''); // true false
console.log(!'str', !!'str'); // false true
console.log(!0, !!0); // true false
console.log(!1, !!1); // false true
console.log(![], !![]); // false true
console.log(!{}, !!{}); // false true
```

Obviously, truthy values become `true` when they are converted to booleans with `!!` (double logical NOT or "NOT NOT") and falsy values become `false`.

## Operator precedence and associativity

Just like in math, operations must be performed in a specific order, following certain conventions. This order is determined by:

- Operator precedence
- Operator associativity

### Operator precedence

Each operator has its own priority. For example, we know that multiplication takes priority over addition and that parentheses allow us to override this natural priority.

```JavaScript
console.log(1 + 2 * 3); // 7
console.log((1 + 2) * 3); // 9
```

Like most programming languages, JavaScript has many operators. The full [reference of operator precedence](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence#Table) is available on MDN.

An interesting point is that the unary `+` has higher precedence than the binary `+`.

```JavaScript
// Conversions are performed BEFORE the addition
console.log(+'41' + +true); // 42
```

### Operator associativity

Unfortunately, operator precedence cannot determine the "absolute" order of all operations. An operation can contain several operators that have the same precedence, so we need a clear direction ("left-to-right" or "right-to-left").

In math, we know that the order does not really matter to perform an addition:

```JavaScript
console.log(1000 + 300 + 37); // 1337
console.log(300 + 1000 + 37); // 1337
console.log(37 + 300 + 1000); // 1337
```

But it is much more problematic with subtraction:

```JavaScript
console.log(100 - 58); // 42
console.log(58 - 100); // -42
```

An interesting point is that the unary `+` has "right-to-left" associativity, whereas the binary `+` has "left-to-right" associativity.

Again, the full [reference of operator associativity](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence#Table) is available on MDN. We could also bring back the previous example:

```JavaScript
// Conversions are performed right-to-left
// The addition is performed left-to-right
console.log(+'41' + +true); // 42
```

## Bracket notation

In JavaScript, all objects have **own** and/or **prototype** properties. These properties can be accessed using the **dot** (`.`) notation or the **bracket** notation (`[]`). JSFuck uses bracket notation, which must not be confused with empty arrays...

```JavaScript
console.log({foo: 'Foo'}.foo); // Foo
console.log({foo: 'Foo'}['foo']); // Foo

console.log([].length); // 0
console.log([]['length']); // 0

console.log(/abc/g.flags); // g
console.log(/abc/g['flags']); // g

console.log(function fn() {}.name); // fn
console.log(function fn() {}['name']); // fn
```

---

## JSFuck basics

[JSFuck](http://www.jsfuck.com/) introduces itself with the following basics:

- **false**       =>  `![]`
- **true**        =>  `!![]`
- **undefined**   =>  `[][[]]`
- **NaN**         =>  `+[![]]`
- **0**           =>  `+[]`
- **1**           =>  `+!+[]`
- **2**           =>  `!+[]+!+[]`
- **10**          =>  `[+!+[]]+[+[]]`
- **Array**       =>  `[]`
- **Number**      =>  `+[]`
- **String**      =>  `[]+[]`
- **Boolean**     =>  `![]`
- **Function**    =>  `[]["filter"]`
- **eval**        =>  `[]["filter"]["constructor"]( CODE )()`
- **window**      =>  `[]["filter"]["constructor"]("return this")()`

With what we know about **type conversion/coercion**, **truthy/falsy values**, **operator precedence/associativity** and **bracket notation**, we can know explain this mess!

### false => `![]`

`[]` is an array. JS arrays are objects and objects are always truthy. So if we negate a truthy value with a logical NOT, it becomes `false`.

### true => `!![]`

`![]` is `false`, but `!![]` is equivalent to `!false`. So it is `true`.

### undefined => `[][[]]`

`[]` is an array. Each array has built-in properties. `length` is a valid one and could be accessed like so: `[]["length"]`. In this case, the array is empty and returns `0`.

But in our case, `[]` or `""` is not a valid property of the array, so `[][[]]` obviously returns `undefined`.

### NaN => `+[![]]`

`![]` is `false`, but `[false]` is truthy (this is an array containing a boolean). `[false].toString()` returns the string `"false"` and `+"false"` returns `NaN`.

### 0 => `+[]`

`[].toString()` gives an empty string and `+""` gives `0`.

### 1 => `+!+[]`

`+[]` gives `0`. `0` is falsy, so `!0` gives `true`. `+true` gives `1`.

### 2 => `!+[]+!+[]`

`!+[]` is `true` because `+[]` is `0` and `0` is falsy (`!0` is `true`). `true+true` = `2` because booleans are coerced to numbers when evaluated in a numerical context (`+false` is `0` and `+true` is `1`).

### 10 => `[+!+[]]+[+[]]`

**Left part:** `+!+[]` = `1`, so `[+!+[]]` = `[1]`.
**Right part:** `+[]` = `0`, so `[+[]]` = `[0]`.

**Concatenation:**
`[1]+[0]` is equivalent to `[1].toString() + [0].toString()` and returns `"10"`.

### Array => `[]`

An array literal (better alternative to `new Array()`). `Array.isArray([])` = `true`.

### Number => `+[]`

`+[] = 0` and `typeof +[]` = `number`.

### String => `[]+[]`

`[]+[]` is equivalent to `[].toString() + [].toString()`. It is actually a concatenation of empty strings and the result is an empty string.

### Boolean => `![]`

`[]` is truthy, `![]` is `false`.

### Function => `[]["filter"]`

`[]` is an array (instance of `Array`) and has access to `Array.prototype` methods. We can then access `Array.prototype.filter` (which is a function) using bracket notation.

### eval => `[]["filter"]["constructor"]( CODE )()`

`[]["filter"]` is a function. In JavaScript, each function is an instance of `Function` and has access to `Function.prototype` properties. `Function.prototype.constructor` is one of these properties (this one is actually a reference to `Function`). Thus, `[]["filter"]["constructor"]` is an equivalent of `Function`. With or without `new`, `Function()` returns a new function. We could then make a simple addition like so:

```JavaScript
Function('a', 'b', 'return a + b')(3, 7) // returns 10
```

### window => `[]["filter"]["constructor"]("return this")()`

`[]["filter"]["constructor"]` is `Function`. When the code is not in *strict mode*, `this` is generally `window` in regular functions. Here is an example:

```JavaScript
function foo() {
  console.log(this); // window
}

foo();
```
For the record, this result would be `undefined` in strict mode:

```JavaScript
function foo() {
  'use strict';
  console.log(this); // undefined
}

foo();
```

It could also be written like this:

```JavaScript
[]["filter"]["constructor"]("'use strict'; console.log(this);")()
```

---

## Challenge

Try to decrypt the following code. Don't cheat, if you can... :P

```JavaScript
(!![]+[])[!+[]+!+[]+!+[]]+([][[]]+[])[+!+[]]+([][[]]+[])[!+[]+!+[]]
```

### Hints

- Identify the main operands
- Break it into small pieces
- The result is a string

Good luck!

### Solution

```
(!![]+[]) => (true+[]) => ("true"+"") => ("true") => "true"
[!+[]+!+[]+!+[]] => [!0+!0+!0] => [true+true+true] => [1+1+1] => [3]

+

([][[]]+[]) => (undefined+[]) => ("undefined"+"") => "undefined"
[+!+[]] => [+!0] => [+true] => [1]

+

([][[]]+[]) => (undefined+[]) => ("undefined"+"") => "undefined"
[!+[]+!+[]] => [!0+!0] => [true+true] => [1+1] => [2]

= "true"[3]+"undefined"[1]+"undefined"[2]
= "e"+"n"+"d"
= "end"
```
