---
title: "My First Post"
date: 2020-03-07T16:59:44Z
draft: false
---


#### What should the path be?

```javascript
import React, { Component } from 'react'

import PropTypes from 'prop-types'
import { withStyles } from '@material-ui/core/styles'
import CircularProgress from '@material-ui/core/CircularProgress'
import Paper from '@material-ui/core/Paper'
import Table from '@material-ui/core/Table'
import TableBody from '@material-ui/core/TableBody'
import TablePagination from '@material-ui/core/TablePagination'

import styles from './styles'
import BookListHeader from './components/book-list-header'
import Book from './components/book'
import columnData from '../app/data-mapping'

function getSorting(order, orderBy) {
  return order === 'desc'
    ? (a, b) => (b[orderBy] < a[orderBy] ? -1 : 1)
    : (a, b) => (a[orderBy] < b[orderBy] ? -1 : 1)
}

class BookList extends Component {
  static propTypes = {
    columnHeaders: PropTypes.arrayOf(PropTypes.object).isRequired,
    books: PropTypes.arrayOf(PropTypes.object).isRequired,
  }

  constructor(props) {
    super(props)
    this.state = {
      order: 'asc',
      orderBy: 'original_title',
      page: 0,
      rowsPerPage: 10,
    }
  }

  handleRequestSort = (_, property) => {
    const orderBy = property
    const { order } = this.state
    let currentOrder = 'desc'

    if (orderBy === property && order === 'desc') {
      currentOrder = 'asc'
    }

    this.setState({ order: currentOrder, orderBy })
  }

  handleChangePage = (_, page) => {
    this.setState({ page })
  }

  handleChangeRowsPerPage = event => {
    this.setState({ rowsPerPage: event.target.value })
  }

  render = () => {
    const { classes, columnHeaders, books, loading, history } = this.props
    const { order, orderBy, rowsPerPage, page } = this.state
    const start = page * rowsPerPage
    const end = start + rowsPerPage
    const rowColumns = [
      ...columnData.actionButtons,
      ...columnData.sortableColumns,
    ]
    return (
      <Paper className={classes.root}>
        <div className={classes.tableWrapper}>
          {loading ? (
            <CircularProgress className={classes.loader} />
          ) : (
            <Table className={classes.table} aria-labelledby="tableTitle">
              <BookListHeader
                columnHeaders={columnHeaders}
                onRequestSort={this.handleRequestSort}
                history={history}
                {...this.state}
              />
              <TableBody>
                {books
                  .sort(getSorting(order, orderBy))
                  .slice(start, end)
                  .map(book => (
                    <Book
                      key={book.id}
                      book={book}
                      typesMapping={rowColumns}
                      history={history}
                    />
                  ))}
              </TableBody>
            </Table>
          )}
        </div>
        <TablePagination
          component="div"
          count={books.length}
          rowsPerPage={rowsPerPage}
          page={page}
          backIconButtonProps={{ 'aria-label': 'Previous Page' }}
          nextIconButtonProps={{ 'aria-label': 'Next Page' }}
          onChangePage={this.handleChangePage}
          onChangeRowsPerPage={this.handleChangeRowsPerPage}
        />
      </Paper>
    )
  }
}

export default withStyles(styles)(BookList)
```
