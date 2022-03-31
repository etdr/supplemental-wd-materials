

# Callbacks


## Preliminaries

In some languages, functions are very different types of things.
In JavaScript, however (and in many other languages), they are just values like numbers or objects.
We can assign them to variables, pass them to other functions, return them from functions, test them for equality, and more.

The *only* real difference between a function and another value is that functions have a special ability:
we can *call* a function, which we do with parentheses.
If we don't use parentheses after the name of a function, we are just pointing to it, not calling it.

```js
function func () {  // this creates a function with the name func
  return 5;
}

console.log(func);  // prints a string representation of the function

console.log(func());  // prints the result of evaluating func(), which entails calling the func function
```

The first `console.log` prints `[Function: func]` since that's how JavaScript represents a function as a string.
The second prints out `5`, because when we call `console.log` on `func()`, we first have to evaluate
the call to `func`, which returns 5. Then 5 is passed into `console.log`.



## What is a callback?

A callback is when we give a function to another function.

Again, the important distinction is between calling a function and passing it another function
vs. calling a function and passing it the *result* of another function.




## A first example

Consider this function:

```js
function runFunc (f) {
  f();
}
```

Let's break down what this function does.
It receives one argument, `f`, and does not return anything.
However, it does call `f`, so `f` must be a function.
But also remember that here `f` is a variable scoped by this function definition.
Hence `f` is a variable that points to a function, and we can run the function it points to by writing `f()`.

Now consider the same function we had above:

```js
function func () {
  return 5;
}
```

We can pass `func` *as an argument* to `runFunc` like this:

```js
runFunc(func);
```

Notice there are no parentheses after `func`.
This is because we don't want to call func immediately and pass the result,
we want to pass `func` *itself*.

Now in this example, `runFunc` will call `f` and not do anything with the result.
It might be more useful to actually return something.

```js
function runFunc2 (f) {
  return f() ** 2;
}
```

`runFunc2` will take a function `f`, call `f`, take the result of that and square it, then take *that* result and return it.
When `func` returns a constant 5, `runFunc2(func)` evaluates to 25.




## A non-example

In contrast to what it says in the modules, this is (usually) *not* a callback:

```js
runFunc(func());
```

The difference is that here we are not passing `runFunc` a function,
but instead the return value of `func` after we call it.
So, here, `runFunc` receives 5 as an argument since this is the value of `func()`.

Sometimes we want this behavior! But this is not a callback,
it's just running one function on the result of another function (the technical term here would be *function composition*).
When you hear "callback" you should think of a function receiving another function as one of its arguments.


Note: I say this is "usually" not a callback because there is the possibility that `func` returns a function,
in which case `runFunc(func())` *would* be a callback because `func()` would be a function.
If this hurts your brain don't think about it, just know that something of the form `f(g())`
is not in general what we mean when we say callback.






## Example: use in array methods

When we write an arrow function as one of the arguments to an array method, we are using a callback.

```js
const arr = [0, 1, 2, 3, 4, 5];

const triples = arr.map(x => 3 * x);
```

The argument to `.map` is a function, except instead of providing it as a variable pointing to a function,
we have written the function directly inside the call to `.map`.
This is a common pattern, especially when the functions are relatively simple.

When `.map` runs on the array with the function we have provided, it has to call the given function multiple times,
once for each of the items in the array. We don't have control over when or how `.map` does this, so we pass it a function
and essentially say "run this whenever you need to and as many times as you have to".




## Why callbacks?

The short answer is that we might not know when or if we need to call a function ourselves,
so we give it to another function that does know.

Imagine a function that takes two functions:

```js
function splitDecision (f1, f2) {
  
  ...

  if (...) {
    return f1();
  } else {
    return f2();
  }
}
```

Here we can imagine that the condition for the `if` statement is something the `splitDecision` function computes
before it hits the `if` (where we have written `...`).
The situation would be that we know we need to run either `func1` or `func2` based on some complex criteria,
so we write `splitDecision` to compute those criteria and run the appropriate function for us.




Callbacks become even more essential when we are dealing with asynchronous functions.
Here we are dealing with processes which can take an arbitrary, unknown amount of time or even result in failure.
We can't run the function until we have the result of the process or, in the case of failure, we can't run the function at all.





## Questions, comments, points of confusion

Please get in touch and I'll be happy to clarify anything,
either on Slack or by making an appointment.






