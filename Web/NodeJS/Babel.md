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
    * [`@babel/polyfill`](https://babeljs.io/docs/en/babel-polyfill)
    * [`@babel/node`](https://babeljs.io/docs/en/babel-node)
    * [`@babel/plugin-transform-runtime`](https://babeljs.io/docs/en/babel-plugin-transform-runtime)
    * [`@babel/register`](https://babeljs.io/docs/en/babel-register)

* [Presets - 配置常用的Plugins集合](https://babeljs.io/docs/en/presets)

    * [@babel/preset-env](https://babeljs.io/docs/en/babel-preset-env)
    * [@babel/preset-flow](https://babeljs.io/docs/en/babel-preset-flow)
    * [@babel/preset-react](https://babeljs.io/docs/en/babel-preset-react)
    * [@babel/preset-typescript](https://babeljs.io/docs/en/babel-preset-typescript)
    * [babel-preset-minify](https://babeljs.io/docs/en/babel-preset-minify)
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