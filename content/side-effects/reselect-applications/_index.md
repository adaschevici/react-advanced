+++
title = "Reselect Applications"
date = 2020-05-04T18:39:35+01:00
weight = 3
draft = false
+++

#### Redux: "All your data are belong to us"

{{<mermaid>}}
stateDiagram
    Redux --> Component1: slice of state
    Component1 --> Redux
    Redux --> Component2: slice of state
    Component2 --> Redux
    Redux --> Component3: slice of state
    Component3 --> Redux
{{</mermaid>}}

The data can be extracted on the fly in each component and access it directly from the component. This opens up a whole
can of worms because in the long run it introduces a change dependency between the state and the component.
When you would eventually change the shape of the state you would also need to change the `mapStateToProps` method on
the component.

Another issue when working with large amounts of data in the redux store is duplication. You need to deal with the
challenges of duplicate data by normalizing the redux store. [Data normalization](#data-normalization) is a great idea but adds extra
complexity when extracting data from the store. One of the more complicated examples in our project can be found in the
`book-list` component. Reselect helps us deal with store data complexity.

```javascript
function mapStateToProps(state) {
  const {
    meta,
    images,
    ratings,
    isLoadingMeta,
    isLoadingImages,
    isLoadingRatings,
    errorImages,
    errorRatings,
    errorMeta,
  } = state.books
  const { error: authError, username } = state.auth
  const authenticated = authError === null
  return {
    meta,
    images,
    ratings,
    isLoading,
    errorImages,
    errorRatings,
    errorMeta,
    authenticated,
    username,
    booksInProgress,
  }
}
```

#### Data Normalization
{{% notice note %}}
While data may be easier to use from components when it is nested, in order to prevent duplicate data you should use
arrays of single level objects and references to represent tree like structures.
{{% /notice %}}

```json
{
    "meta": [
        bookId,
        title,
        year,
        author,
        etc...
    ],
    "images": [
        bookId,
        imageUrl
    ],
    "ratings": [
        bookId,
        ratingValue
    ]
}
```

#### [Reselect](https://github.com/reduxjs/reselect)
Both issues cause the extraction of the data from the redux store to become quite complex over time and as with anything
that is complex it can be difficult to change, especially if it has change dependencies across components. In order to
deal with the complexity of slicing redux state we want to extract that into more specialized module that perform only
the slicing and dicing of the redux state, selectors do exactly that.

The way to structure your application is very much disputed in the community however I am a particular fan of the
`re-ducks` approach where modules are closer to the components' that render the state changes. This allows me to move
reducers closer to the store when the state is shared between more components and also bundle components with
functionality easier. We will add the selectors along side the component.

In `packages/goodreads/src/components/book-list/`
```bash
▾ [ ]book-list/
    [ ]actions.js
    [ ]index.js
    [ ]reducer.js
    [ ]selectors.js
```

{{% notice tip %}}
This approach is opinionated and you should use the approach that feels most natural every time. In React there are no
hard and fast rules.
{{% /notice %}}

#### Our first selector 🎉
```javascript
// packages/goodreads/src/components/book-list/selectors.js
import { createSelector } from 'reselect'

const getMeta = (state) => state.books.meta
const getImages = (state) => state.books.images
const getRatings = (state) => state.books.ratings

export const getBooks = createSelector(
  [getMeta, getImages, getRatings],
  (meta, images, ratings) => {
    return meta.map((bookMeta, idx) => ({
      ...bookMeta,
      ...images[idx],
      ...ratings[idx],
    }))
  }
)
```
This is a memoized selector as described [here](https://github.com/reduxjs/reselect#creating-a-memoized-selector)

## Case study (part II)
- Let's take some time and create a saga to fetch the books in progress and also a selector for the book in progress
  array since it will require some data crunching from the entire books array. In order to add a book to the
  booksInProgress array for a user you need to edit the monkey-api json data file.

```json
  // packages/monkey-api/data/data.average.json
{
  [
  ...
  ],
  "users": [
    {
      "id": "am@bam.com",
      "passwordHash": "$2b$12$t5nytw5tt0hCiIuBu1VhVus1R6sV5ze64lKjjOGVEyDotgfUqkSdi"
    }
  ],
  "book-progress": [
    {
      "id": "am@bam.com",
      "books": [9780312283000]
    }
  ]
}
```

## Bonus points
- have a look at other state slicing and try to simplify the redux state access using selectors

## How can we measure the performance gains?
The benefits of using `reselect` is that it gives us some built in handling of memoization, in other words selectors
have a built in mechanism for caching results so that they are not calclulated multiple times, instead for the same
given values they will fetch the pre calculated results from a key-value structure.

When fine tuning performance you want to apply the Pareto principle very ruthlessly, 20% of the code counts for 80% of
the performance. In order to apply this you need to measure performance in the code. Selectors help
with optimization of JS function calls by memoizing the results it isn't really designed for component rendering
optimization however that may be achieved as a side effect.

We can try to check how much speed-up we gained by using a JS snippet like this:
```javascript
console.time('tag')
// <code to profile>
console.timeEnd('tag')
```

This is the code before adding selectors and included some time profiling
```javascript
console.time('profile before selectors')
const booksCollection = meta.map((bookMeta, idx) => {
  const book = {
    ...bookMeta,
    ...ratings[idx],
    ...images[idx],
  }
  if (booksInProgress.includes(bookMeta.id)) progressData.push(book)
  return book
})
console.timeEnd('profile before selectors')
return (
  <Fragment>
    {authenticated && (
      <Fragment>
        <Artifika>Currently reading</Artifika>
        {progressData.length ? (
          <BookGrid>
            {progressData.map((book) => (
              <BookCard
                key={`${book.id}${book.title}`}
                authenticated={authenticated}
                onStopped={stopBook}
                {...book}
              />
            ))}
          </BookGrid>
        ) : (
          <div>
            <Body tag="h6">Nothing to show here...yet :(</Body>
          </div>
        )}
      </Fragment>
    )}
  </Fragment>
)
```

You can check the profiling after adding the selectors by adding the profiler logs in the `mapStateToProps` method where
the selector functions are called
```javascript
function mapStateToProps(state) {
  const { isLoading, error } = state.books
  const { error: authError, username } = state.authStatus
  const authenticated = authError === null
  console.time('selectors')
  const books = getBooks(state)
  const booksInProgress = getBooksProgress(state)
  console.timeEnd('selectors')
  return {
    books,
    isLoading,
    error,
    authenticated,
    username,
    booksInProgress,
  }
}
```
