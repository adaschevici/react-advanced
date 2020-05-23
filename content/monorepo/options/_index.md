+++
title = "options"
chapter = false
weight = 2
date = 2020-03-07T16:59:44Z
draft = false
+++

#### There are multiple alternatives for a mono repo solution

1. D.I.Y
2. yarn workspaces
3. lerna
4. mix of lerna and yarn

#### Why should you care?

While we will be using `lerna` and `npm` for our setup in the wild projects can
use other approaches and being aware of them helps.

#### do it yourself

`npm` allows you to use `npm link` to use locally available packages. Under the hood
the other solutions make use of this ability, but they also provide you with many other
utils

#### yarn workspaces

They are a fairly recent addition to yarn and are a useful
feature for configuring a monorepo for your app.

Yarn workspaces allow you to create multiple directories
and have each directory contain a package that you can reuse
in other directory as you would an npm package. In order to enable
workspaces you need to have the project configured.

In `<project-root>/package.json`

```json
{
  "private": true,
  "workspaces": [
    <list of package directories>
  ],
  "scripts": {
    ...
  }
}
```

Managing dependencies in the project should be done via `yarn`. Most npm
commands have `yarn` equivalents so it should be straightforward to migrate
unless you are already using `yarn` in which case you won't have to migrate.

Running commands is straightforward and makes use of the `workspace` subcommand.

For example this would add react in a workspace:
`npx yarn workspace <workspace-name> add react`

#### lerna

Lerna has been around longer than yarn workspaces. It has similar features
and it achieves similar goals.

#### lerna and yarn(with or without workspaces)

This is the most convoluted and it requires both a lerna config and a yarn workspace
config. This is more unusual because configs have some degree of redundancy.

#### yarn workspaces vs lerna

1. lerna is more mature and has more features beside monorepo management. It also allows for more sophisticated dependency handlling.
2. you can't install more than one package at once in a package with lerna
3. in yarn workspaces the packages are hoisted automatically so you won't have duplicate
   react dependencies which is a common misshap when using lerna

### The Gist 📌

{{< lazy-image image="monorepo.png" lightbox=false />}}
