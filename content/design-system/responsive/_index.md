---
title: "Responsive"
date: 2020-04-15T13:36:55+01:00
draft: false
---

The most amazing feature of the `styled-system` are the scales.
The true power of scales is apprent when we deal with responsive design.

I for one have always found it to be a real pain when aiming to build
websites that look great on every device. While you hear this quite a lot
in requirements, it's a red herring 🍣.

Before trying to make a website look good on a specific device check your
customer base to see what devices most your users have and make it look
great for the 80%, and style it so that it is not unusable on the other 20%.
There is a wide range of tools to test your responsive design, including
the chrome responsive dev-tool. This will cover most cases however for
fool proof testing you may need to turn to other more professional tools
like [browserstack](https://www.browserstack.com/).

#### `styled-system` scales for responsiveness
In order to cater for different device sizes we need to define a breakpoints
array. Once we define that we can use fontSize and other scaled properties
to define specific properties for some screen sizes.

{{< highlight javascript "hl_lines=27 35" >}}
// http://chir.ag/projects/name-that-color/
const colors = {
  bostonBlue: "#428bca",
  stTropaz: "#2a6496",
  maroonFlush: "#c7254e",
  softPeach: "#f9f2f4",
  mantis: "#6ECE5A",
  citrineWhite: "#FBF7DC",
  blues: [
    '#004170',
    '#006fbe',
    '#2d8fd5',
    '#5aa7de',
  ]
};

const fontFamilies = {
  heading: "Spooky Italic",
  body: "PT Serif, Helvetica, Arial, sans-serif",
  code: "Roboto Mono, monospace"
};

export default {
  textStyles: {
    spooky: {
      tag: "h1",
      fontSize: [12, 13, 14],
      fontWeight: 700,
      lineHeight: ["32px", "36px", "56px"],
      fontFamily: fontFamilies.heading,
      color: colors.blues[2]
    },
  },
  fontSizes: [12, 13, 14, 15, 16, 18, 20, 21, 22, 24, 26, 28, 32, 36, 52],
  breakpoints: ["319px", "599px"],
  colors: {
    ...colors
  }
};
{{< /highlight >}}

The breakpoints, combined with the array of sizes will make the prop
be lowest index from the prop array.
The above config:
- fontSize 32px and lineHeight 32px if device width < 319px
- fontSize 36px and lineHeight 36px if device 599px > width > 319px
- fontSize 52px and lineHeight 52px if device width > 599px

This is a bit strange at first but it is quite useful in the long run to
prevent the responsive design bloat.

Another amazing feature for responsive design available in `styled-system`
is the ability to specify scales for the various breakpoints for containers
ie:

```javascript
<Card
  width={[
    1, // 100% below the smallest breakpoint (all viewports)
    1 / 2, // 50% from the next breakpoint and up
    1 / 4, // 25% from the next breakpoint and up
  ]}
/>
```

Also you don't have to specify the style in the theme, you can also
try it out inline in your components for quick prototyping.
```javascript
// responsive margin
<Text m={[ 0, 1, 2 ]} />

// responsive padding
<Text p={[ 2, 3, 4 ]} />

// responsive font-size
<Text fontSize={[ 3, 4, 5 ]} />
```

The `styled-system` [documentation](https://styled-system.com/) is a great source for tips.
