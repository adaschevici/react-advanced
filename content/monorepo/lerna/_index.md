+++
title = "lerna"
chapter = false
weight = 3
date = 2020-03-07T16:59:44Z
draft = false
+++

#### lerna - there be hydras

{{< lazy-image image="lerna-logo.png" lightbox=false />}}

#### 🎉 lerna is an open source package to manage multi package javascript monorepos 🎉


Lerna is open sourced and available on [Github](https://github.com/lerna/lerna)


#### Fun fact
The seven-headed monster you’ll see at the top of the project’s website and repo depict the Greek mythological creature known as the Lernaean Hydra, or Hydra of Lerna.


#### Getting set up
- create project directory

    ```bash
    mkdir goodreads-v2
    ```

- initialize the npm package

    ```bash
    npm init -y
    ```

    ```json
    {
      "name": "@goodreads/goodreads-v2",
      "version": "1.0.0",
      "description": "",
      "main": "index.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "keywords": [
        "goodreads",
        "react-advanced",
        "monorepo"
      ],
      "author": "<your profile link of choice>",
      "license": "MIT",
      "dependencies": {
        "lerna": "^3.20.2"
      }
    }
    ```
- install lerna so that it is available to the project

    ```bash
    npm i lerna
    ```

- lerna can be used in `goodreads-v2`

    ```bash
    npx lerna [command]
    ```


#### Using lerna to manage your monorepo

- initialize lerna

  ```bash
  npx lerna init
  ```

- add component-library

    ```bash
    npx lerna create @goodreads-v2/component-library
    ```
- the `packages/component-library/package.json` should end up looking like this

    ```json
    {
      "name": "@goodreads-v2/component-library",
      "version": "0.0.0",
      "description": "components library and design system",
      "keywords": [
        "components",
        "design-system"
      ],
      "author": "John Doe <j.doe@gmail.com>",
      "homepage": "",
      "license": "MIT",
      "main": "lib/component.bundle.js",
      "directories": {
        "lib": "lib",
        "test": "__tests__"
      },
      "files": [
        "lib"
      ],
      "publishConfig": {
        "access": "public"
      },
      "scripts": {
        "test": "echo \"Error: run tests from root\" && exit 1"
      }
    }
    ```

- add app package we can do this via `create-react-app`

    - add create-react-app

      ```bash
      npm i create-react-app
      ```

    - create app

        ```bash
        npx create-react-app packages/goodreads
        ```

- add the dependency on the mocked api, we cand do this using git submodules

    ```bash
    git submodule add git@github.com:adaschevici/jungle-jim.git packages/monkey-api
    ```

#### Resulting project structure:

```bash
▾ [ ]packages/
  ▾ [ ]component-library/
    ▸ [ ]__tests__/
    ▸ [ ]lib/
      [ ]package.json
      [ ]README.md
  ▾ [ ]goodreads/
    ▸ [ ]node_modules/
    ▸ [ ]public/
    ▸ [ ]src/
      [ ].gitignore
      [ ]package-lock.json
      [ ]package.json
      [ ]README.md
  ▾ [ ]monkey-api/
    ▸ [ ]data/
    ▸ [ ]src/
      [ ].git
      [ ].gitignore
      [ ].prettierrc
      [ ]ecosystem.config.js
      [ ]LICENSE
      [ ]package-lock.json
      [ ]package.json
      [ ]README.md
  [ ].gitmodules
  [ ]lerna.json
  [ ]package-lock.json
  [ ]package.json
```

#### Cloning projects with submodules
In order to start from the master branch of the repo the clone command is a bit different because it has submodules

```bash
git clone --recurse-submodules git@github.com:adaschevici/goodreads-v2.git
```

When you switch to another branch and the folder of the submodule is not there
```bash
git submodule init
git submodule update --remote
```

#### Add dependencies, and manage them
There are a few commands that we will use throughout that work in bulk.
- clean all node_modules
  ```bash
  npx lerna clean
  ```

- install dependencies in `packages/`
  ```bash
  npx lerna bootstrap
    ```

- locally available packages like `@goodreads-v2/component-library` can be installed via lerna.
  {{% notice tip %}}
  The cool part is that you are not required to publish them but you can work with them as if they were.
  {{% /notice %}}
  ```bash
  npx lerna add @goodreads-v2/component-library --scope=@goodreads-v2/goodreads
  ```
- then we want to add `styled-components` and `styled-system` to the goodreads react app
  ```bash
  npx lerna add styled-components --scope=@goodreads-v2/goodreads
  npx lerna add styled-system --scope=@goodreads-v2/goodreads
  ```
  {{% notice info %}}
  Adding multiple packages at once does not seem to work very well with lerna because lerna does
  multiple things when it installs a package.
  {{% /notice %}}

  {{% notice warning %}}
  If at any point you get an error like this one:
  `lerna ERR! EFILTER No packages remain after filtering [ '<some-package-name>' ]` check the package.json of the
  package you want to add the dependency to and see if the name is exactly what you are instructing lerna to use.
  By default `lerna [command]` uses the package.json defined variables for running.
  {{% /notice %}}

