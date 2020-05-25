+++
title = "Cypress Testing"
date = 2020-05-08T14:40:43+01:00
weight = 3
draft = false
+++

## Integration
There are multiple ways to do integration testing and automating the tests that you could do with manual testing. There
are online services that provide cross browser testing and testing across multiple devices. [Browserstack](https://www.browserstack.com/)
is one of the most popular services for testing on a wide range of browsers.

A few years ago Selenium was the de facto standard for testing web apps but the new kid on the block is Cypress, and
more and more adoption because of the distribution method and easier API. It is distributed as an npm package.

## [Cypress](https://docs.cypress.io)

```bash
npx lerna add cypress --dev --scope=@goodreads-v2/goodreads
```
This installs the cypress package and that is all there is to it. We can now start writing tests. This is much easier
than selenium and webdriver. The cypress tools allow for a great deal of flexibility and configuration and you can try
them out locally quite easily.

```bash
npx cypress open # run this from inside packages/goodreads
```
This will open the browser and test runner for your app. It is very intuitive and has quite a few features. It will
allow you to run your tests on multiple browsers with little to no effort.

{{< lazy-image image="runner-tool.png" lightbox=false />}}

You can see that the cypress folder contains a fair amount of stuff, but it is quite neatly organized. You will notice
the integration, screnshots and videos folders, this is where the videos and screenshots will be stored for your test
runs.
```bash
▾ [ ]cypress/
  ▸ [ ]fixtures/
  ▾ [ ]integration/
    ▸ [ ]examples/
    ▸ [ ]goodreads/
  ▾ [ ]plugins/
      [ ]index.js
  ▸ [ ]screenshots/
  ▸ [ ]support/
  ▸ [ ]videos/
▸ [ ]node_modules/
▸ [ ]public/
▾ [ ]src/
  ▸ [ ]components/
  ▸ [ ]containers/
  ▸ [ ]routes/
  ▸ [ ]store/
  ▸ [ ]test-utils/
    [ ]index.css
    [ ]index.js
    [ ]serviceWorker.js
    [ ]setupTests.js
  [ ].gitignore
  [ ]cypress.json
  [ ]package-lock.json
  [ ]package.json
  [ ]README.md
```

{{% notice tip %}}
The amount of features available in cypress is pretty large and it is quite mature, luckily the documentation is great
as well and rich in [examples](https://docs.cypress.io/examples/examples/recipes.html#Fundamentals), I encourage you to
have a dive into it yourself if you decide to use it.
{{% /notice %}}

I will use it to write some tests that will provide some degree of confidence about the app that the fundamental
scenarios work(this is called a [smoke test](https://en.wikipedia.org/wiki/Smoke_testing_(software))):
- registration works
```javascript
// packages/goodreads/cypress/integration/goodreads/register.spec.js
describe('Registration works', () => {
  beforeEach(() => {
    cy.request('POST', 'http://localhost:3000/test/snapshot', {})
    cy.visit('http://localhost:3000/register')
  })
  afterEach(() => {
    cy.request('POST', 'http://localhost:3000/test/restore-snapshot', {})
  })
  it('Can type into the input and the value changes', () => {
    cy.get('#email')
      .type('fake@email.com')
      .should('have.value', 'fake@email.com')
      .clear()
      .type('slow.typing@email.com', { delay: 100 })
      .should('have.value', 'slow.typing@email.com')
    cy.get('#password').should('have.attr', 'type', 'password')
    cy.get('#password').type('password').should('have.value', 'password')
    cy.get('#re-password').type('password').should('have.value', 'password')

    cy.server()
    // Listen to GET to comments/1
    cy.route('POST', '/auth/register').as('request')

    cy.get('button:contains("Register")').click()

    // wait for GET comments/1
    cy.wait('@request').its('status').should('eq', 201)
  })
})
```
You will notice that this test can only be run once because it alters API state. A good approach is to have an endpoint
on your API that allows you to revert to an older version of the data.json.

#### Brainstorming
Can you think of other tests that would make sense as part of the smoke test? Keep in mind that our focus is to have the
right level of testing at the correct layer.

## Visual testing
Cypress also provides a range of [plugins](https://docs.cypress.io/guides/tooling/plugins-guide.html#Use-Cases) that add
enhancements to the already good testing toolkit cypress provides out of the box. The one thing we've been missing and
makes human intervention valuable is testing the app visually, clicking around, looking at how the elements behave, if
they get cropped on smaller screens etc.

This is tedious and prevents testers from being creative and doing actually the work that can't be automated, like
designing test suites and adapting the overall test suite architecture.

The cypress [visual regression plugin](https://github.com/meinaart/cypress-plugin-snapshots) is the package we will be
using for doing a quick MVP for how we would do visual testing in  our cypress test suites.

#### Getting started
```bash
# add the cypres plugin package
npx lerna add cypress-plugin-snapshots --dev --scope=@goodreads-v2/goodreads
```

The `cypress.json` config file needs to be updated, it allows for various config
[settings](https://github.com/meinaart/cypress-plugin-snapshots#command)
{{< highlight json "hl_lines=5">}}
// update the packages/goodreads/cypress.json to use options for the plugin where to store assets
{
  "nodeVersion": "system",
  "baseUrl": "http://localhost:3000",
  "ignoreTestFiles": ["**/__snapshots__/*", "**/__image_snapshots__/*"]
}
{{< /highlight >}}

```javascript
// in packages/goodreads/cypress/plugins/index.js add a hook for the new plugin
const { initPlugin } = require('cypress-plugin-snapshots/plugin')

module.exports = (on, config) => {
  initPlugin(on, config)
  return config
}
```

Last step
```javascript
// in packages/goodreads/cypress/support/index.js add a command in order to be able to use the plugin from cy
import './commands'
import 'cypress-plugin-snapshots/commands'
```

#### Trying it out
A simple test for checking the `/register` path in the app for visual regressions looks something like this.
```javascript
// packages/goodreads/cypress/integration/goodreads/register.spec.js
  ...
  it('Looks as expected', () => {
    cy.document().toMatchImageSnapshot({
      "createDiffImage": true,                // Should a "diff image" be created, can be disabled for performance
      "threshold": 0.01,                      // Amount in pixels or percentage before snapshot image is invalid
      "name": "custom image name",            // Naming resulting image file with a custom name rather than concatenating test titles
      "separator": "custom image separator",  // Naming resulting image file with a custom separator rather than using the default ` #`
      "thresholdType": "percent",             // Can be either "pixel" or "percent"
    })
  })
  ...
```
In the initial run it will pass automatically as there is no previous snapshot to check against, however that will be
the base image that future runs will try to match against. The resulting directory tree looks like this:
```bash
  ▾ [ ]integration/goodreads/
    ▾ [ ]__image_snapshots__/
        [ ]Registration works  Looks as expected #0.png
      [ ]register.spec.js
```

We will also add a few scripts to make running the scripts quicker
{{< highlight json "hl_lines=19-21" >}}
// package.json
  ...
  "scripts": {
    "build:components": "lerna exec npm run build --scope=@goodreads-v2/component-library",
    "storybook": "lerna exec npm run storybook --scope=@goodreads-v2/component-library",
    "build-storybook": "lerna exec npm run build-storybook --scope=@goodreads-v2/component-library",
    "start:app": "lerna exec npm start --scope=@goodreads-v2/goodreads",
    "start:server": "lerna exec npm run server:dev --scope=jungle-jim",
    "start:goodreads": "npm-run-all --parallel start:server start:app",
    "clean:package-locks": "find . -type f -name 'package-lock.json' -exec rm {} +",
    "clean:lerna": "lerna clean",
    "clean:root-modules": "rm -rf node_modules",
    "clean": "npm-run-all clean:lerna clean:root-modules clean:package-locks",
    "bootstrap": "npm i && lerna bootstrap",
    "test:components": "lerna exec npm test --scope=@goodreads-v2/component-library",
    "test:goodreads": "lerna exec npm test --scope=@goodreads-v2/goodreads",
    "test:goodreads:cov": "lerna exec npm run test:coverage --scope=@goodreads-v2/goodreads",
    "start:goodreads:cy": "lerna exec npm run cypress:open --scope=@goodreads-v2/goodreads",
    "start:server:test": "lerna exec npm run server:test --scope=jungle-jim",
    "start:goodreads:test": "npm-run-all --parallel start:server:test start:app",
    "test:goodreads:cy": "lerna exec npm run cypress:run --scope=@goodreads-v2/goodreads"
  },
  },
  ...
{{< /highlight >}}

{{< highlight json "hl_lines=9-10">}}
// packages/goodreads/package.json
  ...
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "test:coverage": "CI=true react-scripts test --env=jsdom --collect-coverage",
    "eject": "react-scripts eject",
    "cypress:open": "cypress open",
    "cypress:run": "cypress run"
  },
  ...
{{< /highlight >}}

It's a good idea to create npm scripts for delegating from lerna to the packages in the repo as it will make running the
commands easier and doesn't require you to change into their respective directory every time.
{{% notice tip %}}
The great benefit of using this type of testing is that you can actually have a test suite that checks how pages look
on various size devices, check for cropping on smaller sizes etc..., and you only need to do this once 🎉🎉🎉
{{% /notice %}}

{{% notice info %}}
The downside is that these tests are slow to run so you need to keep that in mind when adding more. You also need to
make sure there is some degree of parallell execution in the tests and separate tests by impacted areas. For example you
should probably not run the tests for all pages if you changed a component that is used on a single page. __The good
news__ is that you can test only __individual elements__ on a page.
{{% /notice %}}
