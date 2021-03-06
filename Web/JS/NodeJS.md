# NodeJS

## 语法

* [`exports`、`module.exports` 和 `export`、`export default`讲解](https://juejin.im/post/597ec55a51882556a234fcef)
* Node.js模块系统 - require/exports - 模块加载规则及原理
    * http://nodejs.cn/api/modules.html
    * https://www.runoob.com/nodejs/nodejs-module-system.html
* Node.js 全局对象/全局属性 - `__filename`/`__dirname`/`console`/`process`等
    * http://nodejs.cn/api/globals.html
    * https://www.runoob.com/nodejs/nodejs-global-object.html


## npm

### `npm config ls` 显示当前配置。`npm config ls -l`显示当前配置及`默认配置`。

* `npm help config` - config命令，配置会保存到文件中
* [npm-config](https://docs.npmjs.com/misc/config) - 【非常重要】**命令行使用配置及配置详解**
    * `npm ci --verbose` - `-dd, --verbose、--loglevel verbose`，配置LogLevel

### 配置`prefix`/`cache`参数

  ```bash
  # 全局模块插件存放路径，默认值为安装目录
  npm config set prefix "D:\Program Files\node-v10.15.3"
  
  # 缓存路径，默认值 ~AppData\Roaming\npm-cache
  npm config set cache "D:\Program Files\node-v10.15.3\node_cache"
  ```

### `package.json vars` / `configuration`

The `package.json` fields are tacked onto the `npm_package_` prefix. So, for instance, if you had **{"name":"foo", "version":"1.2.5"}** in your package.json file, then your package scripts would have the `npm_package_name` environment variable set to “foo”, and the `npm_package_version` set to “1.2.5”. You can access these variables in your code with `process.env.npm_package_name`` and process.env.npm_package_version`, and so on for other fields.

Configuration parameters are put in the environment with the `npm_config_` prefix. For instance, you can view the effective `root` config by checking the `npm_config_root` environment variable.

> 使用`npm run env`查看所有的环境变量

### Special: package.json “config” object

The package.json “config” keys are overwritten in the environment if there is a config param of `<name>[@<version>]:<key>`. For example, if the package.json has this:

```json
{ "name" : "foo"
, "config" : { "port" : "8080" }
, "scripts" : { "start" : "node server.js" } }
```
and the server.js is this:
```
http.createServer(...).listen(process.env.npm_package_config_port)
```
then the user could change the behavior by doing:
```
npm config set foo:port 80
```

### `current lifecycle event` / `HOOK SCRIPTS`

The `npm_lifecycle_event` environment variable is set to whichever stage of the cycle is being executed. So, you could have a single script used for different parts of the process which switches based on what’s currently happening.

> [`current lifecycle event` - 文档](https://docs.npmjs.com/misc/scripts#current-lifecycle-event)

> [HOOK SCRIPTS - 文档](https://docs.npmjs.com/misc/scripts#hook-scripts)

### npm 常用命令

* `npm init`创建`package.json`

* 使用`npm help <command>`可查看某条命令的详细帮助，例如npm help install。

* 在package.json所在目录下使用`npm install . -g`可先在本地安装当前命令行程序，可用于发布前的本地测试。

* `npm ci` - 和 `npm install` 类似，但它不会改变 `npm-shrinkwrap.json`、`package-lock.json`，适用于构建服务器、持续集成、自动化部署。
* 使用 `npm help ci` 查看文档解释
* [`npm i` changed my npm-shrinkwrap/package-lock, why?](https://npm.community/t/npm-i-changed-my-npm-shrinkwrap-package-lock-why/190)
* [`package-lock` 和 `npm-shrinkwrap`](https://xwenliang.cn/p/5cbd98b57eb9de0c54000003)
* [`npm install` `模块依赖`算法 -【重要】](https://docs.npmjs.com/cli/install.html#algorithm)

* 使用`npm update <package>`可以把当前目录下node_modules子目录里边的对应模块更新至最新版本。

* 使用`npm update <package> -g`可以把全局安装的对应命令行程序更新至最新版。

* 使用`npm cache clear`可以清空NPM本地缓存，用于对付使用相同版本号发布新版本代码的人。

* 使用`npm unpublish <package>@<version>`可以撤销发布自己发布过的某个版本代码。

* 使用`npx`执行`./node_modules/.bin/`下面的命令，如：`npx babel` => `./node_modules/.bin/babel` 

* `npm run env` - 内嵌脚本命令，用于查看当前所有的环境变量。详情查看`npm help run`。

* `npm help npm-scripts` - 脚本的生命周期

### 包依赖 / `package-lock` / `npm-shrinkwrap`
    
* [`npm install` `模块依赖`算法 -【重要】](https://docs.npmjs.com/cli/install.html#algorithm)
* [`npm i` changed my npm-shrinkwrap/package-lock, why?](https://npm.community/t/npm-i-changed-my-npm-shrinkwrap-package-lock-why/190)
* [package-lock 和 npm-shrinkwrap](https://xwenliang.cn/p/5cbd98b57eb9de0c54000003)
* `npm help package-lock.json`
* `npm help npm-shrinkwrap.json`
* `npm help npm-package-locks`
    * Resolving lockfile conflicts: `npm install --package-lock-only`
* `npm help npm-shrinkwrap`
* `npm help package.json`

* [package.json详解 -【重要】](https://docs.npmjs.com/files/package.json.html)
  * [dependencies配置讲解](https://docs.npmjs.com/files/package.json.html#dependencies)
  * [npm-semver - The semantic versioner for npm(dependencies配置版本详解)](https://docs.npmjs.com/misc/semver.html)
* [Configuring npm -【重要】](https://docs.npmjs.com/cli-documentation/files)

## 常用插件

* [`npm-check`](https://www.npmjs.com/package/npm-check) - `npm install -g npm-check`，检查`过期的`/`不正确的`/`没有使用的`插件、支持交互式更新插件。
* `serve` - 启用 HTTP Server 功能（推荐）
* `http-server` - 启用 HTTP Server 功能

## 常见问题

* [npm install 两个依赖包的Peer Dependencies冲突该怎么解决？](https://segmentfault.com/q/1010000011571000/a-1020000011575690)

* 共享`node_modules`或基础组件
    
    * 使用系统软链接，Windows：`mklink /j "node_modules" "../project1/node_modules"`
    * 使用`npm link`

## 文档

* [Nexus Npm Registry](https://help.sonatype.com/repomanager3/formats/npm-registry) -【重要】在Nexus上使用Npm仓库

* [npm官网](https://docs.npmjs.com/)
    * [npm-scope](https://docs.npmjs.com/misc/scope) - 讲解`scoped packages`发布、安装及配置
    
* [淘宝 NPM 镜像](http://npm.taobao.org/)
* [NodeJS中文文档 - 【重要】](http://nodejs.cn/api/util.html)
* [runoob.com简易文档](https://www.runoob.com/nodejs/nodejs-http-server.html)
* [npm11个提供工作效率的用法](https://leokongwq.github.io/2016/10/21/npm-11-useful-tips.html)

