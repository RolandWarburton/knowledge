# Solving Undefined Callbacks in Methods

When creating a class method that receives a callback function. There is a new scope created for `this` which results in some weird behavior.

This error occurs under the following conditions (in this example).

* class MyClass is created with two methods
* we call `MyClass.callingFunc` and pass in `MyClass.callbackFunc` as a callback
* We cannot access properties of MyClass inside `MyClass.callbackFunc`

## Example One - Failing Example

Note that all these examples are in TypeScript format. The same problem applies to JavaScript as well though.

```ts
class MyClass {
    private secret = "Hello"

    public callbackFunc(){
        console.log(this.secret) //3. called from callingFunc and this becomes out of scope!
    }

    // we call this method
    public callingFunc(cb: Function){
        console.log(this.secret) // 1. prints "Hello"
        cb() // 2. calls the callback function
    }
}

const a = new MyClass()
a.callingFunc(a.callbackFunc)
```

```output
Hello
[ERR]: this is undefined
```

Running this code works like this.

1. Prints "Hello" from `callingFunc` who's scope is `MyClass`
2. `callbackFunc` is called from `callingFunc`
3. Now the scope of `this.secret` becomes undefined (because its no longer in the `MyClass` scope)

## Solving the Missing Scope

So to solve this problem we need to track backwards from `callbackFunc` to inspect the "higher order function" that is enclosing it ([source](https://thenewstack.io/mastering-javascript-callbacks-bind-apply-call/)).

in a callback invoked by its enclosing function (`callingFunc`), the `this` context changes to the function that called the callback. In other words `callingFunc` has passed its `this` context onto `callbackFunc` when it called it.

To correctly identify the true scope of `this` all we need to do is look one layer higher. IE the calling layer above `callbackFunc` is `callingFunc`. Therefore the scope is of `callingFunc`

1. Prints "Hello" from `callingFunc` who's scope is `MyClass` **The scope is MyClass because thats the enclosing function/class**
2. `callbackFunc` is called from `callingFunc` **callingFunc passes its scope to cb**
3. Now the scope of `this.secret` becomes undefined (no longer `MyClass`) **the scope is of callingFunc not MyClass**

## Example Two - Solution One - Arrow Functions

The first and easiest solution to fix this scope problem is to apply the following logic.

> "when a method is created and used as a callback. It needs to be converted to an arrow function to preserve its scope"

So lets convert the the callback to an arrow function that preserves the scope of `MyClass`.

```ts
class MyClass {
    private secret = "Hello"

    // convert this to a callback
    public callbackFunc = () => {
        console.log(this.secret) //3. this is in scope of MyClass because its an arrow function
    }

    // we call this method
    public callingFunc(cb: Function) {
        console.log(this.secret) // 1. prints "Hello"
        cb() // 2. calls the callback function
    }
}

const a = new MyClass()
a.callingFunc(a.callbackFunc)
```

```output
Hello
Hello
```

## Example Three - Solution Two - Call, Apply and Bind

Another solution is to use the call, apply, and bind keywords to assign an object (and therefore its scope) to the this keyword ([source](https://www.youtube.com/watch?v=rZc7_2YXbP8)).

Using the **call** keyword we call an object and pass in the scope. The function is then called immediately with that objects scope.

```ts
public callingFunc(cb: Function) {
    console.log(this.secret)
    cb.call(this) // call also takes comma delimated arguments after the first parameter
}
```

Using the **bind** keyword we can bind the this object to `cb` and then call it at a later time.

```ts
public callingFunc(cb: Function) {
    console.log(this.secret)
    cb.bind(this)()
}
```

Or written long hand.

```ts
public callingFunc(cb: Function) {
    console.log(this.secret)
    cb.bind(this)
    cb()
}
```

Using the **apply** keyword acts in the same way as the **call** keyword. The difference is that **apply** takes an array of arguments, and **call** takes comma delimited arguments.

```ts
public callingFunc(cb: Function) {
    console.log(this.secret)
    cb.apply(this, ["myArg1", "myArg2"])
}
```

All these keywords result in the same output that also solves the callback scope problem.

```output
Hello
Hello
```
