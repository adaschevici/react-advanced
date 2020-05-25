+++
title = "Circle CI"
date = 2020-05-10T14:51:30+01:00
weight = 4
draft = false
+++


{{% notice tip %}}
Now that we have a firm grasp on all the things related to testing and the testing pyramid we should feel pretty good
about that. The next step is to run tests automatically, on a continuous integration platform.
{{% /notice %}}


## [Circle CI](https://circleci.com/)
In the previous chapters we spent some time adding npm scripts at the root of the project that delegate to the child
packages. We ran the integration tests while also running the app. In an automated environment this won't work. We need
to start the app before running our cypress tests.

A really nifty command line tool that is available as an npm package is `start-server-and-test`. This is the recommmended way to run
your app and once the app is ready to serve requests you can start running the cypress tests.

In order to be able to run the tests you need to run the mocked API, the webpack server and the test runner in the same
command. This is an npm script that looks like this more details can be found in the
[docs](https://docs.cypress.io/guides/guides/continuous-integration.html#Boot-your-server) as well:

In order to run the cypress tests in one go we want to install an alternative server runner in the root of the monorepo:

```bash
npm i -S start-server-and-test
```

```bash
  "scripts": {
    ...
    "start:goodreads:test": "npm-run-all --parallel start:server:test start:goodreads",
    "cypress:open": "lerna exec cypress open --scope=@goodreads-v2/goodreads",
    "cypress:run": "lerna exec cypress run --scope=@goodreads-v2/goodreads",
    "test:integration": "START_SERVER_AND_TEST_INSECURE=1 start-server-and-test start:goodreads:test http-get://localhost:3000 cypress:run"
    ...
  }
```

In order to have the application build on CircleCI we need to add a `.circleci/config.yml` in the root of our repo. The
various npm scripts we have been writing so far will give us a good handle on how we handle writing the actual build
scripts as the commands we will run in the YAML config will map to our existing npm scripts.

{{% notice info %}}
There is one limitation with cypress and CircleCI. In order to be able to call `cy.visit(<url>)` the server that is serving
the application needs to have server side routing enabled, however this is out of scope. One way to work around this is
by having tests navigate through `Link` elements in the UI as this is hooked into the `react-router-dom`.
An even better way to deal with this would be to have __a separate environment__ that has a deployment of the application
that is suited for both exploratory testing and automated cypress tests.
{{% /notice %}}

That aside you can run the automated cypress test suites locally as well quite easily.

## Caching
One of the more interesting and challenging parts in CircleCI is the caching, CircleCI allows you to cache the
`node_modules` so that it is preserved across workflows. This mechanism is applicable for built assets from the multiple
packages in our monorepo, as well for the `node_modules` used in each of the packages.

The [CircleCI](https://circleci.com/docs/2.0/) provide a lot of sample `.circleci/config.yml` you can grab various commands from.
The most challenging part in working with assets in CircleCI josb is to use the right type of caching/asset for specific resources. There are three
types of persistence for assets: workspace, cache and artifact.

- Cache: is preserved across workflows
- Workspace: is preserved in the same workflow
- Artifact: is persisted after the execution of the CircleCI pipeline

## Our CircleCI config file
```yaml
version: 2.1
orbs:
  node: circleci/node@1.1.6
jobs:
  build-and-test-components:
    executor:
      name: node/default
    steps:
      - checkout
      - node/with-cache:
          steps:
            - run: npm run bootstrap
            - run: npm run test:components
            - run: npm run build:components
            - save_cache:
                key: components-lib-{{ .Revision }}
                paths:
                  - packages/component-library/lib
                  - packages/component-library/lib/fonts

  test-goodreads:
    executor:
      name: node/default
    steps:
      - checkout
      - node/with-cache:
          steps:
            - restore_cache:
                keys:
                  - components-lib-{{ .Revision }}
            - run: npm run bootstrap
            - run: npm run test:goodreads
  build-goodreads:
    executor:
      name: node/default
    steps:
      - checkout
      - node/with-cache:
          steps:
            - restore_cache:
                keys:
                  - components-lib-{{ .Revision }}
            - run: npm run bootstrap
            - run: npm run build:goodreads
            - save_cache:
                key: goodreads-v2-{{ .Revision }}
                paths:
                  - packages/goodreads/build
  deploy-goodreads:
    ...
workflows:
    build-components-and-app:
      jobs:
        - build-and-test-components
        - test-goodreads:
            requires:
              - build-and-test-components
        - build-goodreads:
            requires:
              - build-and-test-components
        - deploy-goodreads:
            requires:
              - build-goodreads
```

## Security
The storage of the various secrets needs to be done as pipeline environment variables so that it does not make its way
into git.

{{< lazy-image image="circle-security.png" lightbox=false />}}

They are accessible as `$<VAR_NAME>` in the circleci config and as process variables in  your app via
`process.env.<VAR_NAME>`

## Deployment
Deployment should be done as part of the pipeline and should be the last step, once the tests have
passed and the bundle builds have completed successfully. There are various options for the deployment, we
used a platform called netlify which is very straightforward to integrate for static websites.

The option we used to deploy is as a static website on netlify. You need to create an account with [netlify](https://www.netlify.com/), I would recommend using github to login as we
will be using the netlify app as part of our app deployment.

```bash
npm i -S netlify-cli
npx netlify login # perform oauth2 login to netlify so that the cli has access to the netlify api
```

In order to deploy the site to netlify you will need an `NETLIFY_ACCESS_TOKEN` and a `NETLIFY_SITE_ID`. You can get the
`NETLIFY_SITE_ID` when you create the site like so

```bash
npx netlify sites:create --name goodreads-v2
```

and you can create an access token from in your user settings under Applications.

```yaml
  deploy-goodreads:
    executor:
      name: node/default
    steps:
      - checkout
      - node/with-cache:
          steps:
            - restore_cache:
                keys:
                  - goodreads-v2-{{ .Revision }}
            - run:
                name: Netlify Deploy
                command: npx netlify deploy --site $NETLIFY_SITE_ID --auth $NETLIFY_ACCESS_TOKEN --prod --dir=packages/goodreads/build
```

This will not be enough once we have a backend as we require the backend to be deployed as well, however that is more
related to the backend node ecosystem and is out of scope for our deployment.
