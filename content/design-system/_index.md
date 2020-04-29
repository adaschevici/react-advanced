+++
title = "Design System"
chapter = true
weight = 2
pre = "<b>2. </b>"
date = 2020-03-07T16:59:44Z
draft = false
+++


#### The road so far

- Looked at the various packages that make up our monorepo
- looked at lerna and the other variants for managing monorepos
- installed and configured lerna, storybook, the api git submodule and the main react application
- configured rollup and dependencies for the component library

## Atomic design

{{< google-slides token="1WeS_MJLfj_rNH43a5bexV7XT0pSn-tOImNN8JeOEjco" >}}

## Functional diagram

{{<mermaid>}}
graph LR
    subgraph library
        id1
        id3
        id5
    end
    id1(theme) ==defines==> id2(colors, spacing, breakpoints, font sizes and weights)
    id3(typography) ==defines==> id4(fonts, text styles)
    id5(storybook) ==creates==> id6(playground for working with components)
{{</mermaid>}}


### 🏁 Checkpoint 🏁

```bash
git checkout 01-lerna-dependencies
```
