# Promise Cookbook

Small experiments in vanilla promises to better understand how they work.

## A simple promise

```js
const test = new Promise(() => {
    setTimeout(() => {
        console.log("done")
    }, 500)
});

// wait 500ms
// => done
```

Lets improve it by moving setTimeout into its own callback function.

```js
const cb = () => {
    setTimeout(() => {
        console.log("done")
    }, 500)
}

// Heres a longer way of writing the callback
const test = new Promise((resolve) => {
    resolve(cb())
});

// we can also shorthand this a bit
const test = new Promise((resolve) => resolve(cb()));

// wait 500ms
// => done
```

Because we used `resolve(cb())` we can call a `.next` with the data that we resolved. However have the **setTimeout** has no dependency on `.then`, so the "more data" can be printed immediately.

While i'm not 100% sure about the following, i based it on [this video](https://youtu.be/8aGhZQkoFbQ).

* The setTimeout is placed on the stack
* The console.log is placed on the stack
* The queue cant take setTimeout because it hasn't resolved yet, but console.log has so it goes on the queue. `print "more data"`
* setTimeout resolves and its callback is placed on the queue. `print "data"` is then called

```js
const cb = () => {
    setTimeout(() => {
        console.log("done")
    }, 500)
}


const test = new Promise((resolve) => resolve(cb()))
  .then((data) => console.log("more data"));

// => more data
// wait 500ms
// data
```

Lets modify above so it prints in the right order.

```js
// callback takes the resolve
const cb = (resolve) => {
    // when 500ms elapses
    setTimeout(() => {
        console.log("done") // print "done"
        resolve("more data") // then resolve "more data"
    }, 500)
}

// we pass the resolve function to the callback so it can tell us when its safe to move to .then
const test = new Promise((resolve) => cb(resolve))
  .then((data) => console.log(data));

// wait 500ms
// => done
// => more data
```

Next lets create a practical timeout function using `setTimeout` and the things we learnt above.

```js
const timeout = () => {
  return new Promise((res) => {setTimeout(() => {res()}, 500)})
}

timeout().then(() => {console.log("i'm still here!")})

// wait 500ms
// => i'm still here!
```

Then lets apply it to an async function to make it even more syntactically pretty.

```js
const timeout = () => {
  return new Promise((res) => {setTimeout(() => {res()}, 500)})
}


(async () => {
  console.log("dramatic");
  await timeout()
  console.log("suspense")
})()

// => dramatic
// wait 500ms
// => suspense
```

```js
// You can also make this a function like this
const myAsyncFunction = async () => {
  console.log("dramatic");
  await timeout()
  console.log("suspense")
}

myAsyncFunction()

// => dramatic
// wait 500ms
// => suspense
```

## For await loop

check [this video](https://www.youtube.com/watch?v=unDSLi5zBXU) for more information about the concept of the for-await-of loop.

Lets say we have a factory that generates promises for us.

```js
const asyncFactory = function (timeout) {
  return new Promise((resolve) => {
    // pretend to do some work
    setTimeout(() => resolve(timeout), timeout * 1000);
  });
};
```

We may generate promises at any time! What should we do when we want to consume them?

One option is to call `Promise.all()` and it will wait for all the promises to resolve before continuing. You would call this when you want to ensure that every single promise has resolved.

Another way of resolving promises is to use an for-await-of loop. Generally speaking a for loop will iterate over all its items as fast as possible. The for-await loop will instead iterate in the order of items you gave it, but waits ensures that A is resolved before B.

Writing this for-await loop...

```js
const myPromises = [];

  // push a 2, 1, and 3 second promise
  for (const i of [2, 1, 3]) {
    myPromises.push(asyncFactory(i));
  }

  // Good solution
  for await (const p of myPromises) {
    console.log(p);
  }

// 2000ms
// print 2
// print 1
// 1000ms
// print 3
```

Is **almost** the same as writing this not await-enabled loop...

```js
const myPromises = [];

  // push a 2, 1, and 3 second promise
  for (const i of [2, 1, 3]) {
    myPromises.push(asyncFactory(i));
  }

  // Bad solution
  for (const p of myPromises) {
    p.then((t) => console.log(t));
  }

// 2000ms
// print 2
// 1000ms print 1
// print 1
// 3000ms
// print 3
```

The problem with the non await loop is that it is running our promises synchronously, therefore wasting time between promises that resolve out of order. In other words while we are waiting for the (first) 2000ms promise to resolve the second (1000ms) promise resolves in the background, so by the time the first promise resolves the second one can also resolve immediately from the [event](https://www.youtube.com/watch?v=8aGhZQkoFbQ) stack.
