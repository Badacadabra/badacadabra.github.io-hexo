---
title: JavaScript one-liners to show off
tags:
  - ES5
  - ES6
categories:
  - JavaScript
thumbnail: /images/javascript.png
date: 2017-06-17 22:44:09
---


JavaScript is sometimes quite verbose, isn't it? If conciseness and effectiveness matter to you, a nice programming motto might be: *write less, do more*. (jQuery, I'm looking at you!)

Writing complex and powerful code on a single line is often an exciting experience where the only tangible limit is readability. One-liners in JavaScript represent a key topic that has been boosted by the arrival of ECMAScript 2015 (ES6). We cover this right here.

<!-- more -->

## Ancestral techniques (ES5)

ECMAScript 5 is definitely not the best specification to show off with one-liners. In fact, one-liners are rarely used in ES5 because they tend to be too long due to syntax limitations. ES5 one-liners are generally allowed by simple methods in `Array.prototype` and/or `String.prototype` (e.g. `concat`, `reverse`, `slice`, `join`, `replace`, etc.).

Here is a "WTF example" to illustrate what we are talking about:

```JavaScript
console.log(['foo', 'bar'].concat(['baz', 'qux'].reverse().slice(1)).join(' ').replace(/a/i, 'ee').toUpperCase()); // FOO BEER BAZ
```

Nevertheless, we can go further with ES5 thanks to some well-known JS techniques and idioms that make it possible to write more concise code...

### Currying

Currying is generally used to reduce functions arity (number of arguments), but depending on your code structure, it may be used as a hack to get a "light" invocation pattern.

This is the classical way of doing things:

```JavaScript
function foo() {
  return 'Foo';
}

function bar() {
  return 'Bar';
}

function baz() {
  return 'Baz';
}
```

Calling each function on a single line gives us the following code:

```JavaScript
foo(); bar(); baz();
```

Not so cool, actually. With currying, we could transform our code and make it look like this:

```JavaScript
function foo() {
  var foo = 'Foo';
  console.log(foo);
  return function bar() {
    var bar = 'Bar';
    console.log(bar);
    return function baz() {
      var baz = 'Baz';
      console.log(baz);
      return foo + ' ' + bar + ' ' + baz;
    }
  }
}
```

In this particular situation, invoking each function would be more "one-liner compliant":

```JavaScript
foo()()();
```

That's it! `foo()()()` is much more concise because...

- `foo()` invokes `foo` and `foo` returns `bar`
- `foo()()` invokes `bar` and `bar` returns `baz`
- `foo()()()` invokes `baz` and `baz` returns a string.

Currying is nice, but chances are you will prefer to use the classical approach which is far more readable in most cases...

### Method chaining

Method chaining is a very popular pattern in JavaScript which is frequently implemented in JS libraries. In fact, it is an interesting pattern to flatten method invocation and enable developers to chain long operations (possibly on a single line).

Let's take a very basic and naive example: a counter "class" that has a single attribute (`val`) and a single prototype method (`add`):

```JavaScript
function Counter() {
  this.val = 0;
}

Counter.prototype.add = function () {
  this.val++;
};
```

If we want a counter to be incremented three times, we will end up with the following line:

```JavaScript
var c = new Counter(); c.add(); c.add(); c.add(); console.log(c.val); // 3
```

This code works, but writing it on a single line looks terrible. With method chaining, this is obviously much prettier:

```JavaScript
console.log(new Counter().add().add().add().val); // 3
```

**To get this amazing behavior, `Counter.prototype.add` just have to return `this`!**

### Immediately-Invoked Function Expressions (IIFEs)

Regular functions must be explicitly invoked *before* (thanks to hoisting) or *after* they have been declared to perform some action:

**Before**

```JavaScript
console.log(foo()); // Foo

function foo() {
  return 'Foo';
}
```

**After**

```JavaScript
function foo() {
  return 'Foo';
}

console.log(foo()); // Foo
```

Creating a one-liner with such a function is trivial:

```JavaScript
// Invocation first
console.log(foo()); function foo() { return 'Foo'; } // Foo

// Declaration first
function foo() { return 'Foo'; }; console.log(foo()); // Foo
```

Well... It is not fantastic, is it?

Using an IIFE, that is to say a function expression that calls itself,  would be more convenient:

```JavaScript
console.log((function () { return 'Foo'; })()); // Foo
```

### The ternary operator

This operator can be used as a shortcut for `if`/`else` statements. For instance, this code is a bit too verbose to be refactored on a single line:

```JavaScript
if (Array.isArray(['foo', 'bar'])) {
  console.log('OK');
} else {
  console.log('KO');
}
```

If we try, we can see that the result is painful to read:

```JavaScript
if (Array.isArray(['foo', 'bar'])) { console.log('OK'); } else { console.log('KO'); } // OK
```

With the ternary operator, we do not have this problem:

```JavaScript
console.log(Array.isArray(['foo', 'bar']) ? 'OK' : 'KO'); // OK
```

The ternary operator is, to some extent, a powerful factory of one-liners. But, be careful. Any abuse make the code unreadable:

```JavaScript
console.log('Go!' ? '' ? 'Next' ? 'It works!' : 'It does not work!' : 'KO' : 'Oops!'); // KO
```

### Short-circuit operators

Just like the ternary operator, these operators (`&&` and `||`) are mostly used in a boolean context and are perfect to replace `if`/`else` statements that would be hard to read in a one-liner.

```JavaScript
function isEmptyObject(obj) { return typeof obj === 'object' && obj.toString() === '[object Object]' && !Object.getOwnPropertyNames(obj).length }
```

But since any value can be **truthy** or **falsy** in JavaScript, expressions using short-circuit operators may also return non-boolean values...

```JavaScript
console.log(true && 1 && 'hello' && {} && ['foo', 'bar']); // ['foo', 'bar']
console.log(false || 0 || '' || [].toString() || {foo: 'Foo'}); // {foo: 'Foo'}
console.log(true && 0 || 'hello' || [] && {}); // hello
```

`&&`:

- When the left operand is truthy, the next operand is evaluated
- When the left operand is falsy, the next operand is not evaluated and the current value is returned

`||`:

- When the left operand is truthy, the next operand is not evaluated and the current value is returned
- When the left operand is falsy, the next operand is evaluated

## Modern techniques (ES6)

ES6 has been designed with conciseness in mind and it brought a couple of useful features that tend to encourage one-liners:

- **Arrow functions**
- **The spread operator**
- **Destructuring with enhanced object literals**

### Map-Filter-Reduce

For this example, we have a simple array: `[1, 2, 3]`. We want to:

1. Multiply by 2 each number in this array (`map`)
2. Keep the resulting numbers that are lower than 5 (`filter`)
3. Add up the remaining numbers (`reduce`)

With ES5, a one-liner would be utterly awful:

```JavaScript
[1, 2, 3].map(function (v) { return v * 2 }).filter(function (v) { return v < 5 }).reduce(function (a, v) { return a + v }); // 6
```

Thanks to ES6 **arrow functions**, it becomes much more stylish:

```JavaScript
[1, 2, 3].map(v => v * 2 ).filter(v => v < 5).reduce((a, v) => a + v); // 6
```

### Iterable protocol

ES6 allows us to perform complex transformations on iterable data structures with ease. Let's do something totally useless in practice, but relevant for learning purposes:

```JavaScript
Array.from(new Set(...new Map([[...new Set(['foo', 'bar'])].toString().split(',')]))); // ['foo', 'bar']
```

In this example, we did something really complex. Without the **spread operator**, it would have been much more verbose. Let's try to replace it with `for...of`:

```JavaScript
let set1 = new Set(['foo', 'bar']),
    arr1 = [];

for (const el of set1) {
  arr1.push(el);
}

let set2 = new Set(),
    arr2 = arr1.toString().split(','),
    map = new Map([arr2]);

for (const el of map) {
  set2.add(el[0]).add(el[1]);
}

console.log(Array.from(set2)); // ['foo', 'bar']
```

The thing is that... This verbose version is probably more readable than the one-liner. But if we take a simpler example, the spread operator looks fabulous:

```JavaScript
Math.max(...[4, 8, 15, 16, 23, 42]); // 42
```

### Data extraction

Imagine that you have the following data structure:

```JavaScript
var obj = {
  foo: ['Foo', 'Bar', 'Baz'],
  qux: 'Qux',
  quux: 'Quux'
};
```

With ES5, if you want to extract everything on a single line, it will most likely be a nightmare:

```JavaScript
var foo = obj.foo[0]; var bar = obj.foo[1]; var baz = obj.foo[2]; var qux = obj.qux; var quux = obj.quux;
```

But thanks to **destructuring with enhanced object literals**, it can be achieved with comfort:

```JavaScript
let {foo: [foo, bar, baz], qux, quux} = obj;
```

In both cases, `console.log(foo, bar, baz, qux, quux)` will give `Foo Bar Baz Qux Quux`.

## Practical examples

### Finding the longest word in a given string

```JavaScript
// One-liner
let longestWord = str => str.split(' ').sort((a, b) => b.length - a.length)[0];

// How to use it
console.log(longestWord('This is Sparta!')); // Sparta!
```

### Capitalizing the first letter of each line (for multiline strings)

```JavaScript
// One-liner
let capitalize = str => str[0].split('\n').map(v => `${v.charAt(0).toUpperCase()}${v.substr(1)}`).join('\n');

// How to use it
console.log(capitalize`roses are red,
violets are blue,
sugar is sweet,
and so are you.`);

/*
  Roses are red,
  Violets are blue,
  Sugar is sweet,
  And so are you.
*/
```

### Removing duplicates from an array

```JavaScript
// One-liner
let removeDuplicates = arr => [...new Set(arr)];

// How to use it
console.log(removeDuplicates(['foo', 'bar', 'bar', 'foo', 'bar'])); // ['foo', 'bar']
```

### Checking if all elements of an array are in another one

```JavaScript
// One-liner
let similarArrays = (arr1, arr2)  => arr1.every(v1 => arr2.some(v2 => v1 === v2));

// How to use it
console.log(similarArrays(['foo', 3], [3, false, {}, 'foo'])); // true
console.log(similarArrays(['foo', 3], [1337, 'bar', 3, 'qux'])); // false

// Beware: objects are compared by reference...
```

### Swaping values of object properties

```JavaScript
// One-liner
let swapPropsValues = (o, p1, p2) => o[p1] && o[p2] && ([o[p1], o[p2]] = [o[p2], o[p1]]);

// How to use it
let obj = {foo: 'Foo', bar: 'Bar'};
swapPropsValues(obj, 'foo', 'bar');
console.log(obj); // {foo: 'Bar', bar: 'Foo'}
```
