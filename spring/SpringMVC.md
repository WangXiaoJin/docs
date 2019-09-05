# Spring MVC

## 集成Freemarker

* 查看Freemarker所有可用的变量
    * `AbstractView#render()` - 配置了`exposePathVariables`/`staticAttributes`/`requestContextAttribute`
    * `AbstractTemplateView#renderMergedOutputModel()` - 配置了`exposeRequestAttributes`/`exposeSessionAttributes`/`exposeSpringMacroHelpers`
    * `FreeMarkerView#doRender()` - 注入了`JspTaglibs`/`Session`/`Application`/`Request`/`RequestParameters`等对象

* Freemarker获取变量值顺序如下，直到找到位置。参考：`freemarker.ext.servlet.AllHttpScopesHashModel#get(key)`
    1. 从`Mvc Model`中获取
    2. 通过`request.getAttribute(key)`获取
    3. 通过`session.getAttribute(key)`获取
    4. 通过`context.getAttribute(key)`获取
    5. 还没找到则返回`null`

## `ForwardedHeaderFilter`

从请求头中提取 `Forwarded` 、`X-Forwarded-*` 值， 用于 `getServerName()`、`getServerPort()`、`getScheme()`、`isSecure()`
从对应的 `Forwarded` 头信息中获取，且 `sendRedirect(String)` 也会从`Forwarded`中取相关信息。

> 用于代理更改了域名相关信息的场景，此时应用通过上面方法获取的信息与浏览器最初的请求信息不一致。通过 `ForwardedHeaderFilter` 可解决此问题。
