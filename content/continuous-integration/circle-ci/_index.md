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


## [Circle CI](https://circleci.com/vcs-authorize/)
In the previous chapters we spent some time adding npm scripts at the root of the project that delegate to the child
packages. We ran the integration tests while also running the app. In an automated environment this won't work. We need
to start the app before running our cypress tests.

A really nifty command line tool that is available as an npm package is `start-server-and-test`. This is the recommmended way to run
your app and once the app is ready to serve requests you can start running the cypress tests.

In order to be able to run the tests you need to run the mocked API, the webpack server and the test runner in the same
command. This is an npm script that looks like this more details can be found in the
[docs](https://docs.cypress.io/guides/guides/continuous-integration.html#Boot-your-server) as well:

```bash
  "scripts": {
    ...
    "start:app-test": "npm-run-all --parallel start:test-server start:goodreads",
    "start:cy-app": "lerna exec cypress open --scope=@goodreads-v2/goodreads",
    "test:cy": "lerna exec cypress run --scope=@goodreads-v2/goodreads",
    "test:integration": "START_SERVER_AND_TEST_INSECURE=1 start-server-and-test start:app-test http-get://localhost:3000 test:cy"
    ...
  }
```

In order to have the application build on CircleCI we need to add a `.circleci/config.yml` in the root of our repo. The
various npm scripts we have been writing so far will give us a good handle on how we handle writing the actual build
scripts as the commands we will run in the YAML config will map to our existing npm scripts.
