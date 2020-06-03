# Vue

## `VueJs`

### 指令缩写

```html
<!-- 完整语法 -->
<a v-bind:href="url">...</a>

<!-- 缩写 -->
<a :href="url">...</a>
```

```html
<!-- 完整语法 -->
<a v-on:click="doSomething">...</a>

<!-- 缩写 -->
<a @click="doSomething">...</a>
```

```html
<!-- 完整语法 -->
<base-layout>
  <template v-slot:header>
    <h1>Here might be a page title</h1>
  </template>

  <template v-slot:footer>
    <p>Here's some contact info</p>
  </template>
</base-layout>

<!-- 缩写 -->
<base-layout>
  <template #header>
    <h1>Here might be a page title</h1>
  </template>

  <template #footer>
    <p>Here's some contact info</p>
  </template>
</base-layout>
```

### [`Vue.config.errorHandler`](https://cn.vuejs.org/v2/api/#errorHandler) - 指定组件的渲染和观察期间未捕获错误的处理函数

### `<keep-alive>`缓存及组件名

注意这个 `<keep-alive>` 要求被切换到的组件都有自己的名字，不论是通过组件的 `name` 选项还是局部/全局注册，否则页面缓存无效，
所有的组件都会重新渲染。且在`Vue Router`中不同路由切换时不会触发 `watch.$route`/`activated`/`deactivated`。

> 当`keep-alive`中组件配置了`name`属性时，无论从当前组件切出去或切进来都会触发`watch.$route`。

> 当使用了`keep-alive`的缓存时，切换路由时会触发`watch.$route`，即`$route`对象会改变，那么依赖`$route`对象属性的组件也会触发更新。
所以当切换路由离开此缓存组件时，依赖`this.$route.params`属性的组件会触发更新，此更新如导致向后端发送请求，会引起BUG。  
有一种简陋方案：把`this.$route`中的属性值赋值给组件的`data`属性（`id: this.$route.params.id`），这样切换路由时就不会触发依赖组件更新，
但又一点要注意，使用此方案后，切换路由时`data.id`属性永远不会更新。

> `Vue Router`中同一个路由如果产生多个`tab`项时（如：`/user/:id`形式），即使此路由组件没有启用缓存（如：组件名没配置），
产生的多个tab页面之间切换时也会触发`watch.$route`，与其他路由组件间切换并不会触发`watch.$route`。

* [`keep-alive` API](https://cn.vuejs.org/v2/api/#keep-alive)
* [`name` API](https://cn.vuejs.org/v2/api/#name)
* [在动态组件上使用-keep-alive](https://cn.vuejs.org/v2/guide/components-dynamic-async.html#%E5%9C%A8%E5%8A%A8%E6%80%81%E7%BB%84%E4%BB%B6%E4%B8%8A%E4%BD%BF%E7%94%A8-keep-alive)
* [Vue Router数据获取](https://router.vuejs.org/zh/guide/advanced/data-fetching.html)

### 文档

* [Vue实现`全局事件总线`](https://cn.vuejs.org/v2/guide/migration.html#dispatch-%E5%92%8C-broadcast-%E6%9B%BF%E6%8D%A2)

* [`KeyboardEvent.key` - Web中键盘按键对应的值](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values)
* [KeyCode - 在浏览器上按任意键后显示对应的keyCode](http://keycode.info/)

* [生命周期图示](https://cn.vuejs.org/v2/guide/instance.html#%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%9B%BE%E7%A4%BA)

* [vue-devtools - Vue调试工具](https://github.com/vuejs/vue-devtools)
    * [Get the Chrome Extension](https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd)
    * [Get the Firefox Addon](https://addons.mozilla.org/en-US/firefox/addon/vue-js-devtools/)
    * [Get standalone Electron app (works with any environment!)](https://github.com/vuejs/vue-devtools/blob/master/shells/electron/README.md)

* [awesome-vue - 资源及组件列表](https://github.com/vuejs/awesome-vue)
* [在线搜索Vue相关组件](https://awesomejs.dev/for/vue/)

* [`风格指南` - 提供了很多代码规约](https://cn.vuejs.org/v2/style-guide/)
* [VueJs API](https://cn.vuejs.org/v2/api/)
* [`Vue SSR 指南` - Vue.js 服务器端渲染指南](https://ssr.vuejs.org/zh/)

* [生产环境部署](https://cn.vuejs.org/v2/guide/deployment.html)

* [基于Vue的主题](https://cn.vuejs.org/v2/examples/themes.html)
* [在 VS Code 中调试 Vue](https://cn.vuejs.org/v2/cookbook/debugging-in-vscode.html)
* [避免内存泄漏](https://cn.vuejs.org/v2/cookbook/avoiding-memory-leaks.html)
* [客户端存储](https://cn.vuejs.org/v2/cookbook/client-side-storage.html)
* [Packaging Vue Components for npm](https://cn.vuejs.org/v2/cookbook/packaging-sfc-for-npm.html)
* [Dockerize Vue.js App](https://cn.vuejs.org/v2/cookbook/dockerize-vuejs-app.html)



## `Vue Router`

* [`Vue Router` - 官方文档](https://router.vuejs.org/zh/)
* [Vue Router API](https://router.vuejs.org/zh/api/)
* [`path-to-regexp` - 高级路径匹配模式，支持正则](https://github.com/pillarjs/path-to-regexp)
* [HTML5 History 模式时服务端配置](https://router.vuejs.org/zh/guide/essentials/history-mode.html)
* [路由页面时的滚动行为](https://router.vuejs.org/zh/guide/advanced/scroll-behavior.html)
* [路由时数据获取](https://router.vuejs.org/zh/guide/advanced/data-fetching.html)
* [路由懒加载](https://router.vuejs.org/zh/guide/advanced/lazy-loading.html)

## `Vuex`

* [`Vuex` - 官方文档](https://vuex.vuejs.org/zh/)
* [Vuex API 参考](https://vuex.vuejs.org/zh/api/)
* [Vuex项目结构](https://vuex.vuejs.org/zh/guide/structure.html)
* [Vuex的严格模式](https://vuex.vuejs.org/zh/guide/strict.html)
* [Vuex表单处理 - 双向绑定](https://vuex.vuejs.org/zh/guide/forms.html)
* [Vuex热重载](https://vuex.vuejs.org/zh/guide/hot-reload.html)


## `Vue Loader`

* [`Vue Loader` - 官方文档](https://vue-loader.vuejs.org/zh/)
    * [处理资源路径](https://vue-loader.vuejs.org/zh/guide/asset-url.html)
    * [使用预处理器](https://vue-loader.vuejs.org/zh/guide/pre-processors.html)
    * [Scoped CSS](https://vue-loader.vuejs.org/zh/guide/scoped-css.html)
    * [CSS Modules](https://vue-loader.vuejs.org/zh/guide/css-modules.html)
    * [热重载](https://vue-loader.vuejs.org/zh/guide/hot-reload.html)
    * [自定义块](https://vue-loader.vuejs.org/zh/guide/custom-blocks.html)
    * [CSS 提取](https://vue-loader.vuejs.org/zh/guide/extract-css.html)
    * [代码校验 (Linting)](https://vue-loader.vuejs.org/zh/guide/linting.html)
* [Vue 单文件组件 (SFC) 规范](https://vue-loader.vuejs.org/zh/spec.html)
* [选项参考](https://vue-loader.vuejs.org/zh/options.html)


## `Vue Test Utils`

* [`Vue Test Utils` - Vue.js 官方的单元测试实用工具库](https://vue-test-utils.vuejs.org/zh/)
* [Vue 组件的单元测试](https://cn.vuejs.org/v2/cookbook/unit-testing-vue-components.html)


## `Vue CLI`

* [`Vue CLI` - 指南](https://cli.vuejs.org/zh/guide/)
* [配置参考 - 【重要】](https://cli.vuejs.org/zh/config/)

* [浏览器兼容性](https://cli.vuejs.org/zh/guide/browser-compatibility.html)
* [HTML 和静态资源](https://cli.vuejs.org/zh/guide/html-and-static-assets.html)
* [CSS 相关](https://cli.vuejs.org/zh/guide/css.html)
* [webpack 相关](https://cli.vuejs.org/zh/guide/webpack.html)
* [环境变量和模式](https://cli.vuejs.org/zh/guide/mode-and-env.html)
* [部署](https://cli.vuejs.org/zh/guide/deployment.html)

* 缓存和并行处理
    
    * `cache-loader` - 会默认为 Vue/Babel/TypeScript 编译开启。文件会缓存在 `node_modules/.cache` 中——
    如果你遇到了编译方面的问题，记得先删掉缓存目录之后再试试看。
    
    * `thread-loader` - 会在多核 CPU 的机器上为 Babel/TypeScript 转译开启。

* `npx vue-cli-service inspect` 检查并输出webpack配置。`npx vue-cli-service inspect --help`输出帮助信息。

* Vue CLI 使用了 [`webpack-merge`](https://github.com/survivejs/webpack-merge) 合并webpack配置项 / 
[`webpack-chain`](https://github.com/neutrinojs/webpack-chain) 配置webpack配置项
