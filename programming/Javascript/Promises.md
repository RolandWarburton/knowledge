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
