+++
title = "Case Study Large List"
date = 2020-05-13T17:19:31+01:00
weight = 3
draft = false
+++


## How to work on performance

In order to look at performance you need to look at parts of your app in isolation. For example when dealing with a
large list it helps to focus on improving performance on the list not everything all at once.

Most often if you try to improve performance all at once you will wind up wasting CPU cycles on the most precious
resource you have, your brain 🤣.

Prototyping performance in isolation has several benefits:
- you have less signal to noise ratio of the problem
- you can pull people in for finding a solution if you get stuck and they will be more successful in picking up the
  snippet and diagnosing since there is less prerequisite knowledge about the problem
- it is safe to share openly since it will not have material that could fall under NDA

ProtoTyping checklist:
1) local stripped down version of the problem
2) share and collaborate in Online Playgrounds
   [CodeSandbox](https://codesandbox.io), [CodePen](https://codepen.io/) etc.

A common problem where performance can take a hit is when rendering large lists, in general large data sets. Depending
on the practical use case virtualization may not necessarily be the best thing to do however it is the one thing that is
doable exclusively on the frontend.

In order to isolate the performance issue with rendering a large list of components we trimmed down the fat, and we will
use only the bare essentials. Go ahead and clone the performance repo [here](https://github.com/adaschevici/react-manhattan).
The branch we will be looking at is `react-virtual/02-react-window-classes`.

## Virtualization
The concept of doing lazy rendering based on the viewport is called virtualization. The code we will be working with
also attempts to provide a way of handling variable sized list elements and trying to get their respective sizes at
render time.

At this time `react-window` is the more modern approach to building large lists however it is less feature rich than its
predecessor `react-virtualized`. I used `react-window` because it is more in line with react hooks and the code is a bit
more elegant and the bundle size is much smaller than it is when using `react-virtualized`
