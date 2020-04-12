+++
title = "lerna"
chapter = false
weight = 3
date = 2020-03-07T16:59:44Z
draft = false
+++

#### lerna - there be hydras

{{< lazy-image image="lerna-logo.png" lightbox=true />}}

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
