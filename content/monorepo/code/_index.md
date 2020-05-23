+++
title = "Code, gotchas & practice"
chapter = false
weight = 5
date = 2020-03-07T16:59:44Z
draft = false
+++

{{< lazy-image image="gotcha.jpg" lightbox=false />}}

#### Now let's put to good use what we learned so far

1. We want to build a component with `styled-components`
2. export that component from the component-library and import it into the goodreads app
3. build the storybook showcase
4. build the react app with the added dependency on the component library
5. add the npm scripts to the top level `package.json` in order to allow us to run commands easier (ie clean, bootstrap,
   build component library, build storybook, build app, build storybook)

### Boilerplate

The example component we will use a simple text based component with a little style:

```javascript
import React from 'react'
import PropTypes from 'prop-types'
import styled from 'styled-components'

const AlertComponent = ({ message = 'this is an alert' }) => (
  <span>{message}</span>
)

AlertComponent.propTypes = {
  message: PropTypes.string,
}

const StyledAlertComponent = styled.div`
  color: red;
`

export default (props) => (
  <StyledAlertComponent>
    <AlertComponent {...props} />
  </StyledAlertComponent>
)
`;

export default props => (
  <StyledAlertComponent>
    <AlertComponent {...props} />
  </StyledAlertComponent>
);
```

{{% notice warning %}}
Handling deps in our application is not as straightforward as we would wish. Both
the library and the app will have a react dep so we will get an error similar to this one.
{{% /notice %}}

{{< lazy-image image="react-dualdep-error.png" lightbox=false />}}

#### Individual tasks:

Get points 1 - 4 working. For bonus points add the npm scripts as mentioned in point 5.
