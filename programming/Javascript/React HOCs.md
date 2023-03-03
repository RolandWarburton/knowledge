# React Higher Order Components

A HOC is a way to pass component logic to children components.

A HOC is a `React.FC` that returns a `React.FC`.
It doesn't have to be a FC, it can be a more generic `React.ComponentType`
but for the purposes of consistency and specificity i will be using Functional Components only.

## A Basic Pattern Example

Lets learn through example.

We have a component that needs to do two things: to query,
and render some data from the API. The query component logic is quite generic,
so it will be abstracted to a HOC.
The HOC will return a `React.FC` that contains the relevant query functions in its `props` to use.

Components that use this HOC will then implement their own rendering logic.

This decouples the **query** logic from the **render** logic.

## Creating The Query Logic

Its better to start with the HOC first.

The HOC will be a component that returns a component.

```tsx
// when we use our HOC, we need to give it this
interface Props {
  path: string;
}

// our HOC will return these to the chosen component
interface HocProps {
  query: () => { myPath: string };
  path: string;
}

//                   The returned components props \
function withQuery(Component: React.ComponentType<HocProps>) {
  // We return a component with some component logic
  // The component needs to take some props, in this case a path
  return function WithQuery(props: Props) {                     // -+
    const { path } = props                                      //  |
    // example query that all our components that implement     //  +->
    const doQuery = () => {result: 'hi'}                        //  | this component is
    // return the passed in component with its props (HocProps) //  | returned from the HOC
    return <Component path={path} query={doQuery} />;           //  | with its {query} function
  }                                                             // -+
}


// lets skip ahead a bit if you are still a bit confused
// (you should be as we have not covered the next section yet)
//
// at this point lets assume we have created a component with rendering logic,
// lets call that component <Component>, it will take the HocProps above.
// to use this component with our HOC we will write something like below
// const ComponentWithQuery = withQuery(Component)
// <ComponentWithQuery path="test" />
```

## Creating The Rendering Logic

Next we need to create the `Component` that is passed to `withQuery`
that contains rendering logic.

```tsx
// this can be exported from above, and imported here
// as the interface HocProps will be the same
// these are the Props that your rendering component will get access to
interface HocProps {
  query: () => { myPath: string };
  path: string;
}

// the rendering component
function Component(props: HocProps) {
  // use the logic inherited from the HOC
  const const { query, path } = props;
  const result = query();

  // return the component with some UI
  return (
    <div>
      {path} and {result.result} are now in our rendered component
    </div>
  )
}
```

This component will make use of the `query` given to it.
It will also need to take a `path`.

You can see how both `query` and `path` are required above in `withQuery` and `Component`.
These interfaces are the same in both code blocks and should be exported for reuse.

The `Component` returns the rendered content that makes use of the HocProps
(it can use the generic query).

Next we can tie it all together below.

```tsx
const ComponentWithQuery = withQuery(Component);
<ComponentWithQuery path={'example'}>
```

The `Component` is given to the withQuery, returning `ComponentWithQuery`,
which has both the `path`, and `query` available to it.

This makes sense because Component **requires** that it have a `query` in its `HocProps`,
which is provided by the `withQuery` function. This makes `withQuery` a HOC.
