# Chapter 4. Closures and modules in JavaScript

## Global madness

Global variables are a plague in software engineering. We've been taught to avoid them at all costs although globals are useful in some situations. For example when working with JavaScript in the browser we have access to the global `window` object. `window` has a lot of useful methods like:

```js
window.alert('Hello world'); // Shows an alert
window.setTimeout(callback, 3000); // Delay execution
window.fetch(someUrl); // make XHR requests
window.open(); // Opens a new tab
```

These methods are also available globally like so:

```js
alert('Hello world'); // Shows an alert
setTimeout(callback, 3000); // Delay execution
fetch(someUrl); // make XHR requests
open(); // Opens a new tab
```

That's handy. Redux is another example of "good" globals: the entire application's state is stored in a single JavaScript object which is accessible from the entire app (through Redux). But when writing JavaScript code in a team where maybe 50 developers are supposed to ship features ... how do you cope with code like this:

```js
var arr = [];

function addToArr(element) {
  arr.push(element);
  return element + " added!";
}
```

What are the odds of a colleague creating a new global array named `arr` in another file? Very high if you ask me! Another reason why globals in JavaScript are really bad is that the engine is kind enough to create global variables for us. What I mean is that if you forgot to put `var` before a variable's name, like so:

```js
name = "Valentino";
```

the engine creates a global variable for you! Shudder ... Even worst when the "unintended" variable is created inside a function:

```js
function doStuff() {
  name = "Valentino";
}

doStuff();

console.log(name); // "Valentino"
```

An innocent function ended up polluting the global environment. Luckily there is a way for neutralizing this behavior with "strict mode". The statement `"use strict"` at the very top of every JavaScript file is enough to avoid silly mistakes:

```js
"use strict";

function doStuff() {
  name = "Valentino";
}

doStuff();

console.log(name); // ReferenceError: name is not defined
```

Always use strict! Errors like that are a solved problem these days. Developers are becoming more and more self-disciplined (I hope so) but back in the old days global variables and procedural code was the norm in JavaScript. So we had to find a way for solving the "global" problem and luckily JavaScript has always had a built-in mechanism for that.

## Demystifying closures

So how do we protect our global variables from prying eyes? Let's start with a naive solution, moving `arr` inside a function:

```js
function addToArr(element) {
  var arr = [];
  arr.push(element);
  return element + " added to " + arr;
}
```

Seems reasonable but the result isn't what I was expecting:

```js
var firstPass = addToArr("a");
var secondPass = addToArr("b");
console.log(firstPass); // a added to a
console.log(secondPass); // a added to a
```

Ouch. `arr` keeps resetting at every function invocation. It is a local variable now, while in the first example it was declared as a global. Turns out global variables are "live" and they're not destroyed by the JavaScript engine. Local variables on the other hand are wiped out by the engine as part of a process called garbage collection (I will spare you the nitty gritty but here's an interesting article on the subject: [A tour of V8: Garbage Collection](http://jayconrod.com/posts/55/a-tour-of-v8-garbage-collection)). Seems there is no way to keep local variables from being destroyed? Or there is? Would a closure help? But what's a closure anyway?

It should be no surprise to you that JavaScript functions can contain other functions. What follows is perfectly valid code:

```js
function addToArr(element) {
  var arr = [];

  function push() {
    arr.push(element);
  }

  return element + " added to " + arr;
}
```

The code does nothing but ... ten years ago I was trying to solve a similar problem. I wanted to distribute a simple, self-contained functionality to my colleagues. No way I could handle them a JavaScript file with a bunch of global variables. I had to find a way for protecting those variables while keeping some local state inside the function. I tried literally everything until out of desperation I did some research and come up with something like this:

```js
function addToArr(element) {
  var arr = [];

  return function push() {
    arr.push(element);
    console.log(arr);
  };

  //return element + " added to " + arr;
}
```

The outer function becomes a mere container, returning another function. The second return statement is commented because that code will never be reached. At this point we know that the result of a function invocation can be saved inside a variable like so:

```js
var result = addToArr();
```

Now `result` becomes an executable JavaScript function:

```js
var result = addToArr();
result("a");
result("b");
```

There's just a fix to make, moving the parameter "element" from the outer to the inner function:

```js
function addToArr() {
  var arr = [];

  return function push(element) {
    arr.push(element);
    console.log(arr);
  };

  //return element + " added to " + arr;
}
```

And that's where the magic happens. Here's the complete code:

```js
function addToArr() {
  var arr = [];

  return function push(element) {
    arr.push(element);
    console.log(arr);
  };

  //return element + " added to " + arr;
}

var result = addToArr();
result("a"); // [ 'a' ]
result("b"); // [ 'a', 'b' ]
```

Bingo! What's this sorcery? It's called JavaScript closure: a function which is able to remember its environment. For doing so the inner function must be the return value of an enclosing (outer) function. This programming technique is also called factory function in nerd circles. The code can be tweaked a bit, maybe a better naming for the variables, and the inner function can be anonymous if you want:


```js
function addToArr() {
  var arr = [];

  return function(element) {
    arr.push(element);
    return element + " added to " + arr;
  };
}

var closure = addToArr();
console.log(closure("a")); // a added to a
console.log(closure("b")); // a added to a,b
```

Should be clear now that "closure" is the inner function. But one question needs to be addressed: why are we doing this? What's the real purpose of a JavaScript closure?

## The need for closures

Besides mere "academic" knowledge, JavaScript closures are useful for a number of reasons. They help in:

- giving privacy to otherwise global variables
- preserving variables (state) between function calls

One of the most interesting application for closures in JavaScript is the [module pattern](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#revealingmodulepatternjavascript). Before ECMAScript 2015 which introduced ES Modules there was no way for modularizing JavaScript code and give "privacy" to variables and methods other than wrapping them inside functions. Closures, paired with immediately invoked function expressions were and still are the most natural solution for structuring JavaScript code. A JavaScript module in it's simplest form is a variable capturing the result of an IIFE (an immediately invoked function expression):

```js
var Person = (function() {
  // do stuff
})();
```

Inside the module you can have "private" variables and methods:

```js
var Person = (function() {
  var person = {
    name: "",
    age: 0
  };

  function setName(personName) {
    person.name = personName;
  }

  function setAge(personAge) {
    person.age = personAge;
  }
})();
```

From the outside we can't access `person.name` or `person.age` nor we can call `setName` or `setAge`. Everything inside the module is "private". If we want to expose our methods to the public we can return an object containing references to private methods:

```js
var Person = (function() {
  var person = {
    name: "",
    age: 0
  };

  function setName(personName) {
    person.name = personName;
  }

  function setAge(personAge) {
    person.age = personAge;
  }

  return {
    setName: setName,
    setAge: setAge
  };
})();
```

The `person` object is still hidden away but it's easy to add a method for returning the new person:

```js
var Person = (function() {
  var person = {
    name: "",
    age: 0
  };

  function setName(personName) {
    person.name = personName;
  }

  function setAge(personAge) {
    person.age = personAge;
  }

  function getPerson() {
    return person.name + " " + person.age;
  }

  return {
    setName: setName,
    setAge: setAge,
    getPerson: getPerson
  };
})();

Person.setName("Tom");
Person.setAge(44);
var person = Person.getPerson();
console.log(person); // Tom 44
```

This way `person` is still not directly accessible from the outside:

```js
console.log(Person.person); // undefined
```

The module pattern is not the only way for structuring JavaScript code. With objects we could achieve the same outcome:

```js
var Person = {
  name: "",
  age: 0,
  setName: function(personName) {
    this.name = personName;
  }
  // other methods here
};
```

But now internal properties are leaking as you would expect from a JavaScript object:

```js
var Person = {
  name: "",
  age: 0,
  setName: function(personName) {
    this.name = personName;
  }
  // other methods here
};

Person.setName("Tom");

console.log(Person.name); // Tom
```

That's one of the main selling point for modules. Another boon is that modules help in organizing JavaScript code in a way that it's both idiomatic and readable. In other words almost every JavaScript developer can look at the following code and guess its purpose:

```js
"use strict";

var Person = (function() {
  var person = {
    name: "",
    age: 0
  };

  function setName(personName) {
    person.name = personName;
  }

  function setAge(personAge) {
    person.age = personAge;
  }

  function getPerson() {
    return person.name + " " + person.age;
  }

  return {
    setName: setName,
    setAge: setAge,
    getPerson: getPerson
  };
})();
```

## Conclusions

Global variables are bad and you should avoid them whenever you can. Sometimes are useful but in JavaScript we must take extra care because the engine takes the liberty to create global variables out of thin air. A number of patterns emerged during the years for taming global variables, the module pattern being one of them. The module pattern builds on closures, an innate feature of JavaScript. A closure in JavaScript is a function able to "remember" its variable's environment, even between subsequent function calls. A closure is created when we return a function from another function, a pattern also known as "factory function".