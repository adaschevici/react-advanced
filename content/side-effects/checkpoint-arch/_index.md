+++
title = "Fast forward to redux and sagas"
weight = 1
date = 2020-04-15T13:48:49+01:00
draft = false
+++


#### Architecture checkpoint

In the previous version of the application we wired up the storybook setup with tests and a playground for our
components. We didn't have enough components for consuming the API endpoints so we will take a shortcut and fast forward
to a version which implements a decent number of components to consume our API data.

#### Components and libraries introduced in the migration
- `rebass`, `@rebass/layout`, `@rebass/forms` a primitive set of components built using `styled-components`
- responsive grid making use of `@rebass/layout`
- Login and Register components managing internal state that can be passed onto the action in the app
  callback events
- `redux`, `react-redux`, `redux-saga` for managing side-effects and state
- `axios` from making http requests
- redux connection boilerplate for our app including actions, reducers, sagas, api calls
- npm script at the root of the repo for starting up the API package and the goodreads app
- `reselect` for decoupling components from `redux`

#### Getting started and useful npm scripts
```bash
npm run start:app # brings up both app and api servers
npm run build:components # calls the rollup build for the components
npm run snapshot:components # runs the snapshot tests on the component library via storyshots
npm run snapshot:components -- -- -- -u # update snapshots so that snapshots are up to date
```

{{% notice note %}}
Other npm scripts are available in the root `package.json` and also feel free to build your own if you find them missing
some functionality
{{% /notice %}}

#### The resulting directory structure
```bash
▾ [ ]packages/
  ▾ [ ]component-library/
    ▸ [ ].storybook/
    ▸ [ ]__mocks__/
    ▸ [ ]__tests__/
    ▾ [ ]src/
      ▸ [ ]__snapshots__/
      ▸ [ ]components/
      ▸ [ ]fonts/
      ▸ [ ]theme/
      ▸ [ ]typography/
        [ ]index.js
        [ ]storyshots.test.js
      [ ].gitignore
      [ ]babel.config.js
      [ ]jest.config.js
      [ ]package-lock.json
      [ ]package.json
      [ ]README.md
      [ ]rollup.config.js
  ▾ [ ]goodreads/
    ▸ [ ]public/
    ▾ [ ]src/
      ▾ [ ]components/
        ▸ [ ]app/
        ▸ [ ]book-list/
        ▸ [ ]login/
        ▸ [ ]register/
      ▾ [ ]routes/
          [ ]index.js
      ▾ [ ]store/
        ▸ [ ]sagas/
          [ ]api.js
          [ ]index.js
          [ ]rootReducer.js
        [ ]index.css
        [ ]index.js
```

As you can see there are quite a few more parts to the application, but this is required as we need the
additional complexity in order to have the opportunity to make the performance short-comings apparent. In order to optimize you
need to have the problem in the first place.


{{% notice warning %}}
 😈  __Premature optimization is the root of all evil__ 😈
{{% /notice %}}

#### Fast forward noteworthy concepts
We jumped ahead to a point in our application where we have a navbar, authentication and state management already wired
up in the application in order to do a deeper dive into the way complex side effects are managed using sagas. Side
effects are an important aspect in complex applications that raises quite a few challenges in practice.

Interesting gotchas while jumping ahead:
- adding a navbar component in storybook can cause problems as you need to create a router wrapper around the component.
  Details of the implementation can be found in the
  [component](https://github.com/adaschevici/goodreads-v2/blob/03-fast-forward-branch/packages/component-library/src/components/nav-bar/index.js),
  the [story](https://github.com/adaschevici/goodreads-v2/blob/03-fast-forward-branch/packages/component-library/src/components/nav-bar/index.stories.js). We had
  to add a new storybook addon for routing wrapping ease of use `storybook-react-router`
- if we have two instances of `react-router-dom`, one referenced from the component library and one inside our app then
  we will have yet another package clash so we need to mark it as external in
  `packages/component-library/rollup.config.js` and also hoist it so that the static storybook still builds correctly.
