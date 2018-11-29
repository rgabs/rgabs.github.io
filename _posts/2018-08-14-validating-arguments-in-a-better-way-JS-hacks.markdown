---
layout: post
title: "Validating arguments in a better way: JS hacks 😉"
date: 2018-08-14T01:15:00.000Z
intro: "Since JS is not a strongly typed language, writing checks/if-else-conditions for each argument can/will get messier in the long run. And no smart developer would want to have multiple if-else blocks just to validate arguments. Presenting you the new cooler/better way using ES6 default parameters."
categories: front-end
---


I recently came across an interesting pattern for Validating arguments in JavaScript. I am sure all of you JS developers have came across the need to:
1. Make sure that a function is being called with arguments.
2. Handling the arguments which are mandatory for a function to work.
3. Any other reason(s) you can think of..


### The conventional way

```js
function ensure(value) {
  if(value === undefined) {
    throw new Error('no arguments');
  }

  return value;
}
```

The above code is pretty straightforward. And this is how most of us JS developers usually check for arguments in our functions.

Since JS is **not** a [strongly typed](https://en.wikipedia.org/wiki/Strong_and_weak_typing) language, writing checks/if-else-conditions for each argument can/will get messier in the long run. And no smart developer would want to have multiple if-else blocks just to validate arguments.


### The new cooler/better way (using ES6 [default parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters))

```js
const checkUndefined = () => {
  throw new Error('no arguments');
};

function ensure(value = checkUndefined()) { // An error will be thrown if ensure is called without any argument
  return value;
}

```

Here we use the [default parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters) feature of ES6. If the `ensure` function gets called with **no value or undefined**, the execution will be stopped and the error will be thrown.

In this approach, we have removed the requirement of having un-necessary if-conditions which are not actually part of business logic.

We can even extend this approach to support multiple arguments in the called function.

>NOTE: `null` is a valid value in JS. So if the ensure function gets called with null or any other falsy value (0, null, false), the passed argument will be used and the function will not throw any error!

<div style="text-align:center">
  <img src="/static/img/jest/neat.gif" style="width: 70%;display:inline-block;text-align:'left'" vspace="20">
</div>

#### Pretty neat! Right 😉
