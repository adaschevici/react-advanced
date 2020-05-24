+++
title = "Testing Patterns"
date = 2020-05-06T17:28:47+01:00
weight = 4
draft = false
+++

{{% notice tip %}}
We looked at the libraries that can make our life much easier when working with redux, libraries that help us extract
data from the store, optimize function calls by memoizing and orchestrating complex API calls.
{{% /notice %}}

Creating modules for extracting state and the api calls will make testing much easier as it will enforce that the tests
are built around their respective modules as well. Aside from that sagas and selectors are much more testable than their
thunk equivalent.

We have a few things we want to test:
- the original test we had from the CRA creation has stopped working since we added the store so we want to fix it
- the selectors we added previously
- the sagas
- the reducers
- the actions

### Step 1: Configure jest to allow for testing redux connected components
The recommended approach for the location of the tests is for them to be alongside your components in a `__tests__`
folder.

The jest config provided by CRA has some options that can be overriden but it [is limited](https://create-react-app.dev/docs/running-tests/#configuration).
If you require other options for your jest config then you need to eject the app and craft a custom jest config.

In recent years there have been multiple contending libraries for unittesting react components. The most used library
used to be [`enzyme`](https://enzymejs.github.io/enzyme/) but there is a [new library](https://testing-library.com/) that has had very good adoption in
the  react ecosystem. In our project we will be using the `react-testing-library` because of its adoption by the react
team and that it works with CRA too.

In order to test our redux connected components we need to inject redux state and router functionality into the test renderer provided by the library.
The way to do this is by creating a higher order function for creating an enhanced renderer.

```javascript
// packages/goodreads/test-utils/index.js
import React from 'react'
import { BrowserRouter as Router } from 'react-router-dom'
import { render as rtlRender } from '@testing-library/react'
import { createStore } from 'redux'
import { Provider } from 'react-redux'

const createRenderer = (reducerInitialState, reducer) => (
  ui,
  {
    initialState = reducerInitialState,
    store = createStore(reducer, initialState),
    ...renderOptions
  } = {}
) => {
  function Wrapper({ children }) {
    return (
      <Provider store={store}>
        <Router>{children}</Router>
      </Provider>
    )
  }
  return rtlRender(ui, { wrapper: Wrapper, ...renderOptions })
}

// re-export everything
export * from '@testing-library/react'

// export renderer creator higher order function
export { createRenderer }
```

Once we have this utility method we can test our components both functionally and via snapshots.

For example a simple snapshot test of the App component would look something like this.

```javascript
// packages/goodreads/src/components/app/index.test.js
import React from 'react'
import { createStore } from 'redux'
import { createRenderer } from '../../../test-utils'
import App from '.'
import reducer from './reducer'

describe('test suite for app component', () => {
  let fakeState
  const render = createRenderer(reducer, fakeState)
  beforeEach(() => {
    fakeState = {
      auth: {
        username: 'anonymous',
      },
      books: {
        isLoading: false,
        meta: [],
        images: [],
        booksInProgress: [],
        ratings: []
      },
    }
  })
  it('renders app component and matches snapshot', () => {
    const store = createStore(() => fakeState)
    const component = render(<App />, { store })
    expect(component).toMatchSnapshot()
  })
})
```

Another test utility we should add is a saga runner so that we can test our sagas in isolation, it creates a generator
and returns a list of intercepted actions.

```javascript
// packages/goodreads/src/test-utils/sagas.js
import { runSaga } from 'redux-saga'

export async function executeSaga(saga, initialAction) {
  const dispatched = []

  await runSaga(
    {
      dispatch: (action) => dispatched.push(action),
    },
    saga,
    initialAction
  ).done

  return dispatched
}
```

### Step 2: Test! 🎉🎉🎉

#### Selectors
Because selectors are functions the testing is very straightforward, it has some caveats though because of the
memoization.

{{<  highlight javascript "hl_lines=31-33" >}}
describe('book-list selectors', () => {
  const state = {
    books: {
      meta: [
        {
          id: 9780439023480,
          isbn: '439023483',
          isbn13: '9780439023480',
          authors: 'Suzanne Collins',
          year: 2008,
          title: 'The Hunger Games',
          description:
            'Consequatur perferendis voluptatem sit aut accusantium. Aut accusantium sit consequatur voluptatem perferendis. Sit perferendis aut voluptatem accusantium consequatur.',
        },
      ],
      images: [
        {
          id: 9780439023480,
          image: 'https://images.gr-assets.com/books/1447303603m/2767052.jpg',
        },
      ],
      ratings: [
        {
          id: 9780439023480,
          rating: 4.34,
        },
      ],
    },
  }

  afterEach(() => {
    getBooks.resetRecomputations()
  })

  it('extracts books from state', () => {
    const expected = {
      ...state.books.meta[0],
      ...state.books.ratings[0],
      ...state.books.images[0],
    }
    expect(getBooks(state)).toEqual([expected])
  })
})
{{< /highlight >}}

#### Sagas
Sagas are a bit trickier to test but then again side effects are usually the most difficult part in the testing. We make
use of jest mocking extensively to avoid api roundtrips
```javascript
import * as api from '../../api'
import { LOGIN_SUCCEEDED } from '../../../components/login/actions'
import { executeSaga } from '../../../test-utils/sagas'
import { watchLogin } from '../login'

describe('login saga functionality', () => {
  api.authenticateUser = jest.fn()
  beforeEach(() => {
    jest.resetAllMocks()
  })

  it('logs into the app', async () => {
    const loginSuccess = {
      type: LOGIN_SUCCEEDED,
      payload: {
        msg: 'OK',
      },
    }
    const initialAction = {
      payload: { email: 'test@email.com', password: 'testpass' },
    }
    const { payload } = initialAction
    api.authenticateUser.mockImplementation(() =>
      Promise.resolve({ data: 'OK' })
    )
    const dispatched = await executeSaga(watchLogin, initialAction)
    expect(api.authenticateUser).toHaveBeenCalledWith(payload)
    expect(dispatched).toContainEqual(loginSuccess)
  })
})
```

#### Actions
Testing action is nothing special, usually you would essentially check the shape of the action object
```javascript
// packages/goodreads/src/components/book-list/__tests__/actions.test.js
import { FETCH_BOOKS_REQUEST } from '../actions'
import { fetchBooks } from '../actions'

test('creates a fetch action', () => {
  const expected = { type: FETCH_BOOKS_REQUEST }
  expect(fetchBooks()).toEqual(expected)
})
```
#### Reducers
Reducers are the simplest parts to test. They are pure functions and the test checks whether the function taking in
`(state, action)` will return a mutated version of the state and we then check if it is what we expected.


## Case study (part III)
- add a few more unittests for the `book-list` and `login` components
