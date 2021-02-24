# Javascript Patterns

A collection of cool tricks and useful patterns that i am collecting as i learn javascript and node, basically anything and web tech related.

## Pass by Reference Class Helpers

### Pass by Reference Class Helpers - Problem

You are writing a class and need to refactor its methods to reduce the number or lines in a file

### Pass by Reference Class Helpers - Solution

By passing the class as `this` to another function, your entire object is passed by reference to `helperFunction` and any changes made in the helper to the passed class object will be reflected within the originally passed object.

```js
// Example of pass by reference (pbr)
// pbr is when the caller and callee modify the same variable

// bot is created, and then the helperFunction modifies an internal field
// the entire bot class is pbr to the helper function using the `this` keyword

const helperFunction = (bot, v) => {
  bot.internal = bot.internal+1
  console.log(bot.internal)
  console.log(v)
}

class Bot {
  constructor() {
    this.internal = 1
  }
  foo = () => helperFunction(this);
  bar = (v) => helperFunction(this, v);
}

const bot = new Bot()

// Example 1 - Observing pass by reference
bot.foo() //internal is now 2
console.log(bot) // internal is now 2 in the bot class as well

// Example 2 - optional params
// passing "a" to the helper function through the method of the class
bot.bar("a")
```

## Event Emitter Logger

NodeJS event emitters can make good loggers when abstracted to a separate file that exports the emitter.

### Event Emitter Logger - Problem

You have a well separated program over many files and want to be able to easily log to a file from anywhere.

However you also need to log lots of different types of things, and/or log to different locations (ie you need a very adaptable logger).

### Event Emitter Logger - Solution

The event emitter can be defined in a commonly accessible file called `common.js`

```js
// common.js
const events = require("events");

// create a new event emitter and export it
const logEmitter = new events.EventEmitter();
module.exports.logEmitter = logEmitter;
```

Now somewhere else we can define a listener to do something with any emitted logs

```js
const logEmitter = require("./common.js")

logEmitter.addListener("log", function (message) {
    console.log(message);
    fs.appendFileSync(path.resolve(__dirname, "../log.txt"), message + "\n");
});
```

Somewhere else we can emit a log message for our logEmitter listener to work with.

```js
// important function that emits logs.js
const logEmitter = require("./common.js")

// we emit to the "log" stream on the logEmitter
const dateString = new Date().toISOString()
logEmitter.emit("log", `${dateString}: I did something important bot`);
```

#### Event Emitter Logger - Caveats When Implementing

Say you created a logger pattern like this (exposing the `logEmitter.addListener` function). This causes undesirable behavior (in repl.it at least) where the newly created addListener causes it to print either **itself** (its function guts), or **true** if you include the last `emit` line in the below example.

```js
// This code is BAD

const events = require("events");

// create a new event emitter
const logEmitter = new events.EventEmitter();

logEmitter.addListener("log", (message) => {
    console.log(message)
});

// we emit to the "log" stream on the logEmitter
// This causes the output to be "true"
logEmitter.emit("log", `I did something important`);
```

If we include the last line of above we get `true` at the end.

```output
I did something important
true
```

If we remove the last line we get the event emitter itself.

```output
EventEmitter {
  _events: [Object: null prototype] { log: [Function] },
  _eventsCount: 1,
  _maxListeners: undefined,
  [Symbol(kCapture)]: false
}
```

To fix this you need to place the `logEmitter.addListener` in a class or function.

As a class...

```js
// This is GOOD

const events = require("events");

// create a new event emitter and export it
const logEmitter = new events.EventEmitter();

class Bot {
  constructor() {
    logEmitter.addListener("log", (message) => {
      console.log(message)
    });
  }
  emit() {
    logEmitter.emit("log", `I did something important`);
  }
}

const bot = new Bot()
bot.emit()
```

```output
I did something important
```

As a function...

```js
// This is GOOD
const createListener = () => {
  logEmitter.addListener("log", (message) => {
    console.log(message)
  });
}

const emit = () => {
  logEmitter.emit("log", `I did something important`);
}

createListener();
emit();
```
