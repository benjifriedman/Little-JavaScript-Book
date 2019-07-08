# Chapter 6. This in JavaScript

## Demystifying "this"

The `this` keyword in JavaScript is a thick mystery for beginners and a constant crux for more experience developers. `this` in fact is a moving target, something that could change during code execution, without any apparent reason. But let's see for a moment how the `this` keyword looks like in other programming languages. First, here's a JavaScript class:

```js
class Person {
  constructor(name) {
    this.name = name;
  }

  greet() {
    console.log("Hello " + this.name);
  }
}
```

We know from chapter 5 that there are no classes in JavaScript yet `this` appears like a reference to the class itself. Also, there seems to be a parallel in Python classes called `self`:


```python
class Person:
    def __init__(self, name):
        self.name = name
    
    def greet(self):
        return 'Hello' + self.name
```

In a Python class `self` represent the class's instance: i.e. the new object that gets created starting from the class:

```python
me = Person('Valentino')
```

There is something similar in PHP too:

```php
class Person {
    public $name; 

    public function __construct($name){
        $this->name = $name;
    }

    public function greet(){
        echo 'Hello ' . $this->name;
    }
}
```

Here `$this` is the actual class instance. If we take again our JavaScript class for creating two new objects from it we can see that whenever I call `object.name` I get the correct string back:

```js
class Person {
  constructor(name) {
    this.name = name;
  }

  greet() {
    console.log("Hello " + this.name);
  }
}

const me = new Person("Valentino");
console.log(me.name); // 'Valentino'

const you = new Person("Tom");
console.log(you.name); // 'Tom'
```

Seems to make sense. JavaScript is like Python, Java, PHP and `this` appears to point to the actual class instance? Not really. Let's not forget that JavaScript is not an object oriented language, plus it's really permissive and dynamic, and there are no real classes. `this` has nothing to do with classes and I can prove the point with a simple JavaScript function (try it a browser):

```js
function whoIsThis() {
  console.log(this);
}

whoIsThis();
```

What's the output? Spoiler: when a JavaScript function runs in the so called global context `this` will be a reference to said global. And when running in a browser the global points to `window`.

## Rule number 1: default binding

If you run the following code in a browser:

```js
function whoIsThis() {
  console.log(this);
}

whoIsThis();
```

you'll see the output:

```js
Window {postMessage: ƒ, blur: ƒ, focus: ƒ, close: ƒ, parent: Window, …}
```

Surprising? We're not inside any class and `this` is still there. That's because `this` is always present in regular functions and could assume many forms depending on how the function is called. When a function is called in the global environment it will always point its `this` to the global object, `window` in our case. That's the rule number 1 of `this` in JavaScript and it's called **default binding**. Default binding is like a fallback and most of the times it's undesirable. Any function running in the global environment can "pollute" global variables and wreck havoc to your code. Consider the following example:

```js
function firstDev() {
  window.globalSum = function(a, b) {
    return a + b;
  };
}

function nastyDev() {
  window.globalSum = null;
}

firstDev();
nastyDev();
var result = firstDev();
console.log(result);

// Output: undefined
``` 

The first developer creates a global variable named `globalSum` and assigns a function to it. Later on another developer assigns `null` to the same variable causing malfunction in the code. Messing with global variables is always risky and for this reason JavaScript gained a "safe mode" years ago: strict mode. Strict mode is enabled by putting `"use strict";` at the top of every JavaScript file. Strict mode has many positive effects and one of them is the neutralization of the default binding. In strict mode `this` is `undefined` when trying to access it from the global context:

```js
"use strict";

function whoIsThis() {
  console.log(this);
}

whoIsThis();

// Output: undefined
```

Strict mode makes your JavaScript code more secure and (almost) free from silly bugs like that. So to recap, default binding is the first rule of `this` in JavaScript: when the engine can't figure out what `this` is it falls back to the global object. But that's only half of the story because there are 3 other rules for `this`. Let's see them together.

## Rule number 2: implicit binding

"Implicit binding" is an intimidating term, but the theory behind it is not so complicated. It all narrows down to objects. It's no surprise that JavaScript objects can contain functions:

```js
var widget = {
  items: ["a", "b", "c"],
  printItems: function() {
    console.log(this.items);
  }
};
```

When a function is assigned as an object's property then that object becomes the "home" in which the function runs. In other words the `this` keyword inside the function will point automatically to that object. That's rule number 2 of `this` in JavaScript and goes under the name of **implicit binding**. Implicit binding is in action even when we call a function in the global context:

```js
function whoIsThis() {
  console.log(this);
}

whoIsThis();
```

You can't tell from the code but the JavaScript engine assigns the function to a new property on the global object `window`. Under the hood it's like:

```js
window.whoIsThis = function() {
  console.log(this);
};
```

You can easily confirm the assumption. Run the following code in a browser:

```js
function whoIsThis() {
  console.log(this);
}

console.log(typeof window.whoIsThis)
```

and you'll get back "function". And this point you might be asking: what's the real rule for `this` inside a global function? It's called default binding but in reality is more like of an implicit binding. Confusing right? It's JavaScript after all! Just remember that the JavaScript engine always falls back to the global `this` when in doubt about the context (default binding). On the other hand when a function is defined inside a JavaScript object and is called as part of that object then there is no other way around: `this` refers to the host object (implicit binding). And now let's see what is the third rule for `this` in JavaScript.