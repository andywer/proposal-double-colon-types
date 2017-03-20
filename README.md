# Babel syntax proposal: Double colon type declarations

JavaScript / Flow syntax proposal. Separate declaration and implementation of functions, variables, ... in order to improve developer experience when writing typed JS code.

The idea for this syntax has been inspired by the syntax of [elm](http://elm-lang.org/), [haskell](http://www.haskell.org/) and the comments in the snippets shown in [JavaScript without loops](http://jrsinclair.com/articles/2017/javascript-without-loops/).

Check out [Professor Frisby's Mostly Adequate Guide to Functional Programming, Chapter 7](https://drboolean.gitbooks.io/mostly-adequate-guide/content/ch7.html) if you seek further reading. The guide is about JavaScript, using the same syntax, but just in comments.

*Disclaimer: Like many language features this one is also subject to personal preference and you might or might not like it.*


## What it is

Static typing in JavaScript is a nice and powerful opt-in feature. Both TypeScript and Flow use the same syntax of inlining the type declarations as part of the actual implementation code. While that works well in general, it also tends to make the code slightly harder to read, as it happens in C, C++, Java, ..., too.

There is a different way favored by popular functional programming languages which chose to go a different way: Separating the declaration and the implementation.

This proposal covers adding this syntax to JavaScript using the `::` operator:

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

2. It becomes trivial to track / recognize API changes. As soon as you split declaration and implementation you can track breaking interface changes by just watching the declarations and use this knowledge for semantic versioning. Elm, for example, does automatic semantic versioning based on this.

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


## What it is not

This syntax is intended to be consumed by flow/babel and other code analyzers/transpilers. It is not intended to become an ES core language feature.


## Try it yourself

Check out [gear](https://github.com/andywer/gear). It is a small tool that makes Babel and Flow work with those types. Futhermore its own code uses this very syntax extension, so check out its code to see a working example of code typed this way.


## Manual setup

Babel as well as its parser Babylon had to be patched/extended in order to parse the double-colon type syntax. You can find the Babylon fork [here](https://github.com/andywer/babylon/tree/feature/dblcolon-types). Clone the feature branch and run `yarn && npm run build && npm link`.

The Babel fork can be found [here](https://github.com/andywer/babel/tree/feature/dctypes). Clone the feature branch and run `yarn && lerna bootstrap && npm run build` in the project root directory, then `cd packages/babel-core && rm -rf node_modules/babylon && npm link babylon`. Finally `cd ../babel-types && npm link`.

To set up the samples in `dctypes-test/` in the babel root directory:

```sh
cd dctypes-test/
npm install
rm -rf node_modules/babylon node_modules/babel-types
npm link babylon
npm link babel-types
```

`npm run` will show you the samples to run.


## Available transformations

### babel-plugin-transform-to-flow

#### Source

```js
// test-code.js

name :: string

let name = 'Harry'

add :: (number, number) => number

function add (x, y) {
  return x + y
}
```

#### Command

```sh
babel --plugins=transform-dctypes-to-flow --no-babelrc test-code.js
```

#### Output

```js
/* @flow */

let name: string = 'Harry';

function add(x: number, y: number): number {
  return x + y;
}
```

### babel-plugin-transform-dctypes-comments

#### Source

```js
// test-code.js

add :: (number, number) => number

function add (x, y) {
  return x + y
}
```

#### Command

```sh
babel --plugins=transform-dctypes-comments --no-babelrc test-code.js
```

#### Output

```js

// add :: (number, number) => number
function add(x, y) {
  return x + y;
}
```

### babel-plugin-transform-dctypes-to-flow & babel-plugin-flow-runtime

#### Source

```js
// test-code.js

add :: (number, number) => number

function add (x, y) {
  return x + y
}
```

#### Command

```sh
NODE_ENV=development babel --plugins=transform-dctypes-to-flow,flow-runtime --no-babelrc test-code.js
```

#### Output

```js
/* @flow */import t from "flow-runtime";


function add(x: number, y: number): number {
  let _xType = t.number();

  let _yType = t.number();

  const _returnType = t.return(t.number());

  t.param("x", _xType).assert(x);
  t.param("y", _yType).assert(y);

  return _returnType.assert(x + y);
}
t.annotate(add, t.function(t.param("x", t.number()), t.param("y", t.number()), t.return(t.number())));
```


## Limitations

- Does not yet work with `flow-bin`. The static flow type checker is written in OCAML and it would need to be patched as well.
- Just a proof-of-concept right now. Don't be too mad if it does not work in edge-cases yet.


## Feedback

Having feedback? Feel free to open an issue or pull request 😉
