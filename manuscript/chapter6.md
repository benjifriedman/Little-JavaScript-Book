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

## Rule number 3: explicit binding

There is no JavaScript developer that at some point in her/his career didn't see code like this:

```js
someObject.call(anotherObject);
Someobject.prototype.someMethod.apply(someOtherObject);
```

That's **explicit binding** in action. Another example of binding: with the rise of React as the UI library of choice it is common to see class methods "bound" to the class itself (more on that later):

```js
class Button extends React.Component {
  constructor(props) {
    super(props);
    this.state = { text: "" };
    // bounded method
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(() => {
      return { text: "PROCEED TO CHECKOUT" };
    });
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.text || this.props.text}
      </button>
    );
  }
}
```

Nowadays React hooks make classes almost unnecessary, yet there are a lot of "legacy" React components out there using ES2015 classes. One of the question most beginners ask is why we re-bind class methods in React with "bind"? Those three methods, `call`, `apply`, `bind` belong to `Function.prototype` and are used for the so called **explicit binding** (rule number 3). It's in the name: explicit binding means taking a function and forcing its home, also called context object, to be one provided by you. But why would I explicitly bind or re-bind a function? Consider some legacy JavaScript code wrote by a fictional, less JavaScript-savvy colleague some years ago:

```js
var legacyWidget = {
  html: "",
  init: function() {
    this.html = document.createElement("div");
  },
  showModal: function(htmlElement) {
    var newElement = document.createElement(htmlElement);
    this.html.appendChild(newElement);
    window.document.body.appendChild(this.html);
  }
};
```

`showModal` is a "method" bound to the object `legacyWidget`. It is implicitly bound (rule number 2), almost useless because apparently there is no way to change the HTML element (`this.html`) on which the modal should append the new element. `this.html` is hardcoded inside the method. No worries, we can resort to explicit binding for altering the `this` object on which `showModal` runs. It is a job for `call` which takes the new context object and a list of optional arguments. Now we can create a new "shiny" widget and provide a different HTML element as the starting point:

```js
var legacyWidget = {
  html: "",
  init: function() {
    this.html = document.createElement("div");
  },
  showModal: function(htmlElement) {
    var newElement = document.createElement(htmlElement);
    this.html.appendChild(newElement);
    window.document.body.appendChild(this.html);
  }
};

var shinyNewWidget = {
  html: "",
  init: function() {
    // A different HTML element
    this.html = document.createElement("section");
  }
};
```

At this point you can call `call` on the original method:

```js
var legacyWidget = {
  html: "",
  init: function() {
    this.html = document.createElement("div");
  },
  showModal: function(htmlElement) {
    var newElement = document.createElement(htmlElement);
    this.html.appendChild(newElement);
    window.document.body.appendChild(this.html);
  }
};

var shinyNewWidget = {
  html: "",
  init: function() {
    this.html = document.createElement("section");
  }
};

// Initialize the new widget with a different HTML element
shinyNewWidget.init();

// Run the original method with a new context object
legacyWidget.showModal.call(shinyNewWidget, "p");
```

If you're still confused about explicit binding think of it like a primitive style for reusing code. It looks hacky and buggy to me but that's the best you can do if you've got legacy JavaScript code to refactor. Also, the prototype system is a better candidate for structuring the above code but believe me, bad software like that still exists in production today. At this point you may wonder what `apply` and `bind` are. `apply` has the same effect of using `call` except that the former accepts an array of parameters while the latter expects the parameters to be given as a list. In other words `call` works with a list of parameters:

```js
var obj = {
  version: "0.0.1",
  printParams: function(param1, param2, param3) {
    console.log(this.version, param1, param2, param3);
  }
};

var newObj = {
  version: "0.0.2"
};

obj.printParams.call(newObj, "aa", "bb", "cc");
```

While `apply` expects an array of parameters:

```js
var obj = {
  version: "0.0.1",
  printParams: function(param1, param2, param3) {
    console.log(this.version, param1, param2, param3);
  }
};

var newObj = {
  version: "0.0.2"
};

obj.printParams.apply(newObj, ["aa", "bb", "cc"]);
```

And what about `bind`? That's the most powerful method for binding functions. `bind` still takes a new context object for a given function but it does not just call the function int the new context object. It returns a new function bound to that object permanently:

```js
var obj = {
  version: "0.0.1",
  printParams: function(param1, param2, param3) {
    console.log(this.version, param1, param2, param3);
  }
};

var newObj = {
  version: "0.0.2"
};

var newFunc = obj.printParams.bind(newObj);

newFunc("aa", "bb", "cc");
```

A common use case for `bind` is a permanent re-binding of an original function:

```js
var obj = {
  version: "0.0.1",
  printParams: function(param1, param2, param3) {
    console.log(this.version, param1, param2, param3);
  }
};

var newObj = {
  version: "0.0.2"
};

obj.printParams = obj.printParams.bind(newObj);

obj.printParams("aa", "bb", "cc");
```

From now on `obj.printParams` will always refer `newObj` as the object in which the function is running. At this point should be clear why we re-bind class methods in React with `bind`:

```js
class Button extends React.Component {
  constructor(props) {
    super(props);
    this.state = { text: "" };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(() => {
      return { text: "PROCEED TO CHECKOUT" };
    });
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.text || this.props.text}
      </button>
    );
  }
}
```

When we assign a class method to a React element the method "thinks": "I should run in the context of that element". React elements in fact are nothing more than JavaScript objects. When a function runs inside an object then that object becomes its "home" and the implicit `this` binding kicks in. In the example above the method `handleClick` (assigned to a button element) tries to update the component's state by calling `this.setState()`. When called the method runs in the context of the button element which is no longer the class itself and you get the dreaded "TypeError: Cannot read property 'setState' of undefined". We say that the function loses its binding. React components are most of the times exported as ES2015 modules: `this` is `undefined` because ES modules use strict mode by default, thus disabling the default binding. To solve the problem we use `bind` for making the method stick to the right context, the class itself:

```js
  constructor(props) {
    this.handleClick = this.handleClick.bind(this);
  }
```

Explicit binding is stronger than both implicit binding and default binding. With `apply`, `call`, and `bind` we can bend `this` at our own will by providing a dynamic context object to our functions. If "context object" is still too abstract for you think of it as a box in which JavaScript functions run. The box is always there, but we can change its coordinates at any time with an explicit bind.

## Rule number 4: "new" binding

COMING SOON