---
title: vue源码项目
date: 2017-06-01 19:34:23
tags:
---
## 项目结构
- **`build`**: 包含构建相关的内置文件。在大多数情况下你不需要去动它们。然而，熟悉一下下面的文件可能会有帮助。
- `build/alias.js`: 所有的源码和测试用到的模块引入别名

- `build/config.js`: 包含`dist/`文件夹下面生成的所有文件文件的build配置。这个文件是dist文件夹的源文件生成入口。

- **`dist`**: 包含分发的构建文件。注意这个目录只会在一个release发布的时候才会更新；在development分支的最新变化不会实时反映出来。

See [dist/README.md](https://github.com/vuejs/vue/blob/dev/dist/README.md) for more details on dist files.

- **`flow`**: 包含类型声明，用的是 [Flow](https://flowtype.org/). 这些声明都是**全局**加载的，所以你会看到它们在源码中类型注释的使用。

- **`packages`**: 包含 `vue-server-renderer` 和 `vue-template-compiler`,是分离的NPM包。它们都是从源码里自动生成的，并且与main `vue` package有着同样的版本。

- **`test`**: 包含所有的测试. The unit tests are written with [Jasmine](http://jasmine.github.io/2.3/introduction.html) and run with [Karma](http://karma-runner.github.io/0.13/index.html). The e2e tests are written for and run with [Nightwatch.js](http://nightwatchjs.org/).

- **`src`**: contains the source code, obviously. The codebase is written in ES2015 with [Flow](https://flowtype.org/) type annotations.

  - **`compiler`**: contains code for the template-to-render-function compiler.

    The compiler consists of a parser (converts template strings to element ASTs), an optimizer (detects static trees for vdom render optimization), and a code generator (generate render function code from element ASTs). Note the codegen directly generates code strings from the element AST - it's done this way for smaller code size because the compiler is shipped to the browser in the standalone build.

  - **`core`**: contains universal, platform-agnostic runtime code.

    The Vue 2.0 core is platform-agnostic - which means code inside `core` should be able to run in any JavaScript environment, be it the browser, Node.js, or an embedded JavaScript runtime in native applications.

    - **`observer`**: contains code related to the reactivity system.

    - **`vdom`**: contains code related to vdom element creation and patching.

    - **`instance`**: contains Vue instance constructor and prototype methods.

    - **`global-api`**: as the name suggests.

    - **`components`**: universal abstract components. Currently `keep-alive` is the only one.

  - **`server`**: contains code related to server-side rendering.

  - **`platforms`**: contains platform-specific code.

    Entry files for dist builds are located in their respective platform directory.

    Each platform module contains three parts: `compiler`, `runtime` and `server`, corresponding to the three directories above. Each part contains platform-specific modules/utilities which are then imported and injected to the core counterparts in platform-specific entry files. For example, the code implementing the logic behind `v-bind:class` is in `platforms/web/runtime/modules/class.js` - which is imported in `entries/web-runtime.js` and used to create the browser-specific vdom patching function.

  - **`sfc`**: contains single-file component (`*.vue` files) parsing logic. This is used in the `vue-template-compiler` package.

  - **`shared`**: contains utilities shared across the entire codebase.
