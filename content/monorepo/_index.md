+++
title = "Mono Repo"
chapter = true
weight = 1
pre = "<b>1. </b>"
date = 2020-03-07T16:59:44Z
draft = false
+++


## Mono repos

{{< google-slides token="1LKup2J5QuFAC-Lp9lwfzZHTh7zcdaG2NyHeVIQTbijY" >}}


## Code location:
```bash
https://github.com/adaschevici/goodreads-v2
```

## Auxilliary repo for the mock API
This is a mock API I've developed to mimic poor conenctivity and auth on top of `json-server`
We will use it as a submodule in the app but if you want to use it in other projects you can clone it standalone

```bash
https://github.com/adaschevici/jungle-jim
```

## Let's look at

{{<mermaid>}}
graph TD
    subgraph goodreads-v2
        id1
        id4
    end
    id1(Lerna) ==manages==> id2(Component Library built with rollup)
    id1(Lerna) ==manages==> id3(React Application created with CRA)
    id3 ==uses==> id2
    id4(Git) ==manages==> id5(fake API as submodule)
{{</mermaid>}}
