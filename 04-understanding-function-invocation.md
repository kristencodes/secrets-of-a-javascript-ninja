# Chapter 4: Functions for the journeyman: understanding function invocation

## This Chapter Covers:

- Two implicit function paramaters; `arguments` and `this`
- Ways of invoking functions
- Dealing with problems of function contexts

Two things that the last chapter did not cover were the `this` paramater and the `arguments` paramater.

`this` represents the function context (the object on which our function is invoked) whereas `arguments` represents all argument that are passed in through a function call.

## 4.1 Using Implicit Function Paramaters

_Quick recap: what is the difference between a function `paramater` and a function`argument`?_

**Implicit** indicates that we don't need to list these paramaters in the function definition, but they are silently passed to the function and accessible inside. They can be referenced just like any other function paramater.

### 4.1.1 The Arguments Paramater

**Arguments** are a collection of all arguments passed to a function, regardless of whether they were defined or not.

_The introduction of `rest` paramaters has eliminated most of the need for the `arguments` paramater but it's still important to understand how it works if you happen to be working with legacy code._

The `arguments` object has a property called `length` that indicates the exact number of arguments. You can access individual argumetns by using array indexing notation:

```javascript
arguments[2];
```

Take a look at `code/04/01-arguments.html` to see them in action.

It's important to know that the `arguments` paramater **_is not an array_** even though they have a `length` property and use `index` notation. Think of them as an _array-like construct_.

The `rest` paramater on the other hand does create a real array which has certain advantages.

The arguments paramater allows us to write more flexible code that doesn't depend on knowing what is going to be passed in. Let's take a look at `code/04/02-sum.html` for an example. Below will be the same function rewritten with `rest` paramaters.

#### Gotchas

The `arguments` function aliases function paramaters. What this means is that if we set a new value to `arguments[0]`, the value of the first paramater.

```javascript
function infiltrate(person) {
  console.log(person);

  arguments[0] = "ninja";

  console.log(person);

  // the alias works both ways
  person = "developer";

  console.log(arguments[0]);
}
```

This concept is very confusing, so JavaScript provides a way to opt out of it by using _strict mode_.

> **Strict Mode** <br>
> Strict mode is an ES5 addition to JavaScript that changes the behavior of JS engines so that errors are thrown instead of failing silently. Some unsafe language features are completely banned (more on this later). One of the things it changes is that it disables `arguments` aliasing.

You can enable strict mode by writing `"use strict"` at the top of the file.

### 4.1.2 The _this_ paramater: introducing function context

In addition to `arguments`, an implicit paramater called `this` is passed to the function. It's often termed the _function context_.

In many other lanuages, `this` usually points to an instance of the class within which the method is defined.

As we will see, in JavaScript, invoking a function as a _method_ is only one way that a function can be invoked, and what the `this` paramater points to isn't defined only by how and where the function is defined, it can also be influenced by where the function is _invoked_.

In order to understand this well, it's important to understand all the ways functions can be invoked.

## 4.2 Invoking Functions

We can invoke a function in four ways.

- _As a function_ - `skulk()`, in which the function is invoked in a straightforward manner
- As a _method_ - `ninja.skulk()`, which ties the invocation to an object, enabling object-oriented programming
- As a _constructor_ - `new Ninja()`, in which a new object is brought into being
- _Via the function's `apply` or `call` methods_ - `skulk.call(ninja)` or `skulk.apply(ninja)`

For all but the `call` and `apply` approaches, the function invocation operator is a set of **parentheses** following any **expression** that evaluates to a function reference.

### 4.2.1 Invocation as a function

We say that a function is invoked "as a function" to distinguish it from the other mechanisms (methods, constructors, and apply/call).

This type of invokation uses the `()` operator.

```javascript
// function declaration invoked as a function
function ninja(){};
ninja();

// function expression invoked as a function
var samuri = function(){};
samuri(;

// immediately invoked function expression invoked as a function
(function(){})()
```

When invoked in this manner, the _function context_ (value of the `this`) keyword can be two things:

1.  In nonstrict mode, it will be the global context - the `window` object
2.  in strict mode, it will be `undefined`

### 4.2.2 Invocation as a method

When a function is assigned to a property of an object and the invocation occurs by referencing the function using that property, the function is invoked as a _method_ of that object.

```javascript
var ninja = {};
ninja.skulk = function() {};
ninja.skulk();
```

When a function is invoked in this way, that object becomes the function's context and is available within the function via the `this` paramater.

Let's look at an example of how the context can change in `code/04/03-whats-the-context.html`.

Even though the _same_ function, `whatsMyContext` is referenced throughout the example the function context `this` returns changes depending on how `whatsMyContext` is invoked.

### 4.2.3 Invocation as a constructor

To invoke the function as a constructor, we precede the function invocation with the keyword `new`. For example:

```javascript
function whatsMyContext() {
  return this;
}

// invoked as a constructor

new whatsMyContext();
```

> Note: in chapter 3 we talked about **function constructors** which enable us to construct new functions from strings eg) `new Function('a','b', 'return a + b');` - these are not to be confused with **constructor functions** 🙄🙄🙄 which are used to create and initialize object instances.

#### The superpower of constructors

Function constructors can be used to create new instances of objects that can have all the the same properties.

```javascript
function Ninja() {
  this.skulk = function() {
    return this;
  };
}

var ninja1 = new Ninja();
var ninja2 = new Ninja();

ninja1.skulk(); // >> ninja1
ninja2.skulk(); // >> ninja2
```

In general when a constructor is invoked, a couple special actions take place. the `new` keyword triggers the following:

1.  A new empty object is created
2.  This object is passed to the constructor as the `this` paramater, and thus becomes the constructor function's context.
3.  The newly constructed object is returned as the `new` operator's value (with an exception that we'll touch on soon).

![When calling a function with a keyword new a new empty object is created and set as the context of the constructor funciton, the this paramater](https://dpzbhybb2pdcj.cloudfront.net/maras/Figures/04fig01_alt.jpg)

Functions and methods are generally named with a **verb** that describes what they do and start with a lowercase letter. Constructors on the otherhand are usually named a **noun** that describes the object that is being constructed and start with an uppercase character. This makes it more clear that the function is meant to be invoked as a constructor.

### 4.2.4 Invocation with the apply and call methods

What if we want to make the function context whatever we want?

JavaScript provides a means for us to invoke a function and explicitly specify any object we want as the function context. There are two methods that exist on every function: `apply` and `call`. (Functions have properties just like any other object type, including methods).

To invoke a function using it's `apply` method, we pass two paramaters: the object to be used as function context, and an array of values to be used as the invocation arguments.

The `call` method is used in a similar way, except that the arguments are passed directly in the argument list rather than an array.

We can look at an example in `code/04/05-apply-call.html`

A more practical example where we write our own `forEach` function can be found in `code/04/06-call-callbacks.html`;

### 4.3 Fixing the problem of function contexts

There is a common bug related to event handling (when an event handler is called, the function context is set to the object to which the event was bound - more on this in chapter 13).

Let's look at an example in `code/04/04-binding-context.html`

This code doesn't work! It fails because the context of the `click` function _isn't_ referring to the button object as intended.

If we had called the function via `button.click()` the context _would_ have been the button, but because the event handling system of the browser defines the context of invocation to be the target element of the event, it makes the context the `<button>` element, not the `button` object.

#### 4.3.1 Using arrow functions to get around function contexts

Besides syntactic sugar, arrow functions have another feature that makes them great as callback functions: _arrow functions don't have their own `this` value_. Instead, they remember the value of the `this` paramater at the time of their definition.

Let's look at how we can use them to get around the error with the event listener in `code/04/07-using-arrow-functions.html`

In this case, the arrow function was created inside a constructor function, the `this` paramater is the newly constructed object.

Let's take a look at what would happen if we used an object literal in `code/04-07-arrow-functions-and-object-literals.html`

#### 4.3.2 Using the bind method

In addition to `call` and `apply`, every function also has access to the `bind` method that essentially creates a new funciton. The function has an identical body, but it's context is _always_ bound to a certian object, regardless of the way we envoke it. Let's revisit the last problem in `/code/04/09-binding-to-event-handler.html`.

### 4.4 Summary

- When invoking a funciton, in addition to the paramaters explicitly stated in the function definition, function invocations are passed two implicit paramaters: `arguments` and `this`:

  - The `arguments` paramater is a collection (not an array) of arguments passed to the function
  - it has a `length` property that idnicates how many arguments were passed in
  - in nonstrict mode, arguments object ailases the function paramaters
  - the `this` paramater represents the function context, an object to which the function invocation is associated. How `this` is determined can depend on the way a function is defined as well as on how it's invoked.

- A function can be invoked in four ways:

  - As a function `skulk()`
  - As a method `ninja.skulk()`
  - As a constructor: `new Ninja()`
  - Via its `apply` and `call` methods: `skulk.call(ninja) or skulk.apply(ninja)`

- The way a function is invoked influences the value of the `this` paramater:

  - If a function is invoked as a function, the value of the `this` paramater is usually the global `window` object in nonstrict mode, and `undefined` in strict mode
  - If a function is invoked as a method, the value of the `this` paramater is the object on which the funciton was invoked.
  - If a funciton is invoked as a constructor, the value of the `this` paramater is the newly constructed object.
  - If a function is invoked through `call` and `apply`, the value of the `this` paramater is the first argument supplied to `call` and `apply`.

- Arrow functions don't have their own value of the `this` paramater. Instead they pick it up at the moment of their creation.

- Use the `bind` method, available to all functions, to create a new function that's always bound to the argument of the `bind` method. In all other aspects, the bound function behaves as the original function.

### 4.5 Exercises

1.  The following function calculates the sum of the passed-in arguments by using the `arguments` object:

```javascript
  funciton sum(){
  var sum = 0;
  for(var i = 0; i < arguments.length; i++){
    sum += arguments[i];
  }
  return sum;
}
```

By using the rest paramaters introduced in the previous chapter, rewrite the `sum` function so that it doesn't use the arguments object.

2.  After running the following code, what are the values of the variables `ninja` and `samurai`?

```javascript
function getSamurai(samurai) {
  "use strict";

  arguments[0] = "Ishida";

  return samurai;
}

function getNinja(ninja) {
  arguments[0] = "Fuma";
  return ninja;
}

var samurai = getSamurai("Toyotomi");
var ninja = getNinja("Yoshi");
```

3.  When running the following code, which of the assertions will pass?

```javascript
function whoAmI1() {
  "use strict";
  return this;
}

function whoAmI2() {
  return this;
}

assert(whoAmI1() === window, "Window?");
assert(whoAmI2() === window, "Window?");
```

4.  When runing the following code, which of the assertions will pass?

```javascript
var ninja1 = {
  whoAmI: function() {
    return this;
  }
};

var ninja2 = {
  whoAmI: ninja1.whoAmI
};

var identify = ninja2.whoAmI;

assert(ninja1.whoAmI() === ninja1, "ninja1?");
assert(ninja2.whoAmI() === ninja1, "ninja1 again?");
assert(identify() === ninja1, "ninja1 again?");
assert(ninja1.whoAmI.call(ninja2) === ninja2, "ninja2 here?");
```

5.  When running the following code which of the assertions will pass?

```javascript
function Ninja() {
  this.whoAmI = () => this;
}

var ninja1 = new Ninja();

var ninja2 = {
  whoAmI: ninja1.whoAmI
};

assert(ninja1.whoAmI() === ninja1, "ninja1 here?");
assert(ninja2.whoAmI() === ninja2, "ninja2 here?");
```

6.  Which of the following assertions will pass?

```javascript
function Ninja() {
  this.whoAmI = function() {
    return this;
  }.bind(this);
}

var ninja1 = new Ninja();

var ninja2 = {
  whoAmI: ninja1.whoAmI
};

assert(ninja1.whoAmI() === ninja1, "ninja1 here");
assert(ninja2.whoAmI() === ninja2, "ninja2 here");
```
