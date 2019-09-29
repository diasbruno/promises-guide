#### Promise's Guide

# Creating and using promises

## Promise constructor

The `Promise` constructor is used to execute a task
in an asyncronous way.

It will create an object in which, eventually when resolved or rejected,
you can get its value in a method chain style.

This constructor expects a function that will receive 2 arguments.
Both arguments will be functions that are used to tell which
state the promise will be fullfiled. Those arguments are commonly
called `resolve` and `reject`.

```js
new Promise((resolve, reject) => {
  if ((1 + 1) == 2) {
    resolve(true);
  } else {
    reject(false);
  }
});
```

This function can execute any kind of task you want, even other
promises or asyncronous functions can be called.

In a very simplistic way we can think of the state of the promise as:

```js
Promise {
   status: 'pending' | 'resolved' | 'rejected',
   value: value | reason
}
```

In this guide, we will represent this state as a pair like this:

```
Promise { resolved, value }
Promise { rejected, reason }
```

    NOTE: The use of `{}` is just so we don't get confused
    with function calls.

The `pending` status is when the promise is not yet fullfiled or waiting
to be executed. Once resolved or rejected, it will be ready to call the next
task on the chain.

This constructor is also useful to transform other objects that does
asyncronous works into promises.

```js
const downloadImage = url => new Promise((resolve, reject) => {
  const image = new Image();

  image.onload  = function() { resolve(image); }
  image.onerror = function() { reject("Failed to download."); }

  image.src = url;
});

downloadImage("https://example.com/image.png");
```

Once the image is loaded we use `resolve` to pass the image to the next task.
And if an error occurs, we `reject` with a `reason`.

    Gotcha: Not calling one of those callbacks will cause the promise
    to stall and you will never get the result out of it.

## Promoting value to promises (`Promise.resolve` and `Promise.reject`)

Those are simple function that can convert a value into a promise.

```js
Promise.resolve(true);               // => Promise { resolved, value: true }
Promise.reject(false);               // => Promise { rejected, reason: false }
```

They are useful to create simple functions:

```js
const loggedIn =
  user => user == null ? Promise.resolve(user) : Promise.reject("Not logged in.");
```

    Gotcha: This simple functions can be used to start a pipeline,
    but there is one thing to remember...

    If this function throws an exception, the error doesn't
    propagate thought the pipeline. So, you need to handle
    any possible errors.

```js
const loggedIn =
  user => {
    try {
      if (checkLogin(user)) {
        Promise.resolve(user);
      } else {
        Promise.reject("Not logged in.");
      }
    } catch(exception) {
      Promise.reject(exception);
    }
};
```

## Getting a value of a promise (`.then` and `.catch`)

`f` and `g` will be functions that receives one argument.

`Promise.then(f)` and `Promise.catch(g)` are methods provided by the `Promise`,
so we can chain these tasks, forming a pipeline.

    NOTE: It's important to remember that each method returns a new promise object.
    So, we neeed to think about what each function will return.

From the specification, it works like this:

- `Promise { resolved, value }.then(f)`
   The function `f` will be called with `value` as argument and return a new
   promise object.

- `Promise { resolved, value }.catch(g)`
   The function `g` is not called because it was resolved and
   the return of it will be a new promise object with the previous state.

- `Promise { rejected, reason }.then(f)`
   The function `f` is not called because it was rejected and
   the return of it will be a new promise object with the previous state.

- `Promise { rejected, reason }.catch(g)`
   The function `g` will be called with `reason` as argument.

`f` and `g` can be simple functions or functions that return promises.

Returning values:

```js
let p = null;

p = Promise.resolve(true);    // => Promise { resolved, value: true  }

p = p.then(x => x && false);  // => Promise { resolved, value: false }

p = p.then(
  x => console.log(x)
);                            // => Promose { resolved, value: false }
```

    Gotcha: It's not intuitive, but things works different with `.catch`
    when it's a pure function. When returning a value from a `.catch`
    the next promise will be in `resolved` state.

    Later on, there will be more examples where this can be useful.

```js
let p = null;

p = Promise.reject(false);    // => Promise { rejected, value: false }

p = p.catch(x => x || true);  // => Promise { resolved, value: true  }

p = p.then(
  x => console.log(x)
);                            // => Promose { resolved, value: true  }
```

Returning promises:

```js
let p = null;

p = Promise.resolve(true);                     // => Promise { resolved, value: true  }

p = p.then(x => Promise.resolve(x && false));  // => Promise { resolved, value: false }

p = p.then(
  x => x == false ? Promise.reject(x) : Promise.resolve(x)
);                                             // => Promise { rejected, value: false }

p = p.catch(
  y => console.error(y)
);
```
