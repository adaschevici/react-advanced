+++
title = "Scaling Side Effects"
date = 2020-05-04T16:06:56+01:00
weight = 2
draft = false
+++

One of the most complex aspects you encouter in side effect management is dealing with authentication and authentication
storage on the client side. In order to make sure you are following best security guidelines the token needs to be
stored as `http-only`. That means that you will not be able to access it from the client side code.

There are a few pieces of the data available that can be resolved by authenticating with your api. Because of the
`http-only` approach with our JWT token the way to extract it is by making calls to a `check-token` endpoint which is
part of our auth flow.

In order to avoid code duplication we will use a render prop pattern to create a container for our components that is
rendered before its children and populates the store with the slice of user information retrieved from the auth checker
endpoint.

The container should look something like this 
```javascript
// packages/goodreads/src/containers/auth-checker/index.js
import { Component } from 'react'
import { checkAuth } from './actions'
import { connect } from 'react-redux'

class AuthCheck extends Component {
  componentDidUpdate() {
    const { dispatch } = this.props
    dispatch(checkAuth())
  }
  componentDidMount() {
    const { dispatch } = this.props
    dispatch(checkAuth())
  }

  render() {
    const { children } = this.props
    return children
  }
}

const mapStateToProps = (state) => {
  const { username } = state.auth
  return { username }
}

export default connect(mapStateToProps)(AuthCheck)
```

#### Action flow
{{<mermaid>}}
stateDiagram
    [*] --> Component
    Component --> Action
    Action --> SagaMiddleware
    SagaMiddleware --> Redux
    Redux --> Component : rerender

    state SagaMiddleware {
        state watcherSaga {
            [*] --> api: request data
            api --> [*]: response data
        }
    }
    state Redux {
        [*] --> reducer
        reducer --> [*]
    }
{{</mermaid>}}

If you are familiar with redux and the way the functional cycle from the UI to redux and back works then this should be
fairly straightforward. The novelty is where the sagas fit into this flow and that is in the middleware section. Some
projects will use `thunks` to resolve data however when using thunks exclusively the actions will become quite unwieldy
do to callback hell.

Every request/response cycle has 3 actions associated (request start, request success, request failed). Sagas allow us
to have a very standard development pattern for dealing with any http request.

We already saw what a barebone component looks like. Actions are standard as well.

#### SagaMiddleware

The sagaMiddleware needs to be created and also started for it to work
{{< highlight javascript "hl_lines=12 35" >}}
// packages/goodreads/src/store/index.js
import { createStore, applyMiddleware } from 'redux'
import { composeWithDevTools } from 'redux-devtools-extension'
import createSagaMiddleware from 'redux-saga'
import rootSaga from './sagas'
import rootReducer from './rootReducer'

const composeEnhancers = composeWithDevTools({
  trace: true,
})

const sagaMiddleware = createSagaMiddleware()

const middlewares = [sagaMiddleware]

export default function configureStore(initialState = {}) {
  const store = createStore(
    rootReducer,
    initialState,
    composeEnhancers(applyMiddleware(...middlewares))
  )
  sagaMiddleware.run(rootSaga)
  return store
}
{{< /highlight >}}

The implementation of the api calls and the side effects is implemented closer to the store than if we were to use
thunks. This allows for a higher degree of decoupling.
```bash
   ▾ [ ]store/
     ▾ [ ]sagas/
         [ ]books.js
         [ ]index.js
         [ ]login.js
       [ ]api.js
       [ ]index.js
```

The rootSaga is a collation of all the sagas and is added to the middleware collection. Its task is to map specific
actions to side effect handlers that are called watchers.

{{< highlight javascript >}}
// packages/goodreads/src/store/sagas/index.js
import { takeLatest } from 'redux-saga/effects'
import { CHECK_AUTH_REQUEST } from '../../containers/auth-checker/actions'
...
import { watchLogin, watchRegistration, watchAuth } from './login'

export default function* rootSaga() {
  ...
  yield takeLatest(CHECK_AUTH_REQUEST, watchAuth)
}
{{< /highlight >}}

The watchers are the actual functionality for handling the actions and this is where we will write most of our code for side effects

{{< highlight javascript >}}
// packages/goodreads/src/store/sagas/login.js
import { call, put } from 'redux-saga/effects'
import * as api from '../api'
import {
  CHECK_AUTH_SUCCEEDED,
  CHECK_AUTH_FAILED,
} from '../../containers/auth-checker/actions'
...

export const watchAuth = function* watchAuthCheck() {
  try {
    const { data } = yield call(api.checkToken)
    yield put({
      type: CHECK_AUTH_SUCCEEDED,
      payload: {
        username: data.username,
      },
    })
  } catch (e) {
    yield put({
      type: CHECK_AUTH_FAILED,
      payload: {
        error: 'Not authenticated',
      },
    })
  }
}
{{< /highlight >}}

- Actions: 

```javascript
// packages/goodreads/src/containers/auth-checker/actions.js
export const CHECK_AUTH_REQUEST = 'CHECK_AUTH_REQUEST'
export const CHECK_AUTH_SUCCEEDED = 'CHECK_AUTH_SUCCEEDED'
export const CHECK_AUTH_FAILED = 'CHECK_AUTH_FAILED'

export const checkAuth = () => ({
  type: CHECK_AUTH_REQUEST,
})
```
- Reducers:

```javascript
// packages/goodreads/src/containers/auth-checker/reducer.js
import {
    CHECK_AUTH_REQUEST,
    CHECK_AUTH_SUCCEEDED,
    CHECK_AUTH_FAILED,
  } from './actions'
  
  const initialState = {
    isLoading: false,
    username: null,
    error: 'Not authenticated',
  }
  
  export default function authChecker(state = initialState, action) {
    switch (action.type) {
      case CHECK_AUTH_REQUEST: {
        return {
          ...state,
          error: 'Not authenticated',
          isLoading: true,
        }
      }
      case CHECK_AUTH_SUCCEEDED: {
        return {
          ...state,
          username: action.payload.username,
          error: null,
          isLoading: false,
        }
      }
      case CHECK_AUTH_FAILED: {
        return {
          ...state,
          isLoading: false,
          error: action.payload.error,
        }
      }
      default: {
        return state
      }
    }
  }
```
- Update rootReducer

{{< highlight javascript >}}
// packages/goodreads/src/store/rootReducer.js
import { combineReducers } from 'redux'
import appReducer from '../components/app/reducer'
import booksReducer from '../components/book-list/reducer'
import loginReducer from '../components/login/reducer'
import registerReducer from '../components/register/reducer'
import authReducer from '../containers/auth-checker/reducer'

export default combineReducers({
  app: appReducer,
  books: booksReducer,
  login: loginReducer,
  register: registerReducer,
  auth: authReducer
})
{{< /highlight >}}

- Update app.js

```javascript
import React, { Component } from 'react'
import { connect } from 'react-redux'
import { components } from '@goodreads-v2/component-library'
import './index.css'
import BookList from '../book-list'

const { NavBar } = components

class App extends Component {
  render() {
    const { username, authenticated } = this.props
    return (
      <div className="App">
        <NavBar authenticated={authenticated} username={username} />
        <main style={{ height: '70vh' }}>
          <BookList />
        </main>
      </div>
    )
  }
}

const mapStateToProps = (state) => {
  const { username, error } = state.auth
  const authenticated = error === null
  return { username, authenticated }
}
export default connect(mapStateToProps)(App)
```

The decoupling between the http requests and the actions allows you to combine the responses and avoid the callback hell
alltogether. IMO that is pretty cool especially when working with microservices or even services. If a component
requires data from multiple sources to be able to render itself then the ability to collate the data before sending it
on to the store is pretty much invaluable.

### We have reached a point where fetching data works

We have multiple API sources at this point:
- `/images` - fetch cover images for books
- `/meta` - fetch data for each book
- `/ratings` - fetch ratings for books
- `/auth/authenticate` - login
- `/auth/register` - creating new user account
- `/auth/check-token` - verify http token for validity whenever a component is mounted or receives new props

## Case study (part I)

- Orchestrating the API requests so that we introduce composability for our other watcher sagas. We will be able to
  simply create an array of the existing sagas and apply the call to them. In order to allow the sagas to execute we
  need to wrap the execution in `all` otherwise they won't execute.

  {{% notice info %}}
  `all` from sagas is a bit misleading in its naming and the docs. It will not wait for completion on sagas, only on
  promises. It will not block and wait but rather run the sagas in parallel. If you want to orchestrate results to be available
  at the same time you need to use `all` together with the requests and update the store once the response data is ready
  {{% /notice %}}

## Bonus points
- Add actions, sagas and reducer cases for starting to read a book
