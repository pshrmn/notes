# Compose

```js
d = compose(a, b, c);
d(x) === a(b(c(x)));
```

See more on [Wikipedia](https://en.wikipedia.org/wiki/Function_composition_(computer_science)).
## Terminology

This isn't an official terminology for these functions, but I'm defining them here so that I can consistently refer to the specific types of functions in this tutorial.

* **Composing Function** - The function returned by a call to compose.
* **Composed Function** - A function that was passed as an argument to compose and is called by the composing function.

```js
// in this example, d is a composing function and a, b, and c are composed functions
d = compose(a,b,c);
```

## Basics

Calling compose on a number of functions creates a new composing function that chains together calls to the composed functions. The composed functions are called from right to left, with the return values of a function used as the arguments used to call the function to its left. The arguments used to call the composing function are used as the arguments to the right-most composed function. The result of the composing function is the return value of the left-most function.

```js
d = compose(a, b, c);
// the function d is called with the argument x
result = d(x);
// d calls the right-most composed function
c_val = c(x);
// the result of c is used as the argument to b
b_val = b(c_val);
// and the result of b is used as the argument to a
a_val = a(b_val);
// the result of a is returned as the result of the composed function
result = a_val;
```

A nice effect of using compose is that we can avoid parentheses hell.

```js
compose(1,2,3,...,n-2, n-1, n)(arg) === 1(2(3(...(n-2(n-1(n(arg)))))));
```

## Compose Function

To see how this composition works, it's best to look at the source code of a compose function.

```js
// compose takes a variadic (unspecified/any) number of functions
// as its arguments
function compose(...funcs) {
  // compose returns a composing function that will call those
  // composed functions
  
  // The composing function also takes a variadic number of arguments,
  // which should be the same number of arguments as the right-most
  // composed function expects.
  return (...args) => {
    if (funcs.length === 0) {
      return args[0];
    }

    // the last (right-most) composed function is special because its
    // arguments are the arguments passed in when calling the composing
    // function
    const last = funcs[funcs.length - 1];
    const startValue = last(...args);

    // the other composed functions are called from right to left
    const finalResult = funcs.slice(0, -1).reduceRight(
      // the arguments for all of the other functions are the
      // return value of the composed function to its right
      (composed, f) => f(composed),
      // with the result of the right-most composed function being
      // used as the starting value
      startValue
    );
    // The final result, which is the value returned by the left-most
    // composed function, is returned.
    return finalResult;
  }
}
```

### Arguments and Return Values

For two functions that are composed together, the results of the right one is used as the arguments to the left one. So it is important that:

* The number of values that the right function returns matches the number of arguments that the left function expects.
* The types of values that the right function returns matches the types of arguments that the left function expects.

In JavaScript, functions can only return one value, so composed functions should only take one argument. The one exception to this is that the first (right-most) composed function can take multiple arguments since we are effectively calling it directly instead of it being called by another composed function.

```js
/*
 * mismatched argument and return value lengths
 */
function add(a, b) { return a+b; }
function double(x) { return x*2; }

// BE CAREFUL!
let addAndDouble = compose(double, add);
// Because add is the first function called, we can get away
// with the fact that it takes two arguments, we can just
// call addAndDouble passing it two arguments. This is the
// exception to the one parameter rule.

// BAD!
let doubleAndAdd = compose(add, double);
// In this situation, the composed function first calls double,
// which returns a number. That result is then used to call add,
// but because add expect two arguments and we only give it one,
// b === undefined and we would end up return NaN.

/*
 * mismatched arguments and return value types
 */
function takesNumber(number) {
  return number + 1;
}
function takesArrayOfNumbers(arr) {
  return arr.filter(v => v %2 === 0);
}

// UNINTENDED RESULTS
const unintentionalFunc = compose(takesNumber, takesArrayOfNumbers);
let result = unintentionalFunc([1,2,3,4]);
// Here, the user might intend to get all of the even numbers and add one to them
// expecting a result of [3,5], but instead number + 1 converts the array to a string
// and adds 1 to the end of it, returning "2,41"

// ERRORS
const errorFunc = compose(takesArrayOfNumbers, takesNumber);
result = errorFunc(2);
// In other cases, when types simply don't match up, the composed function will
// throw an error.
```

## Functions as Arguments and Return Values

Since the result of the composing function is the result of the left-most composed function, if the left-most composed function returns a function, the result of the composing function is a function.

```js
function leftMost(arg) {
  return function(otherArg) {
    ...
  }
}
function middle(arg) { ... }
function rightMost(arg) { ... }

const composing = compose(leftMost, middle, rightMost);
// result is what leftMost returns, so result is a function
const result = composing(arg);
```

Importantly, the function that is returned by leftMost has the arg that was used to call leftMost in its scope.

```js
function leftMost(arg) {
  return function(otherArg) {
    console.log(`I know what ${arg} is`);
  }
}
```

What if the function to the right of leftMost also returns a function? Then leftMost's return function would be able to call that function.

```js
function leftMost(fn) {
  return function(otherArg) {
    return fn();
  }
}
function middle(arg) {
  return function(otherArg) {
    return "middle function result";
  }
}
function rightMost(arg) { ... }
const composing = compose(leftMost, middle, rightMost);
const result = composing(arg);
// result() === "middle function result";
```

And if every function takes a function as its argument, then the composing function can take a function as its argument. And when the function that the composing function returns is called, each function can be called in a chain, with the one passed as an argument to the composing function called last.

```js
function leftMost(fn) {
  return function(otherArg) {
    return fn(otherArg);
  }
}
function middle(fn) {
  return function(otherArg) {
    return fn(otherArg);
  }
}
function rightMost(fn) {
  return function(otherArg) {
    return fn(otherArg);
  }
}

const composing = compose(leftMost, middle, rightMost);
const result = composing(val => { return `the value is ${val}`; });
// result("foo") === "the value is foo";
```

It is important to note that while the functions are composed from right to left, the resulting function will be executed from left to right.

### Still Stuck?

If you are having trouble picturing how all of this code is executed, the commented code below attempts to give a step by step order of executing to clear things up.

```js
function first(fn) {
  return function innerFirst(arg) {
    return fn(arg);
  }
}

function second(fn) {
  return function innerSecond(arg) {
    return fn(arg);
  }
}

function third(fn) {
  return function innerThird(arg) {
    return fn(arg);
  }
}

function log(arg) {
  console.log(arg);
  return arg;
}

let composing = compose(first, second, third);
/*
 * At this point, the composing function has an array of
 * functions [first, second, third], but nothing has been
 * called.
 */

let andLog = composing(log);
/*
 * So what happens when we create andLog?
 *
 * 1. The composing function receives log as its argument
 *    and calls third, passing it the log function.
 * 2. third receives log as its argument, keeping it as fn
 *    in its scope and returning innerThird.
 * 3. second receives innerThird as its argument, keeping it
 *    as fn in its scope and returning innerSecond.
 * 4. first receives innerSecond as its argument, keeping it
 *    as fn in its scope and returning innerFirst.
 * 5. composing returns innerFirst, and the variable andLog
 *    is the scoped version of the function innerFirst
 */

let result = andLog(message);
/*
 * And what happens when we call andLog?
 *
 * 1. innerFirst receives the message argument and
 *    calls its scoped fn function, which is innerSecond
 * 2. innerSecond receives the message argument from innerFirst
 *    and calls its scoped fn function, which is innerThird
 * 3. innerThird receives the message argument from innerSecond
 *    and calls its scoped fn function, which is log
 * 4. log logs the message argument and returns it
 * 5. innerThird returns the value returned by log
 * 6. innerSecond returns the value returned by innerThird
 * 7. innerFirst returns the value returned by innerSecond
 */
 ```

## Arrow Functions

While all of the above examples use traditional JavaScript function syntax, arrow functions can also be used.

```js
function toBeComposed(fn) {
  return function(arg) {
    return fn(arg);
  }
}
// is equivalent to
const toBeComposed = fn => {
  return arg => {
    return fn(arg);
  }
}
// which can even more concisely be written as
const toBeComposed = fn => arg => fn(arg);
```
