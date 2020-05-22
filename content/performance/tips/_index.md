+++
title = "What to look for"
date = 2020-05-13T17:18:47+01:00
weight = 2
draft = false
+++

## There are a few to try when targeting optimization

#### Hooks `useCallback` and `useMemo`
- `useCallback` should be used to avoid components creating multiple instances of callback handlers on your component.

Often found in the wild:
```javascript
    <button onClick={(event) => this.handleClick(event)}>Click for fireworks!</button>
```

With `useCallback`
```javascript
    const handleClick = useCallback(
        (event) => console.log(event.target.value),
        []
    )
    ...
    <button onClick={handleClick}>Click for fireworks!</button>
```
The catch with arrow functions used as event handlers is that whenever the component rerenders a new copy of the handler
is created. This can increase memory consumption over time.

#### `react-virtualized` & `react-window`
When working with a large number of components most times the screen will not provide enough real estate or displaying
all of them. This makes rendering all of them quite counter intuitive and at the same time it will burden the UI with
the extra rendering.

There are a few libraries like `react-virtualized` and `react-window` that gives you a faster alternative and more
lightweight API that focuses on rendering the visible portion of the screen rather than everything in one go.

#### React `PureComponent`
Usually when implementing components you want to aim for writing them in a functional manner. This is not required
however the recent developments by the __react core team__ we can see that the trend is to write your components in a
functional manner whenever you can.

A `PureComponent` is very similar to a functional component, given `state`/`props`, you render the same way.
Predictability is fun when working with the UI.

The difference between functional components and PureComponents is that in PureComponents you can make use of React's
lifecycle hooks.

#### Reselect for redux computation
Reselect is the library to use with redux for optimizing state management. We touched on it in the redux chapter.

#### Web Workers
Web Workers are a way to actually parellelize the execution of JS in your browser. This will be the option you have when
you have to optimize computation. This is one of the optimization technique that deals with pure performance. It
involves only hardcore code execution speed up, not any UX trickery.

#### React.memo()
This is the class based version of the `useMemo` hook. You can use this to memoize components. `PureComponent`s are a
great candidate for memoization.

Besides the default benefits of memoization there is another extra feature of `React.memo()` which allows for smarter
caching by specifying the second argument like so:

```javascript
React.memo(Component, [areEqual(prevProps, nextProps)]);
```

#### Lazy loading `React.lazy()`
Lazy loading allows for making your UI feel more snappy and prevents your UI from blocking when you have large
components. This was apparent with our large BookCard list for example.

#### Smarter `shouldComponentUpdate` implementation
When you want to get the most out of your rendering performance you want to drill down and get a good handle on when
your components will update. You do this by providing custom implementations of the `shouldComponentUpdate` lifecycle
hook. You need to make it custom to prevent unnecessary renders.

## Done
This checklist is a good starting point for diagnosing and improving performance, can't stress enough though that you
should focus on performance when it becomes noticeable.

There are some guidelines that you can follow as to not get to a point where it becomes a burden, but it is important to
have a good measurement of performance value for the project you are working on.
