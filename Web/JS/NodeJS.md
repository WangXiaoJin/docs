## NodeJS

#### 语法

* [`exports`、`module.exports` 和 `export`、`export default`讲解](https://juejin.im/post/597ec55a51882556a234fcef)
* Node.js模块系统 - require/exports - 模块加载规则及原理
    * http://nodejs.cn/api/modules.html
    * https://www.runoob.com/nodejs/nodejs-module-system.html
* Node.js 全局对象/全局属性 - `__filename`/`__dirname`/`console`/`process`等
    * http://nodejs.cn/api/globals.html
    * https://www.runoob.com/nodejs/nodejs-global-object.html


#### npm

* `npm config ls` 显示当前配置。`npm config ls -l`显示当前配置及`默认配置`。

* 配置`prefix`/`cache`参数

  ```bash
  # 全局模块插件存放路径，默认值为安装目录
  npm config set prefix "D:\Program Files\node-v10.15.3"
  
  # 缓存路径，默认值 ~AppData\Roaming\npm-cache
  npm config set cache "D:\Program Files\node-v10.15.3\node_cache"
  ```

* npm 常用命令
  
  * `npm init`创建`package.json`
  * 使用npm help <command>可查看某条命令的详细帮助，例如npm help install。
  * 在package.json所在目录下使用npm install . -g可先在本地安装当前命令行程序，可用于发布前的本地测试。
  * 使用npm update <package>可以把当前目录下node_modules子目录里边的对应模块更新至最新版本。
  * 使用npm update <package> -g可以把全局安装的对应命令行程序更新至最新版。
  * 使用npm cache clear可以清空NPM本地缓存，用于对付使用相同版本号发布新版本代码的人。
  * 使用npm unpublish <package>@<version>可以撤销发布自己发布过的某个版本代码。

* [淘宝 NPM 镜像](http://npm.taobao.org/)

* [npm官网](https://docs.npmjs.com/)
* [package.json详解 -【重要】](https://docs.npmjs.com/files/package.json.html)
  * [dependencies配置讲解](https://docs.npmjs.com/files/package.json.html#dependencies)
  * [npm-semver - The semantic versioner for npm(dependencies配置版本详解)](https://docs.npmjs.com/misc/semver.html)
* [Configuring npm -【重要】](https://docs.npmjs.com/cli-documentation/files)

### 常用插件

* [`npm-check`](https://www.npmjs.com/package/npm-check) - `npm install -g npm-check`，检查`过期的`/`不正确的`/`没有使用的`插件、支持交互式更新插件。


### 文档

* [NodeJS中文文档 - 【重要】](http://nodejs.cn/api/util.html)
* [runoob.com简易文档](https://www.runoob.com/nodejs/nodejs-http-server.html)