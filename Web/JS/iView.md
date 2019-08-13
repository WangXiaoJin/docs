# iView

## 常见问题

### RESTful 动态路由匹配问题

两种解决方案：

```javascript
data() {
  return {
    loaded: false
  }
},
created() {
  this.loaded = true;
  this.loadData(this.$route.params);
},
beforeRouteUpdate (to, from, next) {
  // loadData 方法中不能直接使用 this.$route.params，用了只会获取 from 路由的参数，所以这里需要通过传参的方式传给 loadData
  this.loadData(to.params);
  next();
},
activated () {
  // 如果缓存启用时，从别的路由跳到此路由时，会触发 created、activated 钩子函数，所以需要判断有没有加载过数据
  if (!this.loaded) {
    this.loadData(this.$route.params);
  }
  this.loaded = false;
},
```

```javascript
watch: {
  '$route' (to, from) {
    // 1. 当此组件没有配置 name，或者配置的 name 和 to.name 相等，则刷新数据
    // 2. 当此组件不配置 name 时，页面缓存肯定无效，在不同路由间切换时不会触发 watch.$route 函数
    // 3. 当此组件配置了name，且路由名和此值一致，并且缓存开启的情况下，从当前组件路由切换至其他路由时会触发 watch.$route 函数，
    //    所有需要判断出口路由是否是此组件，如果是此组件则才需要刷新数据
    if (!this.$options.name || this.$options.name === to.name) {
      // 此 loadData方法中可以使用 this.$route.params，此值和 to.params 相同，这与 beforeRouteUpdate 不同，
      // 由 watch、beforeRouteUpdate 触发时机决定的。
      this.loadData();
    }
  }
}
```

> [参考文档](https://router.vuejs.org/zh/guide/essentials/dynamic-matching.html)


## 文档

* [`iView` - 基于Vue](https://www.iviewui.com/docs/guide/install)
* [`iView-Admin`](https://github.com/iview/iview-admin)
    * [文档](https://lison16.github.io/iview-admin-doc/#/)

