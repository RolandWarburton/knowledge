# Concurrency in NodeJS

## Concurrency vs paralleism vs synchronous, oh my

Typically javascript on the browser is single threaded only, moreover what is the difference between an ajax request and a concurrent process.

What about concurrency vs asynchronous code? They are the same!
asynchronous code is just another name for "code that runs multiple threads" (gross underexplanation but its good enough for understanding the basics)

What about promises and ajax? Simply put, promises are the **tool** for managing and asynchronous method (eg ajax). Ajax is just a type of asynchronous method, more specifically ajax is achieved by providing a callback for your code to "jump back to" once its resolved, or as discussed prior, through the promise interface inside the browser, the keyword being that its **inside** the browser, ajax does not exist on nodes framework because it doesnt need to - Node comes with concurrent capabilities, ie using multiple threads at the same time already.

Because only one call stack is avaliable to the javascript in your browser. An ajax uses an "execution stack" for each function that is provided, and managed, by the browser to keep track of requests outside of regular javascripts call stack, of which only one of can exist in a FIFO manner.

![diagram](/media/Concurrency.png)

TL;DR

* synchronous (single thread)
* asynchronous (single thread w/ concurrency) <-- nodejs is this
* parallel (multi thread)

## Example time

Here are some code snippits that are heavily inspired by Engineering Man [here](https://www.youtube.com/watch?v=Kizk3a6UTPc) and [here](https://www.youtube.com/watch?v=RXN7169vBGw).

I have added lots of comments about what each one of them does.

### Example 1: setTimeout

setTimeout places `console.log(2)` on the callback queue to execute on later.

setTimeout runs asynchronously and once `console.log(1)`. is run it loops back and checks the callback queue and then runs `console.log(2)`.

Therefore the output is `1,2`

```javascript
setTimeout(() => {
    console.log(2)
}, 0);

console.log(1)
```

### Example 2: Callbacks

This async function will start to read stuff.txt

Readfile is supplied a callback function with (err, data). Once it finishes reading it returns the callback and prints the contents of the file 😃

```javascript
fs.readFile(path.resolve(process.cwd(), 'data/stuff.txt'), (err, data) => {
    console.log(data.toString());
})

console.log('here')
```

### Example 3: Callback resolves randomly random

The callback stack is RANDOM af. you will get callbacks in any random order.

Lets say we have 3 files.

```none
├── 1.txt
├── 2.txt
├── 3.txt
```

This could return in order 1,3,2 or 1,2,3 etc

```javascript
for (let i = 1; i <= 3; i++) {
    const filepath = path.resolve(process.cwd(), 'data/', i + '.txt')
    console.log(filepath)
    fs.readFile(filepath, (err, data) => {
        console.log(data.toString());
    })
}
```

### Example 4: Promises

Promises allow you to avoid callback hell. You can deal with one an eventual value at a time and pass it along after each resolve returns something.

```javascript
const promise = new Promise((resolve, reject) => {
    resolve('good to go')
	// or
    reject('not good to go')
}).then(value => {
    // the resolve is passed down to value
    console.log(value)
    // You can reject values here too
    throw 'really bad error'
}).catch(err => {
    // the reject is passed down to err
    // errors go in a catch block
    console.log(err)
})
```

Here is a more practical example with reading files.

```javascript
const promise = new Promise((resolve, reject) => {
    fs.readFile(path.resolve(process.cwd(), 'data/stuff.txt'), (err, data) => {
        if (!err) resolve(data)
        if (err) reject(err)
    })
}).then(data => {
    console.log(data.toString())
}).catch(err => {
    console.log(err)
})
```

Furthermore, here is an improved version of the above code snippit using `util.promisify` to convert a callback function into a promise based one.

```javascript
const read = util.promisify(fs.readFile)
read(path.resolve(process.cwd(), 'data/stuff.txt'))
    .then(data => {
        console.log(data.toString())
    })
```

### Example 5: Resolve all promises at once

If you have lots of promises to resolve at once, or need to gurentee that an array of promises have been resolved by a certain point in your code.
You can use the following code snippit to take an array of promises and resolve all of them in parallel 😃

```javascript
const read = util.promisify(fs.readFile)
const filepath = path.resolve(process.cwd(), 'data/stuff.txt')
Promise.all([
    read(filepath),
    read(filepath),
    read(filepath)])
    .then(data => {
        // unpack the array of data into 3 arrays.
        // js magic ✨
        const [data1, data2, data3] = data
        console.log(data1.toString())
    })
```

### Example 6: Returning a promise

Back to basics again. Create a function called *a*. *a* returns a promise to `console.log()` which is then resolved to 😃

```javascript
function a() {
    return Promise.resolve('😃')
}

console.log(a())
```

### Example 7: Async/Await

Async/await avoids some issues in callbacks where you can get buried in indentation, aka callback hell where you require function *A* but first require *B*, bur first *C*, etc.

An example of callback hell ([source](https://stackabuse.com/avoiding-callback-hell-in-node-js/)).

```javascript
getData(function(a){
    getMoreData(a, function(b){
        getMoreData(b, function(c){
            getMoreData(c, function(d){
	            getMoreData(d, function(e){
		            ...
		        });
	        });
        });
    });
});
```

Using a simple promise function below, i will write the async equivilent of it.

```javascript
const read = util.promisify(fs.readFile)
const filepath = path.resolve(process.cwd(), 'data/stuff.txt')

// This function is using a promise like above
const runMePromise = () => {
    read(filepath)
        .then(data => {
            console.log(data.toString())
        })
}
```

The following function will produce the same result as the top one but uses async instead.

```javascript
// This is exactly like the above promise except using async/await
const runMeAsync = async () => {
    const myData = await read(filepath)
    // this will wait and then print when await is done
    console.log(myData.toString())
}
```

### Example 8: Error handling

Error handling is done by using try and catch blocks in vanilla js.

```javascript
const runMeAsync = async () => {
    try {
        const myData = await read(filepath)
        // this will wait and then print when await is done
        console.log(myData.toString())
    } catch (err) {
        console.log(err)
    }
```
