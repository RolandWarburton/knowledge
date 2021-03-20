# Promisifying File Read and Write

in node `fs.read` and `fs.write` are functions that implement a callback.

```none
fs.readFile(path[, options], callback)
```

```none
fs.writeFile(file, data[, options], callback)
```

Heres how you would write either of these functions in an example.

```js
const fp = "/usr/src/app/style.css"
const css = fs.readFile(fp, {encoding: "utf8"}, () => {})
```

```js
const output = "/usr/src/dist/style.css"
const css = "* {margin: 0px, padding: 0px}";
fs.writeFile(output, css, {encoding: "utf8"}, () => {})
```

## The Problem

Lets say we want to read a file, then manipulate it, and write it back somewhere else. To write this with the vanilla read and write functions you would write.

```js
const fs = require('fs');

const input = "/usr/src/app/style.css"
const output = "/usr/src/dist/style.css"
fs.readFile(input, {encoding: "utf8"}, (err, data) => {
    // file is now read
    fs.writeFile(output, data, {encoding: "utf8"}, () => {
        // file is now written
    });
})
```

The above example is ok for small amounts of complexity, however when you find yourself in situations where you have many layers of reading and writing, or you are refactoring, then nodes `util` function provides a solution to **promisify** a callback function into a promise.

As we know, the promise API is another way of saying we can use async/await, therefore reaching (in my opinion) the cleanest way of refactoring the above code into a flat structure thats easy to read.

```js
const fs = require('fs');
const {promisify} = require('util');

// turn the callback functions into promises
const readFile = promisify(fs.readFile);
const writeFile = promisify(fs.writeFile);

const input = "/usr/src/app/style.css"
const output = "/usr/src/app/dist/style.css";

// read the file
const css = readFile(input, {encoding: "utf8"});

// write the file
writeFile(output, await css, {encoding: "utf8"});
```
