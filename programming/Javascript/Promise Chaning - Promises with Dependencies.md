# Promise Chaining - Ordering Promises with Dependencies

Sometimes you have a set of functions (or functions that return promises) that you need to run, utilizing `Promise.all()` is a great way of resolving an array on promises all at once. However, i encountered a limitation that i was able to solve with some help from [here](https://stackoverflow.com/questions/30853265/dynamic-chaining-in-javascript-promises) in which a chain of promises need to be run one after the other sequentially (promise 1 resolves before promise 2, etc). In the problem statement below i will explain a use case for why you might want to do this.

## The problem

Lets say you have a data structure that looks like this. Perhaps we get this initial data from an API, or a database.

```js
const myPage = {
    pageName: "my page",
    websitePath: ["pages", "mypage"], // example.com/home/mypage
}
```

During our development we discover that we need to add additional information to the above object for its `parent`, and its `siblings`. Lets say we had some functions that get this information for us, **however** they are promise based and take some time to resolve, for now lets mock these functions.

Heres an example of the data structure with the new fields (yet to be populated).

```js
const myPage = {
    pageName: "my page",
    websitePath: ["pages", "mypage"], // example.com/home/mypage
    parent: undefined, // we fill this out later
    siblings: undefined // we fill this out later
}
```

Next heres a mock for a getParent function that takes `myPage`.

```js
const getParent = (myPage) => {
    return new Promise((resolve, reject) => {
        const temp = myPage;
        temp.pop()
        resolve(temp); // return the parent of /page/mypage
    })
}
```

Next lets develop the getSiblings function. **But Wait!** our getSiblings requires `mypage.parent`! This means that we need to run each promise sequentially.

```js
const getSiblings = (myPage) => {
    return new Promise((resolve, reject) => {
        // lets say we get the siblings through a promise function like below
        const siblings = fetchSiblingsFromDatabase(mypage.parent)
        resolve(siblings)
    })
}
```

So heres the problem. Now we have two promises that can do what we need, however they require a special order. Also, both functions require the `myPage` object, and as we build on more fields and create more dependencies within the object (ie field B depends on data from field A before it can be run) the order matters more and more. So the solution is to run each promise sequentially and chain them together (hencce promise chaining).

Heres our problem summarized.

* Field B (getSiblings) depends on data from Field A (getParent)
* Both `getSiblings` and `getParent` require the `myPage` object
* We have any number of steps to run, we cant hard code steps in a promise chain

Therefore. Because we have dependency between fields within the `myPage` object, `Promise.all` cant work here.

## The solution

The first part of the solution is to create a "list" of instructions that defines the *order* of fields to be created. And a reference to the function that we want to get data from.

```js
// define the steps we want to complete
const templateSteps = [
    {
        name: "parent", // the name of the field we are adding to myPage
        // this step has no dependency because we already have the websitePath
        function: (myPage) => getParent(myPage.websitePath)
    },
    {
        name: "siblings", // the name of the field we are adding to myPage
        // this step has a dependency on the "parent" field that we need to populate in the step above
        function: (myPage) => getSiblings(myPage.parent.join("/"))
    },
]
```

Next lets create the function that orderes these steps sequentially, and does all the stuff we mentioned above.

```js
const steps = templateSteps; // rename this to be clearer

steps.reduce((acc, curr, i, arr) => {
    return acc.then(async (curr) => {
        const step = arr[i]; // get the steps information
        const data = await step.function(curr); // run the build, pass it the current state of myPage
        curr[step.name] = data; // set the data in myPage
        return curr; // return the updated object for the next iteration
    });
}, myPage)
.then((result) => {
    // we are done and have an updated object with our new fields
    console.log(result);
});
```

Lets now break down how `Array.reduce` works.

from [mdn](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce) reduce is a higher order function that has the following parts.

* `acc` - An accumulator (what myPage was previously)
* `curr` - The current state (what myPage is now)
* `i` - The current index
* `arr` - A copy of the array we are reducing (templateSteps)

```js
steps.reduce((acc, curr, i, arr) => {
    // do stuff
}, myPage) // <- acc starts out as myPage

// =====================================================

steps.reduce((acc, curr, i, arr) => {
    // ONLY ONCE the accumulator (the previous myPage object) has resolved from a promise back to JSON
    return acc.then(async (curr) => {
        // then run this code...
        return curr; // return the object for the next iteration
    });
}, myPage) // <- acc starts out as myPage

// =====================================================

steps.reduce((acc, curr, i, arr) => {
    // ONLY ONCE the accumulator (the previous myPage object) has resolved from a promise back to JSON
    return acc.then(async (curr) => {
        // get the steps information
        const step = arr[i];
        
        // Now we run the promise and await its return
        const data = await step.function(curr); // run the build, pass it the current state of myPage

        // set the data in myPage
        curr[step.name] = data;

        // return the object for the next iteration
        return curr;
    });
}, myPage) // <- acc starts out as myPage

// =====================================================

steps.reduce((acc, curr, i, arr) => {
    // ONLY ONCE the accumulator (the previous myPage object) has resolved from a promise back to JSON
    return acc.then(async (curr) => {
        // get the steps information
        const step = arr[i];
        
        // Now we run the promise and await its return
        const data = await step.function(curr); // run the build, pass it the current state of myPage

        // set the data in myPage
        curr[step.name] = data;

        // return the object for the next iteration
        return curr;
    });
}, myPage)
// Finally we just attach a .then() to the reducer to run some code when it completes chaining all the promises
.then((result) => {
    // we are done and have an updated object with our new fields
    console.log(result);
});
```
