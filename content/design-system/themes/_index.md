+++
title: "Themes"
date: 2020-04-15T13:36:45+01:00
weight: 3
draft: false
+++

So far we've included the config in the components. This will only
get us so far as we can eventually see that we wind up reusing the
same colors and fonts over and over so it would actually make sense
to have this unified in a big old `JSON` object.

Combining this centralized config with `styled-system` style scales
gives us the strong foundation we need for developing theming capabilities.

```javascript
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
      fontSize: 12,
      fontWeight: 700,
      lineHeight: ["32px", "36px", "56px"],
      fontFamily: fontFamilies.heading,
      color: colors.blues[2]
    },
  },
  fontSizes: [12, 13, 14, 15, 16, 18, 20, 21, 22, 24, 26, 28, 32, 36, 52],
  colors: {
    ...colors
  }
};

```

Even in smaller applications we can clearly notice the benefits of
having the central theme.
- the colors can be redefined by replacing the definition in a single place
- the font families can be defined with their intended usage

An extra advantage that comes from using `styled-system` is given by the scales.
You will notice the `fontSizes` key in the theme. This contains the scale for
the `font-size` attribute in our text components. This means that the `fontSize: 12`
refers to the index 12(0 based ofc) in the `fontSizes` array. In this case index 12 is
32px in size.

Some design systems use a 5px scale, others may use a 8px scale. The scales in
`styled-system` make switching this very easy.
