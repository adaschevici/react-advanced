---
title: "Typography"
date: 2020-04-15T13:36:31+01:00
weight: 2
draft: false
---

Typography is the set of fonts and font sizes that is used in your pages.
You use it to define the sizes and weights of the different text sections.
While this may seem to be very little responsibility to assign
to a component it is actually very much in line with atomic design concepts in
that it describes one of the most primitive aspects in your library, text.

This seems like an unusual amount of red tape to add some text but creating a standard
around this will keep you mindful about text types available and help you keep track
of the code duplication in your components and highlight the opportunities for refactoring
dependencies.


#### Fonts
Your fonts can be hosted on the CDN of choice or bundled with the application from
font files. Some projects will have fonts that are available from [google fonts](https://fonts.google.com/).

Most times you will have to import custom fonts, `.woff` and `.woff2` files.
This is not something you will do very often when working on a project however
it is one of the big decisions when starting the project out, especially if the
design is a highly custom and potentially exotic one.

The way to create the global import for the fonts is by using the `createGlobalStyle` from `styled-components`
```javascript
// src/fonts/index.js
import { createGlobalStyle } from "styled-components";

import SpookyWoff from "./CCSpookytooth-Italic.woff";
import SpookyWoff2 from "./CCSpookytooth-Italic.woff2";

export default createGlobalStyle`
  @font-face {
    font-family: 'Spooky Italic';
    src: local('Spooky Italic'), local('SpookyItalic'),
    url(${SpookyWoff2}) format('woff2'),
    url(${SpookyWoff}) format('woff');
    font-weight: 300;
    font-style: normal;
  }
`;
```

The way to import it in your app is on its own at the top level in your app
as you would do with a regular font file from CDN.

This also impacts the `rollup.config.js`. Because font files are static assets
you need to add a new rollup plugin to copy the files along with the bundle to allow for importing.

```bash
npx lerna add rollup-plugin-copy -D --scope=@goodreads-v2/component-library
```
`rollup.config.js` should become this. You need to add the `woff*` files as external
so the bundling does not attempt to transpile the font files. The woff files also need
to be copied so that they can be imported from the resulting bundle.

{{< highlight javascript "hl_lines=6 22-23 35-42" >}}
// node-resolve will resolve all the node dependencies
import resolve from "@rollup/plugin-node-resolve";
import commonjs from "@rollup/plugin-commonjs";
import babel from "rollup-plugin-babel";
import { terser } from "rollup-plugin-terser";
import copy from "rollup-plugin-copy";
import bundleSize from "rollup-plugin-bundle-size";

export default {
  input: "src/index.js",
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
    "styled-system",
    "./Artifika-Regular.woff",
    "./Artifika-Regular.woff2"
  ],
  plugins: [
    resolve(),
    babel({
      exclude: ["node_modules/**", "*.woff*"]
    }),
    commonjs({
      namedExports: {
        "react-is": ["typeOf", "isElement", "isValidElementType"]
      }
    }),
    copy({
      targets: [
        {
          src: ["src/fonts/*.woff*"],
          dest: "lib/fonts"
        }
      ]
    }),
    terser(),
    bundleSize()
  ]
};
{{< / highlight >}}

#### Variants
Besides defining and bundling the fonts we also need to declare
the size variants for Headings(h1, h2, h3, etc), body, footer and so on.
While it may seem that the avialable text format tags in html give enough
flexibility in complex projects you will find that typography components
can blow up in number.

In order to avoid an extensive number of font styles you need to have a
primitive type that is highly configurable while also
defining a common base. You can think of it as base OOP classes that implement
functionality that is shared between objects.

For building the variants `styled-components` goes hand in hand with `styled-system`.
`styled-components` provides the css in JS while `styled-system` provides the standards
to define the generic components for your different text styles.

The generic component that is used for text components, as a base looks something like this.

```javascript
import React from "react";
import styled from "styled-components";
import {
  space,
  lineHeight,
  fontSize,
  fontStyle,
  size,
  color,
  colorStyle,
  textStyle,
  fontFamily,
  fontWeight,
  letterSpacing,
  borderRadius
} from "styled-system";

const StyledDynamicComponent = styled.div`
  ${space}
  ${fontSize}
  ${fontStyle}
  ${color}
  ${size}
  ${colorStyle}
  ${textStyle}
  ${lineHeight}
  ${letterSpacing}
  ${fontFamily}
  ${fontWeight}
  ${borderRadius}
`;

const DynamicComponent = ({ tag = "div", children, ...props }) => {
  return (
    <StyledDynamicComponent as={tag} {...props}>
      {children}
    </StyledDynamicComponent>
  );
};

DynamicComponent.defaultProps = {
  tag: "div"
};

export default DynamicComponent;
```

The list of props imported from `design-system` is the list of options that
the components that use this mixin are able to receive and as you can see
it is very configurable.

We then use the `DynamicComponent` to wrap our various text styles.
```javascript
// src/typography/artifika.js
import React from 'react'
import DynamicComponent from './dynamic-component'

const colors = {
  blue: "#004170",
}

const artifika = {
  tag: "h1",
  fontSize: 12,
  fontWeight: 700,
  lineHeight: "32px",
  fontFamily: "Artifika",
  color: colors.blue
}

export default props => (
  <DynamicComponent {...artifika} {...props}>
    {props.children}
  </DynamicComponent>
);
```

Wiring everything together we need to re-export the defined components from
the index file in our typography so that we can use it like this: `import { Spooky } from "typography";`
```javascript
import Artifika from './artifika'

export { Artifika }
```
