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

> 注：可使用`npx browserslist`测试上述的`query`语句或直接在`npx browserslist`命令后添加`query`语句

#### Query Composition

An `or` combiner can use the keyword `or` as well as `,`. `last 1 version or > 1% is` equal to `last 1 version, > 1%`.

`and` query combinations are also supported to perform an intersection of the previous query: `last 1 version and > 1%`.

| Query combiner type | 说明 | Example |
| ------------------- | :----------: | ------- |
|`or`/ `,` combiner <br> (union) | A和B的`并集`  | `'> .5% or last 2 versions'` <br> `'> .5%, last 2 versions'` |
| `and` combiner <br> (intersection) | A和B的`交集` | `'> .5% and last 2 versions'` |
| `not` combiner <br> (relative complement) | `not`语句不能放在最左边（它需要依赖数据源，从数据源中排除特定数据），`not`语句从左侧产生的结果集中排除掉特定的数据 | `'> .5% and not last 2 versions'` <br> `'> .5% or not last 2 versions'` <br> `'> .5%, not last 2 versions'` |

可使用`npx browserslist '> 0.5%, not IE 11'`命令快速验证表达式语句。

> 注：Query表达式语句是从左往右执行的，与`and`/`or`/`not`等关键字无关

#### Query Full List - 通过以下关键字指定浏览器及Node版本（**不区分大小写**）

* `> 5%`: browsers versions selected by global usage statistics.
  `>=`, `<` and `<=` work too.
* `> 5% in US`: uses USA usage statistics. It accepts [two-letter country code].
* `> 5% in alt-AS`: uses Asia region usage statistics. List of all region codes
  can be found at [`caniuse-lite/data/regions`].
* `> 5% in my stats`: uses [custom usage data].
* `cover 99.5%`: most popular browsers that provide coverage.
* `cover 99.5% in US`: same as above, with [two-letter country code].
* `cover 99.5% in my stats`: uses [custom usage data].
* `maintained node versions`: all Node.js versions, which are [still maintained]
  by Node.js Foundation.
* `node 10` and `node 10.4`: selects latest Node.js `10.x.x`
  or `10.4.x` release.
* `current node`: Node.js version used by Browserslist right now.
* `extends browserslist-config-mycompany`: take queries from
  `browserslist-config-mycompany` npm package.
* `ie 6-8`: selects an inclusive range of versions.
* `Firefox > 20`: versions of Firefox newer than 20.
  `>=`, `<` and `<=` work too. It also works with Node.js.
* `iOS 7`: the iOS browser version 7 directly.
* `Firefox ESR`: the latest [Firefox ESR] version.
* `unreleased versions` or `unreleased Chrome versions`:
  alpha and beta versions.
* `last 2 major versions` or `last 2 iOS major versions`:
  all minor/patch releases of last 2 major versions.
* `since 2015` or `last 2 years`: all versions released since year 2015
  (also `since 2015-03` and `since 2015-03-10`).
* `dead`: browsers from `last 2 version` query, but with less than 0.5%
  in global usage statistics and without official support or updates
  for 24 months. Right now it is `IE 10`, `IE_Mob 10`, `BlackBerry 10`,
  `BlackBerry 7`, and `OperaMobile 12.1`.
* `last 2 versions`: the last 2 versions for *each* browser.
* `last 2 Chrome versions`: the last 2 versions of Chrome browser.
* `defaults`: Browserslist’s default browsers
  (`> 0.5%, last 2 versions, Firefox ESR, not dead`).
* `not ie <= 8`: exclude browsers selected by previous queries.

You can add `not ` to any query.

[`caniuse-lite/data/regions`]: https://github.com/ben-eb/caniuse-lite/tree/master/data/regions
[two-letter country code]:     https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements
[custom usage data]:           #custom-usage-data
[still maintained]:            https://github.com/nodejs/Release
[Can I Use]:                   https://caniuse.com/

#### Browsers - 不区分大小写

* `Android` for Android WebView.
* `Baidu` for Baidu Browser.
* `BlackBerry` or `bb` for Blackberry browser.
* `Chrome` for Google Chrome.
* `ChromeAndroid` or `and_chr` for Chrome for Android
* `Edge` for Microsoft Edge.
* `Electron` for Electron framework. It will be converted to Chrome version.
* `Explorer` or `ie` for Internet Explorer.
* `ExplorerMobile` or `ie_mob` for Internet Explorer Mobile.
* `Firefox` or `ff` for Mozilla Firefox.
* `FirefoxAndroid` or `and_ff` for Firefox for Android.
* `iOS` or `ios_saf` for iOS Safari.
* `Node` for Node.js.
* `Opera` for Opera.
* `OperaMini` or `op_mini` for Opera Mini.
* `OperaMobile` or `op_mob` for Opera Mobile.
* `QQAndroid` or `and_qq` for QQ Browser for Android.
* `Safari` for desktop Safari.
* `Samsung` for Samsung Internet.
* `UCAndroid` or `and_uc` for UC Browser for Android.
* `kaios` for KaiOS Browser.

#### `package.json`

If you want to reduce config files in project root, you can specify
browsers in `package.json` with `browserslist` key:

```json
{
  "private": true,
  "dependencies": {
    "autoprefixer": "^6.5.4"
  },
  "browserslist": [
    "last 1 version",
    "> 1%",
    "IE 10"
  ]
}
```


#### Config File

Browserslist config should be named `.browserslistrc` or `browserslist`
and have browsers queries split by a new line. Comments starts with `#` symbol:

```yaml
# Browsers that we support

last 1 version
> 1%
IE 10 # sorry
```

Browserslist will check config in every directory in `path`.
So, if tool process `app/styles/main.css`, you can put config to root,
`app/` or `app/styles`.

You can specify direct path in `BROWSERSLIST_CONFIG` environment variables.


#### Environment Variables

If some tool use Browserslist inside, you can change browsers settings
by [environment variables]:

* `BROWSERSLIST` with browsers queries.

   ```sh
  BROWSERSLIST="> 5%" gulp css
   ```

* `BROWSERSLIST_CONFIG` with path to config file.

   ```sh
  BROWSERSLIST_CONFIG=./config/browserslist gulp css
   ```

* `BROWSERSLIST_ENV` with environments string.

   ```sh
  BROWSERSLIST_ENV="development" gulp css
   ```

* `BROWSERSLIST_STATS` with path to the custom usage data
  for `> 1% in my stats` query.

   ```sh
  BROWSERSLIST_STATS=./config/usage_data.json gulp css
   ```

* `BROWSERSLIST_DISABLE_CACHE` if you want to disable config reading cache.

   ```sh
  BROWSERSLIST_DISABLE_CACHE=1 gulp css
   ```

[environment variables]: https://en.wikipedia.org/wiki/Environment_variable


#### Environments

You can also specify different browser queries for various environments.
Browserslist will choose query according to `BROWSERSLIST_ENV` or `NODE_ENV`
variables. If none of them is declared, Browserslist will firstly look
for `production` queries and then use defaults.

In `package.json`:

```js
  "browserslist": {
    "production": [
      "> 1%",
      "ie 10"
    ],
    "modern": [
      "last 1 chrome version",
      "last 1 firefox version"
    ],
    "ssr": [
      "node 12"
    ]
  }
```

In `.browserslistrc` config:

```ini
[production]
> 1%
ie 10

[modern]
last 1 chrome version
last 1 firefox version

[ssr]
node 12
```


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
    
    
    