+++
title = "Styled components and systems"
date = 2020-03-07T16:59:44Z
weight = 1
draft = false
+++


#### Components and atomic design

{{< lazy-image image="composability.png" lightbox=false />}}

{{% notice tip %}}
When looking at components the atomic components score much higher
on the reusability pyramid(which means they are closer to the base)
and the fun part is that they are also less complex, which is not
to say that they have fewer features, but the features
are more related.
{{% /notice %}}

When building a component library you should look at having more
primitive components rather than complex ones. The amount of context
that is bound to the components in your library should also be very little.

### Tools

The libraries we will use for wiring up our design system are `styled-components`
and `styled-system`
1) [`styled-components`](https://styled-components.com/) are a css in js library for building our UIs
2) [`styled-system`](https://styled-system.com/) is a library for correlating styles and styled-components with numeric scales

#### Styled components
Styled components are one of the solutions for css in js. There are multiple
solutions like this, one of the more popular ones being [`emotion`](https://emotion.sh/docs/styled).
The api is generally the same and migration from one to the other is fairly straightforward. We will
be working with the classical styled-components, but it doesn't hurt to be aware of the other options.

#### Styled system
The `styled-system` is used for defining theme numeric scales for your styled components design system. You can use it to
define various scales and also breakpoints for your components. This helps having a centralized set of
props and breakpoints to use in your components. This approach overall decreases code duplication
and unifies the coding standards in your library.

{{% notice note %}}
We briefly touched on how we need to set up the tools for starting up on our component library and design system.
In this section we will dive into how we can create our reusable component and also look at responsiveness and themes
{{% /notice %}}
