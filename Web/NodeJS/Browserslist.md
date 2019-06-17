## Browserslist

### Tools

* [browserslist-ga](https://github.com/browserslist/browserslist-ga) - 依赖于`Google Analytics`。
downloads your website browsers statistics to use it in `> 0.5% in my stats` query.

* [browserslist-useragent-regexp](https://github.com/browserslist/browserslist-useragent-regexp) - 
用于检测是否支持客户端当前使用的浏览器及版本。compiles Browserslist query to a RegExp to test browser useragent.

* [browserslist-useragent-ruby](https://github.com/browserslist/browserslist-useragent-ruby) - 
is a Ruby library to checks browser by user agent string to match Browserslist.

* [browserslist-browserstack](https://github.com/xeroxinteractive/browserslist-browserstack) - 
runs `BrowserStack` tests for all browsers in Browserslist config.

* [caniuse-api](https://github.com/Nyalab/caniuse-api) - returns browsers which support some specific feature.

* Run `npx browserslist` in your project directory to see project’s target browsers. This CLI tool is built-in and available in any project with Autoprefixer.

### Queries

Browserslist 将从以下来源中查找query查询语句：

1. 当前或父目录的`package.json`文件中的`browserslist`属性（推荐）
2. 当前或父目录的`.browserslistrc`配置文件
3. 当前或父目录的`browserslist`配置文件
4. `BROWSERSLIST`环境变量
5. 如果以上方案没有产生一个有效的`Browserslist`结果，则使用`defaults`：`> 0.5%, last 2 versions, Firefox ESR, not dead`

#### Query Composition

An `or` combiner can use the keyword `or` as well as `,`. last 1 version or > 1% is equal to last 1 version, > 1%.

### 参考文档

* [Github官网 -【重要】](https://github.com/browserslist/browserslist)

* [`Can I Use` -【重要】](https://caniuse.com/) - Web特性在各浏览器中的支持情况

* [`Browserslist Online` -【重要】](https://browserl.ist/) - Browserslist在线`Queries`语句查询，显示`Queries`语句对应的各浏览及版本

* [Browserslist Example](https://github.com/browserslist/browserslist-example)

    * Browserslist 配置可以在`.browserslistrc`文件或`package.json`的`browserslist`属性中定义
        
        ```diff
        {
          "private": true,
        + "browserslist": [
        +   "Edge 16"
        + ],
          "scripts": {
          }
        }
        ```
        
    * `Developers` - 通过Browserslist命令行工具验证支持哪些浏览器
    
        ```bash
        $ npx browserslist
        ```
    * `Autoprefixer` 集成Browserslist
    * `Babel` 集成Browserslist
    * `PostCSS Preset Env` 集成Browserslist
    * `PostCSS Normalize` 集成Browserslist
    * `ESLint` 集成Browserslist
    * `Stylelint` 集成Browserslist
    
    
    