+++
title = "Putting It Together"
date = 2020-04-19T12:04:09+01:00
draft = false
weight = 5
+++

We had a look at the features of `styled-components` and `styled-system` and we saw that all the features are very well
suited for building an ecosystem for our component library that is able to fulfill all our design needs.

One other key factor that helps us develop features quickly besides the support of the component library is the ability
to quickly prototype and iterate on the components we are building. This is where storybook comes into play. `storybook`
lets us build an great playground for our components.

We briefly looked at `storybook` when we first got set up but in order to unlock its true potential we need to use the
[addons](https://storybook.js.org/addons/https://storybook.js.org/addons/).

Addons will give us the following abilities:
- [knobs](https://github.com/storybookjs/storybook/tree/master/addons/knobs) - they let us play with the props on each component and see the changes interactively in the component story
- [viewport](https://github.com/storybookjs/storybook/tree/master/addons/viewport) - gives us a list of devices similar to the browser devtools with a list of devices that we can switch
  between to see how our components would look on different devices
- [storyshots](https://github.com/storybookjs/storybook/tree/master/addons/storyshots) - this allows us to create a blanket of tests around our components. This is the first way we add testing
  to our components and the most seamless approach. It is similar to how react snapshots work but it does so in a
  central manner and you only need to set it up once
- [console](https://github.com/storybookjs/storybook-addon-console), [actions](https://github.com/storybookjs/storybook/tree/master/addons/actions), [links](https://github.com/storybookjs/storybook/tree/master/addons/links) - gives you the features to enable some of the more complex interactivity in our components
  and also diagnostics and troubleshooting in isolation, in the stories

We need to install the addons we will be using:

```bash
npx lerna add @storybook/addon-actions --dev --scope=@goodreads-v2/component-library
npx lerna add @storybook/addon-knobs --dev --scope=@goodreads-v2/component-library
npx lerna add @storybook/addon-notes --dev --scope=@goodreads-v2/component-library
npx lerna add @storybook/addon-viewport --dev --scope=@goodreads-v2/component-library
npx lerna add @storybook/addon-storyshots --dev --scope=@goodreads-v2/component-library
```

To have a look at practical examples in `storybook` a great resource I have found is [here](https://storybooks-official.netlify.app/)

To get a feel for what others are doing you can check out a gallery of storybook deployments [here](https://storybook.js.org/docs/examples/)

Now that we have installed all the storybook addons we need to configure things so that they work together.

In order to get storyshots working with our setup we need to enable testing abilities for our library.

```bash
npx lerna add jest --dev --scope=@goodreads-v2/component-library
npx lerna add babel-jest --dev --scope=@goodreads-v2/component-library
```

Individual exercises:
1) Add an npm script for running tests in component-library
2) Create configuration file for storyshots to enable snapshot creation
3) Add `react-test-renderer` to allow jest to take snapshots
4) Make use of the created `<Artifika />` header in the goodreads app and check that it is used in a reponsive manner
based on the scales defined in the breakpoints

#### ⚠️  Things to watch out for ⚠️:
- you may come across an issue similar to what we saw in the first chapter with a duplicate react instance
  This can be solved by manually hoisting the package causing it to the problem to the root of the repo and removing it
  from its original package.json. Every time you do this you should clean up your environment removing node modules and
  the `package-lock.json` files
- you may run into an issue while running the snapshot tests caused by the static fonts. Static files need to be mocked
  as they will not actually create image snapshots but virtual dom snapshots
- check the console for warnings. you can get useful information, for example if you have two instances of styled
  components the theme is not applied correctly which can lead to unpredictable results
