# Promise Chaining - Ordering Promises with Dependencies

Sometimes you have a set of functions (or functions that return promises) that you need to run, utilizing `Promise.all()` is a great way of resolving an array on promises all at once. However, I encountered a limitation that I was able to solve with some help from [here](https://stackoverflow.com/questions/30853265/dynamic-chaining-in-javascript-promises) in which a chain of promises need to be run one after the other sequentially (promise 1 resolves before promise 2, etc.). In the problem statement below I will explain a use case for why you might want to do this.

## The problem

Let's say you have a data structure that looks like this. Perhaps we get this initial data from an API, or a database.

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

So heres the problem. Now we have two promises that can do what we need, however they require a special order. Also, both functions require the `myPage` object, and as we build on more fields and create more dependencies within the object (i.e. field B depends on data from field A before it can be run) the order matters more and more. So the solution is to run each promise sequentially and chain them together (hence promise chaining).

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

Next lets create the function that orders these steps sequentially, and does all the stuff we mentioned above.

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

From [mdn](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce) reduce is a higher order function that has the following parts.

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

## Bonus! - Chaining without promises

This idea of chaining functions/promises is actually quite useful for non promise functions as well. You could use this design pattern to complete a set of steps based by making use of currying to lend a small code footprint and make your code adaptable.

In the below example i have a `renderMarkdown()` function that takes some markdown and renders it to HTML, `renderMarkdown()` **always** needs to run, however some post processing steps needs to be run afterwards on the resultant html, for example minifying the HTML to make it production ready.

First lets create a curry function by wrapping the a `postprocess` function (defined anonymouslyly as `return (steps) => {...}`) under the `renderMarkdown()` function

```js
// Example A

renderMarkdown(markdown) {
    // render out the markdown
    let html = marked(markdown)

    // we will see where (steps) comes from in a bit
    return (steps) => {
        for (const step of steps) {
            html = step(html)
        }
        // now that post processing is complete, return the markdown
        return html
    }
}
```

Next lets write a similar **instructions** template for `renderMarkdown()` to take and pass to postProcess.

```js
const minifyOptions = {
    // you can define extra things to pass to your postProcessingSteps array of functions
    // in this example, we might need to pass minify() some options
}

const postProcessingSteps = [
    (html) => minify(html, minifyOptions),
];
```

Then lets use the *postProcessingSteps* array to flesh out `renderMarkdown()`. A curried function is a function (renderMarkdown) that returns another function (postProcess, or just `return () => {}` if we choose to use an anonymous function within renderMarkdown like in example A).

```js
const minifyOptions = {
    // you can define extra things to pass to your postProcessingSteps array of functions
    // in this example, we might need to pass minify() some options
}

const postProcessingSteps = [
    (html) => minify(html, minifyOptions),
];

// We render markdown into html
// then a function is returned that accepts an array of functions to apply postProcessing on that html
const markdownOutputHtml = renderMarkdown(markdownOutput)(postProcessingSteps); // <- we call that 2nd returned function immediately

// 🎉 now we have html thats also processed
```

In an alternate universe, we could also do this in a non curried way, but it would take more lines of code.

Instead we would write the postProcess logic by itself in its own `postProcess` function, making it a **non-anyonymous** (i.e defined) function, this decouples the two functions and makes PostProcess reusable elsewhere, even outside of `renderMarkdown`, but costs more lines of code and looks less elegant.

```js
// Example B

const postProcessingSteps = [
    (html) => minify(html),
];


postProcess(html, steps) {
    for (const step of steps) {
        html = step(html)
    }
    return html
}


renderMarkdown(markdown, steps) {
    let html = marked(markdown)
    return postProcess(html, steps)
}
```
