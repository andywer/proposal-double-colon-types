# Babel syntax proposal: Double colon type declarations

JavaScript / Flow syntax proposal. Separate declaration and implementation of functions, variables, ... in order to improve developer experience when writing typed JS code.

The idea for this syntax has been inspired by the syntax of [elm](http://elm-lang.org/), [haskell](http://www.haskell.org/) and the comments in the snippets shown in [JavaScript without loops](http://jrsinclair.com/articles/2017/javascript-without-loops/).

*Disclaimer: Like many language features this one is also subject to personal preference and you might or might not like it.*


## What it is

Static typing in JavaScript is a nice and powerful opt-in feature. Both TypeScript and Flow use the same syntax of inlining the type declarations as part of the actual implementation code. While that works well in general, it also tends to make the code slightly harder to read, as it happens in C, C++, Java, ..., too.

There is a different way favored by popular functional programming languages which chose to go a different way: Separating the declaration and the implementation. This proposal covers adding this syntax to JavaScript using the `::` operator:

```js
minimum :: (...number) => number

function minimum (...args) {
  return args.reduce(
    (min, arg) => arg < min ? arg : min,
    0
  )
}
```

## Benefits

1. It is easier to add to existing non-typed code, since you do not even need to touch the actual code. You just need to add the declaration.

2. It becomes trivial to track / recognize API changes. As soon as you split declaration and implementation you can track breaking interface changes by just watching the declarations and use this knowledge for semantic versioning. Elm, for example, takes great advantage of this, doing semantic versioning automatically.

3. For functions having many parameters with complex types it can increase readability, since a medium-length declaration line and a medium-length implementation line are easier to grasp than a single long line including both.

4. When passing an object as a function parameter you can specify the object properties' types more elegantly. Consider a functional React component, for instance:

```js
Greeting :: ({ name: string }) => React.Element

function Greeting ({ name }) {
  return <div>Hello, {name}!</div>
}

// You could not write `function Greeting ({ name: string }): React.Element {`,
// since `{ name: string }` would be interpreted as a `const string = props.name`.
```


## Available transformations



## Setup



## Limitations

- Does not yet work with `flow-bin`. The static flow type checker is written in OCAML and it would need to be patched as well.
- Just a proof-of-concept right now. Don't be too mad if it does not work in edge-cases yet.
