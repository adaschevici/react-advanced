+++
title = "Performance"
date = 2020-05-12T16:13:36+01:00
weight = 5
pre = "<b>5. </b>"
draft = false
+++

#### The road so far

- looked at how to automate UI testing using Cypress
- looked at CircleCI one of the best CI/CD service providers today
- deployed our app to Netlify

## Now
{{< google-slides token="1uDx_ZfK_FVh95xEO3t9t3gMXQ1hHCS6JMmFyzcocCsc" >}}

#### We will look at

{{< lazy-image image="wannagofast.jpg" lightbox=false />}}

- the slow rendering list of books, the data retrieval happens in reasonable timing, however the rendering is so slow
  that it disrupts the other components, we don't need the entire list to be rendered at once
- use web workers to offload computation on multiple executors thus making execution blazing fast

### 🏁 Checkpoint 🏁

```bash
git checkout 05-continuous-integration
npm run clean:all
npm run bootstrap
```
