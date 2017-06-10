---
title: What if JavaScript was a Java scripting language?
date: 2017-06-08 23:51:28
categories:
  - JavaScript
tags:
  - OOP
  - ES5
  - ES6
thumbnail: /images/javascript.png
---

*Meanwhile, in a parallel universe...*

JavaScript is a scripting language for the Java Platform and JS scripts should strictly apply the key concepts of **object-oriented programming** (OOP).

Our task is to model and implement a simple program that involves living beings. This is of course a vast topic and, for the sake of simplicity, we will not deal with peripheral concepts like generics, synchronized methods, collections or packages. This would bring us too far.

<!-- more -->

---

## Inheritance

Java is a class-based language where inheritance is performed using the `extends` keyword. This is much more complicated in JavaScript...

### ES5

ECMAScript 5 does not have built-in classes. JavaScript is a **prototype-based language**, but classes can be emulated thanks to **constructor functions**. Inheritance is possible, but should be done through a manipulation of prototypes:

```JavaScript
// Parent class
function LivingBeing(species) { // constructor
  this.species = species; // attribute
}

LivingBeing.live = function() { // static method
  return 'Life is life!';
};

// Child class
function Person(firstname, lastname) { // constructor
  LivingBeing.call(this, 'Homo sapiens'); // super("Homo sapiens")
  this.firstname = firstname; // attribute
  this.lastname = lastname; // attribute
}
// extends
Person.prototype = Object.create(LivingBeing.prototype);
Person.prototype.constructor = Person;

Person.prototype.sayHello = function () { // instance method
  return 'Hello, ' + this.firstname + ' ' + this.lastname + '!';
};

// --------------------

var anonymousDog = new LivingBeing('Canis lupus');
console.log(anonymousDog.species); // Canis lupus

var john = new Person('John', 'Doe');
console.log(john.sayHello(), '(' + john.species + ')'); // Hello, John Doe! (Homo sapiens)

console.log(LivingBeing.live()); // Life is life!
```

### ES6

ECMAScript 6 brought syntactic sugar to work more easily with so-called classes and inheritance. This new syntax hides the true prototypal nature of JavaScript, but now the language looks much more familiar for Java developers:

```JavaScript
// Parent class
class LivingBeing {
  constructor(species) {
    this.species = species; // attribute
  }

  static live() { // static method
    return 'Life is life!';
  }
}

// Child class
class Person extends LivingBeing {
  constructor(firstname, lastname) {
    super('Homo sapiens');
    this.firstname = firstname; // attribute
    this.lastname = lastname; // attribute
  }

  sayHello() { // instance method
    return `Hello, ${this.firstname} ${this.lastname}!`;
  }
}

// --------------------

let anonymousDog = new LivingBeing('Canis lupus');
console.log(anonymousDog.species); // Canis lupus

let john = new Person('John', 'Doe');
console.log(john.sayHello(), `(${john.species})`); // Hello, John Doe! (Homo sapiens)

console.log(LivingBeing.live()); // Life is life!
```

---

## Composition

In OOP, a best practice is to favor **composition over inheritance**.

### ES5

Composition in ECMAScript 5 is not so different from composition in Java. A specific class may reference another class using an instance of the latter in one of its properties.

```JavaScript
// Class to be referenced
function LivingBeing(species) { // constructor
  this.species = species; // attribute
}

LivingBeing.live = function() { // static method
  return 'Life is life!';
};

// Main class
function Person(firstname, lastname) { // constructor
  this.livingBeing = new LivingBeing('Homo sapiens'); // attribute (composition)
  this.firstname = firstname; // attribute
  this.lastname = lastname; // attribute
}

Person.prototype.sayHello = function () { // instance method
  return 'Hello, ' + this.firstname + ' ' + this.lastname + '!';
};

// --------------------

var  anonymousDog = new LivingBeing('Canis lupus');
console.log(anonymousDog.species);

var john = new Person('John', 'Doe');
console.log(john.sayHello(), '(' + john.livingBeing.species + ')'); // Hello, John Doe! (Homo sapiens)

console.log(LivingBeing.live()); // Life is life!
```

### ES6

Composition in ECMAScript 6 is even closer to Java, thanks to the syntactic sugar that emulates classes natively:

```JavaScript
// Class to be referenced
class LivingBeing {
  constructor(species) {
    this.species = species; // attribute
  }

  static live() {
    return 'Life is life!';
  }
}

// Main class
class Person {
  constructor(firstname, lastname) {
    this.livingBeing = new LivingBeing('Homo sapiens'); // attribute (composition)
    this.firstname = firstname; // attribute
    this.lastname = lastname; // attribute
  }

  sayHello() {
    return `Hello, ${this.firstname} ${this.lastname}!`;
  }
}

// --------------------

let anonymousDog = new LivingBeing('Canis lupus');
console.log(anonymousDog.species);

let john = new Person('John', 'Doe');
console.log(john.sayHello(), `(${john.livingBeing.species})`); // Hello, John Doe (Homo sapiens)

console.log(LivingBeing.live()); // Life is life!
```

Composition works fine here, but conceptually, inheritance is more meaningful for this particular problem.

---

## Abstraction

Abstraction in Java is based on **abstract classes** and **interfaces**. JavaScript does not have a built-in abstraction mechanism, but like classes in ES5, this concept can be emulated.

### Abstract class

An abstract class is a class that cannot be instantiated, so we cannot create direct objects from an abstract class. In JavaScript, we can prevent instantiation of a particular class using conditions in the constructor.

An abstract class can contain abstract methods which are methods without implementation. In JavaScript, this can be reproduced quite easily if we declare a prototype method that only throws an error. It will remain "abstract" (it will throw an error) as long as it is not redefined in the prototype chain.

#### ES5

To create an abstract class in ES5 (here `LivingBeing`), the best test to check if the constructor has been called directly with `new` is `this.constructor === LivingBeing`. Be careful to use the right comparison operator because `this.constructor !== LivingBeing` would create (roughly) a **final class** (a class that cannot be extended)!

```JavaScript
// Abstract class
function LivingBeing(species) {
  if (this.constructor === LivingBeing) {
    throw new Error('You cannot instantiate an abstract class!');
  }
  this.species = species;
}

LivingBeing.prototype.communicate = function () { // abstract method
  throw new Error('You cannot call an abstract method!');
};

// Concrete class
function Person(firstname, lastname) {
  LivingBeing.call(this, 'Homo sapiens');
  this.firstname = firstname;
  this.lastname = lastname;
}
Person.prototype = Object.create(LivingBeing.prototype);
Person.prototype.constructor = Person;

Person.prototype.sayHello = function () { // concrete method
  return 'Hello, ' + this.firstname + ' ' + this.lastname + '!';
};

// --------------------

var john = new Person('John', 'Doe');
console.log(john.sayHello(), '(' + john.species + ')'); // Hello, John Doe! (Homo sapiens)

john.communicate(); // Error: Your cannot call an abstract method!
new LivingBeing('Canis lupus'); // Error: You cannot instantiate an abstract class!
```

#### ES6

With ES6, it is better to use `new.target` like this: `new.target.name === 'LivingBeing'`. Once again, be careful with the comparison operator: `new.target.name !== 'LivingBeing'` would create a final class.

```JavaScript
// Abstract class
class LivingBeing {
  constructor(species) {
    if (new.target.name === 'LivingBeing') {
      throw new Error('You cannot instantiate an abstract class!');
    }
    this.species = species;
  }

  communicate() { // abstract method
    throw new Error('You cannot call an abstract method!');
  }
}

// Concrete class
class Person extends LivingBeing {
  constructor(firstname, lastname) {
    super('Homo sapiens');
    this.firstname = firstname;
    this.lastname = lastname;
  }

  sayHello() { // concrete method
    return `Hello, ${this.firstname} ${this.lastname}`;
  }
}

// --------------------

let john = new Person('John', 'Doe');
console.log(john.sayHello(), `(${john.species})`); // Hello, John Doe! (Homo sapiens)

john.communicate(); // Error: You cannot call an abstract method!
new LivingBeing('Canis lupus'); // Error: You cannot instantiate an abstract class!
```

### Interface

Interfaces in Java are quite close to 100% abstract classes. They contain `public abstract` methods that must be overriden in classes which implement them.

#### ES5

A solution to get interfaces in ES5 is to create a custom `implements` function in `Function.prototype` and use object literals to declare interfaces.

The custom function must traverse the **own properties** of the interface passed as an argument and test if they exist in the current function instance (presumably a constructor function). If they do not exist, they should be created in the function's prototype. In this case, they remain abstract, meaning that they will throw an error if we try to use them before to override them.

```JavaScript
// Custom "implements" function
Function.prototype.implements = function (iface) {
  if (iface.toString() !== '[object Object]') {
    throw new Error('Invalid argument. An interface must be an object.');
  } else {
    for (var prop in iface) {
      if (iface.hasOwnProperty(prop)) {
        if (typeof iface[prop] === 'function' && !(prop in this.prototype)) {
          this.prototype[prop] = iface[prop];
        }
      }
    }
  }
};

// Interface
var LivingBeingInterface = {
  communicate: function () {
    throw new Error('You cannot call an abstract method!');
  },
  getSpecies: function () {
    throw new Error('You cannot call an abstract method!');
  }
};

// Class
Person.implements(LivingBeingInterface); // cool, isn't it?

function Person(firstname, lastname) {
  this.firstname = firstname;
  this.lastname = lastname;
  this.species = 'Homo sapiens';
}

Person.prototype.sayHello = function () {
  return 'Hello, ' + this.firstname + ' ' + this.lastname + '!';
};

Person.prototype.communicate = function () { // overrides the default implementation
  return 'blah blah blah';
};

// Person.prototype.getSpecies() is not overriden

// --------------------

var john = new Person('John', 'Doe');
console.log(john.sayHello()); // Hello, John Doe!
console.log(john.communicate()); // blah blah blah
console.log(john.getSpecies()); // Error: You cannot call an abstract method!
```

#### ES6

ES6 classes are like coherent blocks of methods and they are not hoisted. Thus, contrary to ES5, we must call our `implements` method after the class declaration.

```JavaScript
// Custom "implements" function
Function.prototype.implements = function (iface) {
  if (iface.toString() !== '[object Object]') {
    throw new Error('Invalid argument. An interface must be an object.');
  } else {
    for (let prop in iface) {
      if (iface.hasOwnProperty(prop)) {
        if (typeof iface[prop] === 'function' && !(prop in this.prototype)) {
          this.prototype[prop] = iface[prop];
        }
      }
    }
  }
};

// Interface
const LivingBeingInterface = {
  communicate() {
    throw new Error('You cannot call an abstract method!');
  },
  getSpecies() {
    throw new Error('You cannot call an abstract method!');
  }
};

// Class
class Person {
  constructor (firstname, lastname) {
    this.firstname = firstname;
    this.lastname = lastname;
    this.species = 'Homo sapiens';
  }

  sayHello() {
    return `Hello, ${this.firstname} ${this.lastname}!`;
  }

  communicate() { // overrides the default implementation
    return 'blah blah blah';
  }

  // Person.prototype.getSpecies() is not overriden
}
Person.implements(LivingBeingInterface); // cool, isn't it?

// --------------------

let john = new Person('John', 'Doe');
console.log(john.sayHello()); // Hello, John Doe!
console.log(john.communicate()); // blah blah blah
console.log(john.getSpecies()); // Error: You cannot call an abstract method!
```

---

## Polymorphism

As you may know, Java is strongly typed. On the contrary, JavaScript is loosely typed. But, in Java, if you create an instance of `Person` that extends `LivingBeing`, this instance is of type `Person` **and** `LivingBeing`. So if you iterate through a collection of `LivingBeing` objects containing people, animals or plants, and call the same method on each object, you will have specific results for each living being thanks to a **dynamic binding**. JavaScript works this way too if we use the **prototype chain** properly.

### ES5

As an example, we could override the original `Object.prototype.toString()` method for people and dogs.

In JavaScript, classes and their instances are objects. By default, when we call `toString()` on an object, we get `[object Object]`. `Person` or `Dog` instances are also indirect instances of `LivingBeing`, so we could define several `toString()` methods in the prototype chain.

```JavaScript
function LivingBeing(species) {
  if (this.constructor === LivingBeing) {
    throw new Error('You cannot instantiate an abstract class!');
  }
  this.species = species;
}

// Overrides Object.prototype.toString()
LivingBeing.prototype.toString = function () {
  // returns the species instead of [object Object]
  return 'Species: ' + this.species;
};

function Person(firstname, lastname) {
  LivingBeing.call(this, 'Homo sapiens');
  this.firstname = firstname;
  this.lastname = lastname;
}
Person.prototype = Object.create(LivingBeing.prototype);
Person.prototype.constructor = Person;

Person.prototype.sayHello = function () {
  return 'Hello, ' + this.firstname + ' ' + this.lastname + '!';
};

// Overrides LivingBeing.prototype.toString()
Person.prototype.toString = function () {
  var str = '\n';
  str += 'First name: ' + this.firstname + '\n';
  str += 'Last name: ' + this.lastname + '\n';
  str += LivingBeing.prototype.toString.call(this); // super.toString()
  return str; // returns a summary instead of the species
};

function Dog(name) {
  LivingBeing.call(this, 'Canis lupus');
  this.name = name;
}

// Overrides LivingBeing.prototype.toString()
Dog.prototype.toString = function () {
  var str = '\n';
  str += 'Name: ' + this.name + '\n';
  str += LivingBeing.prototype.toString.call(this);
  return str; // returns a summary instead of the species
};

// --------------------

var john = new Person('John', 'Doe');
console.log(john.sayHello(), '(' + john.species + ')'); // Hello, John Doe! (Homo sapiens)

var fido = new Dog('Fido');
console.log(fido.name, '(' + fido.species + ')'); // Fido (Canis lupus)

var livingBeings = [john, fido];

for (var i = 0; i < livingBeings.length; i++) {
  console.log(livingBeings[i].toString());
}

// First name: John
// Last name: Doe
// Species: Homo sapiens

// Name: Fido
// Species: Canis lupus
```

### ES6

It is not so different with ES6, except that we can use classes, string interpolation and a `for...of` loop for convenience:

```JavaScript
class LivingBeing {
  constructor(species) {
    if (new.target.name === 'LivingBeing') {
      throw new Error('You cannot instantiate an abstract class!');
    }
    this.species = species;
  }

  toString() { // overrides Object.prototype.toString()
    // returns the species instead of [object Object]
    return `Species: ${this.species}`;
  }
}

class Person extends LivingBeing {
  constructor(firstname, lastname) {
    super('Homo sapiens');
    this.firstname = firstname;
    this.lastname = lastname;
  }

  sayHello() {
    return `Hello, ${this.firstname} ${this.lastname}!`;
  }

  toString() { // overrides LivingBeing.prototype.toString()
    return `
First name: ${this.firstname}
Last name: ${this.lastname}
${super.toString()}`; // returns a summary instead of the species
  }
}

class Dog extends LivingBeing {
  constructor(name) {
    super('Canis lupus');
    this.name = name;
  }

  toString() { // overrides LivingBeing.prototype.toString()
    return `
Name: ${this.name}
${super.toString()}`; // returns a summary instead of the species
  }
}

// --------------------

let john = new Person('John', 'Doe');
console.log(john.sayHello(), `(${john.species})`); // Hello, John Doe! (Homo sapiens)

let fido = new Dog('Fido');
console.log(fido.name, `(${fido.species})`); // Fido (Canis lupus)

let livingBeings = [john, fido];

for (const being of livingBeings) {
  console.log(being.toString());
}

// First name: John
// Last name: Doe
// Species: Homo sapiens

// Name: Fido
// Species: Canis lupus
```

---

## Encapsulation

Encapsulation makes it possible to restrict the access to class members through **access level modifiers** and **getters/setters**.

### Access level modifiers

Java provides useful keywords like `public`, `protected` or `private` to modify access level of class attributes and methods. Unfortunately, JavaScript does not have these access modifiers (they are reserved words but are still unused).

#### ES5

To reproduce the effect of `public`, `protected` or `private`, we have to play with the **scope chain**, **closures** and **IIFEs** (Immediately-Invoked Function Expressions).

- A **public** member is basically a prototype property.
- A **protected** member is a simple variable or function declared inside a surrounding function and that can be accessed from `LivingBeing` and `Person`.
- A **private** member is a simple variable or function declared inside a nested function and that can be accessed from `Person` only.

```JavaScript
var Person = (function () { // public class
  var species = ''; // protected attribute

  function LivingBeing(s) { // protected inner class
    species = s;
  }

  var Person = (function () {
    var firstname = '', // private attribute
        lastname = ''; // private attribute

    function Person(first, last) { // private inner class
      LivingBeing.call(this, 'Homo sapiens');
      firstname = first;
      lastname = last;
    }
    Person.prototype = Object.create(LivingBeing.prototype);
    Person.prototype.constructor = Person;

    Person.prototype.sayHello = function () { // public method
      return 'Hello, ' + firstname + ' ' + lastname + '! (' + species + ')';
    };

    return Person;
  })();

  return Person;
})();

// --------------------

var john = new Person('John', 'Doe');
console.log(john.sayHello()); // Hello, John Doe! (Homo sapiens)
console.log(john.firstname, john.lastname, john.species); // undefined undefined undefined
```

#### ES6

It may be surprising, but ES6 classes do not even have access level modifiers. Here we can use arrow functions for convenience:

```JavaScript
const Person = (() => { // public class
  let species = ''; // protected attribute

  class LivingBeing { // protected inner class
    constructor(s) {
      species = s;
    }
  }

  const Person = (() => {
    let firstname = '', // private attribute
        lastname = ''; // private attribute

    class Person extends LivingBeing { // private inner class
      constructor (first, last) {
        super('Homo sapiens');
        firstname = first;
        lastname = last;
      }

      sayHello() { // public method
        return `Hello, ${firstname} ${lastname}! (${species})`;
      }
    }

    return Person;
  })();

  return Person;
})();

// --------------------

let john = new Person('John', 'Doe');
console.log(john.sayHello()); // Hello, John Doe! (Homo sapiens)
console.log(john.firstname, john.lastname, john.species); // undefined undefined undefined
```

### Getters/Setters

When some fields are private or protected, they cannot be accessed from the outside. To read them or modify them, we need public methods: **getters** and **setters**.

#### ES5

Getters and setters can be defined with the old (now deprecated) `__defineGetter__` or `__defineSetter__` respectively, but this is much better to create a regular function with `get` or `set` in the name.

```JavaScript
var Person = (function () { // public class
  var species = ''; // protected attribute

  function LivingBeing(s) { // protected inner class
    species = s;
  }

  LivingBeing.prototype.getSpecies = function () { // public getter
    return species;
  };

  LivingBeing.prototype.setSpecies = function (s) { // public setter
    species = s;
  };

  var Person = (function () {
    var firstname = '', // private attribute
        lastname = ''; // private attribute

    function Person(first, last) { // private inner class
      LivingBeing.call(this, 'Homo sapiens');
      firstname = first;
      lastname = last;
    }
    Person.prototype = Object.create(LivingBeing.prototype);
    Person.prototype.constructor = Person;

    Person.prototype.sayHello = function () { // public method
      return 'Hello, ' + firstname + ' ' + lastname + '! (' + species + ')';
    };

    Person.prototype.getFirstname = function () { // public getter
      return firstname;
    };

    Person.prototype.setFirstname = function (first) { // public setter
      firstname = first;
    };

    Person.prototype.getLastname = function () { // public getter
      return lastname;
    };

    Person.prototype.setLastname = function (last) { // public setter
      lastname = last;
    };

    return Person;
  })();

  return Person;
})();

var john = new Person('John', 'Doe');
console.log(john.sayHello()); // Hello, John Doe! (Homo sapiens)
console.log(john.getFirstname(), john.getLastname(), '(' + john.getSpecies() + ')'); // John Doe (Homo sapiens)

john.setFirstname('Jane');
console.log(john.getFirstname(), john.getLastname()); // Jane Doe
```

#### ES6

`get` and `set` keywords, which were available in ES5 object literals, are now available in ES6 classes. They can be used like so:

```JavaScript
const Person = (() => { // public class
  let species = ''; // protected attribute

  // Abstract class
  class LivingBeing { // protected inner class
    constructor(s) {
      species = s;
    }

    get species() { // public getter
      return species;
    }

    set species(s) { // public setter
      species = s;
    }
  }

  // Concrete class
  const Person = (() => {
    let firstname = '', // private attribute
        lastname = ''; // private attribute

    class Person extends LivingBeing { // private inner class
      constructor (first, last) {
        super('Homo sapiens');
        firstname = first;
        lastname = last;
      }

      sayHello() { // public method
        return `Hello, ${firstname} ${lastname}! (${species})`;
      }

      get firstname() { // public getter
        return firstname;
      }

      set firstname(first) { // public setter
        firstname = first;
      }

      get lastname() { // public getter
        return lastname;
      }

      set lastname(last) { // public setter
        lastname = last;
      }
    }

    return Person;
  })();

  return Person;
})();

let john = new Person('John', 'Doe');
console.log(john.sayHello()); // Hello, John Doe! (Homo sapiens)
console.log(john.firstname, john.lastname, `(${john.species})`); // John Doe (Homo sapiens)

john.firstname = 'Jane';
console.log(john.firstname, john.lastname); // Jane Doe
```

---

## Back to reality

JavaScript is **NOT** Java. Trying to imitate Java with JavaScript is not necessarily a good idea in real life, but this is extremely interesting for educational purposes.

JavaScript is an object-oriented language (in spite of its prototypal nature), but it is much more flexible and permissive than Java. It does not matter if you do not have a strict class hierarchy. It does not matter if you do not have abstraction. It does not even matter if you do not have a strict encapsulation.

For Java developers who really want to feel at home with JavaScript, use [TypeScript](https://www.typescriptlang.org/).
