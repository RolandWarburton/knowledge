# Javascript Patterns

A collection of cool tricks and useful patterns that i am collecting as i learn javascript and node, basically anything and web tech related.

## Pass by Reference Class Helpers

### Problem

You are writing a class and need to refactor its methods to reduce the number or lines in a file

### Solution

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
