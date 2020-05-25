+++
title = "State mgmt side effects"
chapter = true
weight = 3
pre = "<b>3. </b>"
date = 2020-04-15T13:48:49+01:00
draft = false
+++

#### The road so far

- Looked at how to use storybook to demo new components
- adding tests using the storybook storyshots addon
- creating themes
- adding custom fonts using global styles
- using styled system to provide responsive design

## State management && side-effects

{{< google-slides token="15yy4cBAZvOXDVi_fTFRa8jYtY3HY9dsCUaY6HjnxCOc" >}}

## Let's look at

{{<mermaid>}}
graph LR
subgraph goodreads
id1
id3
end
id1(sagas) ==interact with API==> id2(fetch meta, images and ratings, perform login and registration)
id3(redux and selectors) ==store data==> id4(store book and user login data)
{{</mermaid>}}

{{<mermaid>}}
graph LR
subgraph library
id1
id3
end
id1(login component) ==stateful==> id2(functional components using hooks)
id3(register component) ==stateful==> id2(functional components using hooks)
{{</mermaid>}}

### 🏁 Checkpoint 🏁

```bash
git checkout 03-fast-forward-branch
git submodule add git@github.com:adaschevici/jungle-jim.git packages/monkey-api
npm run clean
npm run bootstrap
```
