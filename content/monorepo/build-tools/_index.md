+++
title = "Build Tools"
chapter = false
weight = 4
date = 2020-03-07T16:59:44Z
draft = false
+++

### Build stages of the project:
- build component library

  We want to be able to build the component library so we can use it in the goodreads app

- build storybook for demo of the design system

  In production ready projects we want to have a design system and
  the ability to deploy that design system in order to be able to do incremental
  demos of the various components in isolation.

- build the app that uses the component library

  this will be the end result of our efforts, the main project 🎉

### Tools

1) for building the component library - [rollup](https://rollupjs.org/guide/en/)
2) for wiring up a demo-able component library - [storybook](https://storybook.js.org/)
3) for building the end to end project - [webpack](https://webpack.js.org/)

{{% notice tip %}}
This is a slightly opinionated approach however it is a recommended approach for
production systems because `rollup.js` is performant and easy to configure.
{{% /notice %}}

#### rollup

rollup will make use of babel in order to transpile es6 code into code that can be
run in the browser. We need to add a babel config for transpiling es6 and we need to
add the required presets and plugins.

The babel config is a standard config for react apps. The presets are collections
of plugins that allow you to transpile react code for example. The plugins list gives
you the ability to specify fine grained support for es features(ie. spread operator support)

```javascript
// packages/component-library/babel.config.js
module.exports = api => {
  api.env(["development", "test"]);
  const plugins = [
    "@babel/plugin-syntax-dynamic-import",
    "@babel/plugin-proposal-class-properties",
    "@babel/plugin-proposal-export-default-from",
    "@babel/plugin-proposal-export-namespace-from",
    "@babel/plugin-proposal-object-rest-spread"
  ];
  const presets = [
    ["@babel/preset-env", { forceAllTransforms: true }],
    "@babel/preset-react"
  ];

  return { plugins, presets };
};
```

along with the babel config we also need to have a `packages/component-library/rollup.config.js` that
is used by rollup for generating the `packages/component-library/lib/component.bundle.js`

rollup has a large collection of plugins that is helpful for bundling the components

```javascript
// node-resolve will resolve all the node dependencies
import resolve from "@rollup/plugin-node-resolve";
import commonjs from "@rollup/plugin-commonjs";
import babel from "rollup-plugin-babel";
import bundleSize from "rollup-plugin-bundle-size";

export default {
  input: "src/components/index.js",
  output: {
    file: "lib/component.bundle.js",
    format: "cjs"
  },
  // All the used libs needs to be here
  external: [
    "react",
    "react-dom",
    "prop-types",
    "styled-components",
    "styled-system"
  ],
  plugins: [
    resolve(),
    babel({
      exclude: "node_modules/**"
    }),
    commonjs({
      namedExports: {
        "react-is": ["typeOf", "isElement", "isValidElementType"]
      }
    }),
    bundleSize()
  ]
};
```

#### rollup config

- `input`: the index file that re-exports the components from the
  library, it's the index that contains all the components in the library
- `output`: the path to the bundle
- `external`: the third party libs that should not included in the final bundle.
  This should be specified in order not to bloat the bundle size.
- `plugins`: features to be available for your build process and your final bundle
  (ie. minification, bundlesize, babel etc)

The standard set of plugins for es6 + React is:
- `@rollup/plugin-node-resolve`
- `@rollup/plugin-commonjs`
- `rollup-plugin-babel`

Extras:
- `rollup-plugin-terser` - minify and uglify, optimizes for bundle size
- `rollup-plugin-sourcemaps` - adds sourcemaps to help with debugging
- `rollup-plugin-bundle-size` - this is pretty cool as it gives you the resulting bundle size

{{% notice tip %}}
The rollup config is javascript so you can combine the configuration arrays using
js to fetch environment variables and combining the options as required so that your bundle
is optimally configured for the environment you are building for.
{{% /notice %}}

### Storybook
