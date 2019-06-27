## Babel

### 总结

* `useBuiltIns: "usage"` - 当你使用`usage`时需要配置对应的`corejs`版本，否则会输出警告日志：
`WARNING: We noticed you're using the `useBuiltIns` option without declaring a core-js version...`

    ```js
    const presets = [
      [
        "@babel/env",
        {
          targets: {
            edge: "17",
            firefox: "60",
            chrome: "67",
            safari: "11.1",
            ie: "10",
          },
          useBuiltIns: "usage",
          corejs: 2
        },
      ],
    ];
    
    module.exports = { presets };
    ```

* Plugins
    
    * [`Plugins详细列表` / `怎么配置Plugins` - 【重要】](https://babeljs.io/docs/en/plugins)
    * [`@babel/cli`](https://babeljs.io/docs/en/babel-cli)
    * [`@babel/polyfill`](https://babeljs.io/docs/en/babel-polyfill) - 
        提供ES6 API（Babel默认只转换ES6语法，API不做处理），兼容于ES5，会污染全局变量。
        This will emulate a full ES2015+ environment (no < Stage 4 proposals) 
        and is intended to be used in an application rather than a library/tool. 
        (this polyfill is **automatically** loaded when using `babel-node`).
        
        > 参考：[Babel学习系列4-polyfill和runtime差别(必看)](https://zhuanlan.zhihu.com/p/58624930)
        
    * [`@babel/plugin-transform-runtime`](https://babeljs.io/docs/en/babel-plugin-transform-runtime) - 
        结合`@babel/runtime`一起使用，功能与`@babel/polyfill`类似，但不会污染全局变量。
        常用于插件和组件开发。此方案不适用于实例方法，例：`"foobar".includes("foo")`。
        
        > 参考：[Babel学习系列4-polyfill和runtime差别(必看)](https://zhuanlan.zhihu.com/p/58624930)
        
    * [`@babel/node`](https://babeljs.io/docs/en/babel-node) - 
        `babel-node`是一个命令行功能，类似`node`命令，在运行node之前通过Babel presets/plugins编译源码。
        不建议用于生产环境，部署生产环境之前建议提前编译好。
        
    * [`@babel/register`](https://babeljs.io/docs/en/babel-register) - 
        通过`require`钩子使用Babel，此钩子会绑定到`node require`，且运行时编译require资源。
        All subsequent files required by node with the extensions `.es6`, `.es`, `.jsx`, `.mjs`, and `.js` will be transformed by Babel.
        

* [Presets - 配置常用的Plugins集合](https://babeljs.io/docs/en/presets)

    * [@babel/preset-env](https://babeljs.io/docs/en/babel-preset-env)
    * [@babel/preset-flow](https://babeljs.io/docs/en/babel-preset-flow)
    * [@babel/preset-react](https://babeljs.io/docs/en/babel-preset-react)
    * [@babel/preset-typescript](https://babeljs.io/docs/en/babel-preset-typescript)
    * [babel-preset-minify](https://babeljs.io/docs/en/babel-preset-minify) - 
        压缩/混淆文件
    * Stage-X (`Experimental Presets`)
        * [Stage 0](https://babeljs.io/docs/en/babel-preset-stage-0) - 
            Strawman: just an idea, possible Babel plugin.
        * [Stage 1](https://babeljs.io/docs/en/babel-preset-stage-1) - 
            Proposal: this is worth working on.
        * [Stage 2](https://babeljs.io/docs/en/babel-preset-stage-2) - 
            Draft: initial spec.
        * [Stage 3](https://babeljs.io/docs/en/babel-preset-stage-3) - 
            Candidate: complete spec and initial browser implementations.
        * Stage 4 - Finished: will be added to the next yearly release.

* [Options - 配置](https://babeljs.io/docs/en/options) - 以下列出需要注意事项：
    * [MatchPattern](https://babeljs.io/docs/en/options#matchpattern)
    * [Merging](https://babeljs.io/docs/en/options#merging)
    * [Plugin/Preset entries - 【重要】](https://babeljs.io/docs/en/options#plugin-preset-entries)
    * [Name Normalization - 【重要】](https://babeljs.io/docs/en/options#name-normalization)

* Config Files
    
    * [Configure Babel -【重要】](https://babeljs.io/docs/en/configuration)

        提供了好几种配置方式（推荐使用`babel.config.js`）：
        * `babel.config.js`
        * `.babelrc`
        * 在`package.json`中配置
        * `.babelrc.js`
        * `Using the CLI (@babel/cli)`
        * `Using the API (@babel/core)`
        
    * [Config Files - 配置文件的加载规则及配置方法 -【重要】](https://babeljs.io/docs/en/config-files)
        
        Babel has two parallel config file formats, which can be used together, or independently.
        
        * Project-wide configuration - Babel 7.x新功能：`babel.config.js`
        * File-relative configuration
            * `.babelrc` (and `.babelrc.js`) files
            * `package.json` files with a `"babel"` key

* Tooling
    
    * [`@babel/core`](https://babeljs.io/docs/en/babel-core) - 
        用于解析、转换代码，依赖`parser`、`code-frame`、`generator`、`helpers`、`types`等组件
    * [`@babel/parser`](https://babeljs.io/docs/en/babel-parser) - 
        解析代码生成AST格式数据
    * [`@babel/generator`](https://babeljs.io/docs/en/babel-generator) -
        依据AST数据生成代码
    * [`@babel/code-frame`](https://babeljs.io/docs/en/babel-code-frame) - 
        输出源码行/列相关信息
    * [`@babel/helpers`](https://babeljs.io/docs/en/babel-helpers) - 
        一系列预定义的`@babel/template`模板方法
    * [`@babel/runtime`](https://babeljs.io/docs/en/babel-runtime) - 
        结合`@babel/plugin-transform-runtime`一起使用
    * [`@babel/template`](https://babeljs.io/docs/en/babel-template) - 
        用于快速创建AST的模板
    * [`@babel/traverse`](https://babeljs.io/docs/en/babel-traverse) - 
        用于遍历操作AST数据 
    * [`@babel/types`](https://babeljs.io/docs/en/babel-types) - 
        AST操作工具库，包括判断、断言、创建3类API

### Online Tool

* [Babel线上代码转换](https://babeljs.io/repl)

* 可以运行Babel的Online Editors
    * [JSFiddle](https://jsfiddle.net/fh5whLfd/)
    * [JSBin](http://jsbin.com/rokimopuse/edit?html,js,console,output)
    * [Codepen](http://codepen.io/anon/pen/dOGgeO)

### 文档

* `Using Babel` - 使用Babel的各种场景：https://babeljs.io/setup.html

* [Usage Guide - 【重要】](https://babeljs.io/docs/en/usage)
    
    > You can use the npm package runner that comes with npm@5.2.0 to shorten that command by 
    replacing `./node_modules/.bin/babel` with `npx babel`

* [Babel快速指南](https://www.colabug.com/4811194.html)

* [ECMAScript 2015 特性](https://babeljs.io/docs/en/learn)

* [Example Node Server With Babel -【重要】](https://github.com/babel/example-node-server)

    1. Simplest
    ```javascript
    "scripts": {
        "build": "babel lib -d dist",
        "start": "npm run build && node dist/index.js"
    }
    ```
    
    2. Watching file changes with `nodemon`
    
    ```bash
    npm install --save-dev nodemon
    ```
    
    ```javascript
    "scripts": {
        "build": "babel lib -d dist",
        "start": "npm run build && nodemon dist/index.js"
    }
    ```
    
    3. Add test
    ```javascript
    "scripts": {
        "start": "nodemon lib/index.js --exec babel-node",
        "build": "babel lib -d dist",
        "serve": "node dist/index.js",
        "test": "mocha --require @babel/register"
    }
    ```