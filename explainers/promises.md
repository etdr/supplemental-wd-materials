

# Basic asynchronous programming in JavaScript


## Why Promises

Basically, most things we do in JS happen fast enough that we don't care if the program stops doing other stuff
while it's executing a specific instruction.
What I mean is, if I write `const x = 3 + 7`, it's going to take a second (actually just like a millisecond) for the program to make a variable
and calculate 3 plus 7 and store the result in the new variable `x`, and while it's doing that, it can't
be doing anything else.
But even more complicated statements usually don't take that long to execute, and even if they do,
we need the program to run them before it moves on to another part of the code.
All these operations are *synchronous*, meaning more or less that they happen in order.


The reason we use promises is as a way to support *asynchronous* programming.
One important thing to know is that JavaScript can only be doing one thing at a time.
So, if something will take a long time to run, or if it *might* take a long time to run,
we don't necessarily want to hold up our entire program while we wait for the result of that operation.
It could be the case that our program *can't* do anything else until the operation completes,
in which case it has no other choice but to wait,
but since this isn't always the case we want to support the program doing other stuff
while it's waiting for the completion of processes that might take a while.
"A while" might just refer to a few hundred milliseconds, but still.


It's worth knowing that promises are just one way of doing asynchronous programming;
other languages use different methods.
Plus, there is also *concurrent* or *parallel* programming (related but not equivalent),
where we take advantage of the fact that modern computers have more than one processor core
to do multiple things at once.
This is not that: we are only ever doing one thing at once, but the order in which we do things
can be different than how it appears in the code.



## Promises are Boxes

### Disclaimer

There are a lot of ways to explain promises and most of them center around a metaphor of some sort.
I will also use a metaphor, but it's going to be a little less embellished.
If some other explanation (like the Christmas-presents one in the modules) makes more sense to you, great!
And if you're still fuzzy about what was in the modules, see if this presentation makes any more sense to you
(it might not, but keep at it with whatever makes more sense).


### What are promises?

Promises are boxes.


### Rules of promises

The rules of these boxes are simple:
- any value can be put inside of a box. But...
- *once a value is inside a box, we cannot touch it directly anymore*. However...
- we *can* give the box a function to apply to the value it contains.
  - When we do this, we get a *new* box containing the result of applying that function to the value in the original box.
- A box is either:
  - *pending* (it is waiting for a value), or
  - *settled* (it is done waiting).
- Once a box is settled, it either has a value inside or it doesn't.
  - If it does, we say the box is *fulfilled*. The box contains a value.
  - If it does not, we say the box is *rejected*. The box now carries with it an error message of what went wrong.

There are a few other rules but that's the gist of it.

You might also hear the term "resolved" which in the modules is used to mean the same as "fulfilled"
but which is actually a little more complicated. I'm not going to get into the difference here, though.
I know all the terminology can be confusing (sorry!).
As long as you can distinguish between *pending*, *fulfilled*, and *rejected* you should be ok though.



### Making a promise

Eventually you might write your own code and functions that return promises instead of "unboxed" values
(*I* still have never done this),
but most of the time we just receive promises back when we run functions like `fetch`,
meaning we don't have to manually construct promises ourselves using the `Promise()` constructor function
as you saw in the modules.
(I honestly don't know why they go into that, it's overly complicated for someone who just needs to learn how to *use* promises.
But I digress.)


A key insight here is that we can make a box to store a value even before we know what that value is,
as long as we know we are starting some process that might lead to a value.
When we write `fetch(url)`, we are creating a box that will eventually be fulfilled or rejected
based on the response from the computer at the other end of that url.
We can then write code that tells the program what to do with that promise once it's settled.


### Other rules of promises

Feel free to skip this section, this is just for completeness.

- If we put a box inside another box, the inner box is "unwrapped" and whatever was inside it is now just in the outer box.
  That is, we only ever have one layer of box.
- We can't make an "empty" box without tying it to some computation that eventually will result in the box being fulfilled or rejected.
  That is, every pending promise has some specified way to eventually be settled one way or another.
- Once a box is settled, it can never be un-settled.





## Chaining and `.then`

So, as stated, we don't have access to any value once it is inside of a promise.
But, say we use `fetch` and get back a promise containing data that we retrieved from a server.
We might need to process that data and then display the information on a webpage,
but it sounds like we won't be able to do that because we can't get directly at anything inside a promise.

The trick is, while we can't access anything in a box directly, we can tell the box what to do with the value inside it
by giving it a function to run on that value after the promise is fulfilled.
We do this with `.then`, which is a method of every promise.
We pass some function as a callback to `.then`, as so:

```js
fetch(url)
  .then(x => {
    console.log(x);  // given this function, .then applies it to the promise we get by calling the fetch function on the variable url
  })
```
Here, I am writing the callback function explicitly inside the parentheses calling `.then`,
but you could also define the function elsewhere and pass the name of the function in instead:

```js
const printResponse = x => { console.log(x); };

fetch(url).then(printResponse);
```

Either way, the `.then` *chains* the promise to create a *new* promise.
That is, the return value of `.then` is another promise.
The value of this new promise is the result of running the given function on the value in the original promise.


Since `.then` returns a promise, and all promises have a `.then` method, we can add *another* `.then` to the end of the first:

```js
fetch(url)
  .then(x => x.json())  // we tell .then to go inside the fetch(url) promise and apply x => x.json() to the value inside
  .then(x => {
    console.log(x);
  })
```

(I was going to have a picture for this here but I need to workshop it more first. Ask if you think a visual would help.)

We can chain with `.then` as many times as we need to.

It's important to know that the function in a `.then` will only be run on a successfully settled promise containing a value.
That is, if the promise is rejected, we do not run the `.then`.
The reason for this is simply that a function we give to `.then` is one we want to run on the value in a fulfilled promise,
but a rejected promise contains no such value.

A rejected promise *does*, however, have a note in it explaining what went wrong in trying to get a value.
To access that error, we can use a similar method, `.catch`.

```js
fetch(url)
  .then(x => x.json())
  .then(x => {
    console.log(x);
  })
  .catch(err => console.error(err));  // if any of the above promises is rejected, we fall to this .catch and run this function on the error
```

If anything goes wrong during the fetch or either of the functions chained to it using `.then`,
that promise is rejected with an error information, and we immediately drop to the function in the `.catch`
which processes that info.


You will also see rejection functions passed as second parameters to `.then`.
As far as I know, this is equivalent to passing them to `.catch`,
though the former might allow you more granular control over how you process errors.
Ask me about this if you're interested.




## `async` and `await`


There is another way to write asynchronous code that was introduced into JavaScript in 2016.
The reason for this change is because it's not always natural to express programs and functions
exclusively using callback functions.



The first thing to do is mark a function as asynchronous using the `async` keyword:

```js
async function aFunc () {
  ...
}

const aFuncArrow = async () => { ... }  // arrow functions can also be asynchronous
```

The very first thing to know about an async function is that it *always returns a promise*.
By just marking the function `async` we know that any value we receive back from it will be wrapped in a promise.
So what's the point of this?

The reason we submit to this rule when we write async functions is this:
*while we are within the async function, we can directly access the values contained in promises*.
<!-- But remember, anything we do will be wrapped in a promise when the function returns. -->

You could think of an asynchronous function as a special room where unboxing is allowed,
as long as when you leave the room anything you have with you has to be inside a box.
If you try to leave with something outside of a box, the room boxes it for you.
(Still working on the best metaphor here.)

```js
async function aFunc (url) {
  // the await keyword gets the value out of the fetch(url) promise and we assign that to result
  const result = await fetch(url);
  // calling .json on the result also creates a promise, so we use await again to get that value out
  const jsonResult = await result.json();
  // jsonResult is a regular value, so it will be boxed in a promise when this function returns
  return jsonResult;
}
```

To get at the values inside promises, we use `await`.
The `await` keyword/operator simply can be thought of as an *unwrap* command.
In front of a promise, it says: open the box and give me the value inside.
(In front of a normal value, it does nothing.)

You can only use the `await` operator inside a function marked as `async`.
*If you're not inside the special room, you can't unbox anything.*

The fact that we're still talking about boxes should tip you off to the fact that we're still working with promises.
The `async`/`await` system is built on top of the existing promise system in JavaScript.
It doesn't allow us to do anything new that we couldn't do before using `.then` and other methods,
but it does allow us to express our functions in a somewhat more readable style.
We can also use `try`/`catch` blocks inside async functions for error handling,
if you're familiar with those.



## Questions, comments, points of confusion

Please get in touch and I'll be happy to clarify anything,
either on Slack or by making an appointment.


















## See also

- [Promise objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) on MDN
- [States and Fates](https://github.com/domenic/promises-unwrapping/blob/master/docs/states-and-fates.md)
- [Promise objects](https://tc39.es/ecma262/#sec-promise-objects) in the ECMAScript specification



