+++
title = "Testing Hierarchy"
date = 2020-05-08T09:30:05+01:00
weight = 1
draft = false
+++

## DevOps

The term DevOps is loosely used and it can have multiple semantics depending on the context. The way we will use it
throughout the course and what it refers to in the development lifecycle is a mindset of having a __robust pipeline for
features development__.

The pipeline relies on tight feedback loops all throughout the different product teams. Imagine if one of the product
features is retired, the benefit of the teams being able to function autonomously is that the delivery is not affected
one bit.

#### In practice

{{< mermaid >}}
graph LR
    subgraph project
        id1(team 1)-->id2(team2)
        id1(team 1)-->id1(team 1)
        id2(team 2)-->id3(team3)
        id2(team 2)-->id2(team 2)
        id3(team 3)-->id3(team 3)
    end
{{< /mermaid >}}

The team feedback loop is the automated test suite. This allows you to get feedback on changes as you
write your code. Because we are working on the frontend we need feedback both on functionality and visuals.
__Storybook__ is a great tool for helping us retrieving visual feedback.

The tight feedback loop at the feature team layer helps with rapid turnaround and faster onboarding for new devs. Standing up a
production environment should be as easy as pushing a button(in dev speak running a script). Having these tighter
feedback loops also allows you to have a clear understanding of the definition of done, both in terms of compliance with
the __build should be green__ mantra and the visual feedback from our design system set up.

#### Testing
As far as functional testing is concerned it can be done partly in storybook to check that callbacks and event handlers
are wired in correctly, however it will require a fair amount of stubbing and mocking. The boundaries between our
features should not be crossed while writing unittests in order to not introduce change coupling.

{{< lazy-image image="test-pyramid.png" lightbox=false />}}

We should focus on having __as many unittests as possible__ for the components, but we need to also be pragmatic about it
and not allow the mocks and stubs ecosystem blow out of proportion. We want to keep the cost of unittests to a minimum
and having to maintain a mocks system can become a separate project over time.

We also want to measure test coverage and periodically delete tests that don't decrease coverage. __If you have tests
that don't add coverage, those should not exist__

#### Test coverage
We should add coverage reports to both our component library and our application. The way to add it is by changing
package.json
```json
// packages/goodreads/package.json
...
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "test:coverage": "CI=true react-scripts test --env=jsdom --collect-coverage",
    "eject": "react-scripts eject"
  },
...
```
After we added the coverage test script in our respective packages we also want to have a shortcut script to run it from
the root. As previously the script will be a lerna command wired into our npm scripts.

{{< highlight json "hl_lines=20">}}
  ...
  "scripts": {
    "clean": "lerna clean",
    "bootstrap": "npm i && lerna bootstrap",
    "clean:root": "rm -rf node_modules",
    "clean:locks": "find . -type f -name 'package-lock.json' -exec rm {} +",
    "clean:all": "npm-run-all clean clean:root clean:locks",
    "bootstrap:all": "npm i && npm run bootstrap",
    "build:components": "lerna exec npm run build --scope=@goodreads-v2/component-library",
    "build:storybook": "lerna exec npm run build-storybook --scope=@goodreads-v2/component-library",
    "build:app": "lerna exec npm run build --scope=@goodreads-v2/goodreads",
    "start:storybook": "lerna exec npm run storybook --scope=@goodreads-v2/component-library",
    "static:storybook": "docker run --rm -p 8080:80 -v $(pwd)/packages/component-library/storybook-static/:/usr/share/nginx/html nginx",
    "static:goodreads": "docker run --rm -p 8080:80 -v $(pwd)/packages/goodreads/build/:/usr/share/nginx/html nginx",
    "start:goodreads": "lerna exec npm start --scope=@goodreads-v2/goodreads",
    "start:server": "lerna exec npm run server:dev --scope=jungle-jim",
    "start:app": "npm-run-all --parallel start:server start:goodreads",
    "snapshot:components": "lerna exec npm run test --scope=@goodreads-v2/component-library",
    "test:app": "lerna exec npm run test --scope=@goodreads-v2/goodreads",
    "test:app-cov": "lerna exec npm run test:coverage --scope=@goodreads-v2/goodreads"
  },
  ...
{{< /highlight >}}

We have some tests that we added in the component library, let's have a go at adding coverage reporting for them.
