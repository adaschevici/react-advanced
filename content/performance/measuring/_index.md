+++
title = "Measuring"
date = 2020-05-12T17:43:09+01:00
weight = 1
draft = false
+++

## Chrome

At this time most performance tools are available as a chrome extensions and chrome provides the best toolbelt for
diagnosing performance in react.

There are 3 amazing tools for helping us track down performance issues:
- Chrome DevTools Performance tab
- React Developer Tools Chrome extension
- React's Profiler API

#### Profiling tools
In order to find the bottleneck in your application the way to do it is by recording a profile of the slow actions in
your app. If you are here then you know roughly where the bottleneck is, what you need to do is find the parts of the UI
that are lagging behind and see how their performance can be improved.

#### Chrome DevTools
They are part of Chrome, they are easy to get started with. What you need to do in order to profile your app is start
the app and start a profile recording. Once you are done you will have a flamegraph with the various methods in the
browser UI that are called as your application goes through the scenario you have profiled.

I found that the timing section in the recorded profile is the most useful one to narrow down the slow components. You
can see which components spend the most time rendering, or functions that take a long time to run. This is not specific
to react but all the JS that is executed on the page so it is more of an all purpose tool.

#### React Developer Tools Chrome extension
This is more React specific and it will allow measurements that will map to your React App components and time the
different phases they go through. This is useful from many aspects:
- you can see the different stages of the components and see where the most time is spent(render/commit)
- only highlights the components that are rerendered as part of the session so that you can focus on them
- caveat: the profiler session will not survive across reloads so you can't really diagnose a loading issue, you need to
  use navigation for problematic rendering

{{< lazy-image image="react-devtools.png" lightbox=false />}}

#### React Profiler API
This is the third option which is probably the most interesting as it can be integrated in our code. We don't want to
add it to production bundles. The cool thing about it is that we could potentially use it to write performance testing
for our components.

Writing performance tests is tricky because they are not reliable and are highly succeptible to environmental changes.
This means that the results may change because of node upgrades, container changes, garbage collection and many
unpredictable factors. If you do need to automate performance tests then a good approach is to have a highly controlled
environment and cover the components that are causing regressions.
